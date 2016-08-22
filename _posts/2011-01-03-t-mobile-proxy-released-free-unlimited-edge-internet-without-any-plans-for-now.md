---
author: yifanlu
comments: true
date: 2011-01-03
layout: post
slug: t-mobile-proxy-released-free-unlimited-edge-internet-without-any-plans-for-now
title: 'Update: T-Mobile Proxy released: Free unlimited EDGE internet without any
  plans (for now)'
wordpress_id: 206
categories:
- Guides
- Releases
---

**EDIT**: This no longer works. If you have the package installed, I would recommend uninstalling it.<!-- more -->

I've talked about how it works [here](/2010/12/10/free-3gedge-internet-on-any-t-mobile-phone-without-a-data-plan/). In short, I've made a proxy server that adds the string "tmobile" to all URL requests on the iPhone because T-Mobile allows internet access to URLs with "tmobile" in it. You can download the deb and install it manually from [here](/p/tmobileproxy). You can also add the repo **/cydia/** to Cydia. This script works for any [unlocked](http://www.ultrasn0w.com/) iPhone running T-Mobile including prepaid phones. However, I'm not responsible if you abuse this and get charged. Let's start the countdown. I predict T-Mobile will have this bug fixed in a month.

Please note that it's only tested and working on one phone. So it's pretty beta-ish. If it doesn't work, please post as much info as you can in the comments, so I can fix it up.

Also, I noticed that it doesn't work on the internet3.voicestream.com APN. I use wap.voicestream.com and it works there. Another thing that breaks this is any "T-Zones $5.99 hack" (which hasn't worked for a while now) is installed. The problem is that if you already have a proxy set for ip1 (EDGE/3G) interface, then this won't work. You can't modify EDGE/3G proxy information from Preferences.app, so if you manually edited your proxy information in preferences.plist, installed a package that did, or installed a mobile config that did revert the changes to use this.

If all else fails, install my configuration profile by going to **/cydia/install.mobileconfig** from your iPhone and install that configuration profile. It will set the APN and proxy for you.
