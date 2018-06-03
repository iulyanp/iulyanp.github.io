---
layout: post
title:  "8 steps to start with Docker containers"
description: "Learn how you can create your first docker container as a pro"
slug: '8-steps-to-start-with-docker-container'
date:   2018-06-02 20:30:00 +0300
category: docker
tags: [docker, container, linux]
icon: link
---

Docker is used by almost everybody, from developers to sysadmins and architects, everybody is using it because is fast, 
clean and very easy to build, ship, and run distributed applications.
In my older [docker posts]({{ site.url }}/blog/tag/#docker) I presented the basics of Docker when it wasn't so mature. 
Today I want to recap some of it's functionalities and discuss more advanced technique that will help you start use 
Docker with more confidence.


### Step 1 - Find a docker image

After you install docker on your local machine, in order to start using it you have to find an image that you'll use 
to start one or more docker containers.

> A container is just a sandboxed process running an application and its dependencies on the host operating kernel. 
Docker containers are very lightweight and fast and we can run multiple containers independently on the same machine.

Let's say you want to use `redis` as a key value store for your project. You first search for the redis image on the 
registry.hub.docker.com/ like this:

```
# you login first to the docker hub with your user and password
$ docker login

# then search for redis docker image
$ docker search redis
```

### Step 2 - Launch a container

Using the search command you will identify that the redis docker image is called `redis`. Because redis is just a 
database we can run it as a background service. By default, Docker will run the container in the foreground, so in 
order to run it in background we can use the `-d` flag.

To launch a container in the background running an instance of Redis based on the official image we can use the `run`
docker command, which will start a new container based on the image we found with the `search` command.


```
$ docker run -d redis
```

By default docker will use the `latest` version available if no version is specified. If we want to use a specific 
version of the image we can ask docker to use the version 3.2 like this

```
$ docker run -d redis:3.2
``` 

Because this is the first time you are using the redis docker image, it will be downloaded on your host machine and 
after that a container will be launched with a random name. We can name the container by the `--name` flag.
  
```
$ docker run -d --name redis-container redis 
```

### Step 3 - Find the running containers

Now the container is running in background, we can verify this with the `ps | container ls` docker commands which will
list all running containers.

```
$ docker ps
$ docker container ls
```

Also if you have stopped containers we can list them with the `-a | --all` options of the `ps | container ls` commands. 

```
$ docker ps -a
$ docker ps --all
$ docker container ls -a
$ docker container ls --all
```

### Step 4 - Inspect docker containers

These commands also display the name and the ID that can be used to inspect a container for more information `docker 
inspect redis`

### Step 5 - Check logs on docker containers

The command `docker logs redis` will display messages the container has written to standard error or standard out.

### Step 6 - Map ports on docker containers

At the moment the redis container is running in background on our local machine but we can't access the redis service
because it's running inside the container. The reason is that as I said at the begining of this post, each container 
is sandboxed. If a service needs to be accessible outside it's container then we have to expose it's port to the host.

Redis is running by default on port `6379` so we can map the container port to a host port via `-p` option.

```
$ docker run -d --name redis-random-port -p 6379 redis
```

While this works, we don't know which host port has been assigned to the redis container port 6379. Thankfully, this is 
discovered via `docker port redis-random-port 6379`.

This is good but it's not enough, we want to be able to bind the container port to the specific `6379` host port so 
that other services be able to find it easily. We can do this like so `-p <host-port>:<container-port>`.

```
$ docker run -d --name redis-port -p 6379:6379 redis
```

### Step 7 - Persisting Data

Now we are able to start using the redis service and store some data into the database. After some tests we see that 
everything works perfect, but if the container is deleted or re-created the data from redis is being automatically 
removed. It would be nice if we would be able to keep data even we remove or re-create the container.

Luckily there is a way to do this with docker by binding volumes (directories) using the `-v <host-dir>:<container-dir>`
option. When we bind a directory, the files from the host directory can be accessed in the container and all the 
changes done to those files inside the container will reflect and be stored on the host.

The redis image stores logs and data into a `/data` directory. In order to bind that directory to the current 
directory we can use the `$PWD` env variable, which will map the current directory to the `/data` one from the 
container. 

```
$ docker run -d --name redis_container -v $PWD:/data redis
```

### Step 8 - Interact with a container

We can also interact with a container and get access to a bash shell inside of it with the `-it` flag.

```
$ docker run -it ubuntu bash
```

Running this command will open an ubuntu container and give us access to the shell inside it. Now we can interact 
with the ubuntu shell as if we would do it on a machine with ubuntu operating system. 
