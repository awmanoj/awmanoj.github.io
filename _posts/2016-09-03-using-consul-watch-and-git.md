---
layout: post
title: Using consul watch and git diff to track key value changes
author: Manoj Awasthi
categories: tech
---

In the [previous post](https://awmanoj.github.io/tech/2016/08/27/service-discovery-configuration-management-with-consul/) I introduced how consul is a simple, distributed and fault tolerant system for service discovery and configuration management.

Now, a common need is how do we track if someone has changed a key value or added or deleted a key? Solution is `consul watch` command. We can use [`consul watch`](https://www.consul.io/docs/agent/watches.html) for registering a handler which gets called whenever there is a key-value change. Its format is: 

{% highlight bash %}
$ consul watch -http-addr consul.service.local:8500 -type keyprefix -prefix global /usr/local/bin/watch_handler.sh
{% endhighlight %} 

But there is a problem - everytime there is a change in a key value e.g. key `global/redis/cacheA` is changed from `192.168.1.1` to `192.168.1.2` then consul watch handler gets invoked with latest values only i.e. `192.168.1.2`. 

We felt that there is a need to know the change in full context i.e. `old_value` and `new_value`. After some searching and not finding a solution we devised a method (rather novel I'd say but I am blinded by bias) by using `git` - the most awesome change control system - in local mode (no `remote` set). 

Idea is simple. 

Following steps are to be done as part of the initial setup: 

* On the node where you want to setup the alert system designate a directory say `/var/consul/data`. 
* Run following command to initialize this directory as a git repo.

{% highlight bash %}
$ git init
{% endhighlight %}

* Run following command to save the latest snapshot of all key values for keys starting with prefix `global` to `kvs.txt`.

{% highlight bash %}
$ consul watch -http-addr consul.service.local:8500 -type keyprefix -prefix global > kvs.txt
{% endhighlight %}
 
* Add and commit to the local git repo

{% highlight bash %}
$ git add kvs.txt 
... 
$ git commit -m "Update kvs.txt"
...
{% endhighlight %}

Once the above setup is done - run the `consul watch` in daemon mode (via upstart preferably): 

{% highlight bash %}
$ consul watch -http-addr consul.service.local:8500 -type keyprefix -prefix global /usr/local/bin/watch_handler.sh
{% endhighlight %}

This ensures that when there is a change in any of the key values under the specified prefixed keys then it will trigger the execution of `watch_handler.sh`. This script does following on each such trigger:

* Change directory to `/var/consul/data/`. 
* Run following command to get the latest (post change) key values: 

{% highlight bash %}
$ consul watch -http-addr consul.service.local:8500 -type keyprefix -prefix global > kvs.txt
{% endhighlight %}

* Execute `git diff`. 

> If empty (no change) then don't do anything else trigger your alert mechanism with the output of `git diff` command (e.g. mail, [slack](https://api.slack.com/incoming-webhooks), [mattermost](https://docs.mattermost.com/developer/webhooks-incoming.html) etc.)

* Now that the alert is already sent - commit the new change locally. 

{% highlight bash %}
$ git add kvs.txt 
... 
$ git commit -m "Update kvs.txt"
...
{% endhighlight %}

Note: You will need to do a bit more actually than just `git diff` since the diff command gives values in `base64 encoding`. To convert that into readable values you will need to use `base64` utility and some play with `sed`, `awk` etc. Following is some code which can be handy: 

{% highlight bash %}
OLD=`git diff | sed -e 's/\"//g' | grep Value | awk -F'Value:'  '{print $2}' | awk -F, '{print $1}' | awk '{print $1}' | head -1 | base64 -d | sed -e 's/^/>/g' | sed -e 's/$/\\n/g'` 
NEW=`git diff | sed -e 's/\"//g' | grep Value | awk -F'Value:'  '{print $2}' | awk -F, '{print $1}' | awk '{print $1}' | sed '2q;d' | base64 -d | sed -e 's/^/>/g' | sed -e 's/$/\\n/g'`
{% endhighlight %}

This is a screenshot of the change alert in our slack: 

![Slack consul change alert screenshot](/public/assets/img/consul-watch.png)

### Epilogue 

In addition to providing the full context of change, this solution is also useful in keeping a local versioned history and is helpful in reverting to older values easily by using `git revert` or `git reset ... ` commands. 

Credit to use of `git` for this goes to [@kernelhacker](https://twitter.com/kernelhacker).

I don't yet know if there is any other builtin technique in `consul-watch` to achieve the same thing. If you know please drop a tweet at [@awmanoj](https://twitter.com/awmanoj). TIA. 

