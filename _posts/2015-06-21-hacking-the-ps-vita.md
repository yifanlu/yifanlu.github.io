---
author: yifanlu
comments: true
date: 2015-06-21
layout: post
slug: hacking-the-ps-vita
title: Hacking the PS Vita
wordpress_id: 1230
categories:
- PS Vita
- Technical
tags:
- aslr
- cil
- coreclr
- debugger
- exploits
- mono
- moonlight
- nid
- psm
- psvita
- sdk
- syscalls
- uvloader
- vita
- vitadefiler
- vitainjector
---

_The following was taken from a series of unpublished posts I wrote back in September 2012 (almost three years ago). The posts not only detail the exploit I found but also the thought process that led me to it. I intended to publish it as soon as the exploit was patched by Sony or after someone found another exploit on the system by examining the memory dumps. However, as of today, the PSM privilege escalation is still the only known way to execute native ARM code on the PS Vita. Apologizes for the outdated references._



### Pathfinding



To start, lets brainstorm the different ways we can attack this black box of a device. Typically, a new device is unlocked in a process that usually involves: 1) dumping the device's RAM/ROM/NAND, 2) analyzing the dumps for information and vulnerabilities, 3) using the vulnerability to create a tool that allows others to easily gain root access.
<!-- more -->

[![A cross section of the system-on-chip shows that the DRAM is stacked on top of the CPU and is impossible to sniff with conventional tools.](/images/2015/06/New-Picture-2-300x157.png)](https://web.archive.org/web/20150426003826/http://www.chipworks.com/cn/technical-competitive-analysis/resources/blog/sonys-ps-vita-uses-chip-on-chip-sip-3d-but-not-3d/)

A cross section of the system-on-chip shows that the DRAM is stacked on top of the CPU and is impossible to sniff with conventional tools.
{: .wp-caption-text}

The first thing we need is information. We have a few sources. First is the [wiki](http://www.vitadevwiki.com/index.php?title=Hardware), which we can use to learn about the hardware. Unfortunately, even though the system uses the ARM architecture, the CPU is a proprietary Sony chipset which we have no information on. There is also the [source code](http://www.scei.co.jp/psvita-license/) for various components of the system. We can use the source code to see if any library used (like libpng) has vulnerabilities. However, even if there are, exploiting will be hard without a RAM dump since we don't know where to inject the code or what code to inject, and the whole point of gathering information is to obtain a RAM dump. Therefore, trying to exploit the system at this point is pointless and would involve mostly guesswork. On a system like the PSP, the process was a bit simpler. 1.00 of the firmware allowed unsigned code to run without any hacks, and the system files were unencrypted and open for analyzing. When the firmware update that patched the hole was released, the information was already leaked and used to build another hack which is used to get more information on 1.50 which is used to hack 2.00 and so on. As you can see, everything depends on the first hack, which we are trying to find. This time, Sony did not make it easy for us. The next obvious avenue of attack is a hardware hack. This is how most of the consoles were first analyzed and how the 3DS is currently being analyzed. However, the Vita does not allow us to easily way to sniff the RAM because it is on the same system-on-chip as the CPU (the way to sniff RAM is to put a sniffer between the path of the RAM and the CPU to see what information is read/written). I am not discounting the possibly of a hardware hack on the Vita, but I do not want to open up my $300 device for the mere potential of getting some information. Plus I don't know anything about hardware, so there's that.

The most "common" exploits (looking at how often the PSP was hacked) are buffer overflows and heap overflows. However, finding either requires some knowledge of the system. For example, a buffer overflows allows you to redirect code execution to anywhere in the memory. But we don't know the layout of the RAM. We don't know where the stack is nor do we know where standard calls are. Of course, there's always room for guessing (based on layout of other ARM platforms or the PSP), but there really isn't even a way to see if a bug you find IS a buffer overflow. All you can tell is that the game/system crashed and you could try to "inject" memory addresses but again, you are guessing the memory addresses also. Multiple unknowns makes guessing an nonviable option. Usually, at this point, someone like me (software people) would step to the side and let the expert hardware reverse engineers do their job decapping the CPU and performing black magic, but there is still hope...

![Playstation Mobile](/images/2015/06/install-sonys-playstation-mobile-psm-store-your-nexus-7-tablet-for-free-mini-games.w654-300x168.jpg)

Sony recently opened up PlayStation Mobile, which is a way to allow indie game developers to write sandboxed software for the Vita and (some) Android phones. This is a good potential path into the system, because instead of needing to find an exploit that can execute code, we only need to find an exploit that escalates privileges, allowing us to bypass the sandbox. Traditionally, this is an easier task. PSM uses a runtime library called Mono (which is based off of .NET Framework used on Windows) which can take "bytecode" (which is compiles one programming language to another, simpler one) and compile it into native ARM code on the device after passing security checks (à la sandbox). The downside of having security like this is that performance is impeded, but we won't get into the details of why PSM is inferior to native homebrew code. The good news here is that because PSM is based off of Mono and Mono is open source, we basically have the source code to the entire security system. The bad news is that Mono is a pretty secure platform (generally open source projects usually have better security because more people are looking for bugs) and the last major vulnerability was fixed back in 2010. However, the chances of exploiting Mono is still better than blindly guessing. At least now we have a goal in sight.



### Plan of Attack



So now we have a more specific goal than "hack the Vita." It is "find a bug with Mono or PSM's implementation of Mono." This was the most tiresome part of the whole process. For the next three weeks, for about three to four hours a day, all I did was read Mono's source code as well as disassemble and read the PSM for Windows app and PSM for Android app. This is the boring aspect of hacking that most people gloss over. Slowly, as I understood more about how Mono VM worked, I begin to form possible attack vectors. Even though most of Mono's native code execution abilities were removed (Pinvoke, loading dynamic libraries, `System.Diagnostics.Process` which is used to run programs), `System.Marshal` and `System.IO` are implemented in PSM (they can't be removed because native code is required for tasks such as file IO). The security system of Mono (and .NET) that is used by PSM is the Moonlight (aka Silverlight) [security system](http://www.mono-project.com/MoonlightSecurity). The way this security system (called [CoreCLR](http://www.mono-project.com/docs/advanced/coreclr/)) works is that all managed code running is determined to be "security transparent" (unsafe) UNLESS it is the system runtime library (mscorlib.dll). In the runtime library, there are code marked "security critical", which cannot be called by unsafe code. These "security critical" methods are usually native calls to tasks like threading, memory allocation, file IO, etc. The ability to call "security critical" methods without restriction is essentially the same as bypassing the sandbox because there exist security critical methods in `System.Marshal` that allow for raw memory access and jumping to arbitrary code in memory. Because of this, access to "security critical" methods is only accessible by "safe critical" methods which are exported by the runtime library to provide a gateway to the "security critical" methods. These gateway functions do all the necessary checks to make sure the "transparent" code does not do anything malicious (and everything we want to do would be considered malicious).

Now that we understood the security system in place, it's just a matter of finding a flaw. My first attempt was to exploit some "safe critical" method. All it takes is one gateway function to miss some security check and we would have unrestricted access. Again, for weeks, it was nothing but reading source code, finding what calls specific methods, what arguments are passed where, and so on. Throughout that time, I've investigated hundreds (not a hyperbole) of different possible attack vectors (for example, trying to trick PSM into thinking our library is the system library, or trying to pass malformed paths to IO functions and so on). However, as I've said before, because Mono is open source, there are likely many security researchers who have audited the code especially since the Moonlight component runs in the web browser like Flash (and it would be an embarrassment to be less secure than Flash). I'm not saying Mono is absolutely secure, as I'm sure someone more talented could find an exploit, but the chances I will find something that experts have overlooked are slim. But, what if we turn that thinking around? What if we look were the experts didn't look because they didn't have to.



### Exploiting PSM



If Moonlight (and its security system) was designed for web browsers, then using it as an all purpose security tool could be problematic. The threat model that Moonlight defends against is not exactly the same as the threat model for PSM. Moonlight is used to keep the user safe from malicious developers. PSM is supposed to keep Sony safe from (I guess they would consider "malicious") developers. How do these two goals compare? Well, both threat models need to prevent access to native system functions and both need managed code to be completely sandboxed. However, there's no reason for Moonlight to be secure against the debugger, right? The purpose of the debugger is to "oversee" applications, so implicitly it would be considered privileged. Under the Moonlight threat model, if the attacker can invoke the debugger, they already have privileges to execute unmanaged native code. But in PSM's threat model, the debugger cannot be privileged since the developer has access to it. That seems like a promising observation, but how can we use the debugger to our advantage? Well, let's see what the Mono debugger [can do](http://www.mono-project.com/SoftDebugger:WireFormat). **INVOKE_METHOD** looks VERY interesting. Its description says it can be used to run a method with the debugger. Could be use it to invoke "security critical" methods? MonoDevelop doesn't have any GUI tools that make use of this feature, so I had to quickly code my [own implementation](http://github.com/yifanlu/VitaInjector) of the Mono debugger client. Sure enough, I can use it to call any method, including the unmanaged memory read/write functions. Using this, I was able to dump all of the memory PSM has access to. I quickly took video of this historic moment. However, what I cannot capture was the feeling of success after months of hard work.

<iframe width="420" height="315" src="https://www.youtube.com/embed/w1GICNXTOhM" frameborder="0" allowfullscreen></iframe>


### Executing Code



Some may be disappointed by the fact that there wasn't a clever software bug that was exploited but I think a mistake in implementation is just as bad as any buffer overflow (if not worse). Now that we have a memory dump, it's time to run some code. Here is another roadblock. ARM has a built in feature called XN ([execute never](http://en.wikipedia.org/wiki/ARM_architecture#No-execute_page_protection)) which basically means that the kernel decides what data in memory can be executed, usually with strict rules. This means we cannot just write code into the stack or heap and expect it to run. Even though we now control code execution, we do not have a way of getting custom code to run. All regions of memory that are executable (like PSM itself) are marked read only, so we cannot just overwrite existing code either. And, of course we don't have kernel access (we got out of the PSM sandbox and jumped into the larger PS Vita system sandbox managed by the kernel). A possible way around this limitation would be [return orientated programming](http://en.wikipedia.org/wiki/Return-oriented_programming) which is how most modern exploits (i.e: iPhone) work around XN. This is feasible now that we have a memory dump to work with. At this point, I got in contact with other (read: smarter) people who are also interested in the Vita and were looking into this. However, this is where we really got lucky. (I have to credit the person who asked about it in either the IRC room or the forums, but I can't recall their name right now.) PSM (Mono) must at some point produce executable code. Remember that the bytecode that the C# compiler produces must be just-in-time compiled into native ARM code on the device after passing security checks. This means PSM has access to special syscalls (with support in the kernel) to allocate executable memory and write to it. With our privilege escalation exploit, we are able to utilize these calls to write our own custom ARM code to memory and execute it. At this point, I pushed tons of assembly-coded payloads to farther inspect the system. I tried to read system files (can't), the application files (encrypted) and also tested various syscalls and API functions to see what they do. This is the first time I or anyone has gotten native code to run on the Vita and it felt like Christmas morning.



#### More Details



The way the privilege escalation works is by finding the hashmap of loaded CLR images and traversing the hashmap to set every loaded image as a "runtime library" in order to grant it permission to make "security critical" calls. This is done with the debugger exploit. Then we use the methods found in the `System.Marshal` class to peek/poke memory and call native functions directly from C# code.



### Writing a Loader



However, even though this means we can "run native code," it would be crazy to expect homebrew developers to learn ARM assembly and write all their games that way. In addition, homebrew needs to be able to make calls to system API functions. Finally, homebrew must be packaged in a file format that does not need to be changed when the system changes with firmware updates. What we need is a loader that does the hard work of loading homebrew ELFs to memory (with our newfound PSM executable memory allocation syscalls) and linking them to system API functions in memory.

This problem was solved on the PSP by [Half Byte Loader](http://wololo.net/hbl/) which cleans up the userspace memory, then loads the homebrew ELFs into the clean memory space, and finally links the API imports by locating them dynamically. This is basically what UVLoader does for the Vita. You, the developer, write C/C++ code, using the same APIs as real registered Vita developers do. Then, using an open SDK, compile that code to the same executable format that the official SDK produces (in theory SDK produced files [not published games], if unencrypted, could also run with UVLoader because the format is the same, but obviously, we don't have access plus it would likely be illegal if we did). UVLoader takes that file, parses the ELF format, loads the homebrew into memory, and finds the API/syscalls in memory and "attaches" them to your homebrew. Now, for official developers, this is all done by the system loader. We don't have access to it since the loader is in the kernel. UVLoader is developed after analyzing the format of the PSM executable loaded in memory. The PSM ELF file itself is encrypted, but artifacts of the ELF can be found in memory. Since the system loader is what we're trying to simulate, we can take a look at the memory dumps to see how things are tied together, what offsets means what, and so on.



### Addendum: Aftermath of the PSM Exploit



[![From Vita SDK specifications, credit to xerpi](/images/2015/06/sceimports-150x150.png)](/images/2015/05/specifications.pdf)

From Vita SDK specifications, credit to xerpi
{: .wp-caption-text}

Okay, let's come back to June 2015. Many things have happened since September 2012; and they are mostly security strengthening of the Vita by Sony. When I first released the source code to UVLoader, I thought that would bring a renaissance of Vita hacking as developers worked on an open SDK and homebrew while hackers continue to find more and more exploits to load UVLoader (like on the PSP). Unfortunately, that was not the case. The scene was mostly quiet with the occasional mummer of some glitch that someone found. Nobody was able to get code execution since the PSM hack and nobody worked on the open SDK until I published a [specification](/2015/05/23/calling-all-coders-we-need-you-to-help-create-an-open-vita-sdk/) for the SCE ELF format a month ago. However, Sony kept busy after analyzing the UVLoader source to make the Vita more secure than before. The only silver lining was that they never discovered the "exploit" (if you can call it that) even after many security patches (Sony is required by LGPL to release the source code of all their PSM DevAssistant updates).



#### Counterattack 1: Stricter Usage of Executable Memory Syscalls



The key to executing code with the PSM hack is through the special syscalls that only PSM has access to. These syscalls allow PSM to allocate executable memory and write to it. The way these syscall works is that you must "unlock" the [memory domain](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0211k/Babjdffh.html) before writing to it and "lock" it afterwards. In the 2.00 firmware, certain system functions check to make sure the executable memory domain is locked or it kills the running app. This means if you leave the domain unlocked (as the first version of UVLoader did), it kills PSM. My guess is that Sony originally thought the exploit was that some function forgot to lock the memory domain and we were overwriting JIT code. This was easily bypassed by adding the relevant lock calls to UVLoader. I guess my laziness was misinterpreted by Sony as cleverness.



#### Counterattack 2: Removing "Dangerous" Methods from Mono's Marshal



Sony likely did an (several?) internal security audit of Mono because with each release, more security "fixes" were added (mostly potential use-after-frees and potential null dereferences). By 0.99.20 (DevAssistant 1.03), Sony removed a function needed by VitaInjector. `Marshal.GetDelegateForFunctionPointer` takes in an arbitrary memory address and allows managed C# code to jump to it. Of course, this method is security critical and had to be called from our exploit, but it was the way we were able to call arbitrary native code. At this point, because of the months of studying Mono, I was pretty familiar with CIL (the bytecode format for C#). There is a bytecode instruction "calli" which provides the functionality of `Marshal.GetDelegateForFunctionPointer`. This instruction, as far as I know, is not used elsewhere in Mono's runtime library so it is likely overlooked by Sony's auditors. "calli" can only be used by security critical functions in the CoreCLR security model, so we just embed it into our own DLL and call it with the PSM exploit.



#### Counterattack 3: Address Space Layout Randomization



Because VitaInjector and UVLoader depended on functions in memory (including the executable memory allocation syscall), Sony decided to randomize the memory layout on PSM DevAssistant 1.08. This, along with the next counterattack placed VitaInjection development on hiatus for a year (before it was reincarnated as VitaDefiler). The way around it is simple in theory but a lot of painstaking work in practice. To bypass ASLR, we find the address of a known function in memory using C# managed code (for example Environment.Exit) since Mono knows the memory addresses of internal calls and we can obtain it with the PSM hack. Then using offsets found previously in a memory dump, we can locate all the necessary functions for the loader to run. This was a lot of work because it all has to be encoded as Mono debugger commands, which there is no documentation for.



#### Counterattack 4: NID Poisoning



The way UVLoader links homebrew API calls is by finding the same API call from loaded libraries in memory and using that information to locate the right syscalls and function addresses. On the Vita, there are no string "symbols" found in traditional ELFs. Instead, it uses four byte integers that serves as a hash of the function name to import. In Vita firmware 2.06, Sony replaced the NID import tables for syscalls of all loaded libraries to be random integers (and since 3.x, this extends to import tables for function calls). I called this "NID poisoning" because it makes the import tables useless and we can no longer locate syscalls. One way around this is to "cache" the NIDs within UVLoader. However, as libraries change from firmware version, the NID order changes too and because of the number of libraries that can be loaded, it would make UVLoader way too bulky. UVLoader 1.0 now does use this technique to some extent. It only caches one library's imports, SceLibKernel, which is required to run the loader itself. Then, when linking homebrew ELFs, UVLoader can use a technique discovered by Proxima and me which I named "NID antidote" (for obvious reasons). Libraries can be lazy-loaded, which means the library will be loaded in memory but not linked (and therefore the import NID table is not poisoned yet). Then, when the application decides it needs to use this shared library, it will make another syscall to "start" the library, which then performs the linking and NID poisoning. Every library can be loaded multiple times too, so what we do is look for the libraries already started, then load them again (but without starting them). The already started library have the API calls resolved but the NID poisoned. The newly loaded library have the API calls unresolved but the NID unpoisoned. We just mash the two tables together and have all the information we need for UVLoader.



### More Details



For those who thirst for more details about how the exploit works, I encourage you to take a look at the source for VitaDefiler's [USB.cs](https://github.com/yifanlu/VitaDefiler/blob/master/USB.cs). I have to apologize for the terrible code quality as most of it was written burning the midnight oil and when "Thread.Sleep(1500)" made something work, I kept it in. `StartDump` is how to dump memory using the PSM exploit. `DefeatASLR` shows how to obtain pointers to important functions, defeating ASLR. `EscalatePrivilege` patches Mono to make every loaded image have runtime library permissions.
