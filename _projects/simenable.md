---
archive: https://sites.google.com/a/yifanlu.com/downloads/com.yifanlu.simenable_1.0.2_iphoneos-arm.deb
author: yifanlu
comments: true
date: 2012-04-07
excerpt: Toggle between the GSM and CDMA on an SIM interposer unlocked iPhone.
layout: project
slug: simenable
source: https://sites.google.com/a/yifanlu.com/downloads/simenable_source.zip
title: SIM Enable
version: 1.0.2
wordpress_id: 557
categories:
- iPhone
- Objective-C
tags:
- cdma
- cydia
- gsm
- iphone
- objective-c
- preferenceloader
- r-sim
- unlock
- verizon
---

This is one of those tools that I wrote mainly for my own use, but I'm releasing just for the one guy with the same problem and arriving here via Google (hi). Basically, this creates a new menu item in your iPhone's Settings app that allows you to switch between the CDMA and GSM networks on an iPhone provided it is activated on a CDMA network (Verizon/Sprint) AND you have a SIM interposer (Gevey/R-SIM) that allows you to use a GSM network (AT&T/T-Mobile).

First, a little backstory. I have a R-SIM v3 for my iPhone 4S Verizon and I also have a month of free 3G internet from AT&T due to a Vita promotion. I do not like browsing the internet on the Vita because the web browser sucks, so I decided to put the SIM in my iPhone. Now, the problem is that the data SIM card does not allow calling, so I would have to remove the SIM card every time I need to make or receive a phone call. I discovered by looking at the package I had to install for R-SIM that all it did was make the Verzion and Sprint carrier bundles on the iPhone unusable (by replacing their carrier data with the one from the Unknown bundle), so the iPhone falls back to the SIM card. So I decided to make a toggle in settings that will do this and restart CommCenter, and have the option to revert the bundle back to their original values (therefore restoring CDMA).

So, because I wrote this for a very specific audience (me), I have only tested it with an iPhone 4S Verizon, activated, running 5.0.1 and a R-SIM v3 with an AT&T SIM card. My CommCenter was also patched using the one provided here: http://v.backspace.jp/repo. All other configuration are untested and can be used at your own risk. However, I KNOW this will not work for iPhones that do not have two modems, which, at the time of writing, is only the iPhone 4S. You must also have your root password for the phone be 'alpine', if you don't know what that means, you're fine as that's the default password. The reason for this is because I hard coded the root password into the app because I was lazy.

To install, add my repo to cydia: **http://cydia.yifan.lu/ **and install the package "SIM Enable". You must have a patched CommCenter (the one I used it provided above) and the root password must be 'alpine' (info above). To use, go to the Settings app and find the option "SIM Enable". After changing the toggle, you might have to wait up to five minutes to see the changes show up. If you get "No Carrier", turn airplane mode on and off and you should get a carrier.

Here's a video of it in action. I left in four minutes of the iPhone searching for a network so you have an idea of how long it takes.



### Changes

* **2012-04-05**: Initial release.
* **2012-04-06**: Fixed bug where "load" commcenter runs before "unload" completes.
* **2012-04-06**: Added a message box to prevent user from exiting screen before the process completes.

### Screenshots

![Screen 0](/images/2012/04/photo-1.png)
![Screen 1](/images/2012/04/photo-2.png)
![Screen 2](/images/2012/04/photo-3.png)

