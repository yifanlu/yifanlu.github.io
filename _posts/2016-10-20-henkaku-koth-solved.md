---
author: yifanlu
comments: true
date: 2016-10-20
layout: post
slug: henkaku-koth-solved
title: HENkaku KOTH Solved
categories:
- Vita
tags:
- vita
- henkaku
- jailbreak
- koth
- rop
- kernel
---

When HENkaku was first released, we posed to the community the [KOTH challenge]({% post_url 2016-08-05-henkaku-koth-challenge %}) to get more hackers interested in the Vita. This week, two individuals have separately completed the challenge and are the new kings of Vita hacking! [Mike H.](https://hexkyz.blogspot.com/2016/10/henkaku-exploit-teardown-stage-3.html) and [st4rk](http://st4rk.net/2016/10/21/henkaku-ps-vita-ctf-the-end/) both proved that they have the final encryption key, showing that they solved the kernel ROP chain. I highly recommend reading their respective posts as they give some great insight into how hacking works. I also know of a third group who might have also completed the challenge but wishes to keep quiet for now. Congratuations to them too!

The Prize
---------
All participants have been given the prize for solving the challenge and in a short time, everyone will get a peek too. Molecule has gotten quite lazy since the release of HENkaku and since we underestimated the amount of time it would take for the challenge to be completed, we are only midway through polishing up the source code for release. The participants and I have agreed to not release anything until the end of the month. As a bonus for waiting, the source will not be for HENkaku as you know it today--it will be for the major update we have been working on. Stay tuned for more details! In the meantime, it would be fun to see if anyone can run their own kernel payload with all the information out today--it should be possible!

HENkaku Kernel ROP
------------------
The rest of this post is dedicated to my own explanation in creating the ROP chain for the challenge. I believe it is the most complex ROP chain ever written (although I haven't seen too many ROP chains that does work beyond copying code and running it). Enjoy!

## Introduction

First we'll define a security model for our system. We assume that code in kernel memory is "secure" and our main asset (what we are trying to protect) is kernel code. It is important to note that we are NOT trying to protect the kernel exploit. In fact, we assume the kernel vulnerability, userland exploit, and all userland ROP code is fully understood by the adversary. This is because, even with obfuscation, it is only a matter of time before one can figure out the vulnerability. However, we observe that knowing the vulnerability is useless without a method of exploiting it. Since our vulnerability allows us to control code flow but does not defeat data execution protection, it is useless to an adversary who do not possess kernel code.

This also means that Sony is not an adversary that our model defends against. Since Sony has all the code and likely debug units, it would not be feasible to write a ROP chain that can be obfuscated against Sony. The key idea is this: we only need to protect against reading out of kernel code. That means without either a clever way of figuring out our ROP chain or a separate kernel read exploit, the adversary cannot make use of our vulnerability.

## Security Model

We will actually consider two security models: the weak model assumes that the adversary does not know any of the gadgets used in our ROP chain and the strong model assumes that the adversary knows all of the gadgets used in our ROP chain (and nothing else, or they can trivially just write their own ROP chain). The strong model is the more interesting case. We are trying to secure kernel code from prying eyes but our ROP chain actually leaks a lot of information about kernel code. We know, for example, a lower bound on the size of the code. We know there are some regions that are Thumb code (LSB of the addresses). From the (in-)frequency and distribution of gadgets, we can guess what are function calls and what are helper gadgets. If we identify gadgets that are function calls, we can also guess at the number of arguments they take and any constants that are passed as arguments. The list goes on and on. So our strong security assumption takes the worst case: the adversary knows exactly what each of the gadgets does.

### Weak Assumption

To protect against the weak security assumption, we did two things. First we obfuscated any useful constants. A keen observer may see `256` and guess that AES-256 encryption is used. Or, if they are knowledgeable at Vita development, they may see `0x1020D006` and wonder if it is a memory type passed to `sceKernelAllocMemBlock`. That's why we hid most constants inside gadgets. The `256`, for example, becomes

```
  .word BASE+0x000232eb @ movs r0, #8 @ bx lr
  ...
  .word BASE+0x0001b571 @ lsls r2, r0, #5 @ bx lr
```

and the "size" parameter for the memory allocation becomes

```
  .word BASE+0x00001e43 @ and r2, r2, #0xf0000 ...
```

where r2 was used for something else earlier in execution. It was harder to craft `0x1020D006` so we had to settle with

```
  .word BASE+0x00000031 @ pop {r0, pc}
  .word      0x08106803 @ r0 = 0x8106803
  .word BASE+0x0001eff1 @ lsls r0, r0, #1 ...
```

because at the end of the day, this will not protect against smarter adversaries and is only meant to slow down analysis. There are some cases where we get obfuscation for free just because of the trickiness of writing ROP chains:

```
  .word BASE+0x0001f2b1 @ r5 = eor sb, r0, #0x40 ...
```

This was the only way to move a value from R0 to SB (which we want for storage because it is callee saved). We store the counter for the decrypt loop in SB so our constant for the payload size (used in a compare) is XORed with 0x40.

```
  .word (ENC_PAYLOAD_SIZE ^ 0x40) @ r4 = (payload size) ^ 0x40
  .word BASE+0x00022a49 @ subs r0, r0, r4 @ pop {r4, pc}
  .word      0xDEADBEEF @ r4 = dummy
  .word BASE+0x00003d73 @ ite ne @ movne r0, r3 @ moveq r0, #0 @ bx lr
```

The second thing we did was to use the dummy data for obfuscation. At times we find the need for gadgets such as 

```
  .word BASE+0x00000ce3 @ pop {r4, r5, r6, r7, pc}
  .word      0xDEADBEEF @ r4 = dummy
  .word      0xDEADBEEF @ r5 = dummy
  .word      0xDEADBEEF @ r6 = dummy
  .word BASE+0x0000587f @ r7 = movs r2, r0 @ pop {r4, pc}
```

in order to set register R7. In fact most gadgets ends up popping data into registers we don't care about. This is one source of difficulty in writing ROP chains: we need gadgets that don't mangle registers we DO care about. About half the data in our ROP chain is junk and we can take advantage of that. If we write the address of gadgets into these junk fields, then the adversary must differentiate between gadgets and junk. We make this especially hard by training a Markov chain to generate junk data. This means the distribution of gadgets is roughly the same before and after obfuscation and since we consider bigrams, the probability that one gadget is used after another is about the same for junk fields. We do this because the adversary may use statistical heuristics to deobfuscate the ROP chain.

In the end, our obfuscation can be defeated by a brute force attack to find the junk data. You can take one field at a time and try to change it to a random value and see if the chain still executes successfully (this may be repeated for more confidence). Since there are only about 200 WORDs of data, this should be feasible (although a bit painful).

### Strong Assumption

The strong security assumption poses a much more difficult problem. We need to satisfy both the following

  1. The ROP chain must be _useful_ and therefore must eventually execute ARM code.
  2. It should be _non-malleable_ so our chain cannot be taken apart and pieced together by the adversary to break our security model. This is especially difficult because by construction, ROP is malleable. Our goal is to only use a subset of gadgets that don't have universal fit. We will call a gadget _non-degenerate_ if its usefulness is dependent on its placement in the chain.

Here is an outline of what the payload has to do in order to be useful: first allocate a block of kernel RW memory. Then we have to get the address to that block (the Vita always requires the user to do this manually). Next, we have to set up the AES engine. Then we need to decrypt our payload using the AES engine in blocks. Finally, we have to remap the memory as RX and jump to it.

We have to protect against the obvious attacks such as removing the decryption step or changing the encryption key. There are also less obvious attacks such as changing the block for the base address or to be remapped. In the next few sections, we will describe each step of the payload and the tricks used in detail.

## Design

The main design decision is to not use any LDR/STR gadgets (other than with SP as the source). This is because of the spirit of goal #2, we do not want to make it easy to reuse the gadgets. LDR/STR is likely to be degenerate. If the adversary is able to get an arbitrary read from kernel memory, it is game over. This filters out a large chunk of potential gadgets we can use. We also do not want to limit the size of the second stage payload (therefore introducing a third stage) because each loader adds to the attack surface. This means we need to have a loop in ROP. This is not easy because we need to conditionally manipulate the stack pointer and also keep variables in harder to use registers such as R8 or LR. From experience, the higher the register, the rarer the gadget (except for R12). You will find that most gadgets operate with R0-R3 and R12 because those are used as scratch registers and for parameter passing in the ABI. R4-R6 are also more common because register allocation in GCC starts at lower registers. This is a double edged sword: if we use higher registers for saving loop variables, then we are less likely to limit the gadgets we can use inside the loop. However, it also means that sometimes we have to get creative to move data around these registers (for example, abusing a EOR instruction by calling it twice on the SB register).

### Allocate Memory

First, we need to call `sceKernelAllocMemBlockForKernel(name = "Magic", type = 0x1020D006, size = 0xA0000, opt = NULL)`. This is fairly straightforward. The name is arbitrary so we just chose some string in memory. The type has to be `0x1020D006` (DRAM, cachable, kernel RW, user NA) and we described the obfuscation trick above. The size just has to be large enough to hold our payload and that particular value is due to the other obfuscation trick described above.

### Get Base Address

`sceKernelGetMemBlockBaseForDriver(id, base)` places the base address in `*base`. The id is the return value from above. The easy way of doing this is to set base to a temporary buffer and then LDR it later. However, this would expose a LDR gadget. Instead we use

```
  .word BASE+0x00019713 @ add r3, sp, #0x28 ...
  .word BASE+0x00001e1d @ mov r0, r3 ...
  .word BASE+0x0001efe1 @ movs r1, r0 ...
```

to put the return value right into the stack. Then immediately after the function call we can retrieve it

```
  .word BASE+0x00001f17 @ sceKernelGetMemBlockBaseForDriver(r0 = id, r1 = base) ...
  ...
  .word BASE+0x00000031 @ pop {r0, pc}
  .word      0xDEADBEEF @ r0 = base address (written to from above)
```

Finally, we save the base address to R7 and note that this prevents us from using any gadgets that touches R7 in the future (if we really have to though, we can always move the data around but that would take work).

### Initialize AES Engine

To setup the AES engine, we need to call `aes_init(ctx = buf, blksize = 128, keysize = 256, key = secret_buf)`. The trick to obfuscate the constant 256 was described above. We will highlight a couple of other tricks. First, we need to make sure the ctx buffer (which contains the expanded key) is not revealed to the user. This is pretty simple: we just place the buffer into the memory block we just allocated. That memory block is not accessible in user mode and our security assumption is that kernel memory is protected. We also store the ctx buffer into R6 for future use. This prevents putting the pointer to sensitive information into the user-modifiable (from the exploit) stack. However, this also makes things harder as from this point onwards, we can no longer use gadgets that corrupt R6. Finally, we use the following non-degenerate gadget to set up the key pointer

```
  .word BASE+0x0001fdc5 @ mov r3, lr ...
```

The return pointer was set from a previous gadget

```
  .word BASE+0x000050e9 @ mov r0, r7 @ blx r3
```

and this intricately links the previous section for getting the memory base to setting the key here. Also note that since the key is in kernel code, as long as the kernel code is safe, our payload code will also be protected in our security model. In practice though, since we are using AES-ECB, we are vulnerable to replay attacks but more on that later...

### Decrypt Loop

The loop was tricky to implement. We have a counter in SB that is incremented in each iteration. To keep things simple we also increment it in the first iteration which puts our payload at offset 0x10. Thankfully this still works as 0x00000000 is a NOP in 32-bit ARM so we can slide into our payload. The loop logic involves conditionally subtracting from the stack pointer. To do this, we first save the "right" stack pointer into R4

```
  .word BASE+0x0001d9eb @ add r2, sp, #0xbc ..,
.Lsp_offset_start:
  ...
  .word BASE+0x00000853 @ pop {r0, r1, pc}
  .word      0xDEADBEEF @ r0 = dummy
  .word (0xbc - (.Lloop_end - .Lsp_offset_start)) @ r1 = 0xbc-sizeof(loop)
  .word BASE+0x000000ab @ subs r2, r2, r1 ...
  ...
  .word BASE+0x0002328b @ movs r1, r2 ...
  ...
  .word BASE+0x000000d1 @ movs r4, r1 ...
```

This puts R4 at ".Lloop_end" which is exactly the value of SP if the loop condition is not met.

```
  .word BASE+0x0001bf1f @ movs r2, r4 ...
  ...
  .word (-(.Lloop_end-decrypt_loop_start)) @ r3 = offset to start of loop
  .word BASE+0x0000039b @ pop {r4, pc}
  .word (ENC_PAYLOAD_SIZE ^ 0x40) @ r4 = (payload size) ^ 0x40
  .word BASE+0x00022a49 @ subs r0, r0, r4 ...
  ...
  .word BASE+0x00003d73 @ ite ne @ movne r0, r3 @ moveq r0, #0 ...
  ...
  @ add either 0 or offset to loop start to r2 (sp at loop end)
  .word BASE+0x000021fd @ add r0, r2 ...
  ...
  .word BASE+0x00000ae1 @ movs r1, r0 ...
  ...
  .word BASE+0x0002a117 @ pop {r2, r5, pc}
  .word BASE+0x00000347 @ r2 = pop {pc}
  .word BASE+0x0001f2b1 @ r5 = ...
  .word BASE+0x00000067 @ mov sp, r1 @ blx r2
.Lloop_end:
```

If the condition is met, then SP is set back to the start of the loop. Cool, right!

### Decrypt Payload

The actual decryption code was perhaps the hardest to write. In theory, it is simple enough: `aes_decrypt(ctx = r6, src = user_buf+counter, r2 = r7+counter)`. However, the difficulty comes from the fact that we cannot use R6, R7, SB and therefore we must let `aes_decrypt` save the callee saved arguments into the stack. However, the PUSH instruction will corrupt our ROP chain. The first attempt was to just leave a chunk of space in the chain (using ADD SP) before calling `aes_decrypt`. That doesn't work however, as LR is pushed to where the gadget to call `aes_decrypt` used to be (breaking the next iteration). Since the gadget to call `aes_decrypt` will be corrupted no matter what, the only way around it is to rewrite that gadget into the chain in each iteration.

We did not have any usable gadgets of the form `STR SP, [Rs, #-IMM]` so the store must happen before the call. The only gadgets of the form `STR SP, [Rs, #IMM]` had IMM at most 0x1C so the write gadget must be very close to the `aes_decrypt` call. However, this brings us back to the original problem that `aes_decrypt` corrupts the 0x18 bytes of the stack above it. To make matters even worse, we also need to set R1 and R2 before making the function call (the arguments). It is impossible to find a gadget that both writes to the stack and also sets R1 and R2 to the right value, so we must setup R1 and R2 beforehand. This means our gadget restoring gadget must also not touch R1 and R2. After hours of searching, the perfect gadget was found

```
  str r5, [sp, #0xc]
  ldr r5, [sp, #0x38]
  str r5, [sp, #0x10]
  blx r4
  add sp, #0x1c
  pop {r4, r5, pc}
```

This gadget is highly non-degenerate. It is almost tailor made for our specific purpose. To prevent confusion (there will be confusion), we will now refer to this as the "magical gadget." Here's how we used it:

```
  .word BASE+0x00001411 @ pop {r4, r5, pc}
  .word BASE+0x00000347 @ r4 = pop {pc}
  .word BASE+0x000209d7 @ r5 = str r5, [sp, #0x10] @ blx r4 @ add sp, #0x1c @ pop {r4, r5, pc}
  .word BASE+0x000209d3 @ str r5, [sp, #0xc] @ ldr r5, [sp, #0x38] @ str r5, [sp, #0x10] @ blx r4
  .word BASE+0x00001411 @ pop {r4, r5, pc}
  .word BASE+0x00000347 @ r4 = pop {pc}
  .word BASE+0x0001baf5 @ r5 = 0xD8678061_aes_decrypt

  @ BEGIN region overwritten by decrypt
  .word      0xDEADBEEF @ becomes str r5, [sp, #0x10] @ blx r4 @ add sp, #0x1c @ pop {r4, r5, pc}
  @ lr = add sp, #0x1c @ pop {r4, r5, pc}
  .word      0xDEADBEEF @ becomes add sp, #0xc @ pop {pc}
  .word      0xDEADBEEF @ dummy
  .word      0xDEADBEEF @ dummy
  .word      0xDEADBEEF @ dummy
  .word      0xDEADBEEF @ becomes 0xD8678061_aes_decrypt(r0 = ctx, r1 = src, r2 = dst) @ bx lr
  @ END region overwritten by decrypt

  .word      0xDEADBEEF @ dummy
  .word      0xDEADBEEF @ dummy
  .word      0xDEADBEEF @ dummy
  .word      0xDEADBEEF @ dummy
  .word      0xDEADBEEF @ dummy
  .word BASE+0x0000652b @ loaded by above: add sp, #0xc @ pop {pc}
  .word      0xDEADBEEF @ dummy
  .word      0xDEADBEEF @ r4 = dummy
  .word      0xDEADBEEF @ r5 = dummy
```

The short of it is that we first write two helper gadgets to the region destroyed by `aes_decrypt`. Those two gadgets will restore the gadget for making the `aes_decrypt` call and then actually call it.

Lets step through this line by line. First we setup R4 and R5. R5 is the first restoring gadget (the magical gadget) and SP+0x38 (below the volatile region) is the second restoring gadget. We then invoke the magical gadget for the first time and write the two restoring gadgets. It then jumps to R4 which we have defined as a simple no-op (pop the next gadget). The next gadget sets up R4 and R5 again for the next phase. The first restoring gadget (the magical gadget) is called to write R5 (now the `aes_decrypt` gadget) to the right location. Then the next one skips the dummy data and executes `aes_decrypt`.

`aes_decrypt` saves the LR value and returns to it at the end. Where is LR? Inside the magical gadget of course (thanks to the `BLX R4`). That means we run

```
  add sp, #0x1c
  pop {r4, r5, pc}
```

which hands control back to the ROP chain. Note that we used this one gadget three different ways here! Crazy!

### Remapping Executable

The final step is to remap the memory region as executable. It is very straightforward. We first use `sceKernelFindMemBlockByAddrForDriver(base = r7, 0)` to retrieve the block id. Then we call `remap_memory(blkid = r0, type = 0x1020D005)`. Note the type is now kernel RX user NA. For our final trick, we obfuscate the type parameter by making it appear the same as the first type

```
  .word      0x08106803 @ r1 = 0x8106803
  .word BASE+0x000233d3 @ lsls r2, r1, #1 ...
  ...
  .word BASE+0x00000433 @ subs r1, r2, #1 ...
```

Then we jump into the executable

```
  .word BASE+0x00011c5f @ blx r7
```

and we're done! Note that we do not perform any authentication on the binary payload. This design decision was made for two reasons: 1) if we introduce more gadgets, we increase the amount of data leaked and 2) the authentication may be vulnerable to an oracle attack since it will likely be very simple. This means, however that we are vulnerable to replay attacks (moving and changing the encrypted blocks around) and we allow the adversary to jump into random code. We believe that either attacks will be very difficult to exploit. Since the block size is large and the payload is small, the attacker does not have many blocks to work with. Jumping into random code will, with high probability, just trigger undefined instruction exception before doing anything useful.

## Breaking the Chain

We will now give one possible solution for solving the challenge _without_ a memory leak (but with lots of luck and intuition). First break the junk data obfuscation with the method described in that section. Working backwards, we wish to redirect the `aes_decrypt` gadget to "decrypt" kernel memory with a known key into the stack buffer (that we can leak with the `sceIoDevctl` exploit). To do that, we have to find the `aes_decrypt` gadget and then figure out the arguments. We know the source argument so we need to find the destination argument (which must be close-by in the chain). There are a lot of different ways of going about this. For example: attempt to modify gadgets addresses with +/-1 or +/-4 in hopes of hitting the counter increment gadget. Unfortunately this will not work here because the +0x10 is done directly by a gadget without any arguments. Eventually we might attempt a timing attack using another processor reading kernel stack while the ROP chain runs. We will then find that at some point, the base address of the decrypt buffer will be placed in the stack. We can now race to change the pointer to point into the stack instead and then have the payload decrypted to kernel stack. However, timing is critical because the remap gadget will fail and the kernel will panic when attempting to execute the code (this may be mitigated by removing the `BLX R7` gadget and replacing it with multiple copies of the chain). We even discover the aes context buffer is now in kernel stack and find the scheduled keys. If we knew that R3 to `aes_init` is a pointer to the key, we could also try to replace the key pointer to kernel stack and change the key to a known one. Then, we can replace the source pointer to be in SceSysmem and "decrypt" the data with a known key into kernel stack. Then, we can "encrypt" that data to get the crown jewels at last.
