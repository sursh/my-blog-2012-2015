---
layout: post
title: "Octopress gotchas"
date: 2013-03-11 19:05
comments: true
categories: [octopress]
---

This blog is powered by Octopress, which is basically a set of `rake` tasks to generate a blog from your posts in Markdown form. It's in turn powered by Jekyll. It

## WTF is going on with the branches?

The documentation is generally pretty good, but this threw me for a loop and isn't clearly explained anywhere. Pretty simple once you dig into the `Rakefile`. Or, you know, one more sentence of documentation. 

``` 
$ git br
* source
```

Huh. Interesting. Locally, you *only* have one branch: `source`. But on github, you have both `master` and `source`. Wat?

Basically, `source` holds your posts in Markdown and other files *before* they are parsed into HTML. Once you run `rake generate`, Octopress will generate all the HTML and CSS, and stick the whole site in `/public`. 

And when you run `rake deploy`, Octopress pushes the contents of `/public` (on *local* branch `source`) to the home directory (on *remote* branch `master`).

So: each time you finish a post, run BOTH `rake deploy` to deploy and `git push origin source` to back up your files. 

## Sublime Text <3 Markdown

You can write your blog posts in Markdown, and if you're using Sublime Text 2 there are a few things you can do to make your life more fun: 

**Custom themes**. For example, [MarkdownEditing](http://brettterpstra.com/2012/05/17/markdown-editing-for-sublime-text-2-humble-beginnings/), a series of custom themes and shortcuts for Markdown. (Hint: Package Control will make this easier)

**Install spellchecking.**
ST2 uses [Hunspell](http://www.sublimetext.com/docs/2/spell_checking.html) for spell checking, the same library used in Word. This process was a bit more convoluted: 

1. Command-shift-p in ST2 to open Package Control
2. Choose "Add Repo". Paste in `http://www.sublimetext.com/docs/2/spell_checking.html`
3. Command-shift-p again. Choose "Install Package," then "Dictionaries"
4. Find your preferences in Preferences > Package Settings > Markdown Editing > Markdown Settings - User
5. Add the following to the settings file: 
        "spell_check": true,
        "dictionary": "Packages/Language - English/en_US.dic"
6. Restart Sublime Text. Observe red squigglies when you edit Markdown. 
7. Profit