---
layout: post
title: A quick primer into isolation levels in databases
author: Manoj Awasthi
categories: tech, database, systems, concurrency
---

Transaction Processing is one big source of complexity in correctly designing concurrent applications working with database systems. 

> A transaction is defined as one indivisible logical entity comprising usually of more than one operations which are meant to be considered a single step that either is success (a 'commit') or a failure (an 'abort'). This unit of work should act as if the operations are atomic. 

This is quite important to keep the state of database in a consistent state at all the time and being adherant with ACID semantics. 

I will not go into the examples of transactions in databases assuming the familiarity of the reader to them. There are zillions of articles talking about concurrent processing of bank accounts and why it is so important to ensure atomic behavior.

In this article, we talk in the context of concurrent processing on the databases systems.

# What is an isolation level? 

An isolation level specifies 'how' and 'when' parts of the transaction can and should become visible to other concurrently executing transactions.

> Isolation levels describe the degree to which transactions are isolated from other concurrently executing transactions, and what kind of anomalies can be expected in each.

# Possible Read and Write anomalies specified SQL standard

## Read anomalies

* A **dirty read** is a situation in which a transaction can read uncommitted changes from other transactions. 
* A **nonrepeatable read** is a situation in which a transaction queries the same row twice and gets different results. For example, a transaction T1 reads a row, then transaction T2 modifies the row and commits this change. Now, if T1 requests the row again then the result will be different from previous read. 
* A **phantom read** is a situation a transaction queries the same *set of rows* twice and receives different results. It is similar to nonrepeatable reads but only for range queries.

## Write anomalies

* A **lost update** happens when transaction T1 and T2 both attempt to update the value of V. T1 and T2 read the value of V. T1 updates V and commits and T2 updates V after that and commits as well. Since the transactions are not aware about each other so the result of T1's update will be overwritten by T2's update.
* A **dirty write** is a situation in which one of the transactions takes an uncommitted value (i.e. a dirty read), modifies it and commits it.
* A **write skew** is a situation when each individual transaction respects the required variants, but their combination does not satisfy these invariants. 

# Isolation Levels 

The lowest isolation level (or the weakest) is **Read Uncommitted** which allows dirty-read.

The next isolation level is **Read committed** that allows transactions only to read committed changes. Althought this isolation level does not allow dirty reads but still suffers from nonrepeatable reads and phantom reads.

The next isolation level is **Repeatable Read** that does not allow dirty read and also takes care of nonrepeatable reads but suffers still from phantom reads.

The strongest isolation level is ofcourse **Serializablle** where all transactions are executed serially. This has significant negative impact on performance and hence not a practical choice.

# Summary 

<img src="/public/assets/img/tx-isolation.png" />

So, for most cases **Repeatable Read** would be the go-to isolation level to operate in. You can read more about [Performing concurrency-safe DB updates in postgres using 'repeatable read' isolation level](https://awmanoj.github.io/tech/2016/11/30/performing-concurrency-safe/). 

Happy and safe coding!
