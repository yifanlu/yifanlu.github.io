---
author: yifanlu
comments: true
date: 2011-11-27 23:45:01-06:00
layout: post
slug: reversing-the-xperia-play-emulator-part-deux
title: Reversing the Xperia Play emulator (part deux)
wordpress_id: 329
categories:
- Assembly
- Technical
- Xperia Play
---

The [last time we spoke](/2011/08/07/reverse-engineering-a-dynamic-library-on-the-xperia-play/), I managed to run any PSX game on the Xperia Play by redirecting some function calls. Well, since then Sony (you could say) fixed it (still don't know how, I should look into it one day, I'm guessing they revoked the certificates for Crash Bandicoot) and people running Android 2.3.4 on the Xperia Play can't use PSXPeria anymore. I've re-patched it a while ago, but never got the chance to modify the patching tool to use the new method (I really hate Java and don't want to use it, so I held back.) until today. As customary to my releases, I will begin by telling more than what you want to know about how it works.<!-- more -->

**Previously on "cracking the emulator"...**

If you haven't read the [last posts](/2011/08/01/analyzing-the-psx-emulator-on-the-xperia-play/) I've made about how I reverse engineered the emulator data format and binary, you may want to, but I'll summarize it in a few words. Basically, the emulator was separated into two binaries bin-one decrypts bin-two and bin-two asks bin-one to decrypt and load the game's table-of-contents which is used to load the game. The TOC is important because anyone can replace the game data files, but it won't load because the TOC contains addresses of the places to decompress in the game data. Well, after the hard part of reversing the formats and finding all this out, the actual patch was fairly easy. All we did was make a new library with the same function name as the one that is used by bin-two to query bin-one for the TOC, and use it to load the TOC for our custom game and make sure that library loads before Sony's and the rest is almost magic. We don't need to overwrite any function pointers or even touch the emulator because the linker looks for the first definition of a function and calls it.

**How Sony made our lives harder**

So version 1 is always easiest to break. This applies for almost everything. The PSP, the iPhone, the DS, etc. Version 2 is where it gets real. So what are the changes? First of all, no more bi-binary system. There is a single binary that does both the decrypting and emulating. Oh, and they removed the symbols so we can no longer search for "GetImageToc" and find where the function is. Also, they've started verifying that ISOs.

**Finding the needle**

Before we can begin to think about patching, we first need to find what to patch. As I've mentioned, Sony removed the symbols, so we no longer know what the function names are. We CAN try to map out the entire binary (10MB) and look for something that does what appears to be decrypting a TOC, but we don't have months or a team of assembly experts. What we DO have is the older version of the binary that has the symbols. Assuming that they didn't rewrite the emulator from scratch, the structure should be similar. We open up the old binary, find the function that calls the ones we want to patch, and look for identifying characteristics. What are they? Well, we look for mentions of unique strings and unique calls to standard functions (unique as in something like atoi, not malloc, which is called every other line). Luckily we have both. It seems like a few lines before the function we are interested in, the program does something with the string "/data/image.ps" and sometimes afterwards, uncompress is called. Now we have the address of the functions we want to patch.

**Patching the function**

Well, here's our second problem. What do we patch the function with? We are only limited to the length of what the function originally is, but I'm sure that's not a problem for experts. I'm not an expert though, so how about we steal what Sony did in version 1? We use dlsym to call the function from a loaded binary in memory. After a quick trip to an assembly reference, I wrote the following code: [https://github.com/yifanlu/PSXPeria-Wrapper/blob/master/jni/java-activity-patch.S](https://github.com/yifanlu/PSXPeria-Wrapper/blob/master/jni/java-activity-patch.S), I would go into more details, but I believe my comments on the code explains it better than I could. The only other thing we need is to manually define the address for "dlsym" and the offset for the name of the function. ARM assembly uses relative address, so I haven't come up with a quick way to do this yet. For now, I'm using a calculator and a piece of paper to find the address of dlsym relative to the patch in the program. Comment if you have a better way.

**Phase 2**

When the game didn't boot and was frozen on screen, I knew it had to be another obstacle. Our code had to have worked because otherwise, it would have crashed. Debugging with GDB, it seems like the program is blocking forever, seemingly on purpose. To double check, I loaded Crash Bandicoot again, but with my patched emulator and it worked. So, I guess there was a check somewhere that only loads Crash Bandicoot. Yes, I could go back into IDA and look for where the check is and NOP it out, but I was tired by then and my short attention span wants me to work on something else, so I took the easy way out and patched the PSX image with the titleid for Crash Bandicoot. As far as I know, this shouldn't affect anything in terms of compatibility, but farther tests are needed.

**Next week on "cracking the emulator"...**

Version 3 of the emulator is already out and is distributed in the PS-Suite games in the Japanese PSN store (on the Play). I already took a look at it, and the emulator did not seem to be updated, so I didn't try hard to patch it. However, it seems that they implemented many new security mechanisms in the PS-Suite PSX games. For starters, there is a public-private key exchange to make sure all the files in the APK are untouched, and I'm pretty sure the PS-Image is now encrypted or the format has changed. Now, Sony did not do all this to prevent us from loading our own games (or maybe they did). I suspect it's to prevent pirates from stealing the PSN games. Which means that if I crack the version 3 emulator, I may be helping piracy. This means, I will most likely not touch the PS-Suite emulators, and if I do, two things have to happen. 1) I need to be sure that the emulator has much better compatibility, and 2) I need a way to make sure that my tool isn't going to be used for piracy. So I guess this may be the last release for a while.

**Links**

[Project Page](/p/psxperia)
[Source](https://github.com/yifanlu/PSXperia)
[Downloads](https://github.com/yifanlu/PSXperia/downloads)
