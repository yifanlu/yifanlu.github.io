---
author: yifanlu
comments: true
date: 2010-02-20 04:25:14-06:00
layout: post
slug: google-apps-user-registration-script
title: Google Apps User Registration Script
wordpress_id: 143
categories:
- PHP
- Releases
tags:
- apps
- gaurs
- gmail
- google
- php
- projects
- script
---

Here's another one of my famous 3-hour-projects. I finally decided to cleanup my email. It's too hard to "clean", so I decided to start from scratch by making a new email account. So, I made a Google Apps account. Google Apps is a great product, but one thing missing is registration for users. (You must make an account manually for your user) So, I decided to make one myself. This PHP script acts as a proxy between you and Google Apps. It allows your users to create their own account with you and your Google Apps. It is composed of a backend and a frontend. The backend does the work of taking your admin credentials and form data from a user and creating an account for the user. The frontend hosts the GUI. I made sure to well-comment the code, so it should be easy to create your own frontend to match the style and code of your site. I'm probity won't work on this project again, but because it's released under GNU v3 (as all my projects), you can take it and add on to it.

[Google Apps User Registration Script](/p/googleappuserregistration)
