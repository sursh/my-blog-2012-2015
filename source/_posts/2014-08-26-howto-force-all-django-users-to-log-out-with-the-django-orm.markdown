---
layout: post
title: "HOWTO force all Django users to log out with the Django ORM"
date: 2014-08-26 09:42
comments: true
categories: [Django, Heroku, HOWTO, technical]
---

I recently hit a bug with [Blaggregator](http://blaggregator.us), a Django app I'm running in production on Heroku, and the fix needed each user to log out and then log back in for it to take effect. 

So how to log out all users at once? A bunch of threads around the internet suggested deleting the Session table, as that stores all session information for users (that they are logged in, shopping cart info, etc). That's easy enough with the `dbshell`, but I'm hitting a weird psql bug on Heroku that I don't have time to diagnose, and plus, working directly in the dbshell makes me feel icky. So here's how to force all Django users to log out on your Heroku-backed app using the ORM, which is one layer of abstraction farther from shooting yourself in the foot.

*Be careful* since this can have adverse effects for your customers (remove items in a shopping cart, etc). It will also confuse and annoy visitors who are currently on your site. There are more graceful ways to accomplish this, but in my case it was quick and effective. I'm not storing anything important in the Session for Blaggregator, and the site has < 1,000 users as it's constrained by the total size of the Hacker School alumni population. That's a small enough number that none of them were on the site yesterday early morning when I was deploying. So this was a quick and dirty way to deploy the fix and make sure it was applied.

To get started, fire up the Heroku Django shell: 

``` bash
$ heroku run python manage.py shell
```

When you get the Django shell prompt (`>>>`), then you can import the Session model. (If you want to go read the code, the module name tells you where to look: *django.contrib.sessions*.**models** looks in your virtualenv's lib/python2.7/site-packages/*django/contrib/sessions/* for **models**.py)

``` bash
>>> from django.contrib.sessions.models import Session
```

Now, you can iterate through all the Session objects and delete them. (caution!)

``` bash
>>> for s in Session.objects.all():
...   s.delete()
```

Or, as [Jacob Kaplan-Moss pointed out](https://twitter.com/jacobian/status/504272832231395328), just delete them straight from that queryset and win at code golf: 

``` bash
>>> Session.objects.all().delete()
```

Congradulations!  Your site is now very lonely. 

