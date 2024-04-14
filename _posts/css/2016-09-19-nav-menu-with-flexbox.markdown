---
layout: post
title:  "Navigation menu with flexbox"
description: 'Build a basic navbar with flexbox'
slug: 'navigation-menu-flexbox'
date:   2016-09-19 12:01:06 +0300
categories: flexbox
tags: [flexbox, css]
icon: bars
---


Welcome back!
I'm not very good at introductions so today I'll skip it and dive directly into the subject. 

### How we can use flexbox to create a navigation menu?

First of all I'm gonna create the html for the menu. Let's say a nav tag with an unordered list and 4 anchor tags.

```html
<body>
<nav>
    <ul>
        <li class="nav-item"><a href="#">Home</a></li>
        <li class="nav-item"><a href="#">About</a></li>
        <li class="nav-item"><a href="#">Services</a></li>
        <li class="nav-item"><a href="#">Contact</a></li>
    </ul>
</nav>
</body>
```

Ok, looks exacly like we imagined it but that's fine, a little bit of styling and everything will look nice.
The first thing I'll do is zero up the margin and padding of body and ul and also clear the list style.
Then, because I want to display the links one after the other I'll use `display: flex;` and of course the result is exacly what we need.

```
html, body {
    padding: 0;
    margin: 0;
}

nav {
    background: #e3e3e3;
}

nav ul {
    margin: 0;
    padding: 0;
    display: flex;
    list-style: none;
}
```

In the last post I showed you how to center things on the page with `justify-content: center;` but now I'll show you the other values of justify-content: `space-around`, `space-between`, `flex-start` and `flex-end`. 
In our example I'll use the first one because I want the links to be equaly distributed on the screen.

```
nav ul {
    margin: 0;
    padding: 0;
    display: flex;
    list-style: none;
    justify-content: space-around;
}
```
<img src="/images/flexbox/space-around.jpg" class="content-img">

But what about the others. If you change it to `justify-content: space-between;` you'll see that the differance is not big but it's there, the spaces between the links are equal.

```
nav ul {
    margin: 0;
    padding: 0;
    display: flex;
    list-style: none;
    justify-content: space-between;
}
```
<img src="/images/flexbox/space-between.jpg" class="content-img">
The `flex-start` and `flex-end` as you can imagine will drag the links to the start or to the end of the nav, very good for displaying the logo and sign up and sign in links on a navbar.

```
nav ul {
    margin: 0;
    padding: 0;
    display: flex;
    list-style: none;
    justify-content: flex-start;
}
```
<img src="/images/flexbox/flex-start.jpg" class="content-img">

```
nav ul {
    margin: 0;
    padding: 0;
    display: flex;
    list-style: none;
    justify-content: flex-end;
}
```
<img src="/images/flexbox/flex-end.jpg" class="content-img">
The final css will look someting like this:

```
html, body {
    padding: 0;
    margin: 0;
    height: 100%;
    font-size: 16px;
    font-family: sans-serif;
}

nav {
    background: #fff;
    border-bottom: 1px solid #e3e3e3;
}

nav ul {
    margin: 0;
    padding: 0;
    display: flex;
    list-style: none;
    justify-content: space-around;
}

.nav-item {
    padding: 20px 0;
}

.nav-item a {
    color: #FF5733;
    text-decoration: none;
    font-weight: bold;
}

.nav-item a:hover {
    color: #C70039;
}

```

Hmm, that was easy, hope you enjoy it. See you next time...