---
layout: post
title: "Turn on Git's Autocomplete"
date: 2013-04-01 18:39
comments: true
categories: [git]
---

**Did you know that you can turn on autocomplete for git?** A quick survey of other Hacker Schoolers turned up that most Linux distros come with this built in, but most Mac users didn't have it installed.

{% img http://i2.kym-cdn.com/entries/icons/original/000/000/248/underpants.jpg Underpants Gnomes Business Plan %}

1. Streamline your tools
2. Save time
3. Profit!

It's super easy. Put [git's autocomplete script](https://github.com/git/git/blob/master/contrib/completion/git-completion.bash) in your home directory. Then add it to your .bashrc by running `source ~/.git-completion.bash`. 

There's more detail [here](http://git-scm.com/book/en/Git-Basics-Tips-and-Tricks), along with aliases, which are a MUST. I added `git unstage FILENAME` which I find much more intuitive than `git reset HEAD -- FILENAME`. Hooray! 

Part of my morning routine here at [Hacker School](http://hackerschool.com) is learning a few new things about Python, bash, and git, since in the long run, **knowing these tools like the back of my hand saves tons of time**. 

Let me know on [Twitter](http://twitter.com/sashalaundy)/Humbug if you want to hear more about this routine. 