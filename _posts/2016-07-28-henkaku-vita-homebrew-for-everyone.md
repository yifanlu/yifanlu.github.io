---
author: yifanlu
comments: true
date: 2016-07-28 09:07:31-06:00
layout: post
slug: henkaku-vita-homebrew-for-everyone
title: 'HENkaku: Vita homebrew for everyone'
wordpress_id: 1477
categories:
- Announcements
- PS Vita
tags:
- henkaku
- homebrew
- rejuvenate
- toolchain
- vita
---

![Photo credits to Davee](/images/2016/07/vita-henkaku-300x154.png){: .alignleft} For the last couple of months, **molecule** (composed of I along with **Davee**, **Proxima**, and **xyz**) have been working hard to bring you an easy-to-use homebrew solution. The result is HENkaku (変革), the first HEN for the Vita. Since the release of [Rejuvenate](/2015/06/14/rejuvenate-native-homebrew-for-psvita/) a year ago, developers have created tons of wonderful emulators, games, and apps for the Vita. Unfortunately, Rejuvenate is hard to set up, has many annoying limitations, and supports only an older firmware version. As a result, we recommended Rejuvenate only to developers who wish for an unofficial way to write apps for the Vita. When I first announced Rejuvenate and the call for an open toolchain, I emphasized that the SDK must be binary compatible with the Vita's native loader. I published the [specifications](https://github.com/vitasdk/vita-toolchain/blob/master/doc/specifications.pdf) document and some gracious developers took up the task and wrote [vita-toolchain](https://github.com/vitasdk/vita-toolchain). At the time, there were some pushback on why I was adamant on binary compatibility when the loader was also written by us. Well, the reason was this: developers (mostly) do not have to make any changes to their code. If your homebrew ran on Rejuvenate, it will run with HENkaku with minimal work. We ask developers to build their code with the [latest toolchain](https://goo.gl/QpX5zM) now for HENkaku compatibility.



## What is HENkaku?



HENkaku simply lets you install homebrew as bubbles in LiveArea. It is a native hack that disables the filesystem sandbox. It installs [molecularShell](https://github.com/henkaku/VitaShell), a fork of VitaShell that lets you access the memory card over FTP and install homebrew packages (which we create as VPK files). With vita-toolchain, developers have access to the same system features licensed developers have access to as well as undocumented features that licensed developers cannot use (including overclocking the processors).



## What is it NOT?



It does not let you install or run Vita "backups", warez, or any pirated content. It does not disable any DRM features. It does not let you decrypt encrypted games. Here's my stance on this: I do not care one way or the other about piracy. I do not judge people who do pirate. I will not act as the police for pirates. However, I will personally not write any tools that aid in piracy. It is my choice just as it is the pirate's choice to steal content.



## FAQ



HENkaku will only work on 3.60 so we recommend that you update to it. We know that updating to 3.60 breaks many current Vita tweaks and hacks so here's a short guide on what HENkaku replaces.





  * 
**Should I update if I already have Rejuvenate?**  

Rejuvenate was a limited hack mainly designed for developers who wish to dip their toes in the water. Rejuvenate will not be supported anymore. HENkaku is superior in every way.



  * 
**Should I update if I am using VHBL or PSP homebrew bubbles?**  

Yes. Since HENkaku gives homebrew full filesystem access, it is possible for a developer to create a "bubble creator" Vita homebrew that generates signed PSP homebrew bubbles. As a bonus, you don't have to purchase any games to do this! We (molecule) will not provide support for PSP/PS1 related stuff though, so all this depends on someone else picking up the baton. If you are highly dependent on PSP homebrew, we suggest that you wait and not update past 3.60.



  * 
**Should I update if I am using eCFW/ARK/TN-V/TN-X?**  

We do not support running PSP ISOs/backups. Your best option would be to wait and see what other developers do.



  * 
**Should I update if I am using FailMail tricks to modify the system (whitelist, themes, etc)?**  

Yes. Vita homebrew will have full filesystem access so you can do app.db modifications as well as change whitelist files and so on. It is even possible for homebrew developers to write apps that do system mods for you so you don't have to mess with sqlite at all.






## Release



HENkaku will be released publicly on **07/29/2016 9:00AM UTC** at [https://henkaku.xyz/](https://henkaku.xyz/). It will only support firmware 3.60, so feel free to update to it now in preparation. Do not update past 3.60, the current firmware at the time of this writing.
