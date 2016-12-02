---
layout: post
title: Performing concurrency-safe DB updates in postgres 9.4+ using 'repeatable read' isolation level
author: Manoj Awasthi
categories: tech
---

Let’s accept it for the nth time again, writing concurrent programs correctly is hard. 

In this post, I discuss about a specific problem while concurrently writing to databases that I encountered recently, how to fix it and I am using that encounter to discuss a general idea about concurrent updates in databases. 

### Concurrent - What’s that? 

It’s sometime good to understand the antithesis to understand something. The opposite of concurrent would be sequential. A sequential program is a program with a single thread of execution so the order of events (instructions) executed in the program are predictable and are as per the program logic. A concurrent program is a program with more than one threads of execution and so the order of events (instructions) executed in the program are not really predictable — order of events within one thread remains predictable but whether an event `x` in thread A will happen before or after an event `y` in thread B or they will be simultaneous is generally unknown. 

So a good definition (quoting from “The Go Programming Language”):

>  When we cannot confidently say that one event happens before the other then the events x and y are concurrent. 

### Concurrency-safe programs 

A function is concurrency-safe if it continues to work correctly even when called concurrently (that is from two or more threads of execution without any additional synchronisation code). 

The analogy can be extended to “programs” when programs are accessing a shared resource for writing e.g. databases. If two or more than two programs are executing concurrently (on one or more than one servers) and are accessing a shared resource for writing then too the definition holds true that programs are concurrency-safe if they keep performing their updates correctly. This is the topic of this post. 

### Example program 

Let’s create a toy program for demonstration which does some updates to a local postgres instance. I will be using `golang` for implementing this but idea is language agnostic. 

We will use a test table in a test database (both table and database are aptly named 'test'). Following is the schema of the table: 

```
test=# \d+ test
                         Table "public.test"
 Column |  Type   | Modifiers | Storage | Stats target | Description 
--------+---------+-----------+---------+--------------+-------------
 id     | integer |           | plain   |              | 
 value  | integer |           | plain   |              | 
```

the example code: 

```
package main

import (
  "database/sql"
  "github.com/jmoiron/sqlx"
  _ "github.com/lib/pq"
  “log"
)

func main() {
  for i := 0; i < 750; i++ {
    err := dofn()
	if err != nil {
		log.Println("err", err)
		continue
	}
  }
}

```

In this program, we are simply calling the function `dofn()`, 750 times. In each call, `dofn` opens a connection to database 'test', reads a value from the table 'test' and then either writes (`INSERT`) the value 1 into table if it does not exist or increments the value (`UPDATE`) in the table. 

Following is the body for `dofn` implementation: 

```
func dofn() error {
	db, err := sqlx.Open("postgres", "host=127.0.0.1 port=5432 user=postgres password=postgres dbname=test sslmode=disable")
	if err != nil {
		return err
	}
	defer db.Close()

	tx, err := db.Begin()
	if err != nil {
		return err
	}

	defer tx.Rollback()

	var isExists bool = true
	var id int
	var value int
	err = tx.QueryRow("SELECT * FROM test WHERE id = 1;").Scan(&id, &value)
	if err != nil {
		if err == sql.ErrNoRows {
			isExists = false
		} else {
			return err
		}
	}

	if !isExists {
		_, err := tx.Exec("INSERT INTO test VALUES(1, 1)")
		if err != nil {
			return err
		}
	} else {
		value = value + 1
		_, err := tx.Exec("UPDATE test SET value = $1 WHERE id = 1", value)
		if err != nil {
			return err
		}
	}

	tx.Commit()
	return nil
}
```

Lets open the `psql` console and delete all rows in the table prior to running this example. 

```
test=# DELETE FROM test; 
DELETE 0
test=# SELECT * FROM test; 
 id | value 
----+-------
(0 rows)

``` 

Now, we compile and run this example, as expected we will get the value updated to 750. This was the sequential run.

