---
layout: post
title: "Fixed Length Random numbers in Ruby"
date: 2008-01-09 18:35
comments: true
categories: [Ruby, Rails]
---

My latest project is almost complete and we're setting up a demo site with lots of fake data already included so I used the faker gem to generate most of it.

One issue I had that I couldn't do with the faker gem out of the box is get a fixed length random number.  I need this for license numbers, credit card numbers, phone numbers and others.  The faker gem does offer a random phone number but the ones in my project don't support extensions or punctuation so rather than generating the number then using string functions to strip out the stuff I didn't want, I decided to find an easier way.

<!-- more -->

I started by Googling for it assuming that someone else had already figured it out and while I'm sure its been done a hundred times before, I couldn't find it anywhere.  So after much thought and digging through the ruby docs, this is what I came up with.

{% codeblock lang:ruby %}
rand(9999999999).to_s.center(10, rand(9).to_s)
{% endcodeblock %}

rand(9999999999) will give me a number between 0 and 9999999999.  Convert it to a string and use the center method to make sure its at least 10 digits long and pad the rest with a random number from 0-9.  rand(9)

If you need a number of say 7 digits in length, just change the first rand to rand(9999999) and change the center method length to 7 so it looks like this.

{% codeblock lang:ruby %}
rand(9999999).to_s.center(7, rand(9).to_s)
{% endcodeblock %}

Hope this is helpful to someone.