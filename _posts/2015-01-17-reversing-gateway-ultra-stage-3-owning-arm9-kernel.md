---
author: yifanlu
comments: true
date: 2015-01-17
layout: post
slug: reversing-gateway-ultra-stage-3-owning-arm9-kernel
title: 'Reversing Gateway Ultra Stage 3: Owning ARM9 Kernel'
wordpress_id: 1028
categories:
- 3DS
- Assembly
- C
- Technical
tags:
- 3ds
- arm
- arm11
- arm9
- exploit
- gateway
- ipc
- kernel
- memory
- reboot
- tocttou
---

First, some background: the 3DS has two main processors. [Last time](/2015/01/15/reversing-gateway-ultra-stage-2-owning-arm11-kernel/), I went over how Gateway Ultra exploited the ARM11 processor. However, most of the interesting (from a security perspective) functionalities are handled by a separate ARM946 processor. The ARM9 processor is in charge of the initial system bootup, some system services, and most importantly all the cryptographic functions such as encryption/decryption and signature/verification. In this post, we will look at how to run (privileged) code on the ARM9 processor with privileged access to the ARM11 processor. Please note that this writeup is a work in progress as I have not _completely_ figured out how the exploit works (only the main parts of it). Specifically there are a couple of things that I do not know if it is done for the sake of the exploit or if it is done purely for stability or obfuscation. From a developer's perspective, it doesn't matter because as long as you perform all the steps, you will achieve code execution. But from a hacker's perspective, the information is not complete unless all aspects are known and understood. I am posting this now as-is because I do not know when I'll have time to work on the 3DS again. However, when I do, I will update the post and hopefully clear up all confusion.<!-- more -->


## Code


For simplicity in description, from this point on, I will use pointers and offset values specific to the 4.x kernel. However, the code is the same for all firmware versions.

```c
void arm11_kernel_entry(void) // pointers specific to 4.x
{
  int (*sub_FFF748C4)(int, int, int, int) = 0xFFF748C4;

  __clrex(); // release any exclusive access
  memcpy(0xF3FFFF00, 0x08F01010, 0x1C);// copy GW specific data
  invalidate_dcache();
  invalidate_icache();
  clear_framebuffer(); // clear screen and saves some GPU registers
  // ARM9 code copied to FCRAM 0x23F00000
  memcpy(0xF3F00000, ARM9_PAYLOAD, ARM9_PAYLOAD_LEN);
  // write function hook at 0xFFFF0C80
  memcpy(0xEFFF4C80, jump_table, FUNC_LEN);
  // write FW specific offsets to copied code buffer
  *(int *)(0xEFFF4C80 + 0x60) = 0xFFFD0000; // PDN regs
  *(int *)(0xEFFF4C80 + 0x64) = 0xFFFD2000; // PXI regs
  *(int *)(0xEFFF4C80 + 0x68) = 0xFFF84DDC; // where to return to from hook
  // patch function 0xFFF84D90 to jump to our hook
  *(int *)(0xFFF84DD4 + 0) = 0xE51FF004; // ldr pc, [pc, #-4]
  *(int *)(0xFFF84DD4 + 4) = 0xFFFF0C80; // jump_table + 0
  // patch reboot start function to jump to our hook
  *(int *)(0xFFFF097C + 0) = 0xE51FF004; // ldr pc, [pc, #-4]
  *(int *)(0xFFFF097C + 4) = 0x1FFF4C84; // jump_table + 4
  invalidate_dcache();
  sub_FFF748C4(0, 0, 2, 0); // trigger reboot
}

// not called directly, offset determines jump
void jump_table(void)
{
  func_patch_hook();
  reboot_func();
}

void func_patch_hook(void)
{
  // data written from entry
  int pdn_regs;
  int pxi_regs;
  int (*func_hook_return)(void);

  // save context
  __asm__ ("stmfd sp!, {r0-r12,lr}")
  // TODO: Why is this needed?
  pxi_send(pxi_regs, 0);
  pxi_sync(pxi_regs);
  pxi_send(pxi_regs, 0x10000);
  pxi_recv(pxi_regs);
  pxi_recv(pxi_regs);
  pxi_recv(pxi_regs);
  // TODO: What does this do?
  *(char *)(pdn_regs + 0x230) = 2;
  for (i = 0; i &lt; 16; i += 2); // busy spin
  *(char *)(pdn_regs + 0x230) = 0;
  for (i = 0; i &lt; 16; i += 2); // busy spin
  // restore context and run the two instructions that were replaced
  __asm__ ("ldmfd sp!, {r0-r12,lr}\t\n"
           "ldr r0, =0x44836\t\n"
           "str r0, [r1]\t\n"
           "ldr pc, %0", func_hook_return);
}

// this is a patched version of function 0xFFFF097C
// stuff found in the original code are skipped
void reboot_func(void)
{
  ... // setup
  // disable all interrupts
  __asm__ ("mrs r0, cpsr\t\n"
           "orr r0, r0, #0x1C0\t\n"
           "msr cpsr_cx, r0" ::: "r0");
  while ( *(char *)0x10140000 &amp; 1 ); // wait for powerup ready
  *(void **)0x2400000C = 0x23F00000; // our ARM9 payload
  ...
}
```


