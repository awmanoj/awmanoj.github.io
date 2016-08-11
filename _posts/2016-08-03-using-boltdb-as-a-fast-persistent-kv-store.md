---
layout: post
title: Using Boltdb as a fast, persistent key value store
---

<hr/>
_Originally posted on [Tokopedia Tech Blog](http://tech.tokopedia.com/blog/using-boltdb-as-a-fast-persistent-kv-store/)_
<hr/>

Recently, we at [Tokopedia](https://www.tokopedia.com/) rearchitected a [golang](https://golang.org/) microservice which maps an Image (as in picture, photo, ..) ID to the fully qualified server name that hosts the image.

### A bit of context is always good..

Historically, we stored images and served images from multiple servers for reasons of scale. The mapping of which image is located on which server was in mongodb. And there was a layer of caching on top of it in redis, because mongo didn't scale very well for the workload.

Gradually, we moved stuff to s3, which means all images uploaded from that point onwards could now be served from a single server, and no such mapping was required. However, old images were still being served in the same way, and they needed the mapping. The difference was, our database was now read only and fixed size. 

We still occasionally suffered with memory spikes and service killed by linux oom-killer leading to both latency and downtimes. 

### Search for alternative.. 

Requirements for the service: 

* Persistent storage
* Fast retrieval (tens of thousand QPS)
* `Read only` usage 

Since our requirement is also "read-heavy" usage on a fixed size database (infact, it is read-only usage) we knew we need a lightweight embedded DB solution. There are multiple solutions that exist for this problem but we zeroed on [Boltdb](https://github.com/boltdb/bolt).

### Why Boltdb?
* written in [go](https://golang.org/) and hence fits with rest of the stack well.
* compact, fast (it based on [LMDB](https://symas.com/products/lightning-memory-mapped-database/). Both use a B+tree, have ACID semantics with fully serializable transactions, and support lock-free MVCC using a single writer and multiple readers.
* LMDB heavily focuses on raw performance while Bolt has focused on simplicity and ease of use.
* fits better for a `read heavy` usage.

### How to use?

In pure sense, `boltdb` is not really a database but simply a memory mapped file but it provides ACID semantics as provided by databases so calling it a DB is not misnomer. It comes as a library so installation is as simple as `importing` a go package.

You can go through the detailed [README](https://github.com/boltdb/bolt) and [GoDoc](https://godoc.org/github.com/boltdb/bolt) to get started.

Opening database is as simple as: 

{% highlight go %}
db, err := bolt.Open("my.db", 0666, &bolt.Options{ReadOnly: true})
if err != nil {
    log.Fatal(err)
}
{% endhighlight %}

Add a key value (`answer` = `42`):

{% highlight go %}
db.Update(func(tx *bolt.Tx) error {
    b := tx.Bucket([]byte("MyBucket"))
    err := b.Put([]byte("answer"), []byte("42"))
    return err
})
{% endhighlight %}

Fetch a value by key: 

{% highlight go %}
db.View(func(tx *bolt.Tx) error {
    b := tx.Bucket([]byte("MyBucket"))
    v := b.Get([]byte("answer"))
    fmt.Printf("The answer is: %s\n", v)
    return nil
})
{% endhighlight %}

### Command line utility - `bolt`
 
Boltdb comes with a command line utility which can be used to inspect the correctness and statistics of a BoltDB file. 

### If life was that easy.. ! 

We imported all `mongodb` data using `mongo-export` utility. It was `~4 GB` in size. We exported it to `boltdb` using an export tool we wrote in `golang`. The output file was a single `bolt.db` file with size `~ 13 GB`. This size increase was expected due to the internal data structure being used by `boltdb`. 

What was not expected (or rather **we** did not expect it) was that export tool became exponentially slower as the `bolt.db` file grew larger. We estimated that each export would take days in this manner (in case we need to).

To solve this problem, we decided to use - you guessed it right - `sharding`. 

- We broke down the initial `mongodb` dump into `N` files ([choosing `N`](http://math.stackexchange.com/questions/183909/why-choose-a-prime-number-as-the-number-of-slots-for-hashing-function-that-uses)).
- We export each of those files into a boltdb file. We now have `N` small boltdb files. 
- When a query arrives for a key `K`, we hash the key into a bucket between `0` and `N-1`. This hash value tells us in `O(1)` time which boltdb to query on.
- Query itself is fast due to storage structure and you're done!
 
This helped us export the data and generate BoltDB files in less than an hour time.  

Following is the output of `free -m` on one of the `3` servers we use:

![free -m](/public/assets/img/free-m.png)

and this is the snippet from `top` on the same server:

![top](/public/assets/img/virtual-mem.png)

Note that the `VIRTUAL` memory is exactly the total size of boltdb. Also note from above two outputs that it is not limited by physical RAM availability. 

### Caveats & Limitations 

* Bolt is good for read intensive workloads. Random writes can be slow especially as the database (file) size grows.
* Bolt uses a B+tree internally so there can be a lot of random page access. SSDs provide a significant performance boost over spinning disks. 
* Bolt can handle databases much larger than the available physical RAM, provided its memory-map fits in the process virtual address space. It may be problematic on 32-bits systems.
* The data structures in the Bolt database are memory mapped so the data file will be endian specific. This means that you cannot copy a Bolt file from a little endian machine to a big endian machine and have it work. For most users this is not a concern since most modern CPUs are little endian.

### Result

We successfully migrated to using `boltdb` for our service. The service handles many thousands of QPS, is not limited by physical RAM on the machine and is working well. 

`BoltDB` looks awesome. Do give it a try if this fits your use-case. 

