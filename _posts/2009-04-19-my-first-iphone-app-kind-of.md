---
author: yifanlu
comments: true
date: 2009-04-19 15:05:14-06:00
layout: post
slug: my-first-iphone-app-kind-of
title: My first iPhone app (kind-of)
wordpress_id: 77
categories:
- PHP
- Releases
tags:
- igradebook
- iphone
- php
---

Well, it took two days to coding, but I finally did it. I made an web-app for the iPhone. The reason I decided to do this was to familiarize myself with advanced aspects of PHP, such as classes, curl, and different sorts of weird functions. I created the KGD (KISD Gradebook Disassembler) from scratch in PHP. What it does is logs in to http://pic.katyisd.org/ (where you check grades), and gets to the gradebook page. It parshes the HTML and generates an array of infomation. This array can be sent to other pages for easy creation of stuff based on it. Some other features Idid was cacheing, so it would download the HTML every time, for maximum speed. It also requires NO connection to any database, so it's very portable and expendable. It may also be one of the _cleanest_ code I've ever written and the first one at is object-oriented (to train myself before I get to iPhone SDK programming).  The first thing I did is iGradebook. It is a cool app with an slick UI that allows you to check your grades on the iPhone. Some other things I'm looking into with the KGD core is maybe writing a desktop alert app or an Facebook plugin. If you would like to try it, check out the projects page. Look forward to the KGD core being released under GNU when I'm done. (Oh yea, this only works for KISD students, as that's the only place I have access to grades)
