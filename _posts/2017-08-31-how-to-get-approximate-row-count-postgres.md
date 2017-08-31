---
layout: post
title: How to get approximate row count in postgres tables?
author: Manoj Awasthi
categories: tech
---

If there is a table with tens of millions of rows in a postgres table then following query will be very slow (non interactive):

```
SELECT COUNT(*) FROM SAMPLE 
```

Following can help us get an "approximate" count of rows in postgres: 

```
SAMPLEDB=> SELECT reltuples::BIGINT AS estimate FROM pg_class WHERE relname = 'SAMPLE'; 
 estimate 
----------
 54296044
(1 row)
```

Ref: [https://wiki.postgresql.org/wiki/Count_estimate](https://wiki.postgresql.org/wiki/Count_estimate)
