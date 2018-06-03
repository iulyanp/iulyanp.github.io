---
layout: post
title:  "What is composer and how to use it?"
description: 'Basics of composer for newcomers'
slug: 'what-is-composer'
date:   2016-10-05 12:01:06 +0300
categories: php
tags: [composer, php, modern php]
icon: spinner
---

Composer is from my point of view one of the most important actor when we're talking about a modern php application.

In order to install composer, open a terminal and execute:

```
$ curl -sS https://getcomposer.org/installer | php
```

This command downloads the Composer installer script with curl, executes the installer script with php, 
and creates a `composer.phar` file in the current working directory. The `composer.phar` file is the Composer binary.

If you would like to run composer globally from your terminal you can execute the following:

```
# Move the composer.phar into /usr/local/bin/composer
$ sudo mv composer.phar /usr/local/bin/composer

# Make the composer file executable
$ sudo chmod +x /usr/local/bin/composer
```
You should now be able to run composer in your terminal from anywhere.

### Packagist

Now that you have composer installed on your machine let's install some PHP components with it.
First of all we have to make a list of the components we need for our project. To do this we can use 
[packagist](https://packagist.org/) to search for components that might be exactly what we need. If we
search on packagist for `flysystem` a popular package we will see that we find a component named 
`league/flysystem`. This name comes from the vendor name which is globally unique and the package name
is the one that uniquely identifies a single package within a vendor. 
This convention is used to avoid the name colisions between components from different vendors.

### How to install a PHP component

A component can have different available versions. Usualy the components are versioned using what we call [semantic 
versioning](http://semver.org/) scheme. You can find the availabe versions for a component on the Packagist 
component page.

### Semantic versioning

The semantic versioning scheme contains three numbers separated by dots (eg. 1.10.0). The first number represents the 
**major release** which means that the changes in that version will break backward compatibility. The second represents 
the **minor release** number which is incremented when the new features are added but the backward compatibility will 
not break. The third is the **patch release** number which is used to mark bug fixes on the component.

To install a component in our new project all we need to do is to run:

```
$ composer require vendor/package

$ mkdir test-composer && cd test-composer
$ composer require league/flysystem
```

This command will tell composer to search and install the most stable version of the `league/flysystem` component. 
To see the result of the command you can open the `composer.json` file that was created into the project folder.
There is also a `composer.lock` file that is generated along with the json file. All that you have to know about
the lock file is that in this file are listed all the PHP components you used in your project toughether with every
version number. This basicaly locks down the project to those specific versions.
It's a good practice to commit both on your version control system.

If you don't want to update the components version when you install the project with composer you just have to run
`$ composer install`.

If you want to update your dependencies you can run `$ composer update`, this will update also the .lock file.

