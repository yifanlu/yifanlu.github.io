---
author: yifanlu
comments: true
date: 2011-09-28
layout: post
slug: analyzing-kindle-4-0
title: Analyzing Kindle 4.0
wordpress_id: 320
categories:
- Kindle
- Technical
---

Well, Amazon might as well have stolen my wallet, because I am going to lose a couple hundreds of dollars. However, what fun is a Kindle if we can't run our own code? (Answer: still pretty fun, but that's besides the point.) Anyways, I haven't gotten my hands on the new Kindles yet, but I got the next best thing: a software update from Amazon (http://www.amazon.com/gp/help/customer/display.html/?nodeId=200774090)

If you want to follow me and others try to crack this thing, visit this thread on [MobileRead](http://www.mobileread.com/forums/showthread.php?p=1762199).

I'll post some of the more important stuff we find on this post, so check back regularly.<!-- more -->



	
  * The update format has changed! No more signatures for each file in the update, the update itself is signed and will refuse to extract unless the signature check passes. That means no more easy way out. To get "kindle_update_tool.py" to recognize and extract the new update, remove the signature (first 0x140 bytes) and change "FC04" to "FC02" (Bytes 0x0 to 0x4 after trimming the signature header). Now delete 4 bytes starting from 0x8 and 6 bytes starting from 0x10. (Offsets depend on the SP01 part removed). Now "kindle_update_tool.py" will recognize it.

	
  * Kindle 4.0 is codenamed "Yoshi" following "Luigi" (3.0) and "Mario" (2.0) (I can't remember 1.0). It is built for the **[iMX50](http://www.freescale.com/webapp/sps/site/taxonomy.jsp?code=IMX50_FAMILY)** (800MHz ARM Cortex A8) platform. The Kindle 3 is iMX35 (532MHz ARM) and the Kindle 2/DX is iMX3 (400MHz ARM).


