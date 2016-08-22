---
archive: https://sites.google.com/a/yifanlu.com/downloads/TmobileFreeInternet.apk
author: yifanlu
comments: true
date: 2011-01-03 17:00:16-06:00
excerpt: Free internet with T-Mobile on iPhone
layout: project
slug: tmobileproxy
title: T-Mobile Proxy
version: 1.0.1
wordpress_id: 414
categories:
- Java
- Scripts
---

I found a trick where if the URL contains the string "tmobile", T-Mobile allows access to that site. While it's easier to make a web proxy at somewhere like  tmobile.example.com, it's slow and a pain to use. So I came up with a little script to redirect all HTTP requests on the iPhone (HTTPS experimental, does not work all the time) from example.com to example.com/?carrier=tmobile or example.com/?request=something&carrier;=tmobile. This works for 99% of all sites, and only doesn't work on HTTP sites that explicitly deny weird GET requests. The proxy server is **Tiny HTTP Proxy** written by [Suzuki Hisao](http://www.oki-osk.jp/esc/python/proxy/). The installation script is written by me. What it does is allow the proxy server to run at startup and add the proxy server to your cellular internet configurations (really hard to do if you ever tried it). **Note:** Because of how the configuration is designed, when you remove it, your internet configuration (including Wifi) will revert to what it was before installation. All internet configuration changes after that point will be lost.

To install, either download the DEB file (listed as ZIP in the downloads below), or add **/cydia/** to your Cydia repos list and install the package from Cydia.

Also make sure your APN settings are correct. I suggest installing a [T-Mobile carrier bundle](http://modmyi.com/forums/t-mobile/656651-t-mobile-usa-carrier-bundles-bluetooth-internet-mms-tethering-2g-3g-3gs-4-a.html).

I noticed that it doesn't work on the internet3.voicestream.com APN. I use wap.voicestream.com and it works there. Another thing that breaks this is any "T-Zones $5.99 hack" (which hasn't worked for a while now) is installed. The problem is that if you already have a proxy set for ip1 (EDGE/3G) interface, then this won't work. You can't modify EDGE/3G proxy information from Preferences.app, so if you manually edited your proxy information in preferences.plist, installed a package that did, or installed a mobile config that did revert the changes to use this.

If all else fails, install my configuration profile by going to [/cydia/freeinternet.mobileconfig](/cydia/freeinternet.mobileconfig) from your iPhone and install that configuration profile. It will set the APN and proxy for you.

Also, if you're linking to this, PLEASE link to this page instead of directly to the DEB, or I'll remove it. Thanks!

### Changes

* **2011-01-09**: 

