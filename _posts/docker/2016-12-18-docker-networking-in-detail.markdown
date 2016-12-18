---
layout: post
title:  "Docker networks"
slug: 'docker-networks'
date: '2016-12-18 12:00:00 +0300'
categories: docker
tags: [docker, docker network]
icon: link
---

Today I want to talk about docker networks. When you install docker it creates 3 networks automatically. To list all the
networks use `docker network ls`.

```
$ docker network ls
NETWORK ID          NAME                 DRIVER              SCOPE
9a383391d20c        bridge               bridge              local
060b58f23947        host                 host                local
cf28ccbe9537        none                 null                local
```

As you can imagine, these 3 networks are part of Docker implementation. When you create a container you have the
`--network` flag that can use to specify in which network your container should run.

The `host` network is used to add a container to the hosts network stack.
The `none` network is used to add a container to a container specific network stack.

You don't need to interact with these two default networks. You can list them and inspect them but you can not remove
them. Because you can create your own networks and use them to connect and make containers communicate to each other I
it's worth to take a look on the default `bridge` network to see how it works.

### Default bridge network

The `bridge` network represents the `docker0` network which is present in all Docker installations. The Docker deamon
connects the new containers to this network by default, unless you specifically tell it otherwise through `--network` flag.
You could see this bridge on linux with `ifconfig`.

```
docker0   Link encap:Ethernet  HWaddr 02:42:b4:53:51:b2
          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

The default bridge network in detail can be seen with the command `docker network inspect`

```
$ docker network inspect bridge

