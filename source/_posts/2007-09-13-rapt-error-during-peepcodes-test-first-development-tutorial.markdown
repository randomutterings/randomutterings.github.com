---
layout: post
title: "Rapt error during peepcodes test first development tutorial"
date: 2007-09-13 13:57
comments: true
categories: [Ruby, Rails]
---

OK, let me start off by saying, I love the tutorials from peepcode, you guys are doing a great job so keep doing it.

Alas, I got stuck while watching the [Test-first-development](http://peepcode.com/products/test-first-development) tutorial.  They were using [Rapt](http://rapt.rubyforge.org/) as an alternative to script/plugin and when I tried to run it I kept getting an error.

<!-- more -->

    Error: RaPT currently does not work outside of a Rails application directory.  Please change to the top level of a Rails application and try again.

This error is misleading but at least it was obviously misleading, after all I know I'm in my Rails application directory.

After some googling, I found a post describing a work around for the problem, but it basically gave instructions on installing the plugin with script/plugin.  But the guys at peepcode had a reason for using rapt over script/plugin (its faster).  So I Google some more.  I finally find a [bug](http://rubyforge.org/tracker/index.php?func=detail&aid=6611&group_id=1889&atid=7384) on the Rapt [Trac](http://rubyforge.org/projects/rapt/) posted in 2006 that explains that the user must first add repositories by executing the following.

    rapt discover

If anyone knows how to have it default to yes without prompting you for every repository, please post a comment.

Alternatively, you can add only the repositories you want to use by executing the following.

    rapt source http://path-to-repository