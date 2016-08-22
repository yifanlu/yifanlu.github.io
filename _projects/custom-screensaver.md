---
archive: https://sites.google.com/a/yifanlu.com/downloads/simple_screensaver_1.0.zip
author: yifanlu
comments: true
date: 2012-01-15 23:53:15-06:00
excerpt: Use any image as a screensaver on the Kindle Touch
layout: project
slug: custom-screensaver
source: https://github.com/yifanlu/OpenBlanket
title: Custom Screensaver for Kindle Touch
version: '2.0'
wordpress_id: 527
categories:
- C
- Kindle
---

This Kindle Touch modification will allow you to use custom images as screensavers. This is not for Special Offer Kindles, nor does it remove the advertisements. By design, this mod will only work with non-ads Kindle Touches. An improvement of this vs the screensaver hacks of previous devices and the simple screensaver hack I released a while ago includes:



	
  * No filename constraint (name the images anything you want)

	
  * Auto update when new files are found

	
  * Scrolls through in alphabetical order (not random order, so you can manage the order of the images)

	
  * If no custom images are found, it will default back to Kindle's original screensavers

	
  * If an invalid file is found, it will be skipped, after 5 invalid files, it will default back to original screensaver (instead of just crashing and requiring a restart)


In addition, images are auto-scaled and auto-converted, which means you are no longer limited to 600x800 grayscale images. Of course, those are the preferred format, but you can now use any PNG file and it will be sized to fit the screen. The only limitation is the image format must be PNG.

Finally, as an added bonus, I've included a custom screensaver, "lockscreen.png". If you place that image in your screensaver folder (and nothing else), instead of a picture screensaver, the Kindle will keep whatever you are reading as the screensaver with a neat lock icon on the top of the screen.

As always, installation and usage directions are found in the readme after download.

All this is made possible by OpenBlanket, my attempt at reversing the APIs for LibBlanket, Amazon's library for drawing full screen content on the Kindle Touch. You can check out the source for an almost identical re-implementation of the device's original screensaver module (decompiled by hand and compiles to almost the exact same machine code). The source for the custom screensaver module is a modification of that source (and is included in the download).

**Please note that you must [jailbreak](/p/kindle-touch-jailbreak/) your Kindle before installing this package.**

**My stance on removing ads: **First of all, I will never stop anyone from removing ads because I believe that a user should do anything they want on a device they own. It's very easy to do, and there are many ways of doing it. However, I will never aid anyone in doing so, so please don't ask me for help. The reason is that our main defense for jailbreaking the device being legal and ethical is that "we aren't doing anything to hurt amazon." However, when you remove the ads without paying Amazon $40, they lose money (even if Amazon is a big, evil corporation taking money from anyone is, in my opinion, bad), which means that if enough people start removing ads to make an impact they could take action against people like me who bring you the ability to unlock your devices. Since technically, the TOS for the Kindle states that you cannot modify or reverse engineer the device. No, Amazon is not enforcing it. No, I do not agree with it. But Amazon has always been passive in stopping jailbreaks (removing the exploit but not touching the hacks; not worrying about people who jailbreak; allowing people with jailbroken devices to have support even though you technically broke the TOS and can be denied support), but if they start losing money they could start actively blocking jailbreaks.

### Changes

* **2011-01-12**: "Simple screensaver" for Kindle Touch. Limited to 99 images of specifically named files.
* **2012-01-14**: More advanced. Much more features.

### Screenshots

![Screen 0](/images/2012/01/screenshot_2012-01-15T17_46_20-0600.gif)

