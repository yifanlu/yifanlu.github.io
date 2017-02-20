---
author: yifanlu
comments: true
math: true
date: 2017-02-19
layout: post
slug: psvimgtools-decrypt-vita-backups
title: "psvimgtools: Decrypt Vita Backups"
categories:
- Vita
tags:
- vita
- hacking
- cryptography
- tools
- releases
---

The Vita's Content Manager allows you to backup and restore games, saves, and system settings. These backups are encrypted (but not signed!) using a [key derived in the F00D processor](https://wiki.henkaku.xyz/vita/PSVIMG). While researching into F00D, **xyz** and **Proxima** stumbled upon a neat trick proposed originally by **plutoo** that lets you obtain this secret key and that has inspired me to write a set of tools to manipulate CMA backups. The upshot is that with these tools, you can modify backups for any Vita system including 3.63 and likely all future firmware. This does not mean you can run homebrew, but does enable certain tricks like disabling the PSTV whitelist or swapping X/O buttons.

## Backup Keys

Because my friends who discovered this are pretty busy with other stuff at the time, I will attempt to document their findings here. The backup encryption process is documented in detail [on the wiki](https://wiki.henkaku.xyz/vita/PSVIMG), but the short version is that your AID (unique to a PSN account) is used to generate a key seed. This key seed is used by the F00D processor (the security coprocessor) to generate a AES256 key, which is passed directly to the hardware crypto device. The ARM (application) processor can access this crypto hardware but cannot read any keys out of it. This means that ARM can use the hardware as a black-box to encrypt backups without knowing the key. Of course you can try to brute force the key since you know both the plaintext and ciphertext thanks to the HENkaku kernel hack, but that would take $$2^{256}$$ time, which is physically impossible. However, since we can hack any Vita on 3.60, it is possible to use the Vita itself as a black box for extracting and modifying backups for other devices on unhackable firmwares, but since the process requires access to a hacked Vita, it is not very useful.

### One Weird Trick

But not all hope is lost! As I've said, the crypto hardware can be accessed by the ARM processor as well as the F00D processor. For certain other non-critical tasks, the ARM processor sets the key directly for the crypto hardware, so we know how the keys are set. There are a few dozen key slots that both processors can write to. The catch is that once the key is written, it cannot be read back.

Let's dive deeper into how keys are passed to the crypto hardware. Note that an AES256 key is 256-bits or 32 bytes wide. Since an ARMv7 processor can only write 4 bytes at a time (okay it _can_ do 8 bytes and also the bus width is usually optimized to be the size of a cache line, but for simplicity, we assume it can only write 4 bytes), a 32 byte key is sent with 8 write requests of 4 bytes. Now, the correct way for a crypto device to handle this is to provide a signaling mechanism to the host so it can indicate when a key slot write is about to occur. Then the host sends all parts of the key. Finally, the host indicates that the key transfer is complete and the crypto device locks the key in place and wipes it when another key transfer is requested for that slot. And for completeness, there should be measures in place to only allow one device to do a key transfer at a time in order to prevent races.

The incorrect way to do this is to naively allow anyone to set any part of the key at any time. Why? Because if we can set part of an unknown key to a known value, we can reduce the time to brute force the complete key dramatically. Let's say we have an unknown 256-bit key that is `22 22 22 22 44 44 44 44 66 66 66 66 88 88 88 88 AA AA AA AA CC CC CC CC EE EE EE EE 11 11 11 11`. Now say we can zero out the first 28 bytes of this key so the crypto engine uses `00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 11 11 11 11` where we still don't know the last 4 bytes.

But now, we pass in a chosen plaintext to the crypto device to do an AES256 operation and we get back the ciphertext. We can then brute force every possible key with the first 28 bytes to be zero. That's $$2^{32} = 4294967296$$ keys, which takes about a minute to compute with a single modern Intel core. We now know the last four bytes of the key and can repeat this procedure for the second to last four bytes and so on. This reduces the search time to $$8 \cdot 2^{32} = 2^{35}$$, which is not only possible but practical as well. Running this brute force optimized on a four core Intel CPU with hardware AES instructions takes about 300 seconds to find the full 256-bit key. In fact, xyz pointed out that you can even precompute all possible "mostly-zero" keys and the storage would "only" be half a TB.

As you might have guessed, the Vita does it the incorrect way, so anyone can retrieve their backup keys.

### psvimg-keyfind

I wrote a tool to do this brute force for you. It is not hyper-optimized but is portable and can find any key on a modern computer in about ten minutes. I have provided a Vita homebrew that generates the chosen ciphertexts on any HENkaku enabled Vita. These "partials", as I call it, can be passed to `psvimg-keyfind` to retrieve a backup key for any PSN AID. The AID is not console unique but is tied to your PSN account. This is the hex sequence you see in your CMA backup path. The idea is that if you have a non-hackable Vita, you can easily send your AID to a friend (or stranger) who can generate the partials for you. You can then use `psvimg-keyfind` to find your backup key and use it to modify settings on your non-hackable Vita. Huge thanks to Proxima for the reference implementation that this is based off of.

## Hacking Backups

What I did is completely reverse how CMA generates and parses the backup format. I have documented [extensively](https://github.com/yifanlu/psvimgtools/blob/master/psvimg.h) how these formats work. I also wrote tools to dump and repack CMA backups and all this works with backups generated from the latest firmware.

Hacking backups isn't as fun as having a hacked system. So, don't update from 3.60 if you have it! You cannot run unsigned code with this, so you are only limited to tricks that can be done on the registry, `app.db`, and other places. This includes:

* [Enabling almost any games to run on the PSTV](http://hackinformer.com/PlayStationGuide/PSV/tutorials/how_to_install_the_pstv_whitelist_patch_v2.html)
* [Swap X/O buttons for out-of-region consoles](http://hackinformer.com/PlayStationGuide/PSV/INFO_REGISTRY.html)
* [Run PSP homebrew with custom bubbles](http://hackinformer.com/PlayStationGuide/PSV/INFO_EPSP_BUBBLES.html)
* and maybe more as people make new discoveries

My hope is that other people will take my tools as building blocks for a user-friendly way of enabling some of the tricks above as currently the processes are pretty involved. This also increases the attack surface for people looking to find Vita exploits as parsing of files that users normally aren't allowed to modify are common weak points.

Additionally, because of how Sony implemented CMA backups and that the key-erase procedure is a hardware vulnerability, this is pretty much impossible to patch in future firmware updates. Unless Sony decides to break all compatibility with backups generated on all firmware up until the current firmware. And that would mean that any backup people made up until this theoretical update comes out would be unusable. Sony is known for pulling stunts like removing Linux from PS3, but I think this is beyond even what they would do.

### Release

I've built versions of this tool for Windows x64, Linux x64, and OSX [here](https://github.com/yifanlu/psvimgtools/releases). Please read the [usage notes](https://github.com/yifanlu/psvimgtools/blob/master/README.md).
