---
layout: post
title: Mattermost - an opensource self-hosted Slack Alternative
author: Manoj Awasthi
categories: tech
---

If you're even remotely in tech you would be knowing about and with high chance using [Slack](https://slack.com/) - now ubiquitous messaging system which has replaced (almost) [IRCs](https://en.wikipedia.org/wiki/Internet_Relay_Chat), [Jabber](https://www.jabber.org/) and other IM-like clients especially in corporates. 

In my current company, we have been using Slack extensively - free version.

Free version of Slack has some limitations (ofcourse, that's why it is free!):

* There is a limit to searchable items (upto 10,000 of your team's most recent messages)
* Upto 10 apps or service integrations 
* 5 GB of total file storage for the team.

Now, these are some serious limitations. Not all channels are made equal e.g. a channel for discussing `emergency response` would have a higher bearing on losing messages (entire history of discussions!) because of some random chats elsewhere than a general channel for discussing company gossip. In our case, we were losing history of the messages as soon as we consume the limits (10K messages for example) and this used to come without warning. 

### Need.. 

Till the time team is small (say ~ 50-100) you don't really have to worry about this as much but once the team grows beyond it then requirements arise to curb these limits. The solution is you pay for the service on Slack. This is exactly what we planned.

The pricing of Slack is `per user` and minimum pricing that is available from Slack is `$6 per user per month`. 

### Saviour.. 

That is when I got to know about [Mattermost](https://www.mattermost.org/) - an opensource self-hosted Slack alternative which is incredibly Slack-compatible. For more than a month, we have been using this alongwith the Slack to get a sense of "if we are not able to do a certain thing". Till now, functionality wise Mattermost is working for us well. 

Our setup of Mattermost is on AWS: 

* `1` `t2.medium` EC2 instance (with `1` `100G` EBS Volume)
* `1` RDS instance for postgres  

It provides following benefits: 

* No limit to number of messages (well, there is - but it is disk!) so no fear of losing out history.
* No limit to number of app of service integrations. 
* No limit to the storage.

(As pointed, limit is elastic - you can always go and upgrade your disk - also, disk is cheap)

It was simple to switch: 

* It is easy to setup. There is an awesome guide to [setup mattermost](https://www.mattermost.org/installation/) on its site.
* It is not only slack compatible - the interface is also similar. So no "learning curve" friction.
* We were able to move all our existing slack integrations to Mattermost because of awesome compatibility. In most cases (except runscope - which I will write about sometime), you will just need to replace Slack webhook url with the Mattermost webhook url. It's that simple. 
* Mattermost comes with both an Android and an iOS App.

![Mattermost](/public/assets/img/mm.png)

### Cost comparison..

The approximate per month cost (as per [this site](http://www.ec2instances.info/)) would be `$50` (EC2) + `$10` (100GB storage) + `$30` (`2GB` RAM RDS postgres instance) = `$90`. This is approximate so let's round it to `$100`.  

This is not per user. :-)  

The strength of my company is 500+ currently and it's growing fast. Even if we assume 50% of them being active on slack (which is an underestimate) we can see quickly that cost of buying the `Standard` Slack would be `$6 * 250 = $1500` per month. As compared to this, the cost of setting up and running Mattermost for communication needs is meagre `$100` per month.

### Downsides..

One problem which we have faced and have heard complaints from teams is around Notifications in Mattermost. Notifications are not as smooth as Slack for Mobile and Desktop both. There is delay and alerting doesn't work sometime in realtime. In worst cases, what I have observed is that with a delay of few minutes we do get notifications. 

But with Mattermost setup, the load on Slack (free version) anyways comes down so you can choose to keep few alerting systems running on free version Slack and move more persistent kind of requirements (like chat, datadog integrations, consul integrations) on Mattermost.

