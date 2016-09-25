---
layout: post
title:  "Flexbox introduction - Center a logo on the screen?"
slug: 'flexbox-introduction'
date:   2016-09-15 12:01:06 +0300
categories: flexbox
tags: [flexbox, css]
icon: th
---



Even I'm not working on frontend sometimes I like to take a pick in what's new and awesome on CSS.
And these days seems like *flexbox* is the shining star on the stage.

### Why you should use flexbox?

If you're wondering why you should care about flexbox the answer it's very very simple: "Flexbox rocks".
I'm serious, you want to learn to use flexbox if you're a frontend developer. 
Let's make a short imagination exercise and think about what steps you would needed to make for perfectly center 
a logo on a responsive page.

1. Ok, so back in 2014 a good way to center a logo was to wrapp the logo in a container and try first to center it 
horizontaly using the image `width` and `margin: 0 auto;` which will do the trick for you. 
2. But now how you can center it verticaly? Hmmm, you would say "no problem, set the container `height: 100%;` and then style the 
logo with `position: absolute;` `top: 50%;`". 
3. But still, the logo isn't perfectly centered and you have to subtract half of its height with `transform: translate(0, -50%)`.

Your code would look someting like this:

```
<div class="container">
   <img src="images/logo.jpg" class="logo">
</div>
```


```
.container {
   width: 123px;
   height: 100%;
   margin: 0 auto;
}              

.logo {
   margin: 0;
   position: absolute;               
   top: 50%;                        
   transform: translate(0, -50%);
   background: green;
}

```
Now I know that there are other ways of doing this but I just wanted to show you that wasn't very clean.
In conclusion, it's pretty anoying, especialy if you have multiple layers. But with *flexbox* not anymore.

Let me show you how easy can be if you use *flexbox*.
Ok, so first we have the html page with just an image tag inside the body.

```
<body>   
   <img src="images/logo.jpg">
</body>
```

In your css file you need to be sure that the body and the html page take all the html page with `height: 100%;`.


```
html, body {
   height: 100%;
}
```

Then you just have to enable `display: flex;` but you'll not see any visible change.
So let's go and set the `align-items: center;` and voila the logo is perfectly verticaly centered. 
But don't make the mistake of thinking that aline-items works only verticaly, if fact it's a little dependent on the primary axes. <br>
Last we just need to center the logo horizontaly with `justify-content: center;` and here we go, it's perfectly aligned in the center of the page.

```
body {
   display: flex;
   align-items: center;
   justify-content: center;
}
```

You can style this however you want but now you have a perfectly aligned logo with just 3 lines of code.


In the next post about flexbox I'll show you how we can create a horizontal navigation menu, so keep in touch.

