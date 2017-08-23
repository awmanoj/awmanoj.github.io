---
layout: post
title: So how slow can golang regular expressions be? 
author: Manoj Awasthi
categories: tech
---

Sometime back, I wrote [on slow regular expressions in golang](https://awmanoj.github.io/tech/2016/09/08/on-slow-regular-expressions-golang/). Recently, I thought about experimenting with regular expressions and use Golang [benchmarking](https://dave.cheney.net/2013/06/30/how-to-write-benchmarks-in-go) to estimate their slowness. 

Disclaimer: it’s not a generalisation (please read the previous post to understand the context of the claim). 

## Experiment 

For the experiment, at hand was a problem to identify whether the connection string to redis followed a certain pattern. 

```
120 // IsRedisConnectionStringType2 checks if connectionString is host:port format and
121 // not the tcp://user:pass@host:port/0?q=1 format.
122 // Following are the types of connection strings which xuyu/goredis supports:
123 //      Dial(&DialConfig{"tcp", "127.0.0.1:6379", 0, "", 10*time.Second, 10})
124 //      DialURL("tcp://auth:password@127.0.0.1:6379/0?timeout=10s&maxidle=1")
125 // Connection type2 example: 192.168.17.196:6379, session-redis.service.biznet.consul:6380
```

Problem is solvable using following regular expression:

```
  8 var s2 = "127.0.0.1:6379”
  9 func IsRedisConnectionStringType2Fn1() {
 10     regexp.MatchString("^(\\d+\\.\\d+\\.\\d+\\.\\d+):(\\d+)$", s2)
 11 }
```

To compare if there will be a gain by replacing this with “less intricate”, “less powerful” but possibly “enough” heuristics based check, wrote this function: 

```
13 func IsRedisConnectionStringType2Fn2(connectionString string) bool {
14     // if starts with tcp:// then not a type2 URL
15     if strings.Index(connectionString, "tcp://") == 0 {
16         return false
17     }
18 
19     // if more than two components separated by ':' then not a type2 URL
20     splits := strings.Split(connectionString, ":")
21     if len(splits) != 2 {
22         return false
23     }
24 
25     return true
26 }
```

We use golang benchmarking support (such a beautiful language!) to benchmark both versions: 

```
import "testing"

func BenchmarkIsRedisConnectionStringType2Fn1(b *testing.B) {
    for n := 0; n < b.N; n++ {
        IsRedisConnectionStringType1()
    }
}

func BenchmarkIsRedisConnectionStringType2Fn2(b *testing.B) {
    for n := 0; n < b.N; n++ {
        IsRedisConnectionStringType2(s2)
    }
}
```

The results: 

```
parallels@ubuntu:~/gocode/src/github.com/awmanoj/test2/test$ go test -bench=.
BenchmarkIsRedisConnectionStringType2Fn1-2    100000	    15139 ns/op
BenchmarkIsRedisConnectionStringType2Fn2-2   5000000	      378 ns/op
PASS
ok  	github.com/tokopedia/test2/test	3.968s
```

Here, strings based heuristics checks is 40x faster than the regex version. Is it equivalent (apple to apple)? not at all. 

## Should we fret over this?

Not necessarily — in fact most probably, need not. 

If your code is not hit that often it doesn’t make sense to fret over something which is still imperceptible (ns per operation here) in terms of runtime. But if it is a code which is expected to be hit very frequently (e.g. for every hit to the server) — you should evaluate if there could be a less stringent and complex alternative. 

Is it equivalent? Not at all. In all probability — this heuristics based check will not be as powerful and comprehensive as the regular expression check so it is not always possible to replace the regex with such heuristics check. But when possible it can be a good replacement. 

## Advice 

Though there is good advice hidden in this post which can be useful at times, always remember this nugget:

> We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil. 
>     -- Donald Knuth 


