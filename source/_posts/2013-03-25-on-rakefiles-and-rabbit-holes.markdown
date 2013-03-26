---
layout: post
title: "On Rakefiles and Rabbit Holes"
date: 2013-03-25 15:34
comments: true
categories: [bash, hackerschool, unix]
---

**TL;DR.** I noticed a bug in Octopress. In fixing it, I found a separate, truly spectacular error, and learned a lot of interesting things about bash. I would never have learned so much outside of Hacker School, since it gives me the time and space to open up the box. 

## The original bug.

Octopress' `rake preview` lets you preview your post in the browser before deploying. It spawns Rack to serve local requests and Guard to watch for file changes. Guard in turn spawns Fsevent to do the actual watching. 

``` bash Handy pstree is handy.
$ pstree
...
 | | \-+= 31340 root login -pf sasha
 | |   \-+= 31341 sasha -bash
 | |     \-+= 31430 sasha ruby /.../ruby-1.9.3-p374/bin/rake 
 | |       |-+- 31433 sasha ruby /.../ruby-1.9.3-p374/bin/gua
 | |       | \--= 31443 sasha /.../ruby-1.9.3-p374/gems/rb-fs
 | |       \--- 31434 sasha (ruby)
...
```

But when you `crtl-C`, Fsevent stays running in the background until you `kill` it or modify a file in the watched dir. This throws a `TCPServer Error` if you try to run `rake preview` a second time. Sad pandas. 

Initially I had no idea what was going on here. I've dabbled in Ruby but not modified a Rakefile before. I learned `ps` and `kill` ten years ago, but didn't know much about what was going on under the hood. Ripe conditions for learning a ton. 

**This is what I love about Hacker School**. I have the time and space to go down these rabbit holes. At a startup, priority goes to shipping code. **Here, shipping code is the means, not the end**

In the end, the actual bug was pretty small: when you interrupt `rake`, it passes `kill 9` to guard. Guard ends itself but doesn't properly terminate its child process fsevent`. 

``` ruby Rake preview catching interrupts
  trap("INT") {
    [guardPid, rackupPid].each { |pid| Process.kill(9, pid) rescue Errno::ESRCH }
    exit 0
  }
```

This can be duct-taped together by sending `3` (QUIT) instead of `9` to Guard, but we're asking the Guard team if this is a known issue. But it gets better! 

## Curiouser and curiouser

Things got *really* weird when I was playing around with the different exit codes. The Ruby documentation says that if you pass `kill` a negative argument, it will kill the entire group of processes, not just the child process. Promising! 

However, it broke very spectacularly. If you want to follow along, grab the development branch of Octopress, currently 2.1: 

    $ git clone -b 2.1 https://github.com/imathis/octopress.git

Find the `preview` task and change the message it passes on interrupt from 9 to -3: 

``` ruby Line 161 as of this writing
  trap("INT") {
    [guardPid, rackupPid].each { |pid| Process.kill(-3, pid) rescue Errno::ESRCH }
    exit 0
  }
```

Then run `rake preview` (you will have to run `rake install` the first time). Wait until you see Guard's prompt:

    16:10:05 - INFO - Guard is now watching at '/Users/sasha/code/octopress'
    [1] guard(main)> 

Then Control-C. You will see some lines of error output, and then BOTH guard and bash's prompt. Wat!

    [2] guard(main)> SASHAs-MacBook-Air-2:octopress sasha$ 

Weird! Press any key that sends input. PRY EXPLODES. Press enter and you get a bash prompt back, but now you can't see anything you type. It's still getting to bash (try `ls <enter>`) but some things, like control-L, no longer work. 

Lolwut?

## Down the rabbit hole

So I started reading about [TTY](http://www.linusakesson.net/programming/tty/) and [POSIX signals](http://en.wikipedia.org/wiki/Unix_signal) and using [stty](http://unixhelp.ed.ac.uk/CGI/man-cgi?stty). Interesting stuff, particularly the history of our terminal evolving from ticker tape outputs.

You can also change all sorts of wacky things about your terminal with `stty`. Try `stty -echo` (`stty echo` to undo it). This explains why I wasn't able to see my own typing after the Pry explosion - when control was reluctantly handed back to bash, the flags on my terminal weren't properly reset, including the flag to use raw (non-canonical) input processing, which is why it won't process things like `control-l` until you hit enter. 

I didn't find all the answers in my reading, but I'm asking _significantly_ better questions: 

- How groups are being used is unclear. I expected the process group to inherit its gid from the parent process and to be killed cleanly by passing a negative argument to `kill`, but that didn't work. Passing `-3` kills Rack but only mortally wounds Guard. 

- One remaining mystery: why is the bash prompt printed to the terminal after the Guard prompt when Guard is still running? Maybe control is passed from Guard to bash and then back to Guard?

- It seems that Pry is getting something unexpected from its stdin, which triggers its explosion, which may or may not be coming from bash. But how to intercept it?

- The Readlines library defines some bash shortcuts, like mapping `ctrl-l` to `clear`, but lets you override them with an .inputrc. So you can do awesome things to your prompt like adjusts how `ctrl-w` works. There are also vim and emacs modes for bash. 

**Stuff I learned:** a little Ruby, Rakefiles, POSIX signals, the history of TTY, more about stdin, stdout, and stderr, stty, working with pids, gids, and the Process module in Ruby. And my first accepted (albeit one-character) pull request of Hacker School, which is certainly the highest learning-to-LOC ratio I can think of.  