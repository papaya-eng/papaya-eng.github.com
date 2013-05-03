---
layout: post
title: "Max memcached expiration time?"
description: ""
category: 
tags: [nosql, memcached]
author: <a href="https://github.com/socrateslee">Chun Li</a>
---
{% include JB/setup %}

Just quote from [https://code.google.com/p/memcached/wiki/NewProgramming#Expiration](https://code.google.com/p/memcached/wiki/NewProgramming#Expiration):

>Expiration times can be set from 0, meaning "never expire", to 30 days. Any time higher than 30 days is interpreted as a unix timestamp date. If you want to expire an object on january 1st of next year, this is how you do that.

So if you set a key with an expiration time higher than _30 * 86400 = 2592000_, it is highly possible the key will be expired immediately since the value represents a timestamp in the past(yet it should be a valid set operation). 
