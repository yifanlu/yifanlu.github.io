---
author: yifanlu
comments: true
date: 2014-01-04
layout: post
slug: ps-vita-nand-pinout-updated
title: PS Vita NAND Pinout (Updated)
wordpress_id: 709
categories:
- PS Vita
- Vita Hardware Hacking
tags:
- emmc
- nand
- test points
- vcc
- vita
---

Since the [last post](/2013/12/19/ps-vita-nand-pinout/) on the eMMC pinout, I found the other two important test points. First is VCCQ, which is the power to the eMMC controller. It needs to be pulled at 1.8V. The other point is VCC, which is the power to the actual NAND flash. It needs to be pulled at 3.3V.<!-- more -->

[![Found on the bottom of the board, above the SoC](/images/2014/01/vita_nand_testpoint_1-300x300.jpg)](/images/2014/01/vita_nand_testpoint_1.jpg)

Found on the bottom of the board, above the SoC
{: .wp-caption-text}



[![Found on the bottom of the board, near the multi-connector](/images/2014/01/vita_nand_testpoint_2-300x187.jpg)](/images/2014/01/vita_nand_testpoint_2.jpg)

Found on the bottom of the board, near the multi-connector
{: .wp-caption-text}



[![For reference, the pad of the removed eMMC on the second Vita](/images/2014/01/IMG_3314-300x225.jpeg)](/images/2014/01/IMG_3314.jpeg)

For reference, the pad of the removed eMMC on the second Vita
{: .wp-caption-text}






