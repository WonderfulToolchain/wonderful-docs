---
author: "asie"
date: 2023-08-27
draft: false
title: "Project Update - August 27th, 2023"
---

In the previous project update, I named many issues that I'd like to resolve in the toolchain. In the month of August, motivated in part by some extra time off I had, I managed to tackle a few of them.

Let's talk, first and foremost, about asset processing and ROM linking.

<!--more-->

## Asset processing

The biggest pain point of the toolchain for the past months was the lack of an official answer to asset processing. In many homebrew toolchains, people end up with limited or even no official facilities to handle this problem; they end up writing unportable shell scripts and missing out on many benefits of plugging them into a build system - incremental updates, automatic change detection, et cetera.

For the Wonderful toolchain, I'm trying out a solution in the form of `wf-process`. It effectively works as a Lua script runner which exposes APIs that allow chaining tools together for various input files, complete with dependency resolution integrated with our Makefiles, and emitting .c/.h files containing symbols for said input files.

Here's some example Lua code for converting all .PNG files in a directory to a format compatible with the WSC:

```lua
local process = require("wf.api.v1.process")
local superfamiconv = require("wf.api.v1.process.tools.superfamiconv")

for i, file in pairs(process.inputs(".png")) do
	local tilemap = superfamiconv.convert_tilemap(
		file,
		superfamiconv.config()
			:mode("wsc"):bpp(4)
			:tile_base(0):palette_base(0)
	)
	process.emit_symbol("gfx_color_" .. process.symbol(file), tilemap)
end
```

The verbose syntax allows for operations to be easy to understand at a glance. In addition, chaining is made possible without worrying about where temporary files end up - for example, feeding a tileset from SuperFamiconv to be compressed using LZSA2:

```lua
local lzsa = require("wf.api.v1.process.tools.lzsa")  

local tilemap_color = superfamiconv.convert_tilemap("title_screen.png", ...)
tilemap_color.tiles = lzsa.compress2(tilemap_color.tiles)
```

Documentation for `wf-process` is not yet complete, but I can refer you to the following sources:

