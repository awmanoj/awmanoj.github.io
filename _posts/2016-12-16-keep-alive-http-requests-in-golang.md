---
layout: post
title: Keep-Alive in http requests in golang
author: Manoj Awasthi
categories: tech
---

The [three-way handshake](https://en.wikipedia.org/wiki/Handshaking#TCP_three-way_handshake) in a TCP connection setup is heavy wrt performance and re-using the already created connections is a considerable optimization. Keep alive is a method to allow the same tcp connection for HTTP conversation instead of opening a new one with each new request. Most servers and clients allow for configurations and options to utilise this optimisation. 

I had a strange encounter with the way this is handled in go. 

I’ve a go application - an [nsq](http://nsq.io/) (a distributed message queue) consumer - which reads some data off nsq and send that data over a HTTP POST to a remote server over SSL. This involves a [TLS handshake](https://www.ssl.com/article/ssl-tls-handshake-overview/) which is even more heavy than non-ssl handshake. 

We found that nsq queue depth for the topics was high and increasing, additionally we kept seeing errors like in nsqd logs: 

```
[nsqd] 2016/12/16 17:48:34.563958 ERROR: [19.68.8.7:55239] - E_FIN_FAILED FIN 0b3caf88 failed ID not in flight - ID not in flight
[nsqd] 2016/12/16 17:48:34.567675 ERROR: [19.68.8.7:55239] - E_FIN_FAILED FIN 0b3ca9f4 failed ID not in flight - ID not in flight
[nsqd] 2016/12/16 17:48:34.571008 ERROR: [19.68.8.7:55239] - E_FIN_FAILED FIN 0b3c955c failed ID not in flight - ID not in flight
[nsqd] 2016/12/16 17:48:34.573917 ERROR: [19.68.8.7:55239] - E_FIN_FAILED FIN 0b3c86f8 failed ID not in flight - ID not in flight
```

Based on the [limited](https://github.com/nsqio/nsq/issues/660) [number](https://github.com/nsqio/nsq/issues/729) [of documents](https://github.com/nsqio/nsq/issues/762) we got over the internet it seemed like this was due to a slow consumer (hints like need for heartbeat from consumer pointed in this direction). Since consumer was very simple - just doing some http posts so it was the only suspect. 

We did a `strace` on the running process to see the `read,connect` system calls. It indeed was doing some certificate interchange (read) for each connect or POST: 

```
[pid 18043] read(90, "g\201\f\1\2\0020|\6\10+\6\1\5\5\7\1\1\4p0n0$\6\10+\6\1\5\5\0070\1\206\30http://ocsp.digicert.com0F\6\10+\6\1\5\5\0070\2\206:http://cacerts.digicert.com/DigiCertSHA2SecureServerCA.crt0\f\6\3U\35\23\1\1\377\4\0020\0000\r\6\t*\206H\206\367\r\1\1\v\5\0\3\202\1\1\0\222\327\361\374\211T\330\363\305~8Z\270\210\313<Z\336o\274?\366\34\252\206Q\332\0044\257'\3751\205\273sa\200\251\325\224\267\\*\221\1/Ws\246\351bl=\330q?\200\256f\21\257\331\246\242w\313\202\245\361\241a?\2\345<a\35\313n\276\220\217k\377C\335}\235\2000+%?… ", 3166) = 1948
```

From the code, we noticed that: 

* a http client was being created for each incoming message in the handler. this was overkill clearly. 
* the http client created was using default transport. default http client does have a keep-alive setting. 

Since keep-alive is already there so we thought #1 is the problem which is causing new connections to be created for every request which in turn leads to the TLS handshake (and eventually to slow consumer). We fixed this by creating a global client: 

```
 14 var client *http.Client
 15 
 16 func init() {
 17     tr := &http.Transport{
 18         MaxIdleConnsPerHost: 1024,
 19         TLSHandshakeTimeout: 0 * time.Second,
 20     }
 21     client = &http.Client{Transport: tr}
 22 }
```

and using this to do the Post: 

```
 31     resp, err := client.Post("https://api.some-web.com/v2/events", "application/json", bytes.NewBuffer(eventJson))
 32     if err != nil {
 33         log.Println("err", err)
 34         return err
 35     }
 36  
 37     defer resp.Body.Close()
```

But we observed that problem persisted still (slowness as evident from errors in nsqd logs). We confirmed by doing `strace` again: 

```
$ sudo strace -s 2000 -f -p 18120 -e 'read,connect' 
[pid 18120] read(90, "g\201\f\1\2\0020|\6\10+\6\1\5\5\7\1\1\4p0n0$\6\10+\6\1\5\5\0070\1\206\30http://ocsp.digicert.com0F\6\10+\6\1\5\5\0070\2\206:http://cacerts.digicert.com/DigiCertSHA2SecureServerCA.crt0\f\6\3U\35\23\1\1\377\4\0020\0000\r\6\t*\206H\206\367\r\1\1\v\5\0\3\202\1\1\0\222\327\361\374\211T\330\363\305~8Z\270\210\313<Z\336o\274?\366\34\252\206Q\332\0044\257'\3751\205\273sa\200\251\325\224\267\\*\221\1/Ws\246\351bl=\330q?\200\256f\21\257\331\246\242w\313\202\245\361\241a?\2\345<a\35\313n\276\220\217k\377C\335}\235\2000+%?… ", 3166) = 1948

```

This appeared for each connection request. This meant that golang was somehow not honouring the keep-alive. This is when [this thread on stack overflow](http://stackoverflow.com/questions/17948827/reusing-http-connections-in-golang
) helped us - 

> You should ensure that you read until the response is complete before calling Close(). 

```
res, _ := client.Do(req)
io.Copy(ioutil.Discard, res.Body)
res.Body.Close()
```

> To ensure http.Client connection reuse be sure to do two things:

* Read until Response is complete (i.e. ioutil.ReadAll(rep.Body))
* Call Body.Close()

So we followed the suggestion: 

```
 31     resp, err := client.Post("https://api.some-web.com/v2/events", "application/json", bytes.NewBuffer(eventJson))
 32     if err != nil {
 33         log.Println("err", err)
 34         return defaultErrStatus, err
 35     }
 36 
 37     io.Copy(ioutil.Discard, resp.Body)   // <= NOTE 
 38 
 39     defer resp.Body.Close()
```

While, as a I see the golang implementation, it does include [a subtle comment in the code](https://github.com/golang/go/blob/master/src/net/http/client.go#L666) but I don’t understand yet why this idiosyncrasy exists. But would be something to be aware of and keep on the back of mind. 

```
 // Post issues a POST to the specified URL.
 //
 // Caller should close resp.Body when done reading from it.
 //
 // If the provided body is an io.Closer, it is closed after the
 // request.
 //
```

### Take Away

* Use a global client as much as possible to avoid new connections. 
* Even when you don't need the response back, read it fully before closing the response body - to take benefit of persistent connections (keep-alive).