```
parallels@ubuntu:~/gocode/src/bb/seq$ ./seq 
parallels@ubuntu:~/gocode/src/bb/seq$ sudo -u postgres psql 
psql (9.4.10)
Type "help" for help.

postgres=# \c test
You are now connected to database "test" as user "postgres".
test=# SELECT * FROM test; 
 id | value 
----+-------
  1 |   750
(1 row)
```

### Let make it a bit concurrent.. 

Now, let’s try to modify the program to do execution with the same `dofn` but now executing the updates concurrently: 

```
package main

import	(
	"log"
    "database/sql"
    "github.com/jmoiron/sqlx"
    _ "github.com/lib/pq"
)

func main() {
	for i := 0; i < 3; i++ {
		go launchDoFn()
	}
    
    for {
        time.Sleep(time.Second)
    }
}

func launchDoFn() {
	for i := 0; i < 250; i++ {
		err := dofn()
		if err != nil {
			log.Println("err", err)
			continue
		}
	}	
}
```

As you see we launch three goroutines - each of which start sequential updates of 250 increments (including one insert). There are basically three concurrent threads you can assume which are updating DB. So let’s compile and run and see the result. 

First, we will again clear the table so that it is empty again: 

```
postgres=# \c test
You are now connected to database "test" as user "postgres".
test=# DELETE FROM test; 
DELETE 1
test=# SELECT * FROM test; 
 id | value 
----+-------
(0 rows)
```

Now run it and check the result: 

```
parallels@ubuntu:~/gocode/src/bb/conc$ ./conc 
2016/12/01 19:06:20 err INSERT pq: duplicate key value violates unique constraint "test_pkey" pq: duplicate key value violates unique constraint "test_pkey"
2016/12/01 19:06:20 err pq: duplicate key value violates unique constraint "test_pkey"
parallels@ubuntu:~/gocode/src/bb/conc$ sudo -u postgres psql 
psql (9.4.10)
Type "help" for help.

postgres=# \c test
You are now connected to database "test" as user "postgres".
test=# SELECT * FROM test; 
 id | value 
----+-------
  1 |   464
(1 row)
```

Note: your output may be different - since it is unpredictable but in all likelihood it would be incorrect i.e. not equal to 750. 


### So what happened? 

Well, something like this: 

time t0: Thread A reads N = 5, Thread B also reads N = 5 
time t1: Thread A increments N = N + 1 = 6, Thread B increments N = N + 1 = 6 
time t2: Thread A writes into DB N = 6
time t3: Thread B writes into DB N = 6

So instead of the final value = 7 we get the value = 6. This is happening multiple times and hence causing the mismatch between expected and actual results. 

Now, we will discuss how to fix this.

### A primer on isolation levels 

