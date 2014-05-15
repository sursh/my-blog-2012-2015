---
layout: post
title: "The Slacker Developer's Guide to the Mavericks Upgrade"
date: 2014-05-14 22:00
comments: true
categories: 
---

I am not an early adopter.

Until last week, I was running 10.7. Yes, it's true. Don't judge. I had code to ship. No time for yak shaving! 

But finally I bit the bullet. After dual redundant spatially-distributed backups, I ran the upgrade. And everything went great at the GUI level! And the Messages app is awesome! But Ruby, virtualenv, and Java were all borked. I don't even Ruby, but I use Octopress and Heroku, so I couldn't even blog or restart Hubot. Sadness! 

Some tips in your upgrade: 

1. Homebrew links will probably be messed up. `brew unlink foo` and then `brew link foo` fixed most of these. Consider scripting it to save time and heartache! 

1. Ruby has a new version, and RVM is now using `.ruby_version` instead of `.rvmrc` to keep track of Ruby versions. It automatically moved these files around without telling me, which is half of what broke my blog. 

1. Ruby 2.whatever will complain about Readlines or Yaml or OpenSSL depending on its mood. Try uninstalling Ruby with RVM, installing whatever it demands, and then reinstalling Ruby and seeing if it works. At this point, I'm basically thinking of Ruby as a demanding newborn that is astonishingly articulate about its needs.  

1. Java is gone because reasons. [Here's how to reinstall it](http://osxdaily.com/2013/11/01/install-java-os-x-mavericks/).

1. I'm still not sure what's going on with my virtualenv but I think it has to do with my questionable decision to use a third-party Python distro. Passing in a path to the system Python with `-p` seems to be a decent workaround. 

1. A bunch of packages wouldn't install through pip, and it turns out that the new default when flags are not recognized by gcc is to CATASTROPHICALLY CRASH rather than keep calm and carry on. Don't have time for this drama? Add the following to your environment: 

``` python
export CFLAGS=-Qunused-arguments 
export CPPFLAGS=-Qunused-arguments
```
 
Back to work! 
