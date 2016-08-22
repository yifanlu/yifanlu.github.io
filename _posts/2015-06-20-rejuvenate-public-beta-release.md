---
author: yifanlu
comments: true
date: 2015-06-20 21:37:52-06:00
layout: post
slug: rejuvenate-public-beta-release
title: Rejuvenate Public Beta Release
wordpress_id: 1212
categories:
- PS Vita
- Releases
tags:
- psm
- psvita
- rejuvenate
- release
- sdk
- uvloader
- vita
- vitadefiler
---

[Rejuvenate](/2015/06/14/rejuvenate-native-homebrew-for-psvita/), announced last week allows users to install unofficial applications and games (homebrew) onto their PS Vita device. Please read that announcement post for more information. Today, the public beta is ready for testing.

**The beta is only for those who were able to obtain a publisher's license (whose application was approved by Sony before the deadline on May 31). For the rest of you who do not have the publisher license (and no friends with a publisher license) but only the DevAssist app on your Vita, please wait for further instructions to come.**

<!-- more -->

Note that currently, the open SDK does not work so you will not be able to develop or run homebrew. You can only run the 'hello world' demo. The main purpose of this open beta is to test VitaDefiler and UVLoader and find bugs in those tools as well as the setup process. Please follow the directions in README.txt. If you encounter any bugs or errors, join the [#vitadev](irc://irc.efnet.net/vitadev) IRC channel on EFNet for support. No support will be provided by me elsewhere (Twitter, here, forums, etc). If you are running a version of PSM DevAssist that is currently not supported, please also join the IRC channel so we can port the tools to your PSM version.



### Creating an App Key for a Friend



For those who have a friend with a PSM Publisher License (and can create app keys), follow the [official guide](https://psm.playstation.net/static/general/all/psm_sdk/1/doc/en/key_management_team_en.html) from PSM to get an app key for your device. Then after you get BallMazeDemo to run on your Vita, you can run SETUP.bat from the Rejuvenate beta release and it will find the right keys. From personal experience, you may have to go through all the steps twice for it to work. I will not provide any support for this workaround, but feel free to ask Sony's [official forums](http://community.eu.playstation.com/t5/PlayStation-Mobile-Studio/bd-p/bPSS_dStudio). If you do not have access to a friend with a Publisher License, more instructions will be posted in the upcoming weeks.



### Developer Help Needed


There are currently two open sdks in the works. [psp2sdk](https://github.com/173210/psp2sdk/) and [vita-toolchain](https://bitbucket.org/cirne/vita-toolchain/). Both require more work to produce valid ELFs that UVLoader can load. If you are a developer, please consider contributing to one or both of these projects. I personally like [vita-toolchain](https://bitbucket.org/cirne/vita-toolchain/) more because it implements my [specifications](/2015/05/23/calling-all-coders-we-need-you-to-help-create-an-open-vita-sdk/) exactly but I am biased that way.



### Sources






	
  * [UVLoader](http://github.com/yifanlu/UVLoader) with 3.x support is on GitHub as always.

	
  * [VitaDefiler](https://github.com/yifanlu/VitaDefiler) sources are now public. Feel free to clone the repository and make changes as desired.

	
  * [VitaInjector](https://github.com/yifanlu/VitaInjector), the precursor to VitaDefiler is also public. This was the tool developed before VitaDefiler and also serves as the dirty and hacked up backend for VitaDefiler. It only supports FW < = 2.50. No support will be provided for this tool, its release is only for historical and archival purposes.




The writeup for the exploits used in VitaDefiler will be published soon. They have been written years ago, so I only need to update and polish them before publishing.



### Download



The download can be [found here](/p/rejuvenate/). For those who like to share, please link to that page instead of the ZIP file because I will be posting updates directly to that page and the current download link would be outdated soon.
