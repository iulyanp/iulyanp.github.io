---
layout: post
title:  "Optimize memory usage of Redis instance"
description: "Learn how you can optimize memory usage of Redis instance"
slug: 'optimize-memory-usage-of-redis-instance'
date:   2021-01-04 10:30:00 +0300
category: redis
tags: [redis, redis-cli, memory]
icon: memory
---

Sometimes **Redis keys** can increase the memory consumption for our Redis instances.
Usually, we should prefer descriptive keys while using Redis, but if we have a very big dataset with millions of long keys then these keys will take a lot of money from our pocket.

## Use Smaller Keys with Redis

If the application was well written, switching to smaller keys usually involves updating a few configurations in the application code.

First we will have to identify all the big keys in our Redis instance and shorten it by removing extra characters from it. 

The easyest way to find what keys are very big in our application is to run the command `redis-cli --bigkeys`. This command will scan a certain set of records and return the big keys from that set.

## Advantages

For example: 

Let's say we have `100,000,000 keys` named like this:

`some-descriptive-large-keyname (30 characters)`

If we change the key name into a smaller one we will have something like this:

`some-keyname (12 characters)`

Just by doing this we save `16 chars` which means `16 bytes`. If we make a small calculation with this values we end up by saving `100,000,000*16 = 1.6GB` of RAM!

## Disadvantages

Because we made the key smaller it's possible that it becomes less descriptive, but in some situations this isn't a very big issue comparing to your cost savings. 
Also you could have a readable mapping for each key in your source code and this issue would be solved.
