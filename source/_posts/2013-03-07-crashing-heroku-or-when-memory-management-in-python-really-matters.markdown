---
layout: post
title: "Crashing Heroku (or When Memory Management in Python Really Matters"
date: 2013-03-07 18:24
comments: true
categories: [hackerschool, python]
---

[ ] fix links
[ ] fix the 'where did we leave it' paragraph
[ ] run it by david

[David] and I paired this week to build a [Markov chain generator](). It analyzes 15MB of old Hacker News and generates completely new headlines that really sound like they came from HN. You can [follow the Twitter bot]() or [check out the source code](). 

[Last time](), we [asked your advice] on how to handle all possible trigrams in 15 MB of seed text - a task that took up about 1 GB of memory in our first implementation, rapidly blasting through Heroku's limit of 512 MB. The culprit? **Dictionaries.**

Our initial data structure was a dict of dicts. The key was a [bigram], and the value was a dict of all of the words that followed that bigram in the training text and the number of times they occured. Wat? Here's an example (entire code [here])

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

 * **Generators** for iterating over the headlines. This helped a lot - got us from 1.3 G down to about 850 MB.
 * **Compression.** Tossing out common values, like `1` for the count of the long tail. Storing words as `int`s instead of `string`s. 
 * **Lists.** They take up less memory than dicts. One variation that we tried (and is the most recent version of [our code]) is creating two dicts of lists rather than one dict of dicts. Described [here].
 * **Trade memory for speed.** A Hacker School alum pointed us to his Stack Overflow answer, where he laid out a few other approaches. They simplify and slim down the memory needs, but move that complexity over to the queries and will therefore take much longer to run. David did some research and this seems to be the current state of the art for researchers. I wonder if Google has a better way?

### So where did we leave it?
[THE DUDE'S] suggestions will certainly help, but we wanted to maximize learning per unit of time at Hacker School and ended up shipping it - just deploying on a machine with more memory. Follow the Twitter bot [here].

### Stuff I learned along the way:

 Generators, classes and methods in python, pickle, regex, defaultdict, Tweepy, Buffer's API, how bad the built in titlecase methods are in Python, some clever way to handle default arguments, some fundamental principles of NLP. 
