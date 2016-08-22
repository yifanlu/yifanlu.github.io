---
author: yifanlu
comments: true
date: 2016-08-21
layout: post
slug: welcome-jekyll
title: Welcome Jekyll!
categories:
- Announcements
tags:
- yifanlu
- jekyll
- site
- web
---

Welcome to the new yifan.lu! I just completed the biggest upgrade of this blog since its inception. What? You don't notice any changes? That's good. Although the changes are drastic (moving from WordPress in dynamic PHP to Jekyll in static HTML), the goal was to make the changes as transparent as possible. Please let me know if you notice anything broken so I can fix it.

The move wasn't easy. I had eight years of posts (although, to be fair, the majority of it is worthless) and lots of customizations to WordPress that I had to migrate. I started with [exitwp](https://github.com/thomasf/exitwp), which converts WordPress exports to Jekyll in the Markdown format. I had to modify it in order to migrate my custom post type ([projects](/projects-archive/)) to a Jekyll collection.

Next, I took [minimal-mistakes](https://github.com/mmistakes/minimal-mistakes) as a template and converted my WordPress theme to Jekyll. Finally, I set up [Staticman](https://staticman.net) to support commenting without requiring a third party service and modified exitwp once more to convert the comments.

In the end, I wanted to switch to Jekyll for three main reasons

  1. **Markdown is sexy.** Plus, built in syntax highlighting and the ability to track the history with git. Free hosting with Github Pages means I no longer have to deal with my free hosting provider which required ads on pages (no more ads now!) and always had random downtime. I tried paid hosting, but Wordpress's RPC seemed to eat up my resources and I didn't have the time to figure out how to stop it.
  2. **No PHP!** No downtime from "hackers." No need to update WordPress all the time. No need to have to set up a complex system of caches just to have reasonable speed.
  3. **HTTPS support.** The blog always supported optional SSL from CloudFlare, but lots of links had HTTP hard coded in. There was no easy way to go in and change all the URLs all over the place. Now, if you access yifan.lu from HTTPS (which I recommend), you will not be prompted about insecure resources or stumble a link that downgrades your connection. Because I believe in choice, HTTPS is not enforced--you still have the option to go visit this blog through HTTP.

For archival purposes, here is my modded exitwp.py with some pretty bad hacks to convert my custom post type to a collection and comments to Staticman:
{% gist e7da11d0f3d2910ce0d24fc1a5ffe6b8 %}
