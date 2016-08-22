---
author: yifanlu
comments: true
date: 2012-03-08 02:22:48-06:00
layout: post
slug: unlocking-t-mobile-4g-hotspot-zte-mf61-a-case-study
title: 'Unlocking T-Mobile 4G Hotspot (ZTE MF61): A case study'
wordpress_id: 546
categories:
- Assembly
- Guides
- Qualcomm
- Technical
tags:
- 4g
- assembly
- at
- disassembly
- flashing
- google
- hexagon
- hotspot
- mdm8200a
- mf61
- modem
- nck
- objdump
- qsdp6
- tmobile
- toolchain
- unlocking
- zte
---

So, I have one of [these MiFi clone](http://www.t-mobile.com/shop/Phones/cell-phone-detail.aspx?cell-phone=T-Mobile-4G-Mobile-HotSpot) from T-Mobile and want to unlock it to use on AT&T (I know that AT&T 4G/3G isn't supported, but I thought maybe I could fix that later). The first thing I tried to do was contact T-Mobile, as they are usually very liberal concerning unlock codes. However, [this time](http://support.t-mobile.com/thread/18034), T-Mobile (or, as they claim, the manufacture) isn't so generous. So I've decided to take it upon myself to do it. I will write down the entire procedure here as a case study on how to "reverse engineer" a new device. However, in no way do I consider myself an expert, so feel free to bash me in the comments on what I did wrong. Also, I have decided against releasing any binaries or patches because phone unlocking is a grey area (although it is legal here), but if you read along you should be able to repeat what I did, even though I will also try to generalize.<!-- more -->

**Getting information**


> The hardest part of any hack is the figuring-out-how-to-start phase. That's always tricky. But... let the games begin.
> 
> -Wheatley, _Portal 2_


So before we can do anything, we need to know what to do. The best place to begin is to look [at the updater](http://tmodc.hcac.com/T-Mobile4GMobileHotSpot/FirmwareMac/Mac.htm). A quick look at the extracted files, we find that the files being flashed have names such as "amss.mbn", "dsp1.mbn", and such. A quick scan with a hex editor, we see that the files are unencrypted and unsigned. That's good news because it means we have the ability to change the code. A quick Google search shows us that these files are firmware files for Qualcomm basebands. Now, we need to find more information on this Qualcomm chip. You may try some more Google-fu, but I took another path and [took apart the device](http://twitter.com/#!/yifanlu/statuses/160574621719080960) (not recommended if it's any more complicated). In this case, I found that we are dealing with a Qualcomm MDM8200A device. Google that and you'll find more information such as there are two DSP processors for the modem and on "apps" ARM processor (presumably for T-Mobile's custom firmare, and is what you see as the web interface). We want to unlock the device, so I assume the work is done in the DSP processor. That's the first problem. QDSP6 (I found this name through more Google skills) is not a supported processor in IDA Pro, my go-to tool, so we need another way to disassemble it.

**Disassembly**

Some more Googling (I'm sure you can see a pattern on how this works now) leads me to [this](https://developer.qualcomm.com/hexagon-processor). QDSP6 is actually called "Hexagon" by Qualcomm and they kindly provided an EBI and programmer's guide. I guessed from the documents that there is a toolchain, but no more information is provided about it. More searching lead me to believe that the in-house toolchain is proprietary, but luckily, there is an [open source implementation](https://www.codeaurora.org/patches/quic/hexagon/4.0/) that is being worked on. Having the toolchain means that we can use "objdump", the 2nd most popular disassembly tool [Citation Needed]. So, it's just a matter of sending dsp1.mbn and dsp2.mbn into objdump -x? Nope. It seems that our friends at ZTE either purposely or automatically (as part of the linker) stripped the "section headers" of the ELF file. I did a quick read of the [ELF specifications](http://www.skyfree.org/linux/references/ELF_Format.pdf) and found that the "section headers" are not required for the program to run, but provides information for linking and such. What we did have was the "program headers", which is sort of a stripped down version of the section headers. (Program headers only tell: 1) where each "section" is located in file and where to load it in memory, 2) is it program or data?, 3) readable? writable?, while section headers give more information like the name of each section and more on what the program/data section's purpose is). What I then did is wrote my own section headers using the program headers as a guide and made up the names and other information (because they are not used in the actual disassembling anyways) with a hex editor. Then I pasted my headers into the file, changed some offsets, and objdump -x surrendered the assembly code. 180MB worth of it.

**Assembly**

So we have 180MB worth of code written in a language that could very well be greek. Luckily, as I've mentioned earlier, Qualcomm released a document detailing the QDSP assembly language and how it's used. Most likely, you would be dealing with a more "popular" processor like ARM or x86 and would have access to more resources. However, for QDSP6/Hexagon, we have two PDF documents and that is basically the Bible that we need to memorize. I then spend a couple of hours learning this new assembly language (assembly isn't that hard once you embrace it) and figured out the basics needed to reverse engineer (that is: jumps, store/loads, and arithmetic). Now, another problem arises. We have literally 3 million lines of assembly code with no function names, no symbols, and no "sections". How do we find where the goal (the function that checks the NCK key and unlocks the device accordantly) without spending the next two years decoding this mess? Here, we need to do some assumptions. First, we know   (through Google) that the AT modem command for inputting the NCK key is AT+ZNCK="keyhere" for ZTE devices. So, let's look for "ZNCK" in the hex editor of dsp1.mbn and dsp2.mbn. (If you are not as lucky and don't know what the AT command is, I would put money that the command will contain the word NCK, so just search that). In dsp2.mbn, we find a couple of results. One of the results is in a group of other AT commands. Each command is next to a 4-byte hex value and a bunch of zero padding. I would guess that it is a jump table and the hex values are the memory locations of the functions to jump to. Doing a quick memory to file offset conversion (from our ELF program header), we locate the offset in our disassembly dump to find that it starts an "allocframe" instruction. That means we are at the beginning of a function so our assumptions must be right. Now, we can get to the crux of the problem, which is figuring out how the keycheck works.

**Mapping out the functions**

We now know where the function of interest starts, but we don't know where it ends. It's easy to find out though, look for a jump to lr (in this case for this processor, it's a instruction to jump r31). We start at the beginning of the function and we copy all the instruction until we see a non-conditional jump. We paste the data into another text file (for easier reference). Then we go to the next location in the disassembly (where it would have jumped to) and copy the instruction until we see another non-conditional jump, and then paste them into the second text file. Keep doing this until you see a jump to r31. We now have most of the function. Notice I kept saying "non-conditional" jumps. That's because first, we just need the code that ALWAYS runs, just to filter out stuff we don't need. Now, we should get the other branches just so we have more information. To do this, just follow each jump or function call in the same way as we did for the main function. I would also recommend writing some labels like "branch1" and "func1" for each jump just so you can easily locate two jumps to the same location and such. I would also recommend only doing this up to three "levels" max (three function calls or three jumps) because it could get real messy real quick, and we will need more information so we can filter out un-needed code, as I will detail in the next section.

**Finding data references**

Right now, we are almost completely blind. All we know is what code is run. We don't know the names of functions or what they do, and it would take forever to "map" every function and every function every function calls (and so on). So we need to obtain some information. The best would be to see what data the code is using. For this processor (and likely many others), a "global pointer" is used to refer to some constant data. So, look for references to "gp" in the disassembly. Searching from the very beginning of the program, we find that the global pointer is set to 0x3500000, and according to the ELF headers, that is a section of the dsp2.mbn file at some file offset. In the section we care about, look for references to "gp" and use the offsets you find to locate the data they refer to. I would recommend adding some comments about them in the code so we don't forget about them. Now, the global pointer isn't everything, we can have regular hard-coded pointers to constant areas of memory. Look for setting of registers to large numbers. These are likely parameters to function calls that are too big to be just numerical data and are more likely pointers. Use the ELF header to translate the memory locations to file offsets. In this case (for this processor), some values may be split into rS.h and rS.l, these are memory locations that are too "large" to be set in the register at once. Just convert rS.h into a 16 bit integer, rS.l into a 16 bit integer (both might require zero padding in front), then combine them into one 32 bit integer where rS.h's value is in front of rS.l's value. For example, we have: r1.h = #384; r1.l = #4624. That will make r1 == 0x1801210. You should also make some comments in the code about the data that is being used. Now, predict standard library calls. This may be the hardest step because it involves guessing and incorrect guessing may make other guess more wrong. You don't have much information to go by, but you know 1) the values of some of the data being passed into function calls, and 2) library calls will usually be near the start of the program, or at least very far away from the current function. This will be harder if the function you are trying to map is already near the beginning of the program. The function I'm mapping is found at 0xf84c54, and most function calls are close to it. When I see a function call to 0xb02760, I know that it might be a library call. 3) Some of the more "common" functions and the types of parameters they accept. You don't need to figure out all of the library calls, just enough to get an idea of what the code is doing so you don't try to map out these functions (trying to map out strcpy, for example will get messy real quick). For example, one function call, I see is taking in a data pointer from a "gp" offset, a string that contains "%s: %d", and some more data. I will assume it is calling fprintf(). I see another function is being called many times throughout the code, and it always accepts two pointers where the second one may be a constant and a number. I will assume it is calling memcpy().

**Translating**

This may be the most boring part. You should have enough information now to try to write a higher language code that does what the assembly code says. I would recommend doing this because it is much easier to see logic this way. I used C and started by doing a "literal" transcription using stuff like "r0-r31" as variable names and using goto. Then go back and try to simplify each section. In my process, I found that how the unlock key is checked is though sort of a hash function. It takes the user input, passes it through a huge algorithm of and/or/add/sub of more than 1000 lines and takes the result and compares it to a hard coded value in the NV ram (storage area for the device). Here, I made a choice to not go through and re-code this algorithm for two reasons. First, it would be of little use, as the key check doesn't use a known value like the IMEI and relies on a hard coded value in the NV ram that you need to extract (which a regular user might have trouble doing). Second, after decoding it, we would have to do the algorithm backwards to find the key from the "known value" in the NV ram (and it could be that it would be impossible to work backwards). So I took the easy way out and made a 4-byte patch in where I let the program compare the known value to itself instead of to the generated hash from the input and flashed it to the device. Then I inputted a random key, and the device was unlocked.

Now, remember at the beginning I said the code was unsigned? Because of that I could easily have reflashed the firmware with my "custom" code. However, if your device has some way of preventing modified code from running, you may have no choice but to decode the algorithm.
