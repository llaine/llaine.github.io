---
layout: post
title: "How to scale with php?"
date: 2015-02-04 12:20:56 +0100
comments: true
categories: php scalability apache high-performance sysadmin
---


## Abstract

So today, a little article on the scalability of PHP legacy applications. 

## Use case

Suppose we have a codebase in PHP which we can not change. 
How to scale the app to 500 request by second ? 

### What we have : 

- PHP 5.3
- Spaghetti code
- Strong dependency coupling 
- MySQL Database 


Okay let's talk a little bit about scalability : 

**Scalability** : is ability of a system, network, or process to handle a growing amount of work in a capable manner or its ability to be enlarged to accommodate that growth._

- Horinzontal scalability : means to add more nodes to a system, such as adding a new computer to a distributed software application
- Vertical scalability : means to add resources to a single node in a system, typically involving the addition of CPUs or memory to a single computer.


## Back to the use case

In order to begin sovling the problem, let's remind something. PHP is based on HTTP which is stateless by default.
Which mean that the whole memory is destroyed after the request. 

That's a pretty good news! :) 

__BUT__ 

Everything is stateless, expect from the session 

{% img /images/meme.jpg  %}


### How to deal with the session ? 

1. Install memcacheD
2. In the _php.ini_
    - `session.save_handler = memcached`
    - `session.save_path = goteMemcacheD :port`
3. Reloading Apahe!

Aaaaaand that's all !! 



## Performance 

#### opCode cache 

- APC for php 5.3 (which you shouldn't use) : `apc.stat = 0` (avoid I/O on disk)

By default include in PHP 5.5 
- `opcache.enable = on` in the _php.ini_
- `opcache.validate_timestamp = 0` (avoid I/O on disk)$


# The database (Oh sh*t)

the database is most of the time the last things that is the problem (is this often your codebase). 

But we can optimize it :) ! 


## How to ? 

- Provide a loooot of ram (much than the database size)
- `innodb_buffer_pool_size` needs to be as high as possible (5-6GB)
- `mysql_query_cache` with a good value.
- Create a looooots of index in column which are you for 'WHERE' statements or joins.  

## Doctrine 2

Use as much as you can eager loading. 



You also can use a mysql_cluster (which is quite complicated to install but it's like the rolls royce of mysql).


{% img /images/mysql_cluster.png  %}


#  Infrastructure


1. Use a varnish, for statics assets. 
2. Provisionning tools like Ansible for servers configuration. 




Aaand that's all :) ! 



