---
author: "asie"
date: 2023-07-30
draft: false
title: "Project Update, The First - July 30th, 2023"
---

Back in May 2022, I've been reminded of a console that I've heard about mostly from its Higan/Ares emulation core. It was 8086-based, an oddity in the console world yet a processor I was [already familiar with](https://blog.asie.pl/2020/08/reconstructing-zzt/), as well as one which had a healthy supply of both classic and modern development tools. At the same time, despite selling 3.5 million units, the console did not have a modern homebrew toolchain - one which would allow you to write in C without too much pain and provide useful functionality out of the box; one which would be easy to install and use.

It is now July 2023, and I'm happy to report that after a lot of research, hacks, head-scratching and side quests, we're finally here. In the first project update of hopefully many, let's talk about where we are so far.

<!--more-->

## The System

<img alt="The WonderSwan Color, in blue." src="/img/wonderswan_color_transparent-fs8.png" width="600"/>

Meet the WonderSwan.

The WonderSwan is a handheld released by Bandai in Japan - and, later, a few other Asian markets - in March 1999, as a competitor to Nintendo's handheld hegemony in the form of the Game Boy and Game Boy Color. It enjoyed decent success, selling over three million hardware units total; though this was [much lower than Bandai's expectations](https://nlab.itmedia.co.jp/games/news/9810/08/news02.html), it still allowed them to hold a small but notable of the Japanese market at its peak.

The WonderSwan is also the brainchild of a small company known as [Koto Laboratory](https://www.koto.co.jp/english/index.html), founded by Gunpei Yokoi after his departure from Nintendo; given his untimely death in late 1997, it can be considered his final contribution to the world of handheld video gaming.

The WonderSwan is also a device powered by an 80186-compatible processor designed by NEC; featuring a 224x144 display, two background layers, up to 128 sprites, four wavetable-based audio channels, 16-64 KB of RAM, as well as the unique design allowing for both horizontal and vertical games. Hardware-wise, I would personally describe it as an "overpowered GBC". If you'd like to learn a little more about the hardware, I recommend starting with The Swan Book's [brief hardware overview](https://wonderful.asie.pl/doc/wswan/guide/tutorial/hardware-overview/).

The WonderSwan is a lot of things, which inspired me to work on...

## The Toolchain

How hard could it be? In the end, it wasn't too difficult to get [gcc-ia16](https://gitlab.com/tkchia/build-ia16/) to build and link code for the WonderSwan; what was considerably more difficult was exposing its strong suits and hiding its weakpoints from the end user.

The benefit of using gcc-ia16 is access to modern C standards (C11) and compiler optimizations; the old "official" toolchain, the [WonderWitch](http://wonderwitch.qute.co.jp), offered a mix of the Japan-specific LSI C compiler and the better-known Turbo C compiler. The drawback is that it's considerably less production-tested than the "old guard", having required [a](https://github.com/tkchia/gcc-ia16/issues/102) [few](https://github.com/tkchia/binutils-ia16/issues/6) [bugfixes](https://github.com/tkchia/gcc-ia16/pull/136) and [improvements](https://github.com/tkchia/gcc-ia16/issues/119) to make it more usable with fewer warts - special thanks to T.K.Chia for being patient with me.

Another notable benefit of choosing this compiler is that, thanks to its optimizer design, it was [easy](https://github.com/tkchia/gcc-ia16/pull/99) to add processor tuning support - this is important on the WonderSwan, as the NEC V30MZ's instruction cycle counts have little to do with standard programming practices for the 8086/80186.

Alongside the compiler and linker, libraries were written - the bare-bones `libc`, the `libws` hardware abstraction library, and the `libwsx` utility library. They are still incomplete, but provide useful helpers for many areas of WonderSwan development. As a side quest, an open source implementation of the WonderWitch's system libraries - `libww` - is also provided. Notably, all of these core libraries of the toolchain are licensed under terms which *do not require attribution in binary builds* - as these are often not provided as part of homebrew software distribution regardless.

Tooling also had to be written: initially in Python, later moved to Lua to allow more standardized packaging. This includes tools such as a linker wrapper that outputs ready-made ROM files (`wf-wswantool romlink`) or a .c/.h binary file converter (`wf-bin2c`).

Finally, on top of this, a packaging solution based around Arch Linux's [Pacman](https://archlinux.org/pacman/) was built; the goal was fully root-less maintenance and installation in a separate directory, without conflicting with or modifying the system's own package management.

All in all, it really was a lot of steps to get from "my compiler works" to "my compiler works for more than one person" - this process took roughly a year, though admittedly on a time schedule far from full-time development. By May 2023, the toolchain was installable by others; by June, the first people other than myself started building things with it.

## Project Spotlight: WonderCell

Speaking of things built by other people than myself!

<a href="https://joesoft.itch.io/wondercell">
<img alt="A screenshot of WonderCell - a FreeCell playing board on a green background." class="pixelated" src="/img/wondercell2.png" width="448"/>
</a>

[WonderCell](https://joesoft.itch.io/wondercell) is an implementation of the FreeCell card game for the WonderSwan,
developed by [Joe Kennedy](https://github.com/joffb) using the Wonderful toolchain. It contains everything you'd expect a game to - graphics, music, gameplay - and as such stands as proof of viability of the tools in the present day.

Seeing people try to develop games with the toolchain is important - it provides valuable feedback and information
on what could work even better across a variety of coding approaches and styles.

## The Future

Having made it to this point, despite people now being able to create homebrew, there's still a lot of things that could be done to make the tooling even better. Here are some highlights:

* There's no official solution for asset conversion - be it images, music, sound samples, or fonts. The idea is to allow writing scripts in Lua which use the already-distributed libraries; this would allow a portable way of writing more advanced asset conversion pipelines than "run tool A on file B with arguments C". However, I'm still not sure on the final design of such a tool.
* The libraries could use more work - the most notable highlights are the lack of a built-in sound driver, as well as wrappers for making some common operations (copying tiles with memcpy, configuring various I/O ports) expressible in a more user-friendly manner.
  * If you'd like to suggest some other areas to cover, or API designs which could work well, there's [an open issue](https://github.com/WonderfulToolchain/target-wswan-syslibs/issues/1) on our GitHub.
* The toolchain is currently limited to 768 KB ROMs. With the largest flash cartridges being able to fit 8 MB, that leaves a lot of room on the table.
* The toolchain is currently only available for Linux. Generic has built a proof-of-concept allowing for execution on Windows based around MSYS2, but the proof-of-concept-level build scripts powering the package build system need to be reworked to implement that cleanly.
    * Mac builds are unlikely to happen, as these require a copy of macOS to compile and develop, which - officially - requires an Apple computer. (While I do own one, I don't think a Power Macintosh G4 will really help here...)
* There are known compiler bugs and drawbacks; even if fairly niche, it would be really cool to not have to deal with them. Unfortunately, GCC is hard; I have more experience with LLVM, but an IA-16 backend for it would be a very expensive undertaking which I cannot justify with the user base's current size.
* The documentation could use more work - I've started work on an actual guide recently, but I'm not the greatest at technical documentation writing.

It would also be nice to support other platforms - I have a bare-bones PS1 toolchain I'd like to integrate into the tooling built here, for example. Some work has been made on this, but remains a relatively low priority.

While I'm happy with where I got so far, I'd love to work on the toolchain further - but I need feedback, and for that I need more users. If you're interested in developing for the WonderSwan with modern tools, now is a better time than ever[^1]!

[^1]: With the exception of development hardware availability, due to the silicon shortage and all.

## Okay, but how do I join?

[The Swan Book](https://wonderful.asie.pl/doc/wswan/guide/) provides a step-by-step guide to creating your first WonderSwan homebrew program. It's still in early development, but it's currently the best place to start!

We also have [a Discord server](https://discord.gg/CR7MCZNurp) where you can chat with other users of the toolchain. It's certainly the fastest way to get a response out of us. (An IRC bridge may be set up given sufficient demand. Please let me know - I already run one for a different community.)

All of our projects' source code is available [on GitHub](https://github.com/WonderfulToolchain/).

## The Credits

I'd like to thank [jix](https://jix.one/) and [Tharo](https://github.com/thar0x29a) for their generous donations to the project. With their exceptional contributions in development hardware, I was able to validate my work and help improve the state of understanding of the WonderSwan's hardware.

I'd also like to thank the WonderSwan Discord community, the many people from the gbdev and gbadev Discord communities, and my friends, for their support and advice throughout the development process.

Let's hope the future for this project is Wonderful. Thank you!
