---
author: yifanlu
comments: true
date: 2011-08-07 23:33:08-06:00
layout: post
slug: reverse-engineering-a-dynamic-library-on-the-xperia-play
title: Reverse engineering a dynamic library on the Xperia Play
wordpress_id: 292
categories:
- Assembly
- Technical
- Xperia Play
tags:
- arm
- assembly
- bios
- c++
- debugging
- deflate
- gdb
- hex
- iso
- java
- jni
- memory
- playstation
- psx
- reverse engineering
- sony
- xperia
- xperia play
- zlib
---

Welcome to part two of my journey to completely reverse the PSX emulator on the Xperia Play. When we [last](/2011/08/01/analyzing-the-psx-emulator-on-the-xperia-play/) left off, I managed to figure out the image.ps format and the basic order of execution of the emulator. It's been a week now, and I have more stuff to reveal.<!-- more -->

**Decrypting the data**

One of the main problems was that most of the important files are encrypted. More specifically, these three files: ps1_rom.bin (BIOS), libdefault.so (the emulator), and image_ps_toc (then unknown data). As I mentioned before, Sony used what's called white box cryptography, which means obfuscating the code to hide the decryption keys. But, we don't need the keys, we just need the decrypted data. The obvious way of extracting the decrypted data is dumping it from the RAM. However, the Android kernel I'm using doesn't support reading /dev/kmem and I don't want to mess with recompiling the kernel. I've also tried dumping with GDB, and it did work, but the data isn't complete and is messy. I used a more unorthodox method of obtaining the decrypted data. After hours of reading and mapping in IDA Pro, I figured out that everything that is decrypted goes through one public function, uncompress(), a part of zlib. This is important, because this means everything that is decrypted is sent to zlib and zlib is open source. That means, I just need to recompile zlib with some extra code in uncompress() that will dump the input and output data. A simple fwrite() will do, as the data is already in a clean, memcpy-able form. (I forgot about LD_PRELOAD at the time, but that might have worked easier). After some trouble getting NDK to compile zlib, I have dumps of both the compressed/decrypted and uncompressed forms of all encrypted content.

**Analyzing the dumps**

The next thing is to find out the meaning of the data that we worked so hard to get. ps1_rom.bin is easy. Surprisingly, it is NOT a PS1 BIOS file, but actually part of a PS2 BIOS dump (part, being only the PS1 part of the PS2 BIOS). Does this mean a PS2 emulator is coming for the Play? I don't know. Next, we have libdefault.so. Plugging it into IDA Pro reveals the juicy details of the PS1 emulator. It's really nothing interesting, but if we ever want multi-disk support or decrypting the manuals, this would be the place to look. Finally, we have image_ps_toc (as it is called in the symbols file). I am actually embarrassed to say it took almost a day for me to figure out that it's a table of contents file. I did guess so at first, but I couldn't see a pattern, but after a night's sleep, I figured out the format of the uncompressed image_ps_toc file. (Offsets are in hex, data are little-endian)


```
0x4 byte header
0x4 byte uncompressed image size
0x12 byte constant (I'm guessing it may have something to do with number of disks and where to cut off)
0x4 byte number of entries
Each entry:
0x4 byte offset in image.ps, where the compressed image is split
```

**Image.ps format**

I actually forgot to mention this in my last post. The "rom" that is loaded by the emulator is a file named image.ps. It is found on the SD card inside the ZPAK. It is unencrypted, and if you delete it, it will be downloaded again from Sony's servers unencrypted. How it works is that an PSX ISO is taken and split into 0x9300 (about 38kb) sections, and each section is compressed using deflate (zlib again) and placed inside image.ps (with a 0x14 byte header). The offsets of each section is stored in the toc file (and encrypted) because although uncompressed, they're perfect 38kb sections, compressed, they're variable sized. I already wrote a tool to convert image.ps to an ISO and back again/

**Putting it all back together**

Now that we've tore apart, analyzed, and understood every element of the PSX emulator on the Xperia Play, what do we do? The ultimate goal is to convert any PSX game to run on the Xperia Play, but how do we do that. There are two main challenges. First of all, libjava-activity.so, which loads everything, expects data to be encrypted. Once again, we need keys. Also, I'm pretty sure it uses a custom encryption technique called "TFIT AES Cipher", because I was not able to find information on it anywhere else. However, since we have the decrypted files, we can patch the library to load the decrypted files directly from memory, and I was halfway into doing that when I realized two more problems. Secondly, if I were to patch the library to load decrypted data, that means every user needs to decrypt the files themselves (because I won't distribute copyrighted code). Third, image_ps_toc is variable sized, which means if the image is too big, it'll break the offsets and refuse to load.

Currently, I'm trying to find the easiest and most legal way of allowing custom image_ps_toc files to work. (Bonus points if I can also load custom BIOS files). What I hope for is to write a wrapper library, libjava-activity-wrapper.so, which loads libjava-activity.so and patches GetImageTOC and GetImageTOCLength to load from a file instead of memory. This means I have to deal with Java and JNI again (ugh), and also do some weird stuff with pointers and memcpy (double ugh). The JNI methods in the library do not have their symbols exported, so I have to find a way of manually load them.

**Bonus: blind patching a binary**

When trying to patch a method for an ARM processor, it's a bit of a pain and I'm too lazy to read about proper GDB debugging techniques. In additions, Sony wasn't kind enough to compile everything with debugging symbols. However, I came up with a hacky-slashy way of changing instructions and seeing what happens. First, open up IDA Pro and find the function you want to modify. For example, I want decrypt_executable() to bypass decryption and just copy data plain. Find the instruction to change, and the opcode to change it to. For example, I want to change a BL instruction to NOP and CMP to CMN (don't jump to decryption process and negate the "return == 0"). I have ARM's NOP memorized by now 00 00 A0 E1. I don't know what CMN's opcode is, but if I look around I can find CMN somewhere and I see it's just a change from a 7 to a 5. After everything's done, copy it over to the phone and run it. If it crashes (and it should), look at the dump. The only important part is the beginning:


```
>     
>     I/DEBUG   (  105): signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 00000054
>     I/DEBUG   (  105):  r0 002d9508  r1 413c103c  r2 2afcc8d0  r3 002d93d8
>     I/DEBUG   (  105):  r4 00000004  r5 002d93e0  r6 6ca9dd68  r7 00000000
>     I/DEBUG   (  105):  r8 7e9dd478  r9 2cbffc70  10 0000aca0  fp 6caa4d48
>     I/DEBUG   (  105):  ip 002d93e8  sp 7e9dd0c0  lr 00000001  pc 4112d01c  cpsr 40000010
> 
> 
```


The error message doesn't help at all "SIGSEGV," but we have a dump of all the registers in the CPU. The important one is the PC (program counter), which shows what offset the last instruction was at offset 0x4112d01c in the memory. To get the program offset, just cat /proc/{pid}/maps to find where libjava-activity.so is loaded in memory. Subtract the offsets, and pop it into IDA pro. Now figure out why it crashed and try again. I need to learn proper debugging techniques one day.
