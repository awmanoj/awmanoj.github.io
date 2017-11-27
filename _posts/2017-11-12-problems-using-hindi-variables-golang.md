---
layout: post
title: Problems using Hindi variables in golang 
author: Manoj Awasthi
categories: tech
---

`Go` is a beautiful programming language. It is not so due to its many features but rather due to lack of them. 

It has managed to remain simple, performant, backward compatible (it's a promise by creators, think [python 2.7 vs 3.0 tussle](https://learntocodewith.me/programming/python/python-2-vs-python-3/)) and has some unique features (compared to popular modern programming languages) like the support for concurrent programming (channels, goroutines). 

Apart from the main features of the language, there are some cute little things which go provides. One of them is that your variable names can be composed of any unicode characters. What does that mean? This means, following program containing simplified chinese characters is a valid program which compiles and runs fine (you can try):

```
import (
    "fmt"
)

func main() {
    量 := 5000    // Amount
    速率 := 10    // Rate
    持续时间 := 6 // Duration

    总 := (量 * 速率 * 持续时间) / 100 // Total

    fmt.Println(总)
}
```

Now, that is some fun. I thought why not try my mothertongue, Hindi (Devanagri script), so coded following: 

```
func main() {
    मूलधन := 5000
    दर := 10
    समय := 6 // वर्षमें

    fmt.Println((मूलधन * दर * समय)/100)
}
```

On compiling: 

```
Manojs-MacBook-Pro:test2 manojawasthi$ go build
# _/Users/manojawasthi/gocode/src/github.com/awmanoj/test2
./main.go:8: invalid identifier character U+0942 'ू'
./main.go:12: invalid identifier character U+0942 'ू'
```

Heartbroken! 

On further search on the error and the issue, I found that I wasn't alone (ofcourse). 

![Hindi golang issue screenshot](/public/assets/img/hindi-golang.png)

This was also [discussed in golang-nuts](https://groups.google.com/forum/#!topic/golang-nuts/yJRJRtEEA0E) and Nathan Kerr added a response: 

> The Go team knows this a problem (see the issues listed by Rob Pike). The difficultly in fixing it is that the ecosystem surrounding Go relies on the current definition for identifiers. There is no way to add the characters you need without breaking other tools like syntax highlighters. It is a socioeconomic problem than a technical one. The next time it can be fixed in Go is for Go 2.0 and is already on the list of issues to be considered.
> If your main goal is to write programs in your language, you might consider using one from https://en.wikipedia.org/wiki/Non-English-based_programming_languages

There are also some good comments by [Bakul](https://github.com/bakul) on this thread. He discusses briefly the intricacies involved and why this would be rather low priority item even for Go 2.0 to support Hindi (while it is in plan). I concur.

For the curious, these are some of the issues related to this (latter added by Rob Pike): 

[https://github.com/golang/go/issues/5167](https://github.com/golang/go/issues/5167)
[https://github.com/golang/go/issues/20706](https://github.com/golang/go/issues/20706)

So, while "not holding my breath", will still be looking out for Go2.0 to support hindi names (Devanagri script).

Note: Without the problematic characters, things worked well. 