### Memory Configurations


A quick side-note on the way that ARM11 talks to ARM9. There is a FIFO with a register interface called the [PXI](http://3dbrew.org/wiki/PXI) and is used to pass data to and from each processor. Additionally, most of the physical [memory mappings](http://3dbrew.org/wiki/Memory_layout) are shared between the two processors. Data stored, for example, in the FCRAM or AXI WRAM can be seen by both processors (provided proper cache coherency). However, there is one region (physical 0x08000000 to 0x081000000) that only the ARM9 processor can see. ARM9 code runs in this region. Another thing to note is that the ARM9 processor only performs a one-to-one virtual memory addressing (aka physical addresses and virtual addresses are the same) but I have been told that it does have memory protection enabled.


### ARM9 Process


The ARM9 processor only (ever) has one process running, Process9, which speaks with the kernel to handle commands from ARM11. Process9 has access to a special [syscall](http://3dbrew.org/wiki/SVC) 0x7B, which takes in a function pointer and executes it in kernel mode. This means that essentially, owning ARM9 usermode is enough to get kernel code execution without any additional exploits.


## Exploit Setup


After doing some housekeeping, the first thing the second stage payload code does is copy the third stage ARM9 code to a known location in FCRAM. Next, it makes patches to two ARM11 kernel functions. First, it patches the function at 0xFFF84D90 (I believe this function performs the kernel reboot) to jump into a function hook early-on. Second, it patches the function at 0xFFFF097C (I believe this function is ran after the ARM11 processor resets) to jump into another function hook. These two hooks are the key to how the exploit works.


## Soft Rebooting


The 3DS supports soft rebooting (resetting the processor state without clearing the memory) in order to switch modes (ex: for DS games) and presumably to enable entering and exiting sleep mode. I believe this is triggered at the end of the the exploit setup by calling the function at 0xFFF748C4. At some point in this function, the subroutine at 0xFFF84D90 is called, which runs the code in our first function hook before continuing the execution.

At the same time in the ARM9 processor, Process9 now waits for a special command, 0x44836 from PXI, in the function at 0x0807B97C. I believe that the first function hook in ARM11 sends a series to commands to put Process9 into function 0x0807B97C, however that is only a guess.

The ARM11 processor continues to talk with ARM9 through the PXI and at some point both agree on a shared buffer in FCRAM at 0x24000000 (EDIT: yellows8 says this is the [FIRM header](http://3dbrew.org/wiki/FIRM)) where some information is stored. At 0x2400000C is a function pointer to what ARM9 should execute after the reset. Process9 verifies that this function pointer is in the ARM9 private memory region 0x08000000-0x08100000 (EDIT: I assume the FIRM header signature check also takes place at this point). ARM11 resets and spinlocks in the function at 0xFFFF097C to wait for ARM9 to finish its tasks and tell ARM11 what to do.

Process9 at this point uses SVC 0x7B to jump into some reset handler at 0x080FF600 in kernel mode. At the end of that function, the ARM9 kernel reads the pointer value at 0x2400000C and jumps to it.


## Reset ToCTToU


The problem here is simple. Process9 checks that the data at 0x2400000C (which is FCRAM, shared by both processors) is a valid pointer to code in ARM9 private memory (that ARM11 cannot access). However, after the check passes and before the function pointer is used, ARM11 can overwrite the value to point to code in FCRAM and ARM9 will execute it when it resets. This time-of-check-to-time-of-use bug is made possible by patching the ARM11 function that runs after reset so that it can wait for the right signal and then quickly overwrite the data in FCRAM before ARM9 uses it.


## Conclusions


I apologize for the vagueness and likely mistakes in parts. I hope that if I don't have the time to finish this analysis, someone else can pick up where I left off. Specifically, there are a couple of main questions that I haven't answered:



	
  1. What is the function at 0xFFF748C4, what do the arguments do, and how does it call into function 0xFFF84D90? I speculate that it's a function that performs the reset, but a more precise description is needed.

	
  2. What is the purpose of the first function hook? Specifically why does it send 0 and 0x10000 through PXI and what does PDN register 0x230 do?

	
  3. How does Process9 enter function 0x0807B97C? I suspect that it may have something to do with the first function hook in ARM11.


I hope that either someone can answer these questions (as well as correct any mistakes I've made) or that I'll have time in the future to continue this analysis. This will also be the end of my journey to reverse Gateway Ultra (but the next release may spark my interest again). I don't particularly care about the later stages (I hear there's a modified MIPS VM and timing based obfuscation) or how Gateway enforces DRM to make sure only their card is used. If I do any more reversing with the 3DS, it would be on the kernel and applications so I can make patches of my own instead of worrying about how Gateway does it.

At this point, the information should be enough for anyone to take complete control of the 3DS (<= 9.2.0). I believe that information on its own is amoral but it takes people to make it immoral. There's no point in arguing if piracy is right or wrong or if making this information public would help or harm pirates. I am not here to ensure the 3DS thrives. I am not here to take business away from Gateway. I am not here to be a moral police. I am only here to make sure that information is available for those who thirst for knowledge as much as I do in a form that is as precise and accurate as I can make it.
