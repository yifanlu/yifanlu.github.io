---
author: yifanlu
comments: true
date: 2015-05-23
layout: post
slug: calling-all-coders-we-need-you-to-help-create-an-open-vita-sdk
title: 'Calling all coders: We need you to help create an open Vita SDK!'
wordpress_id: 1084
categories:
- Information
- PS Vita
tags:
- arm
- elf
- libelf
- ps vita
- sdk
- toolchain
- vita
---

One of largest barrier to native PS Vita homebrew is the lack of an open toolchain and SDK. Essentially, we need something like [pspsdk](https://github.com/pspdev/pspsdk) for the Vita. The reason why we don't have it is because there are people who have an understanding of how the Vita's executable format works but lack the time to code up the tools and there are people who have the time and ability to create such tools but lack the knowledge of Vita's internals. The solution, I believe is to publish a comprehensive document detailing how the Vita's executable format is laid out and the requirements for an open toolchain. Anyone with coding skills can now work on an open SDK; no Vita knowledge required!<!-- more -->

For those who want to help with this endeavor, there are only two prerequisites. First, you must understand how the ELF format works. It is a very simple format and [this document](http://flint.cs.yale.edu/cs422/doc/ELF_Format.pdf) gives all the details. It would also be advantageous to understand the [ARM extensions](http://infocenter.arm.com/help/topic/com.arm.doc.ihi0044e/IHI0044E_aaelf.pdf) to ELF. Once you are caught up with that, you should be able to understand the SCE ELF format that I detail in the attached document. The actual tools that we need can be written in any language as long as it supports the three major platforms and meet the requirements detailed in the specifications. However, if you want a place to start, I would recommend [libelf](http://sourceforge.net/projects/elftoolchain/files/Documentation/libelf-by-example/) and portable C. I hope that the community can get together to work on this. Once someone makes promising progress, I will update this post with a link to the project so others can contribute. Please post any comments or suggestions you have for this document below. [#vitasdk](irc://irc.freenode.net/vitasdk) on FreeNode will be for developer discussion on the SDK.

[PS Vita Open SDK Specifications](/images/2015/05/specifications.pdf)

P.S (in an unrelated subject): Did you get a PSM license? If not, don't fret, we got you covered. Just make sure you have the developer assistant installed on your Vita.

**Update:** Follow the progress here [vita-toolchain](https://bitbucket.org/cirne/vita-toolchain/)


