---
layout: post
title: "Crashing Heroku (or When Memory Management in Python Really Matters)"
date: 2013-03-07 18:24
comments: true
categories: [hackerschool, python]
---

[David](https://github.com/maxlikely) and I paired this week to build a [Markov chain generator](http://www.codinghorror.com/blog/2008/06/markov-and-you.html). It analyzes 15MB of old Hacker News headlines and generates completely new headlines that really sound like they came from HN. You can [follow the Twitter bot](https://twitter.com/HackerNewsOrNot) or [check out the code](https://github.com/sursh/markov-hacker-news). 

[Last time](http://blog.sashalaundy.com/blog/2013/03/05/help-us-plug-our-memory-leaks/), we asked your advice on how to handle all possible trigrams in 15 MB of seed text - a task that took up about 1 GB of memory in our first implementation, rapidly blasting through Heroku's limit of 512 MB. The culprit? **Dictionaries.**

Our initial data structure was a dict of dicts. The key was a _bigram_, or word pair, and the value was a dict of all of the words that followed that bigram in the training text and the number of times they occured. Wat? Here's an example:

``` python Our initial data structure

{ 
  ('how', 'to'): { 'do': 529, 
                   'grow': 202,
                   'know': 134,
                   .....
                 },
  ('to', 'do'): { 'the': 1245,
                  'everything': 149,
                  .....
                },
  ...
}

```

With 15 MB of training text and so much redundancy, this can get out of hand quickly! Every word appears _at least_ 3 times in the matrix, and some appear thousands of times. 

### Possible solutions:

 * **Generators.** for iterating over the headlines. This helped a lot - got us from 1.3 G down to about 850 MB.
 * **Compression.** Tossing out common values, like `1` for the count of the long tail of words. 
 * **Creative use of data types.** E.G. storing words as `int`s instead of `string`s. 
 * **Lists.** They take up less memory than dicts. One variation that we tried (and is the most recent version of [our code](https://github.com/sursh/markov-hacker-news/blob/e584ef95ade08f03a482debfc0c0f47adde67af3/trigrams.py)) is creating two dicts of lists rather than one dict of dicts. 
 * **Trade memory for speed.** Darius, a Hacker School alum, pointed us to [his Stack Overflow answer](http://stackoverflow.com/questions/327223/memory-efficient-alternatives-to-python-dictionaries/327295#327295), where he laid out a few other approaches that mainly trade memory usage for speed. Also, David dug up some recent papers that focus entirely on this problem. Researchers are currently doing some very clever algorithms to get the most out of their data structures.

### So where did we leave it?
Darius' suggestions and the approaches in the literature will certainly help reduce memory usage. We wanted to maximize learning per unit of time at Hacker School and didn't think implementing these approaches teach us enough to be worth it. We just shipped it in its current state by deploying on a machine with more memory. Follow the Twitter bot [here](https://twitter.com/HackerNewsOrNot)!

#### Stuff I learned along the way:

 Generators, classes and methods in python, pickle, regex, defaultdict, Tweepy, Buffer's API, how bad the built in titlecase methods are in Python, some clever way to handle default arguments, some fundamental principles of NLP. 
