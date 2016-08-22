---
author: yifanlu
comments: true
date: 2011-09-17 21:20:57-06:00
layout: post
slug: installing-windows-8-developer-preview-8102-on-a-usb-drive-windows-to-goportable-workspace
title: Installing Windows 8 Developer Preview (8102) on a USB Drive (Windows To Go/Portable
  Workspace)
wordpress_id: 312
categories:
- Guides
- Technical
tags:
- build 2011
- dvd
- hard drive
- portable workspace
- pwcreator
- tip
- usb
- windows
- windows 8
- windows to go
---

This really isn't some technical or hard to do thing, but it's a cool little trick I found that I haven't seen mentioned before. If you don't know what "Windows To Go" (previously "Portable Workspace"), watch [this video from the Build 2011 conference](http://channel9.msdn.com/Events/BUILD/BUILD2011/HW-245T). Basically, it allows you to install a full copy of Windows 8 onto a USB drive/external hard drive and use it on any computer that supports USB booting. Your settings, files, programs, etc go where-ever you go. The feature is in Windows 8 (and the developer preview), but the program to make the drive is not. Luckily, an old leaked build has the program, but you can't just copy and paste it, it won't run. Instead, follow the directions below to get Windows 8 installed to a USB drive. (I used a virtual machine to do the following, therefore I did not need to burn any DVDs. I will give the directions assuming you're using a real computer though).<!-- more -->

**Requirements:**



	
  * [Windows 8 Developer Preview](http://msdn.microsoft.com/en-us/windows/apps/br229516) burned to a DVD (unless you're using virtual machine)

	
  * Windows 8 M1 build 7850 burned to a DVD (unless you're using virtual machine)

	
  * 16GB flash drive or external hard drive (or larger)




**Directions:**








	
  1. Install Windows 8 M1 build 7850. (I tried just copying pwcreator.exe and running it on a later build, but it didn't work.)

	
  2. Open the start menu and type in "pwcreator.exe" and press enter. Alternatively, find and open C:\Windows\System32\pwcreator.exe

	
  3. Choose your USB drive and continue.

	
  4. Insert the Windows 8 M1 build 7850 DVD again and continue.

	
  5. Before starting the build process, take out the Windows 8 M1 build 7850 DVD and insert your Windows 8 Developer Preview build 8102 DVD.

	
  6. Continue and allow the process to finish.




I tested it with the x86 version of the Developer Preview, so I don't know how well or if it will work with the x64 build. When you are asked to activate Windows, you can skip it or enter one of the keys found in the Developer Preview DVD under D:\Sources\product.ini (assuming D: is your DVD). I haven't figured out which key to use yet.







Also, the requirements in pwcreator.exe states that you need a 16GB USB drive. However Windows only really need 12GB to install. I have a 16GB flash drive that shows up as 15GB and it wouldn't work. I used GParted in Ubuntu to copy the partitions from a larger USB drive over after creating the image and it works fine. Just a tip.
