---
layout: post
title:  "Docker volumes"
slug: 'docker-volume'
date: '2016-12-17 12:00:00 +0300'
categories: docker
tags: [docker, docker volume]
icon: hdd-o
---

As docker official gide says data volume is a directory within one or more containers that bypasses the
[UnionFS](https://docs.docker.com/engine/reference/glossary/#union-file-system).
With docker volumes you can share files between host and containers or even between different containers. They provide
multiple useful features for persistent or shared data.

* Data volumes persist even if the container itself is deleted.
* Data volumes can be shared and reused between containers.
* Data volumes are initialized when the container is created and if the container image contains data at the specified
mount point, that data is copied into the volume at the initialization of the volume.

### How can I interact with a volume?

You can add a data volume to a container with the `-v` flag added to the `docker create` or `docker run` commands.

> Note! To add multiple volumes to the container use `-v` multiple times.

```
$ docker run -d --name application -v /var/www iulyanp/alpine-php-fpm entrypoint.sh
15661fc3dbc7d93f9d6786ff05c5f640d52371d4670e2d48b88911b59d394763
```

This command will create `/var/www` folder inside our `application` container.

> Note! You can also use the VOLUME instruction in a Dockerfile to add one or more new volumes to any container created from that image.

If we inspect the container we will see that the volume is mounted and has a `hashed` name. You will also see the `Source`
which is the host location and the `Destination` which is telling us the volume location inside the container.

```
$ docker inspect application
...
"Mounts": [
   {
       "Name": "0e9f0824a292441e138aec50156c245fc69e7a4e697e20311b5376d6a37e6612",
       "Source": "/var/lib/docker/volumes/0e9f0824a292441e138aec50156c245fc69e7a4e697e20311b5376d6a37e6612/_data",
       "Destination": "/var/www",
       "Driver": "local",
       "Mode": "",
       "RW": true,
       "Propagation": ""
   },
]
...
```

If you're using docker on linux you can also check to see what is in that volume by `ls -la` all the content from the
`Source`, but as you can imagine nothing is there yet.

```
$ ls -la /var/lib/docker/volumes/0e9f0824a292441e138aec50156c245fc69e7a4e697e20311b5376d6a37e6612/_data
total 8
drwxr-xr-x 2 root root 4096 dec 17 14:03 .
drwxr-xr-x 3 root root 4096 dec 17 14:03 ..
```

> Note! You should use `sudo` if you get `Permission denied`.

If you add something into the container at `/var/www` it will also appear in the `Source`.

```
$ docker exec -it application sh
/var/www # ls -la
total 8
drwxr-xr-x    2 root     root          4096 Dec 17 12:15 .
drwxr-xr-x   19 root     root          4096 Dec 17 12:15 ..
/var/www # touch index.html
/var/www # exit

$ sudo ls -al /var/lib/docker/volumes/0e9f0824a292441e138aec50156c245fc69e7a4e697e20311b5376d6a37e6612/_data
total 8
drwxr-xr-x 2 root root 4096 dec 18 14:35 .
drwxr-xr-x 3 root root 4096 dec 18 14:15 ..
-rw-r--r-- 1 root root    0 dec 18 14:35 index.html
```

### Mount a host directory to the container

To mount an existing host directory into the container all we have to do is this.

```
$ docker run -d --name application -v /application:/var/www iulyanp/alpine-php-fpm entrypoint.sh
```

The command mounts the host directory `/application` into the container at `/var/www`. If the path already exists in the
container image, the content inside it will not be removed, but the host directory will overlay the container one.

```
$ docker inspect application
...
"Mounts": [
    {
        "Source": "/application",
        "Destination": "/var/www",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    },
]
...

$ docker exec -it application sh
/var/www # ls -la
total 216
drwxr-xr-x   12 1000     1000          4096 Dec 12 20:01 .
drwxr-xr-x   17 root     root          4096 Dec 12 20:23 ..
-rwxrwxrwx    1 1000     1000           533 Dec 12 20:01 index.php
```

This is how we use docker for development, we map the application folder into the docker container and any change we are
doing on the host machine it will be visible into the container.

### Named volumes

Named volumes are those volumes that you want to tag by name so you can use them later or when the container is deleted.
To create a named volume just use the `create` command followed by the local driver and a specific name passed to
the `name` option.

```
$ docker volume create --driver=local --name=application
```

If you want to see all docker volumes you have on your computer, list them with `volume ls` command.

```
$ docker volume ls

DRIVER              VOLUME NAME
local               application
```

When you need to remove a docker volume, the `rm` command comes to the rescue.

```
$ docker volume rm application
```

Sometimes you will see that there are volumes that you don't use and they were never deleted. To delete those volumes you
can use.

```
$ docker volume rm $(docker volume ls -f dangling=true -q)
```

To check where a volume is located you could once again use the `inspect` command to find out more about it.

```
$ docker volume inspect application
```

```
[
    {
        "Name": "application",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/application/_data",
        "Labels": {},
        "Scope": "local"
    }
]

$ sudo ls -la /var/lib/docker/volumes/application/_data
```

### Conclusions

That was all for now. Go ahead and play around with volumes and became familiar with them. I'm sure you'll find this
feature very useful, and from now on you will be able to map data between different containers without any problem.

See you next time!