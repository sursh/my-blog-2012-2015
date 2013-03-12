---
layout: post
title: "Turbocharging Octopress"
date: 2013-03-11 19:05
comments: true
categories: [octopress]
---

This blog is powered by [Octopress](http://octopress.org/), which is basically a set of `rake` tasks, themes, and other add-ons generate a blog from Markdown posts. It's in turn powered by [Jekyll](https://github.com/mojombo/jekyll). 

The documentation is generally pretty good, but they didn't really explain one fundamental thing. It's pretty simple once you dig into the `Rakefile`, but here's a quick explanation if you just want to get up and running. 

### WTF is going on with the branches?

On github, you will have two branches: `source` and `master`. But locally: 

``` 
$ git br
* source
```

Huh. Interesting. Locally, you *only* have one branch: `source`.  Wat?

Basically, `source` holds your posts in Markdown and other files *before* they are transmogrified into HTML. Once you run `rake generate`, Octopress will generate all the HTML & CSS and put it all in `/public`. 

And when you run `rake deploy`, Octopress pushes the contents of `/public` (on *local* branch `source`) to the home directory (on *remote* branch `master`).

So: each time you finish a post, run BOTH `rake deploy` to deploy and `git push origin source` to back up your source files to github. 

### Sublime Text <3 Markdown

So you're writing your blog posts in Markdown like a boss. There are a few things you can do to make Sublime Text 2 a lean, mean blogging machine: 

**Custom themes**. For example, [MarkdownEditing](http://brettterpstra.com/2012/05/17/markdown-editing-for-sublime-text-2-humble-beginnings/), a series of custom themes and shortcuts for Markdown. (Hint: installing Package Control first will make this easier)

**Spell check.**
Sublime Text 2 uses [Hunspell](http://www.sublimetext.com/docs/2/spell_checking.html) for spell checking, the same library used in Word. This process was a bit more convoluted: 

1. Command-shift-p in ST2 to open Package Control
2. Choose "Add Repo". Paste in `http://www.sublimetext.com/docs/2/spell_checking.html`
3. Command-shift-p again. Choose "Install Package," then "Dictionaries"
4. Find your preferences in Preferences > Package Settings > Markdown Editing > Markdown Settings - User
5. Add the following to the settings file: 
        "spell_check": true,
        "dictionary": "Packages/Language - English/en_US.dic"
6. Restart Sublime Text. Observe red squigglies when you edit Markdown. 
7. Profit