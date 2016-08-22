---
author: yifanlu
comments: true
date: 2012-08-18 22:46:05-06:00
layout: post
slug: playstation-vita-progress-report
title: Playstation Vita progress report
wordpress_id: 563
categories:
- PS Vita
- Technical
tags:
- coreclr
- hex
- mono
- playstation
- psm
- ram
- vita
---

For the last couple of weeks I've been trying to get unsigned code running on the Vita. Although I haven't succeeded yet, I'm happy to say I've made some progress.

I've already posted much of my progress on this thread: [http://wololo.net/talk/viewtopic.php?f=52&t=12901](http://wololo.net/talk/viewtopic.php?f=52&t=12901) so I'll just quote what I've wrote.


> I'm sure many have tried looking at PSM as a possible way of getting native/unsigned code running, but I haven't seen any reports on it. I'm hoping that here, we can discuss our findings and such. I read most of the mono codebase (very messy and confusing), disassembled and read PSM for windows and android (because in theory the code on the vita should be similar because the windows and android ones are similar), decompiled PSM's mscorlib.dll and the various sony libraries, and USB sniffed debug traffic.
> 
> First, I want to talk about some of the things I've tried and failed. My first thought is that we should look at mono specific things as implementors may overlook these things. For example, reading [http://www.mono-project.com/Interop_with_Native_Libraries](http://www.mono-project.com/Interop_with_Native_Libraries), we see that mono allows a DLL name of "__Internal" to refer to loaded functions (aka libc and such). (Unfortunately, it's not as easy as loading exec() from __Internal.) There's also dllmap, which can map library names to other names. Specify the config file from the environment variable MONO_CONFIG and you can "rename" libraries. (Unfortunately, not much here either because PSM never loads any libraries through managed code). I've also took at look at assembly signing and how signing with the ECMA key (all zeros except for one byte) allows you to sign system libraries. (Unfortunately, mono doesn't really care about signing, so even if you sign a library as system, nothing really changes). I've tried using .NET Refractor to rename references to mscorlib and see if we can load another library with custom commands. Doesn't work because the name mscorlib.dll is basically hard coded inside mono and references to it is just a formality. I also tried to use reflections to access private and protected methods/fields but it doesn't work because of the security manager.
> 
> Now, here's my attempt at hacking PSM. Please note that this is most likely not the only avenue and I look forward to ideas of other routes.
> 
> So what are the security in place to prevent running unmanaged code? First and most importantly, the CoreCLR security manager is turned on. (In fact it is the very first thing PSM does when it starts). This means that all code running is untrusted and cannot call "SecurityCritical" methods. Next, sony made sure that any method that can lead to unmanaged code running is marked as "SecurityCritical". This means all the methods in System.Runtime.Interop.Marshal. This includes Marshal.PreLink() which is what is called when you do a DllLink() attribute on an extern method. This also include the various bridging methods like reading IntPtr and any method that can read/write unmanaged memory. Obviously, they also disabled methods to start processes and in fact even methods to find what path the current domain is in. Most of the icalls are SecurityCritical too, and those that must be called are usually called indirectly through another system method. So in order to run unmanaged code, we can either a) somehow get mono to think our code is trusted, which is hard because all code loaded by untrusted code is untrusted and our code is first loaded as untrusted. b) somehow disable the security manager, which is harder since this would have to be on the realm of exploiting mono, which is a fairly mature project (The last vulnerability was discovered in 2010 and sony's using an April 2012 version of mono). But the one silver lining is that (AFAIK from looking at android and windows version) there is no sony specific security in place and the only thing preventing unsigned code from running are mono features themselves which we have the source code for.
> 
> I've spent most of my time trying to find how to make mono think our code is trusted. Basically what happens is that the core assemblies are hard coded as trusted and with some exception, all other trusted code is determined by the core libraries. So the idea is, how do we make mono think we are loading a core assembly? What happens is that the implementation of the mono runtime (in our case psm.exe (windows), libdefault.so (android), and EBOOT.bin (vita)). Passes a callback function to libmono and this function is called with the name of the assembly to check if it is verified. The name is either the canonized path of the assembly if it is external, or just the filename if it is internal (AFAIK, the Vita system assemblies is internal).
> 
> Now I've found a tiny bug to work with: glib2 only looks at POSIX-styled path separators: / while the Vita can interprate both Windows-styled \ and POSIX-styled / It's not much, but it's something to start with. In order to disable the security manager, first you must make mono think that the library being loaded is a system library. Sony has hard coded the names "mscorlib.dll", "System.dll", and "Sce.PlayStation.Core.dll" and does a strcmp() with the canonical path of the library being loaded. However, if Mono sees that the basename of the library being loaded is "mscorlib.dll" or any of the other two, it will try to load the embedded version instead of our own. So, what we need to do is feed a path to Assembly.LoadFile() that a) the Vita recognizes. b) gets past the security manager by thinking it's a core library, and c) gets past mono's loader by thinking it's an external library.
> 
> If we get past the security manager, running unsigned code is as simple as calling an extern function. No need to deal with stacks or memory addresses. Here's how you can help. I've basically made an application, almost a game that allows you to test input paths. It includes the actual mono and eglib function used by PSM to test the names. Through either passing some malformed string or somehow exploiting the function themselves and getting all three tests to pass, please tell us. Think of it as a puzzle, using the given bug on path separators and any other bugs, find a string that passes the three conditions. I wish I can offer a prize, but I don't have anything except gratitude :P
> 
> [https://gist.github.com/3278820](https://gist.github.com/3278820)


I have since figured out that trying to bypass CoreCLR that way is a dead end, but most of the information is still valid. I made another post providing more details on CoreCLR.


> So here's where I'm at.
> 
> I've established the "main" security on the vita is CoreCLR.
How they implemented it: [http://www.mono-project.com/CoreClrHowTo](http://www.mono-project.com/CoreClrHowTo)
To summarize, sony wrote an application that embedded mono as a library and made some calls (first) to libmono to enable coreclr and some other calls to libmono to load your app.
> 
> More information on CoreCLR:
> [http://www.mono-project.com/Moonlight2CoreCLR](http://www.mono-project.com/Moonlight2CoreCLR)
> As you can see microsoft made this security manager for Silverlight. So users running untrusted webapps won't execute viruses on your computer. All is good. In fact, CoreCLR is taken very seriously as mono runs security audits on it.
> If you did your reading on how CoreCLR works, good, but if not, here's a quick summery:
> Most "useful" C# function, such as ones that can R/W the filesystem, access unmanaged memory (pointers and indirection and unsafe code), link libraries (extern functions) and so on are marked "SecurityCritical" that means your code cannot call them. However, because of how .NET is implemented, an application must have access to some of these functions. What they did was made gatekeeper functions marked SecuritySafeCritical which can call SecurityCritical functions. These gatekeeper functions CAN be called by your applications, but they make sure you don't abuse those critical functions. (For example, permission checks on file R/W).
> 
> On top of all that, there are a few Sony-specific security "features":
> 1) System.Diagnostics.Process is missing and the internal functions that support it are gone from the vita sdk (aka you can't just add the classes back in). This means no access to exec (which I think the Vita platform doesn't even support, they use sony specific functions, I can't find unistd.h). You won't be able to launch apps from mono.
> 2) dlopen() dlclose() dlsym() are missing from the vita sdk, so even if you get past the SecurityCritical calls and do a [DllImport("myfuncs.prx")] on an extern function, it won't link because there's no dynamic linking calls in mono.
> 3) All file I/O calls goes through sony specific IO functions (mono does not have direct access to fopen(), open(), read(), etc). Now the implementation of these sony specific IO calls is unknown (I'm sure the android version of these calls are different so I never bothered to look), but I can guess through how the FS is layed out, your app is mounted to pss0:/top/ and you only have access to within pss0:/top/ so when your application tries to access "/Application/some.dat" you're accessing pss0:/top/Application/some.dat. AFAIK, you can't access game0:/ which is where PSM itself is located or any other path such as the memory card or flash. (That's not to say it's impossible to break out of the FS sandbox, but we don't know how it works so it'll be hard).
> 4) Although System.Runtime.InteropServices.Marshal has functions to read/write to unmanaged memory, there seems to be something there preventing the functions from working property (even after getting past CoreCLR restrictions).
> 
> So with all that said, here's the "progress". As I've mentioned, I've gotten past CoreCLR, that's PSM's "main" line of defense. I can now call SecurityCritical functions with no restrictions. However, because of the other security features I've listed above and possibly more I haven't found yet, we haven't reached the goal.


By now, I can read and write all of the process memory when PSM assistant is running. I have dumped the PSM Vita executable and I will be dumping other modules and stuff soon. I cannot release my dumps for legal reasons, but just want to let you guys know there's hope for the Vita!
