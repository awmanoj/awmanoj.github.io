---
layout: post
title: 421 Misdirected request error
author: Manoj Awasthi
categories: tech
---

Yesterday, I faced an interesting issue that I wasn't aware of earlier. Let me tell the context first.

We have one server (i.e. 1 IP) which serves multiple domains (websites). Yesterday, we decided to put it behind client side SSL certificates. Everything was hunky dory till we started hearing that some users (thankfully, this was internal website so only colleagues!) complained about `421 Misdirected Request` when they try accessing those websites.

<img src="/public/assets/img/421.png" />

A [couple](http://serverfault.com/questions/772901/421-misdirected-request-using-http-2-and-san-ssl) [of](https://bugs.chromium.org/p/chromium/issues/detail?id=546991) searches over the internet helped understand the problem a bit. This post is a summary.

So, basically when you enable SSL (e.g. set `ssl_verify_client` to `optional`, `on` or `optional_no_ca` in `nginx`) and you serve more than one domains from the same server (i.e. 1 IP address) and one of the domains are using `http2` then HTTP/2 tries to reuse connections to `different` servers.  

Let's first understand what protocol does and how servers and clients are expected to behave with respect to this. Following are snippets from [RFC 7540 (HTTP/2)](https://tools.ietf.org/html/rfc7540#section-9.1.1):

Protocol suggests:

>  An origin server might offer a certificate with multiple "subjectAltName" attributes or names with wildcards, one of which is valid for the authority in the URI.  For example, a certificate with a "subjectAltName" of "*.example.com" might permit the use of the same connection for requests to URIs starting with "https://a.example.com/" and "https://b.example.com/".
> ... 
> In some deployments, reusing a connection for multiple origins can result in requests being directed to the wrong origin server.
> ...
> This means that it is possible for clients to send confidential information to servers that might not be the intended target for the request, even though the server is otherwise authoritative.


So servers are suggested to implement: 

> A server that does not wish clients to reuse connections can indicate that it is not authoritative for a request by sending a 421 (Misdirected Request) status code in response to the request (see Section 9.1.2).

.. and browsers (clients) are suggested to implement: 

> Clients receiving a 421 (Misdirected Request) response from a server MAY retry the request -- whether the request method is idempotent or not -- over a different connection.

In our case, we serve two domains `a.foobar.com` (over http) and `b.foobar.com` (over http2) from 1 IP address. We added SSL to this. So when a user: 

* opens browser and accesses `a.foobar.com`, he gets served the web content successfully. 
* from the same browser, accesses `b.foobar.com`. he gets a `421 Misdirected Request` error in chrome (specifically, `Google chrome 56.0.2924.87`).

What happened? 

Seems like, in second case - HTTP2 tries to use the same connection as the previous - owing to a single IP Address. In case of TLS, it additionally depends on whether certificate is valid for the `host` of the domain being accessed. As mentioned in RFC above the behavior is left to servers. If server finds this TLS request on a different domain than the one for which the connection was used previously and it thinks it is not authoritative then it can indicate so by responding with `421 misdirected request` error. In response to this error code, the client (i.e. the browser) should create a new connection and retry the request. 

So, while some browsers like `Safari`, `Mozilla firefox` behave in the expected manner causing no trouble, `Google Chrome` (particular version is the only one I tested with) doesn't abide with this. 

### Solution: 

* For now, I disabled HTTP/2 for the site since it was internal website and it is ok to miss out on the performance benefits of http2. Everything works fine.

* One post suggests it happens due to [difference in SSL configurations on the two domains](http://serverfault.com/questions/772901/421-misdirected-request-using-http-2-and-san-ssl) - this is what I am yet to investigate. I will keep the post updated.

### Useful links: 

[https://trac.nginx.org/nginx/ticket/848](https://trac.nginx.org/nginx/ticket/848)
[https://forum.nginx.org/read.php?29,267026](https://forum.nginx.org/read.php?29,267026)
[https://bugs.chromium.org/p/chromium/issues/detail?id=546991](https://bugs.chromium.org/p/chromium/issues/detail?id=546991)

