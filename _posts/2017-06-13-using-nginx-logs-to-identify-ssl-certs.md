---
layout: post
title: Using nginx logs to identify SSL certificate details
author: Manoj Awasthi
categories: tech
---

Adding client side SSL certificate is [one of the best ways to restrict access on public internet](https://stackoverflow.com/questions/2164397/why-should-i-authenticate-a-client-using-a-certificate). The way it works is that you create server certificates using a certifying authority (which can be you, yourself) and then create some client certificates. Then it is a matter of putting some settings in nginx (or other server you may be using) to enable it. Post that, you share the client certificate with users who are authorized to access the resource with revocation possible from server side. Your resource can be accessed by only people who have the client certificate installed. You can [read this](https://gist.github.com/shamil/2165160#file-gistfile1-md) and [this](https://rynop.com/2012/11/26/howto-client-side-certificate-auth-with-nginx/) for details of how to generate and use certificates for nginx. 

At my workplace, we have a distributed CRUD system for key values which is protected using client side SSL certificates in similar manner. Initially, we created a common client certificate which was shared to all who needed access. But in this setup, there was a loophole. Since the certificate is shared so whether user A makes the change or user B makes the change, there is no way of identifying. So, we came up with per-user certificates which will have the user id and user email identification. These certificates will be delivered to user's email on request for access. 

Next step (and the subject of this blog post) was to identify through logs who does what actions. On Nginx, it is as simple as updating the `log_format` in `/etc/nginx/nginx.conf` file: 

```
  log_format  main '$remote_addr - $ssl_client_s_dn - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$upstream_response_time"';
  access_log /var/log/nginx/access.log main;
  error_log /var/log/nginx/error.log;
```

You notice that I added `$ssl_client_s_dn` in the log format, This variable returns the “subject DN” string of the client certificate for an established SSL connection according to RFC 2253 (1.11.6);

Following logs are examples of what we see in our logs: 

```
10.0.2.31 - /C=ID/ST=DKI-Jakarta/L=Jakarta/O=PT XXXXXXX/OU=Engineering/CN=XXXX/emailAddress=manoj.awasthi@XXXXXXX.com - - [13/Jun/2017:18:06:38 +0700] "DELETE /services/kv/random/key HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36" "0.001"
```

The value corresponding to the variable `$ssl_client_s_dn` is: 

```
/C=ID/ST=DKI-Jakarta/L=Jakarta/O=PT XXXXXXX/OU=Engineering/CN=XXXX/emailAddress=manoj.awasthi@XXXXXXX.com
```

This uniquely identifies the user who does a certain change e.g. `DELETE` a key. A simple, low cost change control system. :-) 

You can read more about other variables related to SSL certificates which are supported in nginx.
