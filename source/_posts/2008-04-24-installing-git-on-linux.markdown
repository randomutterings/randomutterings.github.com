---
layout: post
title: "Installing git on linux"
date: 2008-04-24 18:41
comments: true
categories: [Infrastructure, Git]
---

Trying to install git from source on our linux server (we're running RHEL4 at work) and I was getting an error running make.

    undefined reference to `libiconv'

Running make like this worked.

    make install CFLAGS="-liconv"

But after the install was finished, running git barfed at me about loading a shared library libiconv.so.2.

I had to edit /etc/ld.so.conf and add /usr/local/lib and run ldconfig.

Credit Mark Turner for the ldconfig tip.  Thanks man.