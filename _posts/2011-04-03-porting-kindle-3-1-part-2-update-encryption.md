---
author: yifanlu
comments: true
date: 2011-04-03 04:44:08-06:00
layout: post
slug: porting-kindle-3-1-part-2-update-encryption
title: 'Porting Kindle 3.1: Part 2 - Update encryption'
wordpress_id: 268
categories:
- Assembly
- Kindle
- Technical
tags:
- binary
- dm
- encryption
- hex
- ida pro
- image
- kindle
- kindle 2
- kindle 3
- md
- port
- update
---

**Overview**

So, on the topic of Kindle (I swear, it's becoming an obsession). I am currently in the process of porting the Kindle 3.1 software to Kindle 2 and DX. I will make a series of posts describing my process while describing how various parts of the Kindle operating system works. Now I've tested 3.1 on my Kindle 2 and it works perfectly fine. All features work (audio, TTS, book reading), and the new features also work without major slowdowns (new PDF reader, new browser, etc).

Where's part one you ask? Well, part one was getting the 3.1 OS to work on the Kindle 2, the rest is making an easy installer. That is a long story that involves custom partition tables, manually creating tar files (checksums are a pain), remote debugging, and more. It's a lot of stuff and most aren't very useful because nobody should have to repeat the process, which is why I'm creating a easy to use installer. If I have time one day, I may write it down for documentation purposes.<!-- more -->

First of all, I will write down the game plan. What I plan to do is create an installer with the least amount of steps for the user. I'm hoping for a two part or three part installer. (Can't be one part because you need a copy of the OS, and distributing it is most likely frowned upon by Amazon). How the installer should work is:



	
  1. User copies a image-creator package on a jail-broken Kindle 2. This package will backup the original OS, and generate a new ext3 image with some required files from the Kindle 2 (drivers and such). It will also update the kernel to support recovery packages.

	
  2. User keeps backup in a safe place and copies the image-creator package and the image generated from the K2 on a jail-broken Kindle 3 and runs the package. The image-creator will scan the filesystem making sure all files exist are are unmodified, then copies the files to the ext3 image. It will then take the ext3 image and generate a Kindle 2 recovery package with the 3.1 OS.

	
  3. User copies the recovery package generated from the Kindle 3 and copies it to the Kindle 2 and restarts. The Kindle will write the ext3 image to the root partition.


**Update Encryption**

Now, Igor Skochinsky wrote a [nice post](http://igorsk.blogspot.com/2007/12/hacking-kindle-part-2-bootloader-and.html) a couple of years ago on the Kindle update encryption algorithm. Basically, to encrypt an update, you take each byte of the file and shift the bits four to the left and OR it with the same bits shifted four to the right. Then you AND the result by 0xFF and XOR it by 0x7A. (Sounds like some computer dance move). Well, Igor also wrote a nice Python script that does the encrypting and decrypting, but I didn't want to port Python to Kindle, so I decided to modify Amazon's update decryption script "dm" and reverse it to make a encryption script "md". I opened up IDA Pro and looked for the encryption. Here it is nicely commented by me into psudocode:


```
BL	getchar // get byte to modify
EOR	R3, R0,	#0x7A // R3 = R0 ^ 0x7A
CMN	R0, #1 // if !(R0 == 1), we are at the end of the file ...
MOV	R0, R3,LSR#4 // R0 = R3 >> 4
AND	R0, R0,	#0xF // R0 = R0 & 0xF
ORR	R0, R0,	R3,LSL#4 // R0 = R0 | R3 << 4
BNE	loc_8470 // ... then jump to end of program
MOV	R0, #0 // clear R0 register
ADD	SP, SP,	#4 // don't care
LDMFD	SP!, {PC} // don't care
```


It was a simple matter of reversing the instructions and registers, but like I said before, IDA Pro does not allow changing instructions directly, so I had to mess around with the machine code in the hex editor until I made the instructions I want. Here's the modified function nicely commented by me in human.


```
BL	getchar // get byte to modify
CMN	R0, #1 // if byte is 0x01, then ...
MOV	R3, R0,LSR#4 // set R0 to R0 right shift 4
AND	R3, R3,	#0xF // set R4 to R4 logical AND 0xF
ORR	R3, R3,	R0,LSL#4 // set R3 to R3 logical OR ( R0 left shift 4 )
EOR	R0, R3,	#0x7A // set R0 to R3 logical exclusive OR 0x7A
BNE	loc_8470 // ... exit program
MOV	R0, #0 // clear register R0
ADD	SP, SP,	#4 // don't care
LDMFD	SP!, {PC} // don't care
```


If you want to try it out, here's the [bspatch](/files/md.bspatch) from "dm" to "md". MD5 of dm is 6725ac822654b97355facd138f86d438 and after patching, md should be 3b650bcf4021b41d70796d93e1aad658. You can copy md to your Kindle's /usr/sbin and test it out:


```bash
echo 'hello world' | md > hello.bin # "md" encrypt 'hello world' and output to hello.bin
cat hello.bin | dm # "dm" decrypt hello.bin and it should output 'hello world'
```

Now that we can create update packages from the Kindle, I can start working on the Kindle 2 image-creator script.
