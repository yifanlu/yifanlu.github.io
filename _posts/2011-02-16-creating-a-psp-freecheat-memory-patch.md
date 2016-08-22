---
author: yifanlu
comments: true
date: 2011-02-16
layout: post
slug: creating-a-psp-freecheat-memory-patch
title: Creating a PSP FreeCheat Memory Patch
wordpress_id: 241
categories:
- Guides
- Technical
tags:
- endian
- freecheat
- hex
- memory
- patch
- psp
- technical
---

FreeCheat is a memory editor and cheat device (like Action Replay) for the PSP. It includes features like a live in-game memory viewer and searcher. One of the feature that intrigued me is the memory patcher. I had no idea what it does, but I assume it does what it says: patches the memory. Problem is: I've searched everywhere, but there seems to be no information on how to create a FreeCheat memory patch for the PSP (only .pat files for Monster Hunter). Well, it's not that hard. After some trial and error, I've found out how to create a FreeCheat .pat memory patch. Note that the following should only be attempted by a person with enough technical knowledge to understand it.<!-- more -->

To create a memory patch, first you need to find out what you want to patch. I suggest using FreeCheat's own memory searcher to find the memory location. Another method if using FreeCheat to dump the memory to a file, and open it on your computer with a hex editor. Once you find something you want to replace, look at the address. On FreeCheat, this is the hex number on the bottom left of the memory viewer box. On your hex editor, it should be listed as "address" or "offset". This should be between 0x0000000 and 0x017FFFFF. Now take this number and add 0x08800000 (hex math please) to it.

You can now create a new file in your hex editor to be the patch. The first four bytes in the file is the memory offset (that you found) in big endian form. The problem is that the offset you found is a little endian number. You need to convert it to a big endian number. Most hex editors allows something like this. I use [0xED](http://www.suavetech.com/0xed/0xed.html) on OSX, so on there (make sure it's set to Edit->Number Mode->Little Endian!), I would type in 00000000, highlight it, and under "32 bit unsigned", I would paste in the offset I found and it would convert it automatically. Then in the rest of the hex document, fill in whatever you want to replace the memory with. Save this as a .pat file and copy it to your PSP at /FreeCheat/PATCH and on the PSP, open up FreeCheat, go to MEM Manager and Load MEM Patch.
