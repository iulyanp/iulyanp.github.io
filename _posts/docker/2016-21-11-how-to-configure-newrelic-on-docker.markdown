---
layout: post
title:  "How to configure New relic agent on Docker"
slug: 'how-to-configure-newrelic-on-docker'
date:   2016-11-21 12:01:06 +0300
categories: docker
tags: [docker, newrelic]
icon: th-large
---

Today I want to talk about [New Relic](https://newrelic.com/) and [Docker](https://www.docker.com/). A few days ago I had to install new relic on our servers and because we are using Docker I needed to install it from a Dockerfile. 

### Installing New Relic in a Dockerfile

Before you run the install commands for new relic php module you need to create some environment variables so you could install new relic with no user interactions.

```
 ENV ENVIRONMENT dev
 ENV NR_INSTALL_SILENT true
 ENV NR_INSTALL_KEY XXX
```

The first one `ENVIRONMENT` is not required, it's just a variable that I used in order to configure the application name into `newrelic.ini` file.
The other two variables are required. You will want to run the new relic install in silent mode `NR_INSTALL_SILENT` and also provide the `NR_INSTALL_KEY`.

> **Note!** Don't forget to replace `XXX` from `ENV NR_INSTALL_KEY XXX` with your new relic key.


Next you can install new relic php module from the tar archive. I used this approach because our dockers are based on [Alpine Linux](https://alpinelinux.org/).

> **Note!** From New Relic documentation: "Download the appropriate tar distribution file from the New Relic website. For example, for SmartOS download newrelic-php5-X.X.X.X-solaris.tar.gz or for Alpine Linux download newrelic-php5-X.X.X.X-linux-musl.tar.gz."

```
RUN mkdir /opt && cd /opt \
	&& wget https://download.newrelic.com/php_agent/release/newrelic-php5-6.8.0.177-linux-musl.tar.gz --no-check-certificate \
	&& tar -xzvf newrelic-php5-6.8.0.177-linux-musl.tar.gz \
	&& ./newrelic-php5-6.8.0.177-linux-musl/newrelic-install install \
	&& echo 'Configure the application name'
	&& sed -i 's/newrelic.appname = "PHP Application"/newrelic.appname = "PIM '${ENVIRONMENT}'"/g' /etc/php/conf.d/newrelic.ini
```

It's as simple as that! Thanks for reading, see you next time.
