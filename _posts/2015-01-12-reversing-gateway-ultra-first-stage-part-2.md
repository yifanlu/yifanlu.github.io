---
author: yifanlu
comments: true
date: 2015-01-12
layout: post
slug: reversing-gateway-ultra-first-stage-part-2
title: Reversing Gateway Ultra First Stage (Part 2)
wordpress_id: 1007
categories:
- 3DS
- Assembly
- Technical
tags:
- 3ds
- arm
- dma
- gateway
- gpu
- payload
- physical
- rop
---

When we [last left off](/2015/01/10/reversing-gateway-ultra-first-stage-part-1/), we looked at the ROP code that loaded a larger second-part of the payload. Now we will walk through what was loaded and how userland native code execution was achieved. I am still an amateur at 3DS hacking so I am sure to get some things wrong, so please post any corrections you have in the comments and I will update the post as needed.<!-- more -->


#### Pseudocode


Some of the hard coded addresses are inside the stack payload loaded by the first part from Launcher.dat (at 0x08F01000).

```c
int GX_SetTextureCopy(void *input_buffer, void *output_buffer, unsigned int size, 
int in_x, int in_y, int out_x, int out_y, int flags);
int GSPGPU_FlushDataCache(void *addr, unsigned int len);
int svcSleepThread(unsigned long long nanoseconds);
void memcpy(void *dst, const void *src, unsigned int len);

// There are offsets and addresses specific to each FW version inside of 
// the first stage that is used by both the first and second stage payloads
struct // example for 4.1.0
{
    void (*payload_code)(void); // 0x009D2000
    unsigned int unk_4; // 0x252D3000
    unsigned int orig_code; // 0x1E5F8FFD
    void *payload_target; // 0x192D3000
    unsigned int unk_10; // 0xEFF83C97
    unsigned int unk_14; // 0xF0000000
    unsigned int unk_18; // 0xE8000000
    unsigned int unk_1C; // 0xEFFF4C80
    unsigned int unk_20; // 0xEFFE4DD4
    unsigned int unk_24; // 0xFFF84DDC
    unsigned int unk_28; // 0xFFF748C4
    unsigned int unk_2C; // 0xEFFF497C
    unsigned int unk_30; // 0x1FFF4C84
    unsigned int unk_34; // 0xFFFD0000
    unsigned int unk_38; // 0xFFFD2000
    unsigned int unk_3C; // 0xFFFD4000
    unsigned int unk_40; // 0xFFFCE000
} fw_specific_data;

void payload() // base at 0x08F01000
{
    int i;
    unsigned int kversion;
    struct fw_specific_data *data;
    int code_not_copied;

    // part 1, some setup
    *(int*)0x08000838 = 0x08F02B3C;
    svcSleepThread (0x400000LL);
    svcSleepThread (0x400000LL);
    svcSleepThread (0x400000LL);
    for (i = 0; i < 3; i++) // do 3 times to be safe
    {
        GSPGPU_FlushDataCache (0x18000000, 0x00038400);
        GX_SetTextureCopy (0x18000000, 0x1F48F000, 0x00038400, 0, 0, 0, 0, 8);
        svcSleepThread (0x400000LL);
        GSPGPU_FlushDataCache (0x18000000, 0x00038400);
        GX_SetTextureCopy (0x18000000, 0x1F4C7800, 0x00038400, 0, 0, 0, 0, 8);
        svcSleepThread (0x400000LL);
    }

    kversion = *(unsigned int *)0x1FF80000; // KERNEL_VERSION register
    data = 0x08F02894; // buffer to store FW specific data

    // part 2, get kernel specific data from our buffer
    if (kversion == 0x02220000) // 2.34-0 4.1.0
    {
        memcpy (data, 0x08F028D8, 0x44);
    }
    else if (kversion == 0x02230600) // 2.35-6 5.0.0
    {
        memcpy (data, 0x08F0291C, 0x44);
    }
    else if (kversion == 0x02240000) // 2.36-0 5.1.0
    {
        memcpy (data, 0x08F02960, 0x44);
    }
    else if (kversion == 0x02250000) // 2.37-0 6.0.0
    {
        memcpy (data, 0x08F029A4, 0x44);
    }
    else if (kversion == 0x02260000) // 2.38-0 6.1.0
    {
        memcpy (data, 0x08F029E8, 0x44);
    }
    else if (kversion == 0x02270400) // 2.39-4 7.0.0
    {
        memcpy (data, 0x08F02A2C, 0x44);
    }
    else if (kversion == 0x02280000) // 2.40-0 7.2.0
    {
        memcpy (data, 0x08F02A70, 0x44);
    }
    else if (kversion == 0x022C0600) // 2.44-6 8.0.0
    {
        memcpy (data, 0x08F02AB4, 0x44);
    }

    // part 3, execute code
    do
    {
        // if the function has it's original code, we try again
        code_not_copied = *(unsigned int *)data->payload_code + data->orig_code == 0;
        // copy second stage to FCRAM
        memcpy (0x18410000, 0x08F02B90, 0x000021F0);
        // make sure data is written and cache flushed || attempted GW obfuscation
        memcpy (0x18410000, 0x18410000, 0x00010000);
        memcpy (0x18410000, 0x18410000, 0x00010000);
        GSPGPU_FlushDataCache (0x18410000, 0x000021F0);
        // copy the second stage code
        GX_SetTextureCopy (0x18410000, data-&gt;payload_target, 0x000021F0, 0, 0, 0, 0, 8);
        svcSleepThread (0x400000LL);
        memcpy (0x18410000, 0x18410000, 0x00010000);
    } while (code_not_copied);

    (void(*)() 0x009D2000)();
    // I think it was originally data-&gt;payload_code but later they hard coded it 
    // for some reason
}
```


