---
author: yifanlu
comments: true
date: 2010-12-10
layout: post
slug: free-3gedge-internet-on-any-t-mobile-phone-without-a-data-plan
title: Free 3G/EDGE internet on any T-Mobile phone without a data plan
wordpress_id: 196
categories:
- Guides
- Information
- Technical
tags:
- free
- internet
- proxy
- tmobile
---

Well, the secret is out. I refreshed my iPhone's Cydia to find somebody selling "free T-Mobile Internet access". I knew immediately what the method was, as I've been using it for almost a year now. Since it's now public, and T-Mobile will close the hole anyways, I might as well help you save your money from these crappy "services".

So what's the "bug" that allows free internet? It seems like the stupidest thing in the world, and I'm almost certain that some technician left it in on purpose. Basically, any URL with the word "tmobile" is accessible without a data plan. (as long as your APN is set to epc.tmobile.net) So all you have to do is make a proxy site (aka PHProxy) with "tmobile" somewhere in the URL (tmobile.yoursite.com or freehost.com/tmobileproxy) and it would be accessible via your phone.

It gets better. As far as I know, the above is the only thing that's "leaked". Here's some new information: the method above only allows web site browsing, there is a way to 1) not use a slow and unreliable proxy, and 2) work with all HTTP apps on the phone other then web browsers. If you append the string "?tmobile" at the end of the URL, it loads without fail. So just install a local proxy (like Privoxy, or a custom one) on your iPhone (or whatever smartphone) which adds "?tmobile" to the end of the URL (or "&tmobile;" for pages with GET requests) and it will work.

If you don't get a word I said, don't worry. When I have time, I'll post my custom proxy written in Python, or even post an iPhone Cydia package.

P.S: This method only works with HTTP requests (not HTTPS, or any other protocol). I have another, slower method of getting access to everything, but I'm not ready to reveal it yet.
