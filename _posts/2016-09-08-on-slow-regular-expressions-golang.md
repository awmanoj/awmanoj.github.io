---
layout: post
title: On slow regular expressions in golang
author: Manoj Awasthi
categories: tech
---

Few months back, a colleague mentioned over lunch [how slow golang regular expressions are](http://benchmarksgame.alioth.debian.org/u64q/compare.php?lang=go&lang2=python3) - this was not only compared to awesome performance of Perl regex or PCRE but also other languages. Recently, while looking for threads discussing Go vs Python wrt regex I got hold of following comments (by [Russ Cox](https://swtch.com/~rsc/)): 

>First of all, Ruby and Python are using C implementations of the regexp search, so Go is being beat by C, not by Ruby.

More importantly, 

>Second, Go is using a different algorithm for regexp matching than the C implementations in those other languages. The algorithm Go uses guarantees to complete in time that is linear in the length of the input. The algorithm that Ruby/Python/etc are using can take time exponential in the length of the input, although on trivial cases it typically runs quite fast. In order to guarantee the linear time bound, Go's algorithm's best case speed a little slower than the optimistic Ruby/Python/etc algorithm. On the other hand, there are inputs for which Go will return quickly and Ruby/Python/etc need more time than is left before the heat death of the universe. It's a decent tradeoff.

Take away 1: Fair enough!

Take away 2: Prefer using `strings` functions if your job gets done with that. Didn't authors already tell us Go is an opinionated language! 
