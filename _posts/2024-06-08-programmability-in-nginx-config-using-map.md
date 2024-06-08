---
layout: post
title: Programmability in nginx config using map
author: Manoj Awasthi
categories: tech
---

NGINX is one of the most beautiful piece of tech that I explored briefly in 2011-12 and heavily 2016 onwards while scaling an ecommerce platform at work in Indonesia. It is so simple and so powerful that it is hard to not fall in love with it. Primarily, it is an HTTP reverse proxy server and can be used to serve static and dynamic content behind web services with excellent inbuilt support for load balancing, caching, logging, rate limiting, and even request mirroring. 

For a more elaborate understanding of the feature set it comes with, head to [NGINX documentation](https://nginx.org/en/).

Most of the features in NGINX are controlled through easily configurable NGINX configuration that usually sits at `/etc/nginx/nginx.conf` and extended by included files in the configuration (including `/etc/nginx/<sites>/<site.example.com>`). This ability to modify the conf file followed by a simple config test and reload as follows is what makes it so easy to operate:

```
$ nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
..
$ nginx -s reload
```

Don't get disheartened. This post is not a general manual for NGINX. It is about something I discovered today while experimenting on something for a friend. This person wants to run a simple web tool and serve it on different end points for different employees.

```
http://example.com/it/review-1
http://example.com/it/review-2
http://example.com/it/review-3
```

All the three sites are similar in nature. They have some logic to fetch some data and present on the web page in a form, which when submitted post review gets submitted to backend and redirected back to the original endpoint for fetching more data for review.

Initially they set up three endpoints in the config as following:

```
	location /it/review-1/ {
		proxy_pass http://localhost:5000/?param=1;
    	}
	location /it/review-2/ {
		proxy_pass http://localhost:5000/?param=2;
    	}
	location /it/review-3/ {
		proxy_pass http://localhost:5000/?param=3;
    	}
```

Now they want to expand it to 10 endpoints. So, one way was to simply proceed with duplicating the endpoints in NGINX config. So, he asked me if there is a more elegant solution to this. And as I researched I found about `map` in NGINX which I'd never used. 

## `map` in NGINX

`map` coupled with excellent support for regex in NGINX made it possible to avoid all duplication and enable a very elegant solution. 

```
# Define this in http block but outside the server block
# Define a map to extract the 'param' parameter based on the URL path
map $request_uri $param {
	default 0; # Set default value to 0
	~^/it/review-(\d+)/ $1;
}
```

This assigns `param` variable to a default value of `0` and otherwise parse it from the `request_uri`. So a request to `/it/review-2` will assign `param` a value of `2`.

Next step is to use it for the `location` block:

```
        location ~ ^/it/review-(\d+)/ {
            # Extract the param parameter
            set $param_value $param;
	    
            # Use the odd parameter in the proxy_pass directive
            proxy_pass http://127.0.0.1:5000/?param=$param_value;
	}
```

But then I realised that this will lead to a situation where even unintended endpoints like beyond ten (`10`) will work e.g. `/it/review-12`. For handling that I needed to check for the value and return `501 (Not Implemented)` for the requests of that nature.

So, the question was HOW!?
 
And again the superhero to the rescue is our `map`. Following code helped do the validation and set a variable for every request if the `param` value is greater than `10`.
```
# Define a map to check if param_value is >= 10
map $param $param_ge_10 {
	default 0;
	~^[1-9]$ 0; # Param value is less than 10
	~^1[0-9]$ 1; # Param value is greater than or equal to 10
}
```

Finally, we can use this for additional logic in the location block:

```
        location ~ ^/it/review-(\d+)/ {
            # Extract the param parameter
            set $param_value $param;
	
            # Return 501 Not Implemented if param_value is >= 10
            if ($param_ge_10) {
                return 501;
            }
    
            # Use the odd parameter in the proxy_pass directive
            proxy_pass http://127.0.0.1:5000/?param=$param_value;
	}
```

And that was it. Quite a learning, followed by the excitement to share it and hence this post!!
