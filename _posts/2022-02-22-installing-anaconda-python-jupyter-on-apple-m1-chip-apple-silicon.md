---
layout: post
title: Installing Anaconda for Python/Jupyter on Apple M1 chip (Apple Silicon)
author: Manoj Awasthi
categories: tech
---

While the world is busy celebrating the little joys like the fact that today is a palindrom date (22022022 when written as DDMMYYYY; this is likely why aliens don't visit us!), I ended up trying to set up Python/Jupyter on my new Macbook Pro with Apple Silicon.

The machine is fantastic and I love that it is quieter, cooler (despite running more tabs on chrome than the previous macbook pro on Intel chip) and faster. The only challenge is now I have to specifically look for ARM64 support or M1 support etc when I look for installation packages. Not a big deal though. Most software packages are making it a point to ensure a page or section dedicated for M1 support. 

I already set up an ubuntu running on my Parallels virtual machine on the mac. Ubuntu is non negotiable! wait, let me not digress.

--- 

So following is a fork for original github page lest it disappear.

https://github.com/awmanoj/miniforge

This page contains conda packages for M1 Silicon.

Direct link - https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-ppc64le.sh

Install pip3:

```
$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
$ python3 get-pip.py
```

Install jupyterlab:

```
$ pip3 install jupyterlab
```

Install notebook:

```
$ pip3 install notebook
```


Have fun!

