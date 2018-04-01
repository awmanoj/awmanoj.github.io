---
layout: post
title: Undoing git reset hard (remove local commits)
author: Manoj Awasthi
categories: tech
---

Although this post is being published on 01 April celebrated as April Fool's Day and known for pranks all around, you can trust this post.

This was a long weekend and I sat down to code something that I planned for quite sometime but didn't get time to. After finishing with the coding and testing I prepared the git repo and pushed the code in there. 

```
$ git add . 
$ git commit -m "Initial commit." 
... 
$ git push origin master 
...
```

This was taking long time and I realized, inadvertently, I committed some binary embedded databases (big size for git). That is when I thought of quickly removing the commit and re-commit again: 

```
$ git reset --hard HEAD~1 
```

As a side effect, all my local files were gone too. Yes, all work that went into over the weekend. :-(

For sometime I was shaken but then a quick search over google led to [this QnA](https://stackoverflow.com/questions/5473/how-can-i-undo-git-reset-hard-head1/6636) on stackoverflow. This helped! :-) 

Following taken verbatim from the above QnA post summarizes what to do: 

```
$ git init
Initialized empty Git repository in .git/

$ echo "testing reset" > file1
$ git add file1
$ git commit -m 'added file1'
Created initial commit 1a75c1d: added file1
 1 files changed, 1 insertions(+), 0 deletions(-)
 create mode 100644 file1

$ echo "added new file" > file2
$ git add file2
$ git commit -m 'added file2'
Created commit f6e5064: added file2
 1 files changed, 1 insertions(+), 0 deletions(-)
 create mode 100644 file2

$ git reset --hard HEAD^
HEAD is now at 1a75c1d... added file1

$ cat file2
cat: file2: No such file or directory

$ git reflog
1a75c1d... HEAD@{0}: reset --hard HEAD^: updating HEAD
f6e5064... HEAD@{1}: commit: added file2

$ git reset --hard f6e5064
HEAD is now at f6e5064... added file2

$ cat file2
added new file
```

Particularly: 

```
$ git reflog
1a75c1d... HEAD@{0}: reset --hard HEAD^: updating HEAD
f6e5064... HEAD@{1}: commit: added file2

$ git reset --hard f6e5064
HEAD is now at f6e5064... added file2
```

Life saver! :-) 

