---
layout: post
title:  "Graceful restart php-fpm"
slug: 'graceful-restart-php-fpm'
date:   2018-02-01 12:01:06 +0300
categories: php
tags: [php, php-fpm, modern php]
icon: code
---

Php-Fpm allows for a graceful restart of child, usually with the reload keyword instead of restart on the init script.

### Setup

All you have to do is to set `process_control_timeout` to something other than 0 for this to work.
`innodb_lock_wait_timeout` defaults to 50 seconds, it's a good practice to set `process_control_timeout` with the default of `innodb_lock_wait_timeout`.

> process_control_timeout mixed
Time limit for child processes to wait for a reaction on signals from master. Available units: s(econds), m(inutes), h(ours), or d(ays) Default Unit: seconds. Default value: 0.

To see if the php-fpm process is running on Centos you can use `sudo systemctl status php-fpm`

```bash
● php-fpm.service - The PHP FastCGI Process Manager
   Loaded: loaded (/usr/lib/systemd/system/php-fpm.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-01-01 11:57:37 EEST; 1 months 0 days ago
  Process: 9294 ExecReload=/bin/kill -USR2 $MAINPID (code=exited, status=0/SUCCESS)
 Main PID: 32688 (php-fpm)
   Status: "Processes active: 0, idle: 2, Requests: 246, slow: 0, Traffic: 0.6req/sec"
    Tasks: 3
   Memory: 73.6M
   CGroup: /system.slice/php-fpm.service
           ├─ 9301 php-fpm: pool www
           ├─ 9302 php-fpm: pool www
           └─32688 php-fpm: master process (/etc/php-fpm.conf)
```

You can reload the php-fpm process just by running `sudo /bin/systemctl reload php-fpm`.
You can find out what systemd reload php-fpm will do by looking at the ExecReload= option in the [Service] section in the php-fpm.service unit file.

```ini
[Unit]
Description=The PHP FastCGI Process Manager
After=syslog.target network.target

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/php-fpm
ExecStart=/usr/sbin/php-fpm --nodaemonize
ExecReload=/bin/kill -USR2 $MAINPID
PrivateTmp=true
RuntimeDirectory=php-fpm
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
```

Or by running

```bash
sudo systemctl show php-fpm --property=ExecReload
```

Output

```bash
ExecReload={ path=/bin/kill ; argv[]=/bin/kill -USR2 $MAINPID ; ignore_errors=no ; start_time=[Fri 2021-12-10 21:56:37 EET] ; stop_time=[Fri 2021-12-10 21:56:37 EET] ; pid=9294 ; code=exited ; status=0 }
```

So the -s HUP means configuration reload, start the new worker processes with a new configuration. Gracefully shutdown the old worker processes.
