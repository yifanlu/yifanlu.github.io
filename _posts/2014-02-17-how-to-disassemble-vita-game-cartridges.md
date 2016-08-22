---
author: yifanlu
comments: true
date: 2014-02-17 06:57:35-06:00
layout: post
slug: how-to-disassemble-vita-game-cartridges
title: How to Disassemble Vita Game Cartridges
wordpress_id: 731
categories:
- Guides
- PS Vita
- Vita Hardware Hacking
tags:
- cartridges
- emmc
- game
- hardware
- nand
- nandway
- sd
- soldering
- vita
---

A hacker named katsu recently released a [method for dumping Vita games](http://wololo.net/talk/viewtopic.php?f=64&t=36827). As a developer, I am completely against piracy, but as a reverse engineer I can't shy away from taking apart perfectly working devices. However, most pictures I see of Vita game carts taken apart show the game cart casing damaged beyond repair or completely destroyed. I managed to take apart a game cart and put it together with no obvious signs of damage, and I thought I would share my (simple) method here.<!-- more -->

[![Photo Feb 16, 7 48 07 PM](/images/2014/02/Photo-Feb-16-7-48-07-PM-300x225.jpg)](/images/2014/02/Photo-Feb-16-7-48-07-PM.jpg)



If you take a look at the top right or left corner of the game cart, you can see a line of where the two halves of the plastic was glued together. Locate the upper left corner and, with a sharp knife, push the blade into the line on the corner until you have a small dent. Then, move the knife downwards and wiggle the knife until you loosen the glue for the entire left side of the cart. Then keep moving the knife down and when you hit the bottom of the cart, turn and lose about half the bottom edge of the cart. Now you can use your fingers to spread the two halves apart (but be careful not to use too much force and tear the glue from the other two edges), and you can either shake the memory chip out or use a pair of tweezers.

[![Photo Feb 16, 7 42 47 PM](/images/2014/02/Photo-Feb-16-7-42-47-PM-300x225.jpg)](/images/2014/02/Photo-Feb-16-7-42-47-PM.jpg)



If you were to follow katsu's pinout, you need to solder to the copper pads. A trick for doing so is to first flux up the points and then melt a pea-sized blob of solder in middle of all the points. Then take your iron and spread the blob around until all the pads are soldered up. Then just make the the remaining blob is not on top of any copper and you can easily remove it.

[![Photo Feb 16, 8 29 57 PM](/images/2014/02/Photo-Feb-16-8-29-57-PM-300x225.jpg)](/images/2014/02/Photo-Feb-16-8-29-57-PM.jpg)



Then you can solder wires onto the points to your heart's content. After you're done with everything, you can easily put the memory chip back into the casing and there is enough glue to keep the two halves of the case together (along with the memory chip). You can then continue to play the game.

[![](http://i.hizliresim.com/emz1z2.jpg)](http://wololo.net/talk/viewtopic.php?f=64&t=36827)

Pinout for Vita game cart. Credits to katsu.
{: .wp-caption-text}



If you were to follow the pinout, you can see that it appears to be a standard NAND pinout (not eMMC and not Memory Stick Duo). I have not tested this, but I believe this means you can use [NANDWay](https://github.com/hjudges/NORway) or any other NAND dumping technique (there's lots for PS3 and Xbox 360) provided you attach to the right pins. I suspect that the Vita communicates with the game cart through the SD protocol with an additional line for a security interface, but that is just speculation. If that were the case, having one-to-one dumps would not allow you to create clone games. Regardless, I will not be looking too much into game carts because they are so closely tied with piracy.
