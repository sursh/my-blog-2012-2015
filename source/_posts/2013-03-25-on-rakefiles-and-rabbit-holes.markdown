---
layout: post
title: "On Rakefiles and Rabbit Holes"
date: 2013-03-25 15:34
comments: true
categories: [bash, hackerschool]
---

TL;DR. I noticed a bug in Octopress that throws a `TCPServer Error` under certain conditions. I dug in and found a truly spectacular error, and found a bunch of interesting things about bash. Hacker School is a place where I can indulge this kind of curiosity. 

## The original bug.

Octopress' `rake preview` lets you preview your post in the browser before deploying. It spawns `rack` to serve local requests to `localhost` and `guard` to watch for changes to your blog files. `Guard` in turn spawns `fsevent` to do the actual watching. But when you `crtl-C`, `fsevent` stays running in the background until you `kill` it or modify a file in the dir that it's watching. This throws a `TCPServer Error` if you try to run `rake preview` a second time. 

```
$ pstree
...
 | | \-+= 31340 root login -pf sasha
 | |   \-+= 31341 sasha -bash
 | |     \-+= 31430 sasha ruby /Users/sasha/.rvm/gems/ruby-1.9.3-p374/bin/rake 
 | |       |-+- 31433 sasha ruby /Users/sasha/.rvm/gems/ruby-1.9.3-p374/bin/gua
 | |       | \--= 31443 sasha /Users/sasha/.rvm/gems/ruby-1.9.3-p374/gems/rb-fs
 | |       \--- 31434 sasha (ruby)
...

Initially I had no idea what was going on here. I've dabbled in Ruby but not modified a Rakefile before. I learned `ps` and `kill` ten years ago, but didn't know much about what was going on under the hood. Ripe conditions for learning a ton. 

**This is what I love about Hacker School**. I have the time and space to dig into these rabbit holes out of pure curiosity. At a startup, priority goes to shipping code. **Here, shipping code is the means, not the end**

The actual bug was pretty small: when you interupt `rake`, it passes `kill 9` to guard. Guard ends itself but doesn't properly terminate ``fsevent`. 

  trap("INT") {
    [guardPid, rackupPid].each { |pid| Process.kill(9, pid) rescue Errno::ESRCH }
    exit 0
  }

This can be duct-taped together by sending 3 (quit) instead of 9 to `guard`, but we're asking the guard team if this is a known issue. In the process of figuring this out, I got sidetracked by an even more interesting bug. 

## Curiouser and curiouser

Things got *really* weird when I was playing around with the different exit codes. The Ruby documentation says that if you pass `kill` a negative argument, it will kill the entire group, not just the child process. Promising! 

However, it broke very spectacularly. If you want to follow along, grab the development branch of Octopress, currently 2.1: 

    git clone -b 2.1 https://github.com/imathis/octopress.git

Find the `preview` task and change the message it passes on interrupt from 9 to -3: 

  trap("INT") {
    [guardPid, rackupPid].each { |pid| Process.kill(-3, pid) rescue Errno::ESRCH }
    exit 0
  }

Then run `rake preview` (you will have to run `rake install` the first time). Wait until you see Guard's prompt:

    16:10:05 - INFO - Guard is now watching at '/Users/sasha/code/octopress'
    [1] guard(main)> 

Then Control-C. You will see some lines of error output, and then BOTH guard and bash's prompt: 

    [2] guard(main)> SASHAs-MacBook-Air-2:octopress sasha$ 

Weird! Press any key that sends input. PRY EXPLODES. Press enter and you get a bash prompt back, but now you can't see anything you type. It's still getting to bash (try `ls <enter>`) but some things, like control-L, no longer work. 

Lolwut?

## Down the rabbit hole

So I started reading about [TTY](http://www.linusakesson.net/programming/tty/) and [POSIX signals](http://en.wikipedia.org/wiki/Unix_signal) and [stty](http://unixhelp.ed.ac.uk/CGI/man-cgi?stty). Interesting stuff, particularly the history of our terminal evolving from ticker tape outputs.

I didn't get all the answers, but I'm asking _significantly_ better questions: 

- How groups are being used is unclear. I expected the process group to inherit its gid from the parent process and to be killed cleanly by passing a negative argument to `kill`, but that didn't work. Passing `-3` kills rack but only mortally wounds guard. 
- It's clear that when control is handed back to bash, some flags are changed. `-echo` for one, which is what suppressed the typed output. And it switches to raw (non-canonical) mode, which is why it won't process things like `control-l` until you hit enter. Read more about this at `stty(1)`. 

    bash The flags that get changed.
    $ diff stty.before stty.after

- One remaining mystery: why is the bash prompt printed to the terminal after the guard prompt when guard is still running? Maybe control is passed from guard to bash and then back to guard?
- And why are the flags on TTY changed at all as control is passed around? Is guard/pry using /tty/dev00X?
- it seems that pry is getting something it's not expecting from its stdin. not sure how to dig into this further. 
- Useful thing learned: the Readlines library defines some bash shortcuts, like mapping `ctrl-l` to `clear`, but lets you override them with an .inputrc. So you can do things like adjusts how `ctrl-w` works. There are also vim and emacs modes for bash. 

Stuff I learned: a little Ruby, Rakefiles, POSIX signals, the history of TTY, more about stdin, stdout, and stderr, stty, working with pids, gids, and the Process module in Ruby. And my first accepted (albeit one-character) pull request of Hacker School. Highest learning-to-LOC ratio ever! 