Since this is a common problem, SQL standard defines a way to solve this. Basically there are four levels of transaction isolation that are defined. As mentioned on [postgres documentation page](https://www.postgresql.org/docs/9.1/static/transaction-iso.html):

The most strict (isolation level) is Serializable, which is defined by the standard in a paragraph which says that any concurrent execution of a set of Serializable transactions is guaranteed to produce the same effect as running them one at a time in some order. The other three levels are defined in terms of phenomena, resulting from interaction between concurrent transactions, which must not occur at each level.

* dirty read: A transaction reads data written by a concurrent uncommitted transaction.

* nonrepeatable read: A transaction re-reads data it has previously read and finds that data has been modified by another transaction (that committed since the initial read).

* phantom read: A transaction re-executes a query returning a set of rows that satisfy a search condition and finds that the set of rows satisfying the condition has changed due to another recently-committed transaction.


| Isolation Level | Dirty Read | Non repeatable Read | Phantom Read | 
| --------------- | ---------- | :-----------------: | ------------:|
| Read Uncommitted | Possible | Possible | Possible | 
| Read committed | Not possible | Possible | Possible | 
| Repeatable Read | Not Possible | Not Possible | Possible |  
| Serialisable | Not Possible | Not Possible | Not Possible | 

In PostgreSQL, you can request any of the four standard transaction isolation levels. But internally, there are only three distinct isolation levels, which correspond to the levels Read Committed, Repeatable Read, and Serializable. 

Also, 

> .. phantom reads are not possible in the PostgreSQL implementation of Repeatable Read, so the actual isolation level might be stricter than what you select.

This is permitted by the SQL standard: the four isolation levels only define which phenomena must not happen, they do not define which phenomena must happen. 

### Take away! 

* When we set the isolation level to serializable then behind the scene all concurrent updates to a database are performed in serial fashion i.e. one after the other. This clearly translates into bad performance.
* When we set the isolation level to Repeatable read then behind the scene, database makes sure that of all transactions which are currently executing on a record read in "past" and trying to update the record (pay attention here: say A, B are executing, both read record #3 in table (i.e. did a 'select'), then try writing into database updated values) - only one of them is guaranteed to succeed and others will fail with "could not serialize access due to concurrent update" error. This is better performant than serializable, fixes the problem of correctness but means that you have to repeat the transactions which fail.
* Read committed is the default mode of postgres. It is safe when you are concurrently reading but not safe for concurrent updates.

### Fixing using repeatable read isolation level 

As mentioned above, in postgres repeatable read is as strict as serialisable and is more performant than serialisable. So, we will use this isolation level to fix the problem we faced while running concurrent updates. 

We re-write our code for dofn():

```
func dofn() error {
	db, err := sqlx.Open("postgres", "host=127.0.0.1 port=5432 user=postgres password=postgres dbname=tokopedia-advertise-aws sslmode=disable")
	if err != nil {
		log.Println("err", err)
		return err
	}

	defer db.Close()

	var has_committed bool = false

	for ok := true; ok; ok = !has_committed {
		tx, err := db.Begin()
   		if err != nil {
       		log.Println("err", "error begining transaction in postgres", err)
       		return err 
   		}

		defer tx.Rollback()
 		_, err = tx.Exec(`set transaction isolation level repeatable read`)   // <=== SET ISOLATION LEVEL
		if err != nil {
   			return err
		}

		var isExists bool = true
		var id int 
		var value int
		err = tx.QueryRow("select * from test1 where id = 1;").Scan(&id, &value)
		if err != nil {
			if err == sql.ErrNoRows {
				isExists = false
			} else {
				return err
			}
		}

		if !isExists {
			_, err := tx.Exec("INSERT INTO test VALUES(1, 1)")
			if err != nil {
           		 return err
            }
		} else {
			value = value + 1
			_, err := tx.Exec("UPDATE test set value = $1 where id = 1", value)
			if err != nil {
			if strings.Contains(err.Error(), "could not serialize access due to concurrent update") {
                continue
            } else {
            	return err
            }
		}
	}

	tx.Commit()
	has_committed = true
	}
	return nil
}
```

Let's again empty the table, compile and run this and see results: 

```
test=# DELETE FROM test; 
DELETE 1
test=# SELECT * FROM test; 
 id | value 
----+-------
(0 rows)
```
```
parallels@ubuntu:~/gocode/src/bb/conc$ ./conc
2016/12/01 19:28:06 err UPDATE pq: could not serialize access due to concurrent update pq: could not serialize access due to concurrent update
2016/12/01 19:28:06 err UPDATE pq: could not serialize access due to concurrent update pq: could not serialize access due to concurrent update
...
...
... -- many errors and retries --- 
... 
parallels@ubuntu:~/gocode/src/bb/conc$ sudo -u postgres psql 
psql (9.4.10)
Type "help" for help.

postgres=# \c test
You are now connected to database "test" as user "postgres".
test=# SELECT * FROM test; 
 id | value 
----+-------
  1 |   750
(1 row)
```

Hence, we see the issue of correctness is resolved and with this change our program is concurrency safe wrt database update. 


