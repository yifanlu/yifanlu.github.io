---
archive: https://sites.google.com/a/yifanlu.com/downloads/igradebook.zip
author: yifanlu
comments: true
date: 2009-04-21
excerpt: Based on KGC, allows KISD students to browse their grades online.
layout: project
published: false
slug: igradebook
source: https://sites.google.com/a/yifanlu.com/downloads/igradebook_source.zip
title: iGradebook
version: '1.1'
wordpress_id: 423
categories:
- PHP
---

What is this site? Well, if you're a Katy ISD student (if not, go away, this isn't for you), you probably know that the online gradebook is not accessible from your iPhone or iPod touch. Most kids would just go two feet to their closest computer and forget about it, but not me. I don't have a life therefore, it was only natural that I spent the next few hours writing a disassembler core for the whole gradebook system. So what does it do? It first downloads the login page and finds the unique id that you must have to log on, then it "feeds" the logon information to the system and tricks the site into thinking an actual human is accessing the site though some complex computer magic. It then lets the site decrypt the SSL page where grades are stored (since it thinks the system is a human), and shows the grade. The program then takes that web page, and recodes it to an array of infomation. The array is stored in a session, and it shows up on your iPhone with a cool UI.

This project is retired as the school blocked it, plus the gradebook site now works on the iPhone so it is no longer needed.

### Changes

* **2009-04-20**: 

### Screenshots

![Screen 0](/images/2012/01/igradebook_screen.png)

