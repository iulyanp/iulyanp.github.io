---
layout: post
title:  "Delete All Data from Redis"
description: "Learn how you can delete all data from Redis"
slug: 'delete-all-data-from-redis'
date: 2021-01-10 12:31:04 +0300
category: redis
tags: [redis, redis-cli, data]
icon: memory
---

Some advantages of Redis being a NoSQL and in-memory system is that you can perform some tasks very easily compared with relational database systems.

One such capability is **deleting everything** in your database or even all databases!

> NOTE! DO NOT TRY THIS IN PRODUCTION ENV.

## Start Redis instance

Using the `redis-server` command from your shell prompt manually start your Redis server.

```
$  redis-server
14382:C 10 Jan 2021 18:25:12.578 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
14382:C 10 Jan 2021 18:25:12.578 # Redis version=5.0.4, bits=64, commit=00000000, modified=0, pid=14382, just started
14382:C 10 Jan 2021 18:25:12.578 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
14382:M 25 Jun 2021 18:25:12.579 * Increased maximum number of open files to 10032 (it was originally set to 256).
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 5.0.4 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 14382
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           https://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

14382:M 25 Jun 2021 18:25:12.581 # Server initialized
14382:M 25 Jun 2021 18:25:12.582 * Ready to accept connections
```
You should see an output from Redis similar with the one above, indicating the server is running and which port it is attached to.


### Accessing the Redis Command Line Interface

Redis come with the Redis Command Line Interface, which can be accessed simply by running from the command line the `redis-cli` command.

```
$ redis-cli
127.0.0.1:6379>
```

If you were able to connect, you should see the `redis-cli` prompt with the `host` and `port` specified, as seen above.

### Deleting a single database from Redis

In Redis if you have multiple databases, you can access them by their unique `index` number.

You may connect to a different database by entering the `select index` command:

```
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]>
```

Notice that the `redis-cli` prompt now indicates you are connected to database `[1]`.

Now in order to destroy a the database, we can issue the FLUSHDB command:

```
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> FLUSHDB
OK
```

### Deleting ALL Databases from Redis

If you just played around with Redis and wish to destroy everything in the end, you can use the FLUSHALL command:

```
127.0.0.1:6379> FLUSHALL
OK
```

Very simple, but that is how you can quickly delete everything in Redis.