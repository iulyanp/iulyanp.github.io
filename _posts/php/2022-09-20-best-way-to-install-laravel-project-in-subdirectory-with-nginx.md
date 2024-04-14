---
layout: post
title:  "Install laravel project in a subdirectory with nginx"
description: 'Correct way to install laravel project in a subdirectory with nginx'
slug: 'install-laravel-project-in-subdirectory-with-nginx'
date:   2022-09-20 12:11:06 +0300
categories: php
tags: [laravel, nginx, subdirectory]
icon: code
---

We will go over how to install your laravel application under a subdirectory of another website.

For instance, we might require a second application for a `blog` to run at `iulianpopa.ro/blog` if we already have one running at `iulianpopa.ro`.

Although it seems like it should be straightforward, this is actually more complicated and full of perplexing Nginx configurations! An immediate Google search turns up a TON of unclear, ineffective examples.

## Basic nginx setup for Laravel

Here is how to set up two Laravel apps to run together when one of them is in a subdirectory of another.

First let's take a look on the default laravel nginx config.

```
server {
    listen 80;
    server_name main.com;
    root /var/www/main/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

## What we want to achieve

Let's say we don't want to install the blog code inside of our main laravel application. The "blog" application will be at `/var/www/blog`, while the main application is located in `/var/www/main`.

This means will start from the basic configuration and extend it to support our second application which lives outside the main directory.

If we remove the boilerplate from the default nginx vhost configuration, we can better see how nginx and php-fpm pair spin a laravel project.

```
server {
    listen 80;
    server_name main.com;
    root /var/www/main/public;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }
}
```

This basic nginx configuration doesn't include everything you may want in a production app, such as cache headers for static assets, but it's sufficient to get a PHP application up and running.

These sections of the manual are most important:

- `root` - we define the web root that is utilized the main application

- `index` - if no file is mentioned in the URI, we define `index.php` as one of the default files to seek for.

- `location { try_files ... }` - In a block that captures all URIs, we utilize the tried-and-true try files command. This checks the root directory to determine if a file or directory that was sent in the URI already exists. The index.php file, which is unquestionably located in the root directory, is the next step if it doesn't already exist.

## Adding an App in a directory outside our main project

When attempting to run a second application within a subdirectory outside our main project, we encountered some issues. Let's go over them. Initially, we have identified that the desired subdirectory for the second application is `/blog` so we create a new location block inside the nginx vhost.

```
location /blog {

}
```

We are aware that the blog project has a distinct web root. Although you might assume that pointing Nginx to `/var/www/blog/public` would work, it actually does not. Instead, we must use an alias.
We are aware that the blog project has a distinct web root. Although you might assume that pointing Nginx to `/var/www/blog/public` would work, it actually does not. Instead, we must use an alias.

```
location /blog {
    alias /var/www/blog/public;
}
```

To locate a file on the disk drive, Nginx combines the given URI with the root. For instance, if the URI is `/laravel/test`, Nginx will seek to serve `/var/www/main/public/laravel/test`. As this directory is not present, the `try_files` segment directs Nginx to forward the request to `/var/www/top/public/index.php`.


The behavior of an `alias` is not identical to `root`. In our case, the `/blog` URI is linked to `/var/www/blog/public`. Therefore, a URI of `/blog` will search for a file to serve in `/var/www/blog/public`. On the other hand, a URI of `/blog/laravel` will direct Nginx to search in `/var/www/blog/public/laravel`, without including the `/blog` part.

Finally we have to add the `try_files` in order to serve URI like `/blog/laravel` from file `/var/www/blog/public/index.php`. 

```
location /blog {
    alias /var/www/blog/public;
    try_files $uri $uri/ /index.php$is_args$args;

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $request_filename;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }
}
```

We have nested the PHP handler location block inside the location `/blog {...}` block to allow for some customization.

It's worth noting that I included something different in the PHP location block. I adjusted the `SCRIPT_FILENAME` FastCGI parameter to `$request_filename`. By default, this parameter is set to `$document_root$fastcgi_script_name`, which would always point to `/var/www/main/public/index.php` in our configuration - which is not the correct application!

Therefore, `$request_filename` considers the `alias` accurately and will transmit the appropriate path to the `index.php` file within our nested application.

Nevertheless, if our application has a specific route, such as `laravel`, and we attempt to access it via URI `/blog/laravel`, we encounter a 404 error.

To get around this issue, we should rewrite the URI like so:

```
location @blog {
    rewrite /blog/(.*)$ /blog/index.php last;
}
```

> P.S. In some other posts I found around the internet the rewrite block looked something like this:

```
location @blog {
    rewrite /blog/(.*)$ /blog/index.php?/$1 last;
}
```

This statement will forward the matched path in the regex to the `index.php` as a query argument.
However, with Laravel, this will append the path as a query string in the following manner: `/blog/laravel/test`.

```
// laravel
url()->full()

// returns 
/blog/laravel/test?%2Flaravel%2Ftest
```

The Laravel `index.php` does not require the path as an argument. Instead, it searches the Server `Request_URI` for path information. Therefore, a routing like the following should suffice:

```
// when fastcgi_params is included;
rewrite ^/demo1/(.*)$ /demo1/index.php last;

// or when don't have fastcgi_params included
rewrite ^/demo1/(.*)$ /demo1/index.php?$query_string last;
```

In my view, this is the optimal setup for this scenario since the majority of PHP applications employ a public directory as their web root. This can make it quite complicated to layer application files, where one project's content is placed inside another. 
Naturally, depending on the application, your situation may differ slightly, but this guide should require minimal modification.

The final result:

```
server {
    listen 80;
    server_name main.com;
    root /var/www/main/public;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location /blog {
        alias /var/www/blog/public;
        try_files $uri $uri/ @blog;

        location  ~* \.php$ {
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        }
    }

    location @blog {
        rewrite /blog/(.*)$ /blog/index.php last;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
    }
}
```