[
    {
        "Name": "bridge",
        "Id": "9a383391d20c1d8b94f2f74068240e143a42fe1c5cf60e187af5b408d05acb00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

The container is automatically added to the default network through `docker run` command. In the same time it creates a
`Subnet` and a `Getway` to the network.

```
$ docker run -itd --name=container busybox
82d6575acb647a0110d560e0c3841f5ac07a300025fecc0f99b3a606afc61d49
```
You can attach to a running container and investigate its configuration through the `attach` command.

```
$ docker attach container
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:acff:fe11:2/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:63 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:9902 (9.6 KiB)  TX bytes:648 (648.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	82d6575acb64
```

> Note: You can detach from the `container` and leave it running with CTRL-p CTRL-q.

You can also see the new container added to the default network if you inspect it once again. As you can see it is added in
the "Containers" section.

```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "9a383391d20c1d8b94f2f74068240e143a42fe1c5cf60e187af5b408d05acb00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "82d6575acb647a0110d560e0c3841f5ac07a300025fecc0f99b3a606afc61d49": {
                "Name": "container",
                "EndpointID": "abb28479dcc6e48af7b208cd0a6c852c7ebcc834dfdca052015dee06c653fcad",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

As you can imagine all the containers inside the default network can communicate to each other, but only by their IP
address. Docker does not support automatic service discovery on the default bridge network.

> Note! If you want to communicate between containers through the container names you have to use the legacy
`docker run --link` but you should avoid this technique and instead create your own bridge networks.

So let's say you create another container in the default network `second-container` and you try to ping the
`container` from inside it. You would do something like this.

```
$ docker run -itd --name=second-container busybox
ae6c2924137537438167149d2f0aa83c5de3b74e9e200069c4cc1f6b18ba7e7e

$ docker attach second-container
/ # ping -w3 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.172 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.149 ms
64 bytes from 172.17.0.2: seq=2 ttl=64 time=0.123 ms

--- 172.17.0.2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.123/0.148/0.172 ms
```

You can off course disconnet a container from the network by using `docker network disconnect` command.

```
$ docker network disconnect bridge second-container
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "9a383391d20c1d8b94f2f74068240e143a42fe1c5cf60e187af5b408d05acb00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "82d6575acb647a0110d560e0c3841f5ac07a300025fecc0f99b3a606afc61d49": {
                "Name": "container",
                "EndpointID": "abb28479dcc6e48af7b208cd0a6c852c7ebcc834dfdca052015dee06c653fcad",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

As you can see the `second-container` isn't in the `bridge` network anymore. If you list the containers `docker ps` you will
still see it.

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
ae6c29241375        busybox             "sh"                30 minutes ago      Up 30 minutes                           second-container
82d6575acb64        busybox             "sh"                About an hour ago   Up About an hour                        container
```

If you try again to ping the `container` from inside `second-container` you will get this.

```
$ docker attach second-container
/ # ping -w3 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
ping: sendto: Network is unreachable
```

### Most used `docker network` commands

List all docker networks

```
$ docker network ls
```

Create a docker network

```
$ docker network create --driver=bridge <name>
```

Connect a container to a network

```
$ docker network connect <name> <container>
```

Disconnect a docker container from a network

```
$ docker network disconnect <name> <container>
```

Remove a docker network

```
$ docker network rm appnetwork
```
Inspect a docker network

```
$ docker network inspect <name>
```

Now that we know all this new commands, let's create a new network for an app that you build.

```
$ docker network create --driver=bridge appnetwork
aa22406b9c055ecce93fd3d1f99ef26a1f9683c63a801a15d62d960ea746281c

$ docker network list
NETWORK ID          NAME                 DRIVER              SCOPE
aa22406b9c05        appnetwork           bridge              local
9a383391d20c        bridge               bridge              local
060b58f23947        host                 host                local
cf28ccbe9537        none                 null                local

$ docker network inspect appnetwork
[
    {
        "Name": "appnetwork",
        "Id": "1efe567faea63dd5d0f98993cd4b3fe09265dd8083bd0500b21af782b184a62e",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1/16"
                }
            ]
        },
        "Internal": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

After we create the appnetwork, we can launch containers on it using the `docker run --network=<NETWORK>` option.

```
$ docker run --network=appnetwork -itd --name=third-container busybox
a9d2489bff114170747deb78d1d7300575bc6b74da36d52120dacb56266e812f
$ docker network inspect appnetwork
[
    {
        "Name": "appnetwork",
        "Id": "aa22406b9c055ecce93fd3d1f99ef26a1f9683c63a801a15d62d960ea746281c",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.23.0.0/16",
                    "Gateway": "172.23.0.1/16"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "a9d2489bff114170747deb78d1d7300575bc6b74da36d52120dacb56266e812f": {
                "Name": "third-container",
                "EndpointID": "02f935901427d3b944c4c40ebd95c0d59250469f5546b1c30600b9cc60322bd2",
                "MacAddress": "02:42:ac:17:00:02",
                "IPv4Address": "172.23.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

And off course we can connect existing containers to our new network.

```
$ docker network connect appnetwork second-container
$ docker network inspect appnetwork
[
    {
        "Name": "appnetwork",
        "Id": "aa22406b9c055ecce93fd3d1f99ef26a1f9683c63a801a15d62d960ea746281c",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.23.0.0/16",
                    "Gateway": "172.23.0.1/16"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "89dd8ffec9978b7bd8479cd476bc2e52e7b98d70d3fc047422fbd9499b498a8d": {
                "Name": "second-container",
                "EndpointID": "e3fad0af4496a640aee1a00a427aec4ad96d25ecd8cc73f2e694eddede6e2a3e",
                "MacAddress": "02:42:ac:17:00:03",
                "IPv4Address": "172.23.0.3/16",
                "IPv6Address": ""
            },
            "a9d2489bff114170747deb78d1d7300575bc6b74da36d52120dacb56266e812f": {
                "Name": "third-container",
                "EndpointID": "02f935901427d3b944c4c40ebd95c0d59250469f5546b1c30600b9cc60322bd2",
                "MacAddress": "02:42:ac:17:00:02",
                "IPv4Address": "172.23.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

In our custom network we can communicate between containers using their names like this.

```
$ docker attach second-container
/ # ping -w3 third-container
PING third-container (172.23.0.2): 56 data bytes
64 bytes from 172.23.0.2: seq=0 ttl=64 time=0.123 ms
64 bytes from 172.23.0.2: seq=1 ttl=64 time=0.105 ms
64 bytes from 172.23.0.2: seq=2 ttl=64 time=0.127 ms

--- third-container ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.105/0.118/0.127 ms
```

### Conclusion

Docker network new feature it's amazing. Before it you could use the Docker link feature to allow containers to discover
each other. With the introduction of Docker networks, containers can communicate with each other by its name automatically.
You can still use `--link` option to bind containers but as I said before this is not recommended anymore.  For more
information about `docker network`, please refer to the official [page](https://docs.docker.com/engine/userguide/networking/).