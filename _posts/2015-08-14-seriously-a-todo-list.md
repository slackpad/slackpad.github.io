---
layout: post
title:  "Seriously, a todo list?"
categories: startups hashtagtodo programming
---

Origin Story
------------

A while ago a friend set me up for lunch with a person he had met at a mixer. This guy was looking for a developer to partner with to help him build out a small web app. I was a little skeptical because I didn't know him, and I was even more skeptical as he started pitching me on a todo list app. Surely the world didn't need another one of these! He was super sincere and passionate, though, so I decided to hear him out.

He had put a lot of thought into why there are so many todo list apps and came to the conclusion that people start excited about them but since it's another place you need to look every day they kind of fall out of mind and stop using them. He and his wife had been hacking their Google Calendar as a todo list, adding simple `[ ]` check boxes to regular calendar events, and manually dragging them forward if they didn't get done. My wife does this too, minus the check boxes, so it seemed familiar. I pointed out that Google Calendar already has a built in todo list, but we spent a few minutes playing with it and realized how bad it was (and it's another place you need to look). His idea for the app was basically something that would move todo events forward if you didn't mark them off.

I was interested enough to shake hands and partner up; it seemed like a fun side project.

_tl;dr: if you are curious and want to see the final product, check out [HashtagTodo](https://hashtagtodo.appspot.com/)_.

Building the App
----------------

The basic strategy was to have users authenticate with their Google Account and grant our app offline access to their calendar via OAuth. Whenever they create new calendar events, we'd look for `#todo` in the event title, and if we see that, add a `[ ]` check box. At midnight for whatever time zone the calendar is in, we'd push any overdue events forward. Users literally type an `[ x ]` to check off an event, which is a little cheesy, but if they already live inside their Google Calendar and have it on all their devices, then it's pretty simple and works everywhere. No app to install and no extra place to check!

I chose Google App Engine and Python/Flask since I had experience with those and they seemed like a good fit for something integrating with Google Calendar. Surprisingly, I found Google's native support for OAuth poorly documented and a little tricky to integrate into my Flask app, so I opted for [Authomatic](https://github.com/peterhudec/authomatic) which was super easy to use. There's a magical `WerkzeugAdapter` that you can pass your request and response objects into and it will do the OAuth dance for you. If we ever decide to support other calendar providers this will be a good layer to have. Other notable Python libraries are [pytz](http://pytz.sourceforge.net/) for dealing with time zones and [Arrow](http://crsmithdev.com/arrow/) for humanizing times for our summary lists (eg. "due in 2 days").

_Update 2015-09-09 - it looks like Google has released a [Flask-compatible OAuth client library](http://oauth2client.readthedocs.org/en/latest/source/oauth2client.flask_util.html). I'm going to give that a shot._

We talk to the Google Calendar API using Google's [api-client-library](https://developers.google.com/api-client-library/python/) which works super well, but is also super generic. It parses a definition file that's fetched from Google to build all the request and response objects. This is slick in that a single dependency can make a Python client for any Google API, but the code itself if pretty obtuse. At times I was left wishing there was code specific to the Google Calendar API that I could reference to learn a little more about different request options, such as what format they expect, since Google's documentation is a little light in some places. It wasn't a huge problem, but we did have to resort to some experimentation.

When a user signs up, we scan their list of calendars and then register for push notifications. The push notifications basically say "something on this calendar changed" so we need to scan over their events looking for todos to process. It's a little unfortunate that we can't get push notifications for individual events that we care about, so we spend most of our time scanning over calendars for changes that aren't relevant to our app. This is especially true for users with a lot of business churn on their work calendars.

Rolling events forward is done using the Google App Engine cron facility, where we set up jobs that run a few minutes after each hour and look for calendars that have just crossed midnight. We then look for uncompleted todo events and push them forward to the next day. You can make a todo at a particular time during the day so you get a reminder, such as putting some things on your lunch break. When we roll them forward we will make them all-day events on the next day, so they show up at the top of the calendar. Another cool thing is you can make recurring todos and they will get check boxes added as they come into the week ahead, and then will start moving forward if they become overdue. A lot of users use these for monthly things they need to do, like remember to pay someone.

HashtagTodo
-----------

We've got a small but growing set of users, and some people are totally managing their life with this simple app. My wife and I often drop a `#todo` on each other's calendars, and I use it for all things that I absolutely must not lose track of.

We hope this app is useful to people and that it continues to be discovered by folks who live by their Google Calendars. Check it out - [HashtagTodo](https://hashtagtodo.appspot.com/).
