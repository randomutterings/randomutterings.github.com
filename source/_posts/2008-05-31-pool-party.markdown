---
layout: post
title: "Pool-party"
date: 2008-05-31 18:43
comments: false
categories: [Infrastructure, Ruby, Rails, EC2]
---

![jump in](http://blog.randomutterings.com/images/articles/489184472_214de5bfee.jpg)

Photo by [hrlndspnks]("http://www.flickr.com/photos/mancake/489184472/)

Having a great time at Railsconf08.  Meeting some great people and learning a lot.  I plan to post in more detail about a lot of this after the conference but I wanted to go ahead and mention one of the coolest presentations I've seen so far.

Ari Lerner has created a ruby gem called pool-party.  It auto-scales your ec2 cluster based on criteria you set in the config file like average requests per second.  Set your minimum requests per second and if the average of all the servers on your cluster goes over the maximum, pool-party will launch another instance for you.  It scales down in the same way.  It doesn't care what you're using your cluster for and a its going to support plugins soon so you can add in your own hook like functionality.  Pool-party is using s3fuse to mount an s3 bucket on the instance  at startup.  Specify a bucket in pool-party and it gets mounted at /data on each instance.  This solves the data persistence issue with ec2.  You're not required to use it though, just don't specify a bucket in pool-party and nothing gets mounted.

Ari open-sourced Pool Party as of 2 days ago.  Check it out at poolpartyrb.com.  If you want to contribute, its on github at [http://github.com/auser/pool-party](http://github.com/auser/pool-party).