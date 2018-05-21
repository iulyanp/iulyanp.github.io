---
layout: post
title:  "Run PHPUnit on multiple PHP versions using Docker"
slug: 'run-phpunit-on-multiple-php-versions-using-docker'
date:  2018-05-20 09:00:00 +0300
category: docker
tags: [docker, phpunit, php]
icon: external-link
---

One of the biggest advantage using Docker is the ability to switch between tools versions. Personally I think the 
easiest way to demonstrate how you can benefit from this advantage is to run PHPUnit tests against different PHP 
versions.

### Prepare the application

For this example I will use a package build on top of [Respect Validation](https://github.com/Respect/Validation) 
validator called [flex-validator](git@github.com:iulyanp/flex-validator.git) which offers a simpler interface to 
validate multiple inputs together and return all the errors at once.

We will clone the repository, install all it's dependencies and finally run all it's tests on 7.0 and 7.2 versions of
PHP. 

##### Clone the repo

```
$ git clone git@github.com:iulyanp/flex-validator.git
```

##### Install the dependencies using Docker

```
$ docker run --rm -v $(pwd):/var/www composer install
```

### Running UnitTests

Now it's time to run the tests on PHP 7.0 version. 

```bash
$ docker run --rm -v $(pwd):/var/www -w /var/www php:7.0 vendor/bin/phpunit

PHPUnit 6.5.6 by Sebastian Bergmann and contributors.

.....................                                             21 / 21 (100%)

Time: 624 ms, Memory: 4.00MB

OK (21 tests, 46 assertions)
```

#### What happened?

- downloaded the `php:7.0` docker image
- started a container from that image
- mapped the current folder into the `/var/www` directory `-v $(pwd):/var/www`
- made the `/var/www` directory the current one in the container `-w /var/www`
- run the `vendor/bin/phpunit` command inside the `/var/www` directory
- after tests executed the container closed and removed itself `--rm`

Or you can run it on PHP 7.2 version to see everything will work nicely with the latest PHP version.

```bash
$ docker run --rm -v $(pwd):/var/www -w /var/www php:7.2 vendor/bin/phpunit

PHPUnit 6.5.6 by Sebastian Bergmann and contributors.

.....................                                             21 / 21 (100%)

Time: 751 ms, Memory: 4.00MB

OK (21 tests, 46 assertions)
```

As you can see this is a very clean way to test our application or package on different PHP versions without installing 
them locally.
