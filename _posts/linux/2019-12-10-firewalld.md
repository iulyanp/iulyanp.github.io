---
layout: post
title:  "Use firewalld-cmd to manage your firewall"
slug: 'firewalld-cmd'
date:   2019-06-14 12:01:06 +0300
category: linux
tags: [linux, tail, less, logs]
icon: angle-right
---

Firewalld is a firewall management solution available for many Linux distributions. It is a nice tool that offers a friendly interface for the iptables packet filtering system provided by the Linux kernel. 

In this post, we will cover how to set up a firewall for your server and show you the basics of managing the firewall with the `firewall-cmd` administrative tool.

The command-line tool `firewall-cmd` is part of the firewalld application. It can be used to make permanent and non-permanent runtime changes.

```
### Installing firewalld
```

By default, firewalld is included, but in case it is not installed, you can always install it using yum or apt package managers.

```
# yum install -y firewalld
```

To enable the firewalld you should run:

```
# systemctl enable firewalld
```

As you already anticipating you can restart the firewalld service like so:

```
# systemctl restart firewalld
```

A list of all the available options with firewall-cmd command could be found by running the help command:

```
# firewall-cmd --help

Usage: firewall-cmd [OPTIONS...]

General Options
  -h, --help           Prints a short help text and exists
  -V, --version        Print the version string of firewalld
  -q, --quiet          Do not print status messages

Status Options
  --state                  Return and print firewalld state
  --reload                 Reload firewall and keep state information
  --complete-reload        Reload firewall and lose state information
  --runtime-to-permanent   Create permanent from runtime configuration

```

The firewall-cmd command has a lot of options that you could use, such as General, Status, Permanent, Zone, IcmpType, Service, Adapt and Query Zones, Direct, Lockdown, Lockdown Whitelist, and Panic. For more information about each option you could refer to the firewall-cmd man page.

The more important and used commands are the following.

1. List all zones

Use the following command to list information for all zones.

```
# firewall-cmd --list-all-zones

block
  target: %%REJECT%%
  icmp-block-inversion: no
  interfaces:
  sources:
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

drop
  target: DROP
  icmp-block-inversion: no
  interfaces:
  sources:
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

internal (active)
  target: default
  icmp-block-inversion: no
  interfaces:
  sources: 192.168.XXX.XXX
  services: dhcpv6-client mdns samba-client ssh
  ports: 4730/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: dhcpv6-client http https ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

```

Public is the default zone set, if you do not change it. To check the currently set default zone use the below command:

```
# firewall-cmd --get-default-zone

public
```

2. List allowed service and ports on the system

To show currently allowed service on your system use the below command.

```
# firewall-cmd --list-services

dhcpv6-client https ssh
```

You can check the list of ports that are open on your system.

```
# firewall-cmd --list-ports
```

1. To Enable all the incoming ports for a service

You can also open the required ports for a service by using the `-–add-service option`. 
Let's give access by HTTP clients for the public zone.

```
# firewall-cmd --zone=public --add-service=http

success
```

To list services that are allowed for the public zone:

```
# firewall-cmd --zone=public --list-services

dhcpv6-client http https ssh
```

Using this command only changes the Runtime configuration and does not update the configuration files. 
If we restart the firewalld service we can notice that the configuration changes made in Runtime configuration mode are lost.

```
# systemctl restart firewalld
# firewall-cmd --zone=public --list-services

dhcpv6-client ssh
```

To make changes permanent, use the `-–permanent` option. Example:

```
# firewall-cmd --permanent --zone=public --add-service=http

success
```

Changes made in Permanent configuration mode are not implemented immediately. Example:

```
# firewall-cmd --zone=public --list-services

dhcpv6-client ssh
```

However, changes made in a Permanent configuration are written to configuration files which means are persistent and 
even if we restart the firewalld service, it will read the configuration files and implements the changes.

```
# systemctl restart firewalld
# firewall-cmd --zone=public --list-services

dhcpv6-client http ssh
```

4. Allow traffic on an incoming port

The command below will open the port 2222 effective immediately:

```
# firewall-cmd --add-port=[PORT]/tcp
```

But as with the services, the changes will not persist across reboots

```
# firewall-cmd --add-port=2222/tcp
```

To create a persistent rule use the `--permanent` flag again, but will not take effect immediately:

```
# firewall-cmd --permanent --add-port=2222/tcp
```

You can list the open ports like so:

```
# firewall-cmd –-list-ports

2222/tcp
```

5. Start and stop firewalld service

As with any other linux service, in order to start/stop/status firewalld service use the following commands:

```
# systemctl start firewalld.service
# systemctl stop firewalld.service
# systemctl status firewalld.service
```