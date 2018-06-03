---
layout: post
title:  "Compile your assets with Symfony Encore"
description: 'How to compile assets with Symfony Encore and Webpack'
slug: 'symfony-encore'
date:   2017-06-18 12:01:06 +0300
category: symfony
tags: [symfony, encore, webpack, manage assets]
icon: align-center
---


These days I'm playing with Symfony Encore, the new tool for managing assets in Symfony 3.3. It is a tool inspired from
[Laravel Mix](https://github.com/JeffreyWay/laravel-mix) which is the asset manager from Laravel Framework, build by
Jeffrey Way the owner of [Laracasts](https://laracasts.com).
At the time Jeff created Mix, I wanted to use it on Symfony projects, so I decided to wrap laravel-mix package into
a symfony bundle. So [elixir-mix-bundle](https://packagist.org/packages/iulyanp/elixir-mix-bundle) was created.
If you used it already, Symfony Encore will look a lot like it.

A few days ago Ryan Weaver the creator of [KNPUniversity](knpuniversity.com) and a Symfony evangelist, announced
[Symfony Encore](https://github.com/symfony/webpack-encore) a simple API around Webpack for compiling assets and the
[Assetic](http://symfony.com/doc/current/assetic.html) replacement.
The official documentation could be found on [Symfony website](http://symfony.com/doc/current/frontend.html) which in
fact is already using Encore.

## How to install Symfony Encore
First you have to install [Node.js](https://nodejs.org/en/download/) and also [Yarn](https://yarnpkg.com/lang/en/docs/install/)
or [NPM](https://www.npmjs.com/).
When you're ready you can install encore by running on your symfony root project:

```
$ yarn add @symfony/webpack-encore --dev

or using npm

$ npm init
$ npm install @symfony/webpack-encore --save-dev
```

You will notice that a `package.json` file was generated and all the dependencies were downloaded into a new directory
`node_modules`. You should version the json file but ignore the node_modules.

## How to use Encore

First thing to do is to create a `webpack.config.js` file at the root of your project.
I personally keep all my assets into the `app/Resources/public` folder so let's say we have a sass and a js file:

* `app/Resources/public/sass/global.scss`
* `app/Resources/public/js/app.js`

The `webpack.config.js` file will then look like this:

```
// webpack.config.js
var Encore = require('@symfony/webpack-encore');

Encore
    // directory where all compiled assets will be stored
    .setOutputPath('web/build/')

    // what's the public path to this directory (relative to your project's document root dir)
    .setPublicPath('/build')

    // empty the outputPath dir before each build
    .cleanupOutputBeforeBuild()

    // will output as web/build/app.js
    .addEntry('app', './app/Resources/public/js/app.js')

    // will output as web/build/global.css
    .addStyleEntry('main', './app/Resources/public/sass/main.scss')

    // allow sass/scss files to be processed
    .enableSassLoader()

    // allow legacy applications to use $/jQuery as a global variable
    .autoProvidejQuery()

    .enableSourceMaps(!Encore.isProduction())

    // create hashed filenames (e.g. app.abc123.css)
    //.enableVersioning()
;

// export the final configuration
module.exports = Encore.getWebpackConfig();
```

This configuration of Encore is already complex.
In order to compile everything up you should execute encore:

```
# compile assets
$ ./node_modules/.bin/encore dev

# watch when assets change and recompile them automatically
$ ./node_modules/.bin/encore dev --watch

# compile, minify and optimize assets for production use
$ ./node_modules/.bin/encore production
```

Because sometimes I'm lazy I like to set those commands in the `package.json`:

```
{
  "devDependencies": {},
  "scripts": {
    "dev-server": "./node_modules/.bin/encore dev-server",
    "dev": "./node_modules/.bin/encore dev",
    "webpack": "./node_modules/.bin/encore dev --watch",
    "prod": "./node_modules/.bin/encore production"
  },
}
```

Then you can execute encore like `$ npm run dev` for development or `$ npm run prod` for production.

At first, because you are trying to compile `sass` file it will fail and ask you to install `sass-loader` and `node-sass`.

After Encore will compile your files you can find in the `web/build/` directory all the generated files. You will
have an `app.js` file, next there will be a `global.css` file and also you will see the `manifest.json` which contains
the real name for each asset. This is very useful when you'll use versioning with Encore, will explain more later.

You can now include your new compiled files into the html.

{% raw %}
```
{# layout.html.twig #}
<!DOCTYPE html>
<html>
    <head>
        <!-- ... -->
        <link rel="stylesheet" href="{{ asset('build/global.css') }}">
    </head>
    <body>
        <!-- ... -->
        <script src="{{ asset('build/app.js') }}"></script>
    </body>
</html>
```
{% endraw %}

## The main Encore methods

### Compile Js & Sass
As you've already seen, Encore can compile Js and Sass files through `.addEntry()` and `.addStyleEntry()` methods.
Sass files are compiled and output as CSS in `build/css/global.css`, because of `.enableSassLoader()` method which you
should have in your `webpack.config.js`.

You can also compile both js and sass files at the same time with `.addEntry()` method.

```
.addEntry('app', [
    './app/Resources/public/js/app.js',
    './app/Resources/public/sass/main.scss'
])
```

The output will be an `build/app.js` and an `build/app.css` file.

### Compile Less
You could also compile `less` files if you'd like. All you have to do is install `less-loader` and `less` packages, then
enable them in `webpack.config.js` like so `Encore.enableLessLoader()`.


## How to use Source Maps

[Source Maps](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map) helps you to easily debug
your `sass`, `less`, `typescript` files allowing browsers to point to the original file related to some asset. Let's say
you have a `main.scss` and `pages.scss` files that you compile into `main.css` stylesheet. When browser will interpret
your `main.css` file if you're using Source Maps, it will point to the `pages.scss` file if you are debugging something
from that file not at the generated `main.css` file.

Encore includes by default source maps only in development mode `.enableSourceMaps(!Encore.isProduction())` but you could
change this anytime.

## Assets Versioning

If you would like to version your assets, encore gives you the best way to do so. Just by calling `.enableVersioning()`
method into your `webpack.config.js` file, each asset name you are compiling with encore will contain a hash
`app.abc432.js` instead of `app.js`. Whenever your `app.js` content will change, encore will generate a new hash on your
filename and your browser will need to load your new changes.

But wait, how could you know which hash encore used for your new generated file. Should you change the asset name every
time you compile your files?
The answer is "No way!". If you remember a few moments before I said something about a `manifest.json` file that is
generated together with your compiled assets.
This file is the magic behind finding your asset real name.

```
{
  "build/app.css": "/build/app.bf4fd3bb74643601faf6b4f985f9a7d7.js",
  "build/app.js": "/build/global.9bb94772fa673c01ebf7.css"
}
```

In Symfony this file is used by the `json_manifest_file` versioning strategy from `config.yml`.

```
# app/config/config.yml
framework:
    # ...
    assets:
        # feature is supported in Symfony 3.3 and higher
        json_manifest_path: '%kernel.project_dir%/web/build/manifest.json'
```

## The Twig asset() function

That's how it works, just be sure to use Twig {% raw %}{{ asset() }}{% endraw %} function when you want to load js or 
css files.

{% raw %}
```
<link href="{{ asset('build/main.css') }}" rel="stylesheet" />

<script src="{{ asset('build/app.js') }}"></script>
```
{% endraw %}

Remember that if you're not using cache busting you don't have to use `asset()` function, but as a best practice you
always should use it because it will make it a lot easier when you decide to add asset versioning.

## Using bootstrap Js & Css

I know many projects are using bootstrap so here's how it works with Encore. First install `bootstrap-sass`:

```
$ npm install bootstrap-sass --dev
```

### Import Bootstrap from sass

After that bootstrap is downloaded into your `node_modules` directory and you can import it in any of your sass or
javascript files.

```
// app/Resources/public/sass/main.scss
@import '~bootstrap-sass/assets/stylesheets/bootstrap';
```

After you include bootstrap-sass the Webpack builds might become slower but you can fix that with `resolve_url_loader`
option:

```
Encore
    .enableSassLoader({
        resolve_url_loader: false
    })

// For Encore v0.15.0 you have to pass as a first parameter a closure that receives an options parameter and 
// as a second parameter the object with `resolveUrlLoader` option:

Encore
    .enableSassLoader(function(options) {
        // options.includePaths = [...]
    }, {
        resolveUrlLoader: false
    })
```

If you want to learn more about what this is about you can check [problems with url()](https://github.com/webpack-contrib/sass-loader#problems-with-url)

### Import Bootstrap from Js

As you already know `bootstrap.js` requires jquery so you have to install it also.

```
$ npm install jquery --dev
```

You should also include in `webpack.config.js` the `.autoProvidejQuery()` method because Bootstrap needs jQuery as a
global variable. All this does is expose the `$` and `jQuery` global variables so that Bootstrap could find them.

```
WebpackConfig.autoProvideVariables({
   $: 'jquery',
   jQuery: 'jquery'
});
```

Finally you can require bootstrap in your custom js file:

```
var $ = require('jquery');
require('bootstrap-sass');

(function() {
    $('body').html('Using jQuery');
})();
```

## Extracting your vendors

Sometimes you end up with the scenario in which you want your vendors code saved in a different file then your custom
code. For example it's possible that you'd want that jquery and bootstrap js files saved in a `vendors.js` because that
code is on all your pages and will not change often, but your custom js code saved in the `app.js` file.
For that encore comes with `.createSharedEntry()` function which does exactly that.

```
Encore
    // ...
    .addEntry('app', [
        'app/Resources/public/js/custom.js',
        'app/Resources/public/js/anotherCustom.js'
    ])
    .createSharedEntry('vendor', ['jquery', 'bootstrap'])
```

This configuration will compile and extract in a `vendor.js` the jquery and bootstrap libraries. This file will need to
be included into your html, but beside this another `manifest.js` file is created and need to be also loaded.

{% raw %}
```
<!-- included in your layout.html.twig -->
<script src="{{ asset('build/manifest.js') }}"></script>
<script src="{{ asset('build/vendor.js') }}"></script>
<script src="{{ asset('build/app.js') }}"></script>
```
{% endraw %}

The `manifest.js` file is used by Webpack in order to know how to load the shared modules.

As I already said, the benefit of going with this strategy is long-term caching, meaning that the browsers will cache
your `vendors.js` because it's code will not change often, but also have your custom code busting the browser cache when
it changes.

Also this approach can be useful when you want to create page-specific css/js.

```
Encore
    // ...
    .addEntry('app', [
        'app/Resources/public/js/custom-global.js',
        'app/Resources/public/sass/main.scss'
    ])

    .createSharedEntry('vendor', ['jquery', 'bootstrap'])

    .addEntry('checkout', [
        'app/Resources/public/js/checkout-page.js',
        'app/Resources/public/css/checkout-page.scss'
    ])
```

{% raw %}
```
<!-- included in your layout.html.twig -->
<script src="{{ asset('build/manifest.js') }}"></script>
<script src="{{ asset('build/vendor.js') }}"></script>
<script src="{{ asset('build/app.js') }}"></script>

<!-- include in your checkout file -->
<script src="{{ asset('build/checkout.css') }}"></script>
<script src="{{ asset('build/checkout.js') }}"></script>
```
{% endraw %}

## In conclusion

Symfony Encore is the new tool that replaces Assetic in Symfony. Of course if you prefer Laravel Mix you could use
[elixir-mix-bundle](https://packagist.org/packages/iulyanp/elixir-mix-bundle), but now that Encore is out there you can
make your own mind and use whatever feels great to you. Go build great things.