#### Details


The first part, I'm not too sure about. I think it's either some required housekeeping or needless calls to obfuscate the exploit (found later). I couldn't find any documentation on the 0x1F4XXXXX region except that is it in the [VRAM](http://3dbrew.org/wiki/Memory_layout#ARM11_User-land_memory_regions). (EDIT: plutoo tells me it's the framebuffer. Likely the screen is cleared black for debugging or something.) I am also unsure of the use of setting 0x08000838 to some location in the payload that is filled with "0x002CAFE4". In the second part, version specific information for each released kernel version is copied to a global space for use by both the first stage and the second stage exploit code. (This includes specific kernel addresses and stuff).

The meat of the exploit is an unchecked GPU DMA write that allows the attacker to overwrite read-only executable pages in memory. This is the same exploit used by smealum in his ninjhax and he gives a much better explanation of "gspwn" [in his blog](http://smealum.net/?p=517). In short, certain areas of the physical memory are mapped at some virtual address as read-only executable (EDIT: yellows8 tells me specifically, this is in a CRO, which is something like shared libraries for 3DS) but when the physical address of the same location is written to by the GPU, it does not go through the CPU's MMU (since it is a different device) and can write to it. The need for thread sleep (and maybe the weird useless memcpys) is because the CPU's various levels of cache needs some time to see the changes that it did not expect from the GPU.

The second stage of the payload is the ARM code copied from Launcher.dat (3.0.0) offset 0x1B90 for a length of 0x21F0 (remember to decrypt it using the "add"-pad stream cipher described in the first post).


#### Raw ROP Payload Annotated


It is a huge mess, but for those who are curious, [here it is](http://pastebin.com/ym6jmb4K). The bulk of the code are useless obfuscation (for example, it would pop 9 registers full of junk data and then fill the same 9 registers with more junk data afterwards). However, the obfuscation is easy to get past if you just ignore everything except gadgets that do 1) memory loads, 2) memory stores, 3) set flags, or 4) function call. Every other gadget is useless. They also do this weird thing where they "memcpy" one part of the stack to another part (which goes past the current SP). However, comparing the two blocks of data (before and after the copy) shows nothing different aside from some garbage values.
