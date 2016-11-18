---
author: yifanlu
comments: true
date: 2016-11-17
layout: post
slug: designing-a-cfw-framework
title: "Designing taiHEN: A CFW Framework"
categories:
- Vita
tags:
- vita
- henkaku
- taihen
- design
- software
---

I take software design very seriously. I believe that the architecture side of software is a far more difficult problem than the implementation side. As I've touch upon in my [last post]({% post_url 2016-11-01-taihen-cfw-framework-for-ps-vita %}), console hackers are usually very bad at writing good code. The code that runs with hacks are usually ill performing and unstable leading to diminished battery life and worse performance. In creating taiHEN, I wanted to do most of the hard work in writing custom firmwares: patching code, loading plugins, managing multiple hooks from different sources so hackers can focus on reverse engineering and adding functionality. 

A nice companion piece to this would be my previous article on designing a CFW for the [3DS]({% post_url 2016-03-28-3ds-code-injection-through-loader %}). However, even my 3DS CFW was lacking as it required hard coding offsets to patches (for each firmware version). A workaround is to do pattern matching, which is what many 3DS CFW do, but that is only a half-measure. A key observation: most desired patches will have a user-observable effect. A second key observation: most observable effects span multiple modules. What does this mean? On the Vita, the kernel is modular and applications are also modules. Every functionality is contained in its own module, with its own set of permissions, and its own interface. For example, SceKernelThreadmgr handles threading related stuff and SceIoFilemgr handles file IO. If we treat each module as a black box and only worry about the _interface_ between modules, we can still do some powerful modifications. For example, if we wish to change the data from a file accessed from a game, we only need to hook the interface between the game and SceIoFilemgr and write logic to redirect the file if some conditions match (typically a string compare with the path). This way, we do not have to dig inside the game and find the specific offset of the function that accesses the file of interest. This is also a good approach because the kernel has built in support for finding exported and imported functions (it is used by SceModulemgr to do dynamic linking). Additionally, since modules share the same identifiers for exports/imports across firmware versions (for compatibility), this removes the need for the hacker to have to try to manually find offsets or do messy pattern matching. In practice, the majority of firmware patches can be done this way. To modify functionality affecting just that one module, hook an import function. To modify functionality affecting all other modules, hook an export function. Hooking the interfaces between modules is much cleaner than injecting code into the modules themselves.

This was the main goal in taiHEN: give developers a way to easily add hooks to module interfaces. But that is not enough for a _good_ custom firmware framework. I also wanted to satisfy the following goals

* The framework should be simple. Nobody likes reading documentation and figuring out which init functions to call and what flags to pass in. The API should be simple to read and intuit.
* It should be thread-safe. This is a requirement for any kernel code running on a modern system.
* It should be robust. I do not want to introduce new bugs into the kernel.
* It should be fast. Again, a basic requirement for kernel code. Specifically, inserting hooks should be fast and more importantly, executing the hooks should not require a ton of overhead.

To explain how these goals are met, I will dive into the low-level details of certain design decisions and my justification for doing it that way.

## Data Structure

The most important aspect of the design is the underlying data structure. In order to choose the right data structure for the job at hand, you have to consider what operations you wish to optimize for. In this case

* Given a process id, check if we have patches for the process.
* Given a process id and an address (and size), query the structure to see if a patch already exists.
* Add and remove patches
* For function hooks, store the original function so it can be called

![Hashmap layout]({{ site.url }}/images/2016/11/proc_map.svg){: .alignleft} To start out, we use a standard hashmap to store `tai_proc_t` structures. This maps process ids to `tai_proc_t`, so we can quickly get information for a given process id. Since process ids are efficiently random, they serve as a good hash function as well.

Now it gets tricky. We support two kinds of patches: injection and hooks. An injection is simple--it's a direct write to the target memory. A typical use case might be to overwrite a string in read-only memory. An injection also claims exclusive access: another attempt at injecting the same address will fail. A hook, on the other hand, is designed to redirect a function. For any given imported/exported function, a hook will redirect control flow to the patch function. This is done thanks to [libsubstitute](https://github.com/comex/substitute). There might be the case where multiple plugins wish to hook a single function (for example, two different plugins wish to redirect two different files). That's where the hook chain comes in. The patched function can invoke `TAI_CONTINUE` to call the next patch function in the chain (the last one in the chain would be the original function). This allows the patch function to manipulate the inputs before continuing the chain and manipulate the output after continuing the chain.

