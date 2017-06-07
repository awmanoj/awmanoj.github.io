---
layout: post
title: Why do (some) engineers write bad code?
author: Manoj Awasthi
categories: tech
---

I've been reading <a target="_blank" href="https://www.amazon.com/gp/product/059600284X/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=059600284X&linkCode=as2&tag=awmanoj-20&linkId=8ee0e4238a7b93669231b2bff3c6939e">System Performance Tuning, 2nd Edition</a><img src="//ir-na.amazon-adsystem.com/e/ir?t=awmanoj-20&l=am2&o=1&a=059600284X" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> recently. Though the content is a bit dated as a reference (was written while Linux kernel 2.4 was coming up) but this is still a fantastic book on refreshing the workings of the components of a computer and knowing the tweaks for solaris and linux.

This post though, is focussed on a topic that is touched towards the end of the book. Why do we end up writing bad code?

Bad code could mean many things - bad design or architecture decision, badly written code that is harder to maintain (duplication, tightly coupled components, less modular etc.), code that is bad at performance (time or space) and so on. The book talks about following reasons for badly written code (most of it is verbatim with few modifications here and there):

* **Writing bad code is much faster than writing good code.** Good software engineering practices often trade an increase in initial development time for a decrease in the time required to maintain the software by eliminating bugs in the design phase or for a decrease in application runtime by integrating performance analysis into the development process. When faced with a deadline and irate management, it’s often easier to just get it done rather than do it well. 

> TIP: Learn to say No to "can we do this quickly" in favor of better thought of and more solid code or design. You will thank yourself for it more often than not. 

* **Implementing good algorithms is technically challenging.** This coupled with management’s desire for a product yesterday and the time to implement such an algorithm, often the good algorithms lose out.

> TIP: Invest in learning about common algorithms & data structures and weigh upon choices when you're faced with a problem. Almost always the first solution that strikes one's mind is not the best.

* **Good software engineers are difficult to find.** Software engineering is more than writing good source code. It involves architecture development, soliciting support from other development organisations, formal code review, ongoing documentation, version control etc. Concepts like pair programming are just beginning to move from the academic world into the mainstream and aren’t yet heavily practised. 

* **Legacy is hard to maintain.** It’s quite rare that a group of engineers sit down to develop a piece of software from scratch. More likely, they’re adding onto an existing framework, or developing the next version of the current product. The mistakes have already been made, and it’s not time effective (or cost effective) to throw everything and start over. Solution can be refactoring which makes the existing code more amicable to catching bugs, bottlenecks and improvements. 

Also, experience wise I find that, bad code gets copied more than good code (remember ‘it is implemented the same way in that file / module / project etc.’? at the workplace). While re-writes are hard, it’s always possible to refactor the code as and when possible to make it more friendlier to improvements. 

> TIP: The basic idea here is whenever you write a line of code, it should be refactored. If you’re rewriting pieces of legacy code, those should be refactored as well.

Good night!

