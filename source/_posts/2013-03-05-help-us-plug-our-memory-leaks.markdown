---
layout: post
title: "Help us plug our memory leaks"
date: 2013-03-05 00:08
comments: true
categories: [python, hackerschool]
---

David and I are running our app on Heroku, but it uses too much memory to run! **Got any clever optimization suggestions?**

We're building a Markov chain generator that is trained on a corpus of 15MB of old Hacker News headlines. The app currently indexes the training corpus and constructs a matrix of all the possibilities, then queries that matrix to generate a hopefully entertaining new headline. Preview it [here](http://twitter.com/HackerNewsOrNot).

**However, as we first wrote it, it used 1+ GB of memory when we deployed to Heroku, where the limit is 512 MB!** We wrangled it down to 570 MB today and will tackle it again tomorrow morning, but need more ideas.

Some things we've tried: 

 * Pickling the matrix. Not pickling the matrix. (didn't help)
 * Storing the words as `int`s in our matrix rather than `string`s (helped)
 * Tweaked the data structure (helped, could probably do more with this)
 * We ran out of time, but want to try some sort of data store. Redis? 

**What should we try next?** Our code after the jump: 

<!-- more -->

Very unfinished code: 

``` python The code + a few half-finished experiments 
import sys
import time
import re
import collections
import numpy
import string
import cPickle as pickle
import twitterclient

class Markov(object):

  def read(self, filename):
    with open(filename) as f:
      for line in f:
        headline = map(intern, re.split('\s+', line.lower().strip()))
    if headline[-1] == '': del(headline[-1]) # clear lingering null entries
    if headline[0] == '': del(headline[0])
        yield ['^'] + headline + ['$']

  def generateTrigrams(self, tokens):
    ''' Create a list of tuples, where each tuple is a trigram '''
    for idx, item in enumerate(tokens[:-2]):
      yield (item, tokens[idx+1], tokens[idx+2])

  def generateMatrix(self, filename):
    ''' Run through the list of trigrams and add them to the occurence matrix '''

    self.matrix = {}
    self.dictionary = {}

    try: # load as pickle

      with open('matrix.p', 'rb') as f:
        self.matrix = pickle.load(f)

    except: # interpret file as newline separated headlines

      print 'reading headlines'
      headlines = self.read(filename)

      for headline in headlines:

        trigrams = self.generateTrigrams(headline)
        
        for first_word, second_word, third_word in trigrams:
          self.dictionary.setdefault(first_word, len(self.dictionary))
          self.dictionary.setdefault(second_word, len(self.dictionary))
          self.dictionary.setdefault(third_word, len(self.dictionary))

        trigrams = self.generateTrigrams(headline)
        
        for first_word, second_word, third_word in trigrams:
          first_word  = self.dictionary[first_word]
          second_word = self.dictionary[second_word]
          third_word  = self.dictionary[third_word]

          self.matrix.setdefault( (first_word, second_word), collections.defaultdict(int)) 
          self.matrix[(first_word, second_word)][third_word] += 1

      print sum(len(d) for d in self.matrix.itervalues())
      #print("DUMPING %d" % len(self.matrix))
      #pickle.dump(self.matrix, open("matrix.p", "wb"))


  def generateNextWord(self, prev_word, current_word):

    conditional_words = self.matrix[ (prev_word, current_word) ]
    words, counts = zip(*conditional_words.items())

    # pick one of the possibilities, with probability weighted by frequency in training corpus
    cumcounts = numpy.cumsum(counts)
    coin = numpy.random.randint(cumcounts[-1])
    for index, item in enumerate(cumcounts):
      if item > coin:
        return words[index]


  def generateParagraph(self, seed2 = 'i', seed1 = '^',):
    
    seed1 = self.dictionary[seed1]
    seed2 = self.dictionary[seed2]

    prev_word = seed1
    current_word = seed2
    paragraph = [ seed1 ]

    while (self.dictionary[current_word] != '$' and len(paragraph) < 20): 
      paragraph.append(current_word)
      prev_word, current_word = current_word, self.generateNextWord( prev_word, current_word )

    #paragraph = []
    paragraph = ' '.join(paragraph[1:])  # strip off leading caret
    return string.capwords(paragraph)


  def __str__(self):
    s = ''
    for word, cfd in self.matrix.iteritems():
      for word2, count in cfd.iteritems():
        s += '%s %s: %d\n' % (word, word2, count)
    return s


def main():

  if len(sys.argv) != 2:
    print 'usage: ./markov.py inputFile'
    sys.exit(1)

  filename = sys.argv[1]
  
  m = Markov()
  m.generateMatrix(filename)

  while True:
    tweet = m.generateParagraph('how') # todo: new seed
    break
    if len(tweet) < 120:
      break

  print("Tweeting %s" % tweet)
  twitterclient.postTweet(tweet)

if __name__ == '__main__':
  main()

```
