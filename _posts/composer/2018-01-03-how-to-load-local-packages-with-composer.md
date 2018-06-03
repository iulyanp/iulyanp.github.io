---
layout: post
title:  "How to properly load local dependencies with composer"
description: 'How to work on applications with local packages and composer'
slug: 'how-to-properly-load-local-dependencies-with-composer'
date:  2018-1-3 19:00:00 +0300
category: composer
tags: [composer, local dependencies]
icon: external-link
---

Sometime you may find yourself working on an application and needed to load some local package, component or fork in 
order for you to be able to **temporarily** try some things and keep everything clean and organized while you're 
developing.
You're first thought could be to copy the package content into your `vendor` directory or even symlink it. But that's 
the ugly way to do this, fortunately there's a better way.

Let's take a simple example and consider an application with the following structure:

```
apps/
    app/
        composer.json
    
component/
    package1/
        composer.json
    package2/
        composer.json
```

Composer has the concept of **[repositories](https://getcomposer.org/doc/05-repositories.md#repository){:target="_blank"}** 
which are nothing more than a package source. As the official composer documentation says, composer will look inside all 
the repositories for the packages your application needs.
By default composer has one primary repository configured and you're already using it: Packagist. But of course you can 
configure other repositories in your application `composer.json` file.

> **! Note:**
> Repositories are available **only** for the root package, because [composer does not load repositories recursively](https://getcomposer.org/doc/faqs/why-can't-composer-load-repositories-recursively.md){:target="_blank"}.

Your `composer.json` file should look like this:

```
  ...
  "repositories": [
    {
      "type": "path",
      "url": "../component/package1"
    }
  ],
  "require": {
    "vendor/package1": "*"
  }
  ...
```

Composer will try first to symlink the **package** if possible, else the package will be copied. You can also force 
composer to use symlink with `"symlink": true`.

```
  ...
  "repositories": [
    {
      "type": "path",
      "url": "../component/package1",
      "options": {
        "symlink": true
      }
    }
  ],
  "require": {
    "vendor/package1": "*"
  }
  ...
```

> **! Note:**
> If you want to disable the default repository [Packagist](https://packagist.org){:target="_blank"}, you can do it by adding the following in your 
> composer.json file.
  
  ```
  "repositories": [
      {
          "packagist.org": false
      }
  ]
  ```
  