* [a basic image display example for the toolchain](https://github.com/WonderfulToolchain/target-wswan-examples/tree/main/examples/basic_display),
* [a port of joffb's WonderCell to use wf-process](https://github.com/asiekierka/wondercell/tree/feature/wf-process),
* [a reference manual for the wf-process API](https://wonderful.asie.pl/doc/wf-lua/).

## New linker

Last month, I said:

> The toolchain is currently limited to 768 KB ROMs. With the largest flash cartridges being able to fit 8 MB, that leaves a lot of room on the table.

Fixing this adequately ended with the creation of a new linker - or, well, half of one.

Previously, the ROM linking process worked by calling the GNU linker and copying  out the resulting binary data twice:

* first with the ROM at the linear address `0x40000`,
* second with the ROM moved to the end of the address space, that is `(0xFFFF0 - first_stage_size)`.

This wasn't optimal, but worked well enough and allowed other development work to progress. Later on, my experiences with contributing to the llvm-mos SDK allowed me to observe that controlling some of the linking process can unlock new types of functionality. While the GNU linker offers many features that can be used (or misused) into handling things like bankable ROMs and non-contiguous memory allocation, I felt that the additional control granted by writing a custom solution would be beneficial. At the same time, I didn't want to write all the code responsible for actually gluing object files together myself - it's a lot of work to maintain, especially if the toolchain were to ever support features like link-time optimization.

Today, the ROM linking process works by:

* calling the GNU linker to create a *relocatable* .ELF file - with all the symbols in one place, but the memory layout undecided;
* calling `wf-wswantool build rom` to *relocate* this .ELF file into a ROM, complete with appropriate bank allocation.

Full control over the memory layout, in addition to an extension in the form of special section names, allowed unlocking new functionality, such as:

* non-contiguous console RAM allocation, complete with confining certain types of data (wavetables, screen maps) to appropriate locations:

```c  
// Old
#define screen_1 ((ws_scr_entry_t __wf_iram*) 0x3000)
#define screen_2 ((ws_scr_entry_t __wf_iram*) 0x3800)
#define sprites ((ws_sprite_t __wf_iram*) 0x2e00)

// New - notice no hardcoded addresses!
__attribute__((section(".iramx_screen.1")))
ws_screen_cell_t __wf_iram screen_1[32*32];
__attribute__((section(".iramx_screen.2")))
ws_screen_cell_t __wf_iram screen_2[32*32];
__attribute__((section(".iramx_sprite")))
ws_sprite_t __wf_iram sprites[128];  
  
// Note: We also need to reserve space for tiles now.  
// _2000 refers to memory location 0x2000.
__attribute__((section(".iramx_2bpp_2000")))
ws_tile_t __wf_iram tiles_2bpp[320];
```

* proper ROM allocation, complete with data being placeable in the same bank regardless of them being accessible from a linear 768 KB memory window or a 64 KB one,
* special `__bank_[symbol]` symbols, resolving to the bank `[symbol]` was placed in:

```c
__attribute__((section(".rom0.gfx_font"))) // any bank, "ROM bank 0"
const uint8_t __wf_rom gfx_font[1024] = { ... }; // array "gfx_font"

extern const void *__bank_gfx_font; // address = bank for "gfx_font"
```

* working around a long-standing bug with relocating debug information - `addr2line` and `objdump -S` work correctly now!

This required a lot of work, and should be fairly stable now - though edge case bugs may appear with something like this; don't hesitate to report them! The `wf-process` asset processing system also supports placing things in banks.

One issue does remain, however. With the linker written in Lua, whose string primitives are immutable, the current cost of modifying data for relocation purposes is fairly high, leading to sub-par performance. This can be improved by implementing a dedicated "typed array" type. In addition, the WonderWitch target still uses the old linking method for now, without these new features - more understanding of the FreyaOS platform is required to ensure appropriate adaptation.

This set of improvements also has much more documentation available. On top of the projects linked above, there's [a section of the Swan Book](https://wonderful.asie.pl/doc/wswan/guide/topics/memory-management/) dedicated to memory layouting and management.

## Other improvements

There have also been many other, smaller improvements to the toolchain:

* The C library has received fixes to `atoi()` and `atol()`, as well as implementations of `bsearch()` and `qsort()` ported from [PDCLIB](https://github.com/DevSolar/pdclib).
* The hardware library has received fixes and optimizations to UART/serial port handling, as well as a minor optimization to `ws_dma_copy_words`.
* An interesting new target has appeared: the MobileWonderGate! It appears that the web browser bundled with the mobile adapter will accept binaries with an `application/wondergate` MIME type. Unfortunately, there's currently a ~13 KB filesize limit, and the method of alleviating and/or circumventing that is unknown.
* A central location for toolchain issues and feature requests [has been created](https://github.com/WonderfulToolchain/wf-issues/issues). I have also populated it with some of the things that I've had in mind for the toolchain's future.

## WSdev Wiki

Another long-standing issue with the WonderSwan as a target was the lack of a central location for hardware documentation. The two primary community-made manuals - WSMan and STSWS - did not have full overlap; due to their maintainers being busy, new discoveries would often end up lost as a mere mention on a personal GitHub repository or even a Discord channel.

I'm happy to report that I've spent some time collaborating with veterans of homebrew wiki maintenance - NESdev - to build an equivalent resource, one that will hopefully combine all the available information: [the WSdev Wiki](https://ws.nesdev.org/wiki/Main_Page). Contributions and suggestions welcome!

## The Future

I'm fairly happy with the toolchain's standing right now, especially given that I cannot keep focusing on it forever - it is a rather niche platform. However, there's still some things I'd like to work on. As I've mentioned them in the previous update, I'll just summarize here:

* More user library functions - especially regarding audio;
* Integrating Generic's proof-of-concept Windows support;
* Better toolchain documentation - more examples, more articles.

Thank you for reading. The potential future of the console which was now appears very bright; it remains to be seen if others will be interested in trying it out.