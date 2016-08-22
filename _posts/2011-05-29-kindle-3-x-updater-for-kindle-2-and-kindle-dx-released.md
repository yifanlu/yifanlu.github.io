---
author: yifanlu
comments: true
date: 2011-05-29
layout: post
slug: kindle-3-x-updater-for-kindle-2-and-kindle-dx-released
title: Kindle 3.X updater for Kindle 2 and Kindle DX released
wordpress_id: 276
categories:
- Kindle
- Releases
tags:
- '3.0'
- dx
- jailbreak
- k2
- kernel
- kindle
- package
- script
- update
- updater
---

After a month and a half of testing thanks to the community of [MobileRead](http://www.mobileread.com/forums/showthread.php?p=1473632), I can finally release the first stable version of the Kindle 3.X software updater (help me come up with a better name, please). If you haven't read my [last few Kindle-related posts](/tag/kindle/) (read them if you want more technical details of this script), you should know that this allows you to use all the cool new features of the Kindle 3 on a K2 or DX device. Installation is easy and is only three steps: 1) Use "prepare-kindle" script on old Kindle to back up and flash recovery kernel, 2) Copy generated files to Kindle 3 along with "create-updater" script and run it, 3) Copy generated update package back to old Kindle and restart. If that sounds confusing, don't worry, the readme contains very detailed directions and even how to recover in case anything goes wrong. Speaking of recovery, a "side effect" of using this is that the custom kernel that you flash in order to run the update package allows recovering without a serial cable and the installation of unsigned recovery packages.<!-- more -->

[Here it is.](/p/kindleupdater)

Oh, and in case anyone is wondering why I'm not just distributing a full 3.X update package and making you generate it by yourself, it's because the Kindle framework and OS are proprietary code. I believe that Amazon didn't release 3.0 for the DX and K2 because they don't want to lose business for the Kindle 3. So, by making you have a Kindle 3 in order to use this, I can keep Amazon happy.
