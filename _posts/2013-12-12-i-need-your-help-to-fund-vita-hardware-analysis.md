---
author: yifanlu
comments: true
date: 2013-12-12
layout: post
slug: i-need-your-help-to-fund-vita-hardware-analysis
title: I need your help to fund Vita hardware analysis
wordpress_id: 634
categories:
- Announcements
- PS Vita
tags:
- digital signals
- fundraising
- hardware
- homebrew
- jtag
- logic board
- nand
- soc
- tools
- vita
- vitatv
---

It's been a little more than a year since I demonstrated the first Vita running [unsigned code](/2012/12/12/playstation-vita-the-progress-and-the-plan/), and it's been dead silent since then. There is a lot of work on the [PSP emulator](http://wololo.net/2013/12/02/official-tn-v4-changelog-revealed/) but it's been pretty quiet on the Vita front. In fact, there hasn't even been any new userland exploits found (by me or others) for a year. I made a post a while ago saying that [progress through hardware](/2013/09/10/why-hacking-the-vita-is-hard-or-a-history-of-first-hacks/) was one of the few options we haven't looked extensively at, and the reason for that is because hardware hacking is an expensive endeavor. All this time I've been sitting and waiting for progress to be made by some unknown genius or some Chinese piracy company (sadly, for some scenes *cough* 3DS *cough*, this is the way devices get hacked since these companies have the money to do it); progress that would allow people like me to continue with the software work. Unfortunately, as of today, I have not heard of any ongoing work on Vita hardware hacking (PLEASE tell me if you are so we can collaborate). In fact, one of the simplest thing to do (hardware-wise), dumping the NAND, hasn't been done (or publicly stated to be done) yet. Meanwhile, the [PS4](http://www.psdevwiki.com/ps4/JAISPI) has gotten its NAND dumped in a couple of weeks. Since nobody else seem to be serious about getting this device unlocked and poked at by hobbyists, I feel like it's time for me to learn how to stop fearing and love the hardware. And I need your help.<!-- more -->


### Disclaimer


Before we talk business, I want to be as open and honest as possible. I am not a hardware hacker. I have very minimal experience with hardware (I know how to solder and I know what resistors look like), so by no means am I the best person for this job. In fact, I wish there was someone else doing this. My only qualification is the small amount of knowledge I have running userland Vita code and exploring the [USB MTP protocol](/2012/02/18/playstation-vitas-usb-mtp-connection-analyzed/). It could turn out that I'm completely incompetent and not get anything useful. It could turn out that everything works out but my goals were set in the wrong direction. It could also take a very long time before any results are found (since this is a hobby after all). But, I will always be as open as possible; documenting any small discoveries I make and posting details and guides about what I'm doing. I'll post any large transaction that takes place within the scope of this project and admit any mistakes I'll definitely make. I won't be able to release data I obtain from the device for legal reasons (including any actual dumps made) but I will post instruction for reproducing everything I do. I have seen other "scene" fundraisers and the problems that arises in them (mostly lack of response from the developer(s)) and will try to avoid making such mistakes. If you still believe in me, read on.


### Funding the Project


I never ask for donations before I complete a project because I don't like taking money for just expectations. I believe that the user should only donate once they try something and love it. I turned down many requests to donate money in the past and always asked for unwanted/broken hardware donations instead, however, it seems that there are more people willing to donate money than donate devices. In a perfect world, I would fund this project with my own money, but in a perfect world, I would be rich. Since this is the first time I'm looking seriously at hardware, I'm going to need to buy tools and devices to do research that would benefit the community (hopefully). I hesitantly and sincerely ask for your help. There are two main goals, the first one will let me get a hardware setup working so I have to tools to work with. The second will allow me to get hardware to test using the tools. If I end up going over the estimated amount, I will pay out of my own pocket. Any remaining money after the project is fully funded will be donated to the EFF. All your money will benefit the homebrew community. Also, all of the prices are estimated (with fees calculated in) with simple searches so if you can find a better deal or if you can get me the item directly, please [contact me](/contact/)!


### Goals


To be honest, there is no clear roadmap at this point. The first thing is to dump the NAND, try to map out signals from the CPU/SoC, and look at the data IO from the memory card, game card, and connectors. From there, I hope to get a better idea of how the hardware works and find where to go from there. I promise that I will not ask for more money once this is funded and any additional venture will come out of my own pockets.


## Donate




Thanks to everyone who donated! The goal was met in less than a week. I'm currently in the process of buying the supplies and will post an update as soon as I can. If you have a broken Vita hardware, please consider donating it as more hardware to work with is better and there are other people I'm working with who can benifit also from having a logic board to work with.




## Goal 1: Setup and Finding Traces ($80)


Before we can dump the NAND, we need to sacrifice a logic board to remove the NAND and trace the BGA points and find test points to solder. The board has to be sacrificed because realistically, it's very hard to reflow such a tiny chip. In addition, the SoC would also be removed to see if there's any interesting test points coming out of the CPU (potentially to see if there's any JTAG or other debugging ports coming out, which is unlikely). I would need:

#### Vita Logic Board - $20

![Vita Logic Board](/images/2013/12/vita_logic-150x150.jpg)

It does not have to be fully working. On eBay, people are selling Vita logic boards with broken connectors for around this price (after shipping).

#### Heat Gun - $21

![Heat Gun](/images/2013/12/heat_gun.jpg)

A heat gun is needed to remove the surface mounted NAND and SoC. It's also why reattaching it almost impossible because the hot air will blow the components around.


#### Soldering Tools - $20

![Soldering Tools](/images/2013/12/soldering_tools-150x150.jpg)

I do have basic soldering tools, but throughout the project, there will be tasks that require more precision, so I would need a magnifying soldering station (cheapest is $15 on Amazon), soldering flux (about $5 on Amazon), and some small tools.


#### Digital Multimeter - $10

![Multimeter](/images/2013/12/multimeter-150x150.jpg)

A cheap one will do. I only need it for continuity testing and reading resistor values.

#### Saleae Logic Analyzer (clone) - $10

![Saleae Clone](/images/2013/12/saleae_clone-150x150.jpg)

Although a real Saleae logic is $150 (for 8 ports) or $300 (for 16 ports), there's some cheap clones on eBay going for about $10. This would allow me to find signals coming out of a running Vita and, for example, verify that the test points found are indeed data driven.

## Goal 2: Dumping the NAND and Testing ($250)


After getting all the tools and finding the traces, the first thing to do is to dump the NAND from a working console. This should be easy once the trace is found since the NAND is eMMC (can be dumped using an SD card reader). Next, I want to explore the signals coming into and out of the Vita (USB, multi-connector, mystery port, memory card, game card). Then depending on what I find, I'll go from there.


#### PlayStation Vita Console - $100-150

![Vita](/images/2013/12/vita.jpeg)

This would be the working console that I will test with. First, I will dump the NAND with the test points found. Then I will try to analyze the game card and memory card traffic using the logic analyzer. Although the console should be working, to save money, I may get one with a broken screen, which goes for around $100 on eBay or a used unit for $150 on CowBoom. If you own a broken Vita, and want to donate it instead of money, please [contact me](/contact/).

#### PlayStation VitaTV Console - $120

![VitaTV](/images/2013/12/vitatv-150x150.jpg)

First a NAND dump of the VitaTV would be interesting to see if there's any differences (assuming it's not encrypted). Also, I would like to see how the HDMI port is connected (4gamer suspect that HDMI out comes directly from the SoC) and see if I can get a regular Vita to output HDMI (most likely not possible without software and hardware modifications). I also want to do some software tests on the VitaTV as the introduction of USB host may also introduce new bugs into the system (remember how the PS3 was hacked). It seems to be about $120 after shipping from Nippon-Yasan. If you want to donate a VitaTV directly instead of money, please [contact me](/contact/).

#### PlayStation Vita Cradle - $15

![vita_cradle](/images/2013/12/vita_cradle-150x150.jpg)

The Vita cradle is a good pin-out interface for the Vita multi-connector. By soldering to the cradle, it would minimize the risk of damage to the Vita directly. Exploring the multi-connector is a good way to start since there are 16 pins and only a few of them are figured out.


#### (Optional) PlayStation Vita PCH-2000 - $220

![vita_pch-2000](/images/2013/12/vita_pch-2000-150x150.jpeg)This is purely optional and only if someone generous would like to donate the console to me directly. There's not much I want to do here except dump the NAND and trace the microUSB signals.