To support this "hook chain" idea, the easiest implementation would be to generate a "dispatcher" function. We can keep track of every hook in the chain, and then the dispatcher calls them all in order. Removing a hook from the chain would be simple enough, just remove the reference from the dispatcher. This would introduce a lot of overhead though. If there is just one function in the chain, we would have to still generate and run the dispatcher. Here is a better solution: when the first hook is inserted, we use libsubstitute to redirect the function call directly to that patch function. We then store a pointer to the original function along with some other information in `tai_hook_t` in the process's address space. `TAI_CONTINUE` will use that pointer to find the next patch function to call--no dispatcher needed. `tai_hook_t` is therefore a node in a linked list. When we add another hook to the chain, we simply allocate another `tai_hook_t` and add it to the linked list. The only catch is that if we remove the head hook, we must re-patch the original function with libsubstitute to jump to the new head hook.

So the layout is as follows: Each `tai_proc_t` points to a sorted linked list of `tai_patch_t`. Each `tai_patch_t` is either an injection or a linked list of hooks. Because the patches are sorted by their address, it makes insertion and deletion simple.

Here's a visual representation of what the data structure looks like.

![Data structure layout]({{ site.url }}/images/2016/11/structure_expanded.svg){: .aligncenter}

The last thing to note is the slab allocator. There are two times where we need to allocate memory accessible directly from the process's address space. First, libsubstitute needs to save the first couple of bytes of the function it is hooking. This is so you can call back into the original function from your patch. Second, as mentioned above, we need the actual hook data to be in user address space in order for one hook to find the next one in the chain without potentially having to call down into the kernel to find it. The easy way of doing this is to allocate a new page for each request, but that wastes a lot of memory because we need only about 20 bytes for both the hook data and the saved original function. A slab allocator is perfect for this situation because each request is small and about the same size. This makes allocations both fast and have little overhead. The allocator I chose to use was [this one](https://github.com/bbu/userland-slab-allocator) because it was simple and has no external dependencies.

## Designing for Testability

The hard part of any project--especially writing kernel code in a system with no debugging facilities--is testing it. My only "debugging" tool is `printf` so a project that is multithreaded, operates with complex custom data structures, and interacts intimately with the kernel makes testing a daunting task. One of my goals is for taiHEN to be robust. That is why from the start, I designed it to be built from blocks that are self contained.

![Building blocks]({{ site.url }}/images/2016/11/building_blocks.svg){: .aligncenter}

In the bottom layer, we have `tai_proc_map` which is a hash map with some special add/query functions (as described above). Because this data structure doesn't depend on any Vita-specific functionality, I was able to write a unit test on my Intel x86 machine. There, I spawned hundreds of threads each performing random operations with the hash map. Once that worked without and crashes or leaks, it gave me enough confidence to use it as a building block. Since the slab allocator and libsubstitute are both external projects with their own unit tests, I can just rely on those. In the next level, I have tests for the NID resolver and the hook chain system (described above). Each gets their own test. Once those were passing with enough seeds and iterations, I hooked everything together into the APIs exposed to the developer. The final tests I wrote only interfaces with the public APIs (in multiple threads) to ensure that the overall system was working.

This is not the perfect testbench but it is the bare minimum that I believe all complex software projects should do. With more time, I would have written more unit tests, directed tests, and randomized tests. My minimum testbench though saved me from a lot of headaches. When something breaks, I didn't have to wonder what component has the bug or if multiple components were all breaking at once. I can quickly root cause the crash and fix it. I think, for developers, the most frustrating situation is not being able to proceed: not knowing how to debug is more despairing than having to fix a complicated bug.

## Conclusion

You can learn more about taiHEN [at the dedicated site](https://tai.henkaku.xyz/) and you can read about the APIs [here](https://tai.henkaku.xyz/docs/group__taihen.html). I hope that people will build some really neat stuff with this framework. I also hope that people will build upon it as well. I know that the PS4's kernel is structured very similarly and it just may be possible to port taiHEN there. If anyone would like to pick it up, check out the [GitHub page](https://github.com/yifanlu/taiHEN).
