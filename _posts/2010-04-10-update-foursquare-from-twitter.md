---
author: yifanlu
comments: true
date: 2010-04-10
layout: post
slug: update-foursquare-from-twitter
title: Update Foursquare from Twitter
wordpress_id: 153
categories:
- Java
- Releases
tags:
- foursquare
- http
- java
- spaghetti code
- tweetbot
- twitter
---

Ok, so I THOUGHT this was going to be a quick one hour project. I want to update foursquare from Twitter because my cell phone plan ONLY allows access to Twitter and MS Exchange (why, I don't know). The goal was to write a application that sits in the background and waits for "command" tweets. My original plan was to do it in C++, however, networking & sockets in C++ is too complicated for such a small project, plus no good libraries are available for Twitter in C++. Ok, so I moved to Python. It has great networking tools right? Plus a wonderful Twitter API library. I was halfway through when I found that Foursquare support was crappy. Finally, I went to the language I hate the most. Java. Also, note that I've been messing around for hours now. Fuck. So I quickly wrote this in Java, tired and angry. The result is the worst code I ever written. I am the only person who would ever make use of this, so I didn't care. I'm only releasing it for archival purposes, for some laughs to random strangers, and as an example of what you should NOT do. It's a great example of "it compiles, ship it".

So, the only features are: search foursquare, checkin to foursquare, and greet the user. To use it, you need to set up two twitter accounts, one for the client (you will tweet commands from here) and one for the server (the server will use this account to tweet). Make the client follow the server and the server follow the client. Make sure your twitter client supports geotags. If your tweet doesn't have a geotag, the listener will crash. (I know, stupid)

Again, you **MUST **have a geotag with every tweet.

[/p/tweetbot](/p/tweetbot)

To search, tweet:


> Search for "restaurant"


**Note**: You MUST put the quotes for the search to work. For all commands, the server only reads "key" words. That's the first word and any word in quotes. You can type in


> Search for some shitty "restaurant" up in this bitch


and it'll work just fine. For search, if you don't have any words in quotes, then you'll just get a list of 10 closest venues.

Now, the server will return top ten results near you, in two tweets (a random character appearing at the end of server tweets is not a bug, I did that because Twitter rejects any two tweets that are identical).


> 0: Random Chinese Restaurant
> 
> 1: Some Mexican Restaurant
> 
> 2: Another Crappy Restaurant


and so on, to check in to "Another Crappy Restaurant", tweet


> Checkin 2


If you know the name of the venue, you don't have to search, you can just tweet.


> Checkin to "another crappy restaurant"


That's basically it.

Now, about the spaghetti code, here's some of the things I used: depreciated methods, tons of try-catch just to bypass errors (no actual error handling), bad variable name and no documentation, bad code flow, usage of Runtime.getRuntime().exec() to call cURL because I was too lazy to write a proper HTTP controller which is a security and stability issue, use of reflections with data across the internet, and so much more. My hope is that someone will take it and fix it up or something, because I'm too tired to do anything about it.
