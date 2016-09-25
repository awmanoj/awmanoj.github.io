---
layout: post
title: Runscope Integration with Mattermost
author: Manoj Awasthi
categories: tech
---

[Runscope](https://www.runscope.com/) is an API monitoring and testing tool. It allows you to test your web service endpoints periodically (say every 5 mins) for uptime & errors with a good support for alert and notifications. 

We are using Runscope at my workplace to monitor our web services. It has great [integration with Slack](https://www.runscope.com/docs/api-testing/slack) so any Runscope tests that results in error would post a notification on one of our Slack channels.

As I wrote earlier, we are [replacing Slack](https://awmanoj.github.io/tech/2016/09/11/mattermost-opensource-selfhosted-slack-alternative/) with an open source alternative - Mattermost. 

Now the problem is that Runscope does not provide any ready-to-use integration with Mattermost. So, we need to do more toil. Basically we can use [Webhook Notifications](https://www.runscope.com/docs/alerts/notifications#webhook). To do this, you run a web service on public IP (or behind ELB) and register it with RunScope for tests you are interested in. This web service will receive a callback from Runscope (and `HTTP POST`) in a certain format containing all data about the test that ran. Your web service can take appropriate action for the callback - e.g. in our case - if we detect there is an error we send a `HTTP POST` to Mattermost with data in same format as we were getting on Slack. 

I have written some example code in `go` - [runscope-mattermost-hook](https://github.com/awmanoj/runscope-mattermost-hook) for the integration service which you may refer or use. If you have feedback please [tweet](https://twitter.com/awmanoj) to me.

Ref: [https://github.com/awmanoj/runscope-mattermost-hook](https://github.com/awmanoj/runscope-mattermost-hook)
