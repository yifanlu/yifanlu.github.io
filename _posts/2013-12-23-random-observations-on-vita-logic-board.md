---
author: yifanlu
comments: true
date: 2013-12-23 02:09:19-06:00
layout: post
slug: random-observations-on-vita-logic-board
title: Random observations on Vita logic board
wordpress_id: 684
categories:
- PS Vita
- Vita Hardware Hacking
tags:
- connector
- emmc
- hardware
- hdmi
- logic board
- nand
- syscon
- usb
- video
- vita
- vitatv
---

While I'm waiting for more tools to arrive, here's some things I've found while playing around with the continuity test on a multimeter. There is no stunning discovery here, just bits and pieces of thoughts that may not be completely accurate.<!-- more -->


#### On Video Out


The unfilled pads next to the eMMC has something to do with video. The direction of the trace goes from the SoC to the video connector. A continuity test shows that all the pads comes from the SoC and leads to some point on the video connector. Could they be pads used for testing video in factory? Looking at the VitaTV teardown from [4gamer.net](http://www.4gamer.net/games/990/G999021/20131115043/screenshot.html?num=022) shows that traces in a similar location coming out of the SoC goes through similar looking components and then into the Op-Amp and to the HDMI connector. This is a stretch, but could these traces output HDMI if connected properly? As a side note, I could not find any direct connection between anything on the video connector to either the mystery port or the multi-connector. If Sony were to ever produce a video-out cable, there needs to be a software update as there doesn’t seem to be hardware support.


#### On the Mystery Port and USB


The first two pins on the mystery port appear to be ground (or Vdd and Vss). The last pin could be a power source. Pins 3 and 4 goes through a component and directly into the SoC. What’s interesting is that the D+/D- USB line from the multi-connector on the bottom goes through a similar looking component and that they are very close to the pins that handle the mystery port. Looking at [4gamer.net](http://www.4gamer.net/games/990/G999021/20131115043/screenshot.html?num=025)’s VitaTV teardown again, we see that the USB input port has two lines that go through very similar paths (the various components that it goes through) as the Vita’s USB output, but the position of the traces going into the SoC on the VitaTV is the same position of the trace on the Vita coming from the mystery port. Could the mystery port be a common USB host/USB OTG port with a custom plug?


#### On the Mystery Chip


Also [4gamer.net](http://www.4gamer.net/games/990/G999021/20131115043/screenshot.html?num=027) speculates the SCEI chip on the top of the board has something to do with USB, but I think that’s not true because USB lines go directly into the SoC. Which means that we still don’t know what the SCEI chip does (it is the only chip on the board that has yet been identified by [any source](www.techinsights.com/teardowns/sony-playstation-vita-teardown/)). My completely baseless hypothesis is that it’s syscon because it would be reasonable to assume that the syscon is outside of the SoC since it would decide when to power own the SoC.


#### On the eMMC


This may be public knowledge but the Vita’s eMMC NAND is 4GB (same as VitaTV and Vita Slim). The new Vitas do not have any additional storage chips. This also means that the 1GB internal storage on the new Vitas is just another partition or something on that NAND (no hardware changes).
