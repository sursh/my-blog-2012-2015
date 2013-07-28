---
layout: post
title: "Tracking an encoding bug through many applications layers"
date: 2013-07-28 17:02
comments: true
categories: 
---

AKA How to decode HTML entities in your djago app

so allison said systemat debugging is important at hacker school. [link to her blog post]. better than randomly changing things. fuzzy feeling, dig into it to understand underlying concepts. benefit: learn those concetps. ANd your instincts get better for new problems.

having this bug:
link to his blog post with chars 
show screenshot—chars appear as html entities. wat?

a few layers: 

wordpress feed -> feedparser -> feedergrabber (by dpb) -> crawlposts -> db -> django view -> browser rendering html.

could be a problem at any of those layers! what if my db isn't using unicode?? etc. 

i've heard of encoding but didn't really understand it. heard of ascii and unicode and knew ascii is older and has a smaller charset, and that's it. Read joel's [extremely entertaining] blog post on encoding. 

confirmed that encoding wsan't the issue as we were'nt getting question marks, etc. also type(title) confirmed that the strings were unicode, as expected. had stuff that looked like HTML entities, which solve the problem of displaying the char '<a>' on a website instead of the browser interpreting that as a link. 

but where in that chain are they getting HTML encoded into entities? I started at crawlposts and worked my way backwards. at each level, had HTML entities. even looked at how wordpress generates title,s [link to CDATA info].

so, wordpress is handing us htmlentities. tihs is acutally a really good idea, because it stops injection of malicious code. what if david had listened to the devil on his shoulder and titled his blog post '<script>somethingbad()</script>'? 

so, wordpress is handing us htmlentities, which is actually a good idea. we need to handle them on our end. a little googling lead me to an undocumented method HTMLParser, which takes entities and puts them back into HTML.

but is that safe?? what abotu Evil David? Django is a step ahead of us. Jacob Kaplan-Moss, creater of django, spoke to us at hacker school and told us about the design decision to escape chars by default so you wouldn't forget. [power of defaults]. to play with this, changed the title in the DB to <script>alert('thotuh')</script> but was foiled. Displayed as literals. you can see this in the [page source]. note: confused for a bit because the Inspect Element in chrome renders those escaped chars. so i went down a wrong path trying to figure out why HTML wasn't rendering tags nested inside <a> elements. It does. Thanks, Obama.

So, long story short, I'm going to decode the HTML entities with HTMLParser and pass them into Django whose templating language will escape them properly. 

[screenshot of successful result]