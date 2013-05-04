---
layout: post
title: "Changing images with JQuery and the fade effect"
date: 2009-02-15 18:47
comments: false
categories: []
---

![me and lydia](http://farm9.staticflickr.com/8405/8706164820_edf4dc7753.jpg)

Photo by [me!](http://www.flickr.com/photos/randomutterings/)

It took me a bit of digging to figure out how to fade out a picture, change the src, then fade it back in.  I started out trying to use Scriptaculous but I got stuck trying to get the queue setup to wait until the image was faded out to change the src.  I ended up using jQuery which also gave me a bonus, the solution is unabtrusive.

Here are the application.js and show.html.haml files from my project.

<!-- more -->

<script src="http://gist.github.com/64723.js"></script>