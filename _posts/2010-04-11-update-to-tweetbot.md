---
author: yifanlu
comments: true
date: 2010-04-11 17:19:49-06:00
layout: post
slug: update-to-tweetbot
title: Update to TweetBot
wordpress_id: 155
categories:
- Java
- Releases
---

After a good night's sleep, I "fixed up" TweetBot. No, the code is still junk, but I made it a bit better. New features:

-Doesn't crash if no geolocation is found

-Multiple commands separated by period.

-Update shout

-Update Twitter/Facebook

-Removed debug logging (it showed things down a bit)

-Commands that begin with "set" ignores the word "set", this way your sentences can flow better.

Some new commands:

Set a shout


> Status is "I'm shopping"


Update Twitter, facebook, or both


> Show twitter
> 
> Show facebook
> 
> Show twitter and facebook


Multiple commands


> Search around me for "best buy". Set status to "buying a new computer". Show twitter & facebook. Checkin to result 0.


Remember, you can always throw in extra words outside of quotes. Only the first word, quotes, and (if no quotes), the last word are read.

[/p/tweetbot](/p/tweetbot)
