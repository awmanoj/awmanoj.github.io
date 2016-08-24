---
layout: post
title: Simple service discovery and configuration management with Consul 
date:   2016-08-22 07:20:22 +0700
author: Manoj Awasthi
categories: tech
---

[Consul](https://www.consul.io/) is a service discovery and configuration management solution by Hashicorp, the company which also developed [Vagrant, Vault and many other popular products](https://www.hashicorp.com/#products). Consul is written in golang, is unbelievably simple to setup, scalable and fault-tolerant. While it is in the same league of softwares as [etcd](https://coreos.com/etcd/), [zookeeper](https://zookeeper.apache.org/), etc. it has a distinguishing support for multiple data centres. 

In this post I will touch upon how can you setup a consul cluster and consul web UI. I’ll also talk briefly about how can you use it as a configuration management solution using consul-template. 

### Quick reference to concepts 

* A Consul cluster consists of one or more nodes running consul service in `server` mode. 
* A Consul service instance can run in either `server` or `client` mode.
* A Consul cluster needs a quorum (majority) of nodes for master election.
* At the minimum, you should create a cluster of 3-5 nodes for high availability.
* Consul cluster uses [Gossip protocol](https://www.consul.io/docs/internals/gossip.html) for cluster communication (membership, broadcasting) and it is an eventually consistent protocol.

### Setting up Consul 

Consul comes as a prebuilt binary and can be downloaded from [the download page at Hashicorp](https://www.consul.io/downloads.html) (choose the binary per host operating system). You can follow the [quickstart guide](https://www.consul.io/intro/getting-started/agent.html) on consul.io site to play with consul. 

If you're setting it up on production then I will recommend a very detailed guide by digital ocean on [setting up consul in production environment for ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-configure-consul-in-a-production-environment-on-ubuntu-14-04). 

### Setting up consul web UI 

You can choose to setup webUI on one or all of the consul server machines putting them behind a load balancer OR you can choose a separate machine for it. We preferred a separate instance. 

Consul web UI requires a tar.gz file that can be downloaded from [here](https://releases.hashicorp.com/consul/0.6.4/consul_0.6.4_web_ui.zip). Extract it on some location on your web UI machine or server. Then start consul service as “agent” with -ui flag set in the config. 

Additionally you may want to put it behind nginx (which is almost always useful). You also need to put it behind load balancer and ensure proper security policies. 

Once the setup is done, open your favourite browser and browse through the webUI. 

### Using Web UI 

![consul-web-ui](/public/assets/img/consul_default.png)

* From the UI you can inspect all services. The first service which you already notice running is consul itself. You can quickly inspect from command line the DNS and HTTP interface to inspect these: 

* You can also add and remove key value pairs. these are particular to a data centre. but there are ways to replicate them across data centres if the need be - you just need to run another service - [consul-replicate](https://github.com/hashicorp/consul-replicate).

* Web UI also displays other useful information about consul cluster, services e.g. health of nodes based on check, health of a service (are all or some nodes providing the service are down?), etc. 

### Using consul for configuration management (introducing consul-template)

Once the consul is setup, we can use it to store key value pairs. Complete CRUD for the key value store is available via consul web UI which makes it super easy to operate consul. If you prefer, there is an HTTP based REST interface too for the CRUD operations. 

Keys have hierarchical structure like a file path. So either a key component is a `folder` or a `key` (leaf). A `folder` can contains other `folders` or other `keys`. A `key` contains a `value`. 

It's a general practise in software development to keep services' IP addresses, application specific values which can be changed at runtime, access keys etc. externally configurable. This is desirable since you don't want to recompile and release application all the time (actually, you want to minimize that to absolute minimum rather). Let's take a concrete example of where consul can help you. 

E.g. suppose we have two redis instances 192.168.1.1:6379 and 192.168.1.2:6379 and your configuration looks something like this: 

file: db.ini
{%highlight bash%}
[redis]
  cacheA = 192.168.1.1:6379
  cacheB = 192.168.1.2:6379

[mongo]
  ... 
{% endhighlight %}

In this case, anytime there is a change in the redis IP addresses then either: 
 
* you have to go manually edit this file on each of the instances on which your app runs OR 
* you need to repackage the application and release it. 

Both are painful and error prone.

Let's now see the scenario if we use consul. 

Now, we can instead create two keys: `service/redis/cacheA=192.168.1.1:6379` and `service/redis/cacheB=192.168.1.2:6379`. We also remove the config file `db.ini` altogether (or rename it to a `db.ini.sample` for backup). We add instead a template file (`go template`) like following:

file: db.ini.ctmpl
{% highlight bash %}
{% raw %}
[redis]
  cacheA = {{ key service/redis/cacheA }}
  cacheB = {{ key service/redis/cacheB }}

[mongo]
  ... 
{% endraw %}
{% endhighlight %}

Now, on each of the app servers we run a daemon `consul-template` ([download it here](https://github.com/hashicorp/consul-template)) with following format: 

{% highlight bash %}
$ consul-template -consul 192.168.4.5:8500 -template="db.ini.ctmpl:/path/to/db.ini" 
{% endhighlight %}

This daemon generates the `/path/to/db.ini` dynamically whenever there is a change in values referenced and hence keeps the configuration updated all the time. 

I can read your mind - config changed but who reloads the application? Well, you will need to write a post-step in a configuration file for that. [Read](https://github.com/hashicorp/consul-template) consul-template github README in detail - it has all the answers and more. 

Go templating language is very comprehensive and you can generate almost any configuration that you want. In addition there is a support for plugins written in golang which bring almost infinite power to what you can achieve. 

I will write about service discovery with consul, consul watch and consul-datadog integrations in some other post later (this is already getting long). 

If you have any questions on this and are not able to find easily over internet - you can shoot your question at me and I may try to find the answer for you - reach me on [twitter](https://twitter.com/awmanoj) and mail (last_name . first_name @ gmail.com). 

Happy Consul!
