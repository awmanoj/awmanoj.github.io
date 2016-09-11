---
layout: post
title: Consul Integration with Opsgenie
author: Manoj Awasthi
categories: tech
---

In past I discussed about using [consul as a service discovery and configuration management solution]((https://awmanoj.github.io/tech/2016/08/27/service-discovery-configuration-management-with-consul/) and how to [configure for alerts on key value changes](https://awmanoj.github.io/tech/2016/09/03/using-consul-watch-and-git/). 

[Opsgenie](https://www.opsgenie.com/) is a must-have tool for [SRE team](https://landing.google.com/sre/interview/ben-treynor.html) of any company beyond a certain scale. It enables alerts on some events (services down for example) via email, SMS, mobile push and even phone calls (so this is the enabler for dreaded pager service).

At my company we use opsgenie and SRE team is on a constant roster - so on a rotation basis different engineers are on-call. 

There was a request to integrate service health on consul to opsgenie. Quick search over internet revealed that there is a solution called [consul-alerts](https://github.com/AcalephStorage/consul-alerts) which is also the official [recommendation from opsgenie](https://www.opsgenie.com/docs/integrations/consul-integration).

After installing it and trying to figure out - to be frank - I couldn't comprehend it well (couldn't get it deliver notifications successfully rather). So I thought, let me get to first principles and use the native `consul watch` command and some `bash` to do the simple job (isn't KISS something never to be forgotten?) and came up with this simple(st possible) script which does the job well. This script is invoked from an `upstart` script (ubuntu 14.04) which takes care of re-running it if it gets killed somehow. 

{% highlight bash %}
#!/bin/bash 

OLD_SERVICES_CRITICAL=
while [ 1 ]; do 
	rm -f /tmp/consul/services.txt
	consul-template -consul <___CONSUL_SERVICE_IP_HERE____>:8500 -template="/tmp/consul/services.alert.ctmpl:/tmp/consul/services.txt" -once
	SERVICES_CRITICAL=`cat /tmp/consul/services.txt | grep -i critical | awk '{print "["$1" "$2"]"}'`
	
	if [ ! -z "$OLD_SERVICES_CRITICAL" ] && [ -z "$SERVICES_CRITICAL" ]; 
		OLD_SERVICES_CRITICAL=""
	fi

	if [ ! -z "$SERVICES_CRITICAL" ] && [ $SERVICES_CRITICAL -ne $OLD_SERVICES_CRITICAL ]; then
		curl -XPOST 'https://api.opsgenie.com/v1/json/alert' -d"
		{
   		\"apiKey\": \"______YOUR_OPSGENIE_KEY_HERE______\", 
   		\"message\": \"Services down: $SERVICES_CRITICAL.\", 
  	 	\"teams\": [\"______YOUR_SRE_TEAM_NAME_HERE_____\"]
		}"

		# Add an alert to Slack.
		#... 

		# Add an alert to Mattermost.
		#... 

		OLD_SERVICES_CRITICAL=$SERVICES_CRITICAL
	fi

	## every 10 seconds
	sleep 10
done
{% endhighlight %}

`service.alert.ctmpl` is simple - it lists down all service names, their IP, their port and current status (passing, warning, critical etc.): 

{% highlight bash %}
{{ range services }}{{ range service .Name "any" }}{{.Name}} {{.Address}} {{.Port}} {{.Status}}
{{end}}{{ end }}
{% endhighlight %}

It took less than 5 minutes to get this running. 
