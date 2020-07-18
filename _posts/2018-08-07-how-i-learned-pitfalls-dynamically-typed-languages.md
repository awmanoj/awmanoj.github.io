---
layout: post
title: How I learned hard way the pitfalls of a dynamically typed languages? 
author: Manoj Awasthi
categories: tech
---

Dynamically typed (weak types) languages (like Perl, Python, Ruby etc) are great for developer productivity or so they say. I agree to large extent but then for a large scale system with growing components and services this may become a nightmare to maintain. This is based on some of the learnings being on both sides.  

In this post, want to discuss one interesting issue that I got reminded of while going through some archives and boils down to one of the perils of using a dynamically type language. 

For the context, we are a marketplace with millions of sellers and buyers on the platform. Sellers can open a shop on the platform and start selling while buyers can discover the products and shops they want to get access to. Marketplace also provides many payment methods and escrow so that your payment is safe. 

Marketplace platform was originally written in Perl and though we modularised it from Monolith that it was to a huge number of micro services while taking the opportunity to write them in a modern statically typed language (golang), we still had some parts of it written in Perl. We have been moving these parts to golang fast. 

So one fine day last year, suddenly we started receiving an ever increasing number of merchant complaints that they were getting an error while adding products (hence failed uploads of products) which resolved to the following message: 

```
"category not complete"
```

Yes, not that useful but this was not supposed to occur so much, a highly unlikely message and hence never got much attention from the product managers. 

At the peak of the error, the frequency of reported issues was ~ a couple of thousands per hour. 

After assembling multiple teams involved and tracking the root cause, we found that the bug itself was pretty interesting. 

There was a sql query to check if a category is leaf or not - try finding out categories whose parent is the given category and see the number of results returned - if zero then ok other wise throw error.  It looked something like this in the perl code: 

```
                     $str_sql = qq~
                         select
                             d_id
                         from ws_department
                         where parent = <:n=$val_dep:>
                         and status = <:n=1:>
                     ~;
```

`$val_dep` i.e. category ID was fetched in a previous code block using another `SELECT` query and was expected to be a single integer (equivalent of `LIMIT 1`). But somehow `$val_dep`, instead of being the category_id (i.e. `int`) ended up being the array (multiple values) whose address would look like `0x245bf20`. then this function

```
$svq->sql($str_sql);
```

will convert the query into

```
                     $str_sql = qq~
                         select
                             d_id
                         from ws_department
                         where parent = 24520 
                         and status = <:n=1:>
                     ~;
```

So, note how Perl showed its `intelligence` in converting the `0x245bf20` to `24520` (silently). Grr.

This caused the intermittently occurring bug. Sometimes that memory address would resolve to a valid category ID in the marketplace and many times it won’t. When it doesn’t it leads to the error that leaf category ID is not present which in turn led to the “bad” error message “category not complete”. 

Side effect of dynamically typed language. 

Statically typed languages (C, C++, Java, Scala, Go etc) ensure type safety and would help avoid such pitfalls. 

So be mindful about using the language carefully. If it is dynamically typed then you can expect no type safety. So ensure proper validation (err towards more of it) to write rock solid code (if there is anything like that).

:-) 


