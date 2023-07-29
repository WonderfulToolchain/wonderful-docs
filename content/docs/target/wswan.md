---
title: "wswan"
---

# "wswan" target (WonderSwan/WonderWitch)

## Installation

`wf-pacman -S target-wswan`

Examples and project templates are available in the [wf-wswan-examples](https://github.com/WonderfulToolchain/wf-wswan-examples) repository.

## Toolchain library documentation

* **[The Swan Book](/doc/wswan/guide)** - (very early WIP) guide to developing homebrew for the WonderSwan with Wonderful

### Library references

* [libws](/doc/libws) - hardware access library
* [libwsx](/doc/libwsx) - additional utilities library
* [libww](/doc/libww) - FreyaBIOS/FreyaOS wrapper library for WonderWitch

## Wonderful-maintained tools

* [BootFriend](/ws/bootfriend) - custom console firmware allowing fastboot and running code from WSC/SC internal RAM without a flash cartridge,
* [CartFriend](https://github.com/WonderfullToolchain/ws-cartfriend) - custom flash cartridge firmware with a homebrew-centric design,
* [ExtFriend](https://github.com/WonderfulToolchain/ws-extfriend) - DIY serial/headphone capture USB device, based on the RP2040/Raspberry Pi Pico.

## Brief hardware overview

The WonderSwan is a handheld console released by Bandai in March 1999, with an enhanced Color model released later that year.

* NEC V30MZ (80186-compatible) CPU @ 3.072 MHz
    * Note: The V30MZ performs many key operations in 1-3 cycles. It also features a prefetch queue and simple instruction pipeline. It is overall more powerful than a 3 MHz Intel 80186 chip would be.
* 16KB (WS) / 64KB (WSC) of RAM - shared between software data, video data and audio wavetables
* Graphics:
    * ~75.47 Hz, 224x144 display + six developer-controlled LCD segments
    * Mono:
        * 2BPP tiles: 8 palettes x 4 shades and 8 palettes x 3 shades + transparency
        * Palette: 8 out of 16 shades
        * Up to 512 tiles
    * Color:
        * 2BPP tiles: 8 palettes x 4 colors and 8 palettes x 3 colors + transparency, 12-bit RGB
        * 4BPP tiles: 16 palettes x 15 colors + transparency
            * Planar and chunky storage supported
        * Palette: 12-bit RGB
        * Up to 1024 tiles (512 available for sprites)
    * Two 32x32 background tile layers ("screens")
    * Up to 128 8x8 sprites, up to 32 per line, built-in shadow memory
    * Hardware horizontal/vertical flipping
* Sound:
    * Four wavetable (32 x 4-bit) channels
    * 24000 Hz
        * Internal speaker: 8-bit PCM
        * Digital headphone output: 16-bit PCM
    * Channel 2: optional sample mode 
    * Channel 3: hardware sweep
    * Channel 4: optional LFSR noise mode
    * (Color) Hyper Voice: fifth "channel", stereo samples, headphone output only
* Eleven keypad inputs (X1-X4, Y1-Y4, A, B, Start)
* 9600/38400 bps UART (for console, computer or peripheral connectivity)
* Cartridge:
    * Up to 16MB (2001 mapper) or 64MB (2003 mapper) of ROM data
    * Banking: two 64KB ROM banks + one 768KB ROM bank
    * Optional SRAM (one 64KB bank) or EEPROM save data
    * Optional real-time clock

More detailed hardware documentation is listed as part of [awesome-wsdev](https://github.com/WonderfulToolchain/awesome-wsdev).

## Toolchain overview

### Memory models

8086-class CPUs feature *segmentation* - a practice of using *segment registers* (CS - Code Segment, DS - Data Segment, SS - Stack Segment, ES) to extend the address space from 16 bits to 20 bits. This means that we can distinguish two class of pointers:

* *near* pointers - containing only a 16-bit offset; these allow accessing up to 64KB of data;
* *far* pointers - containing a 16-bit segment and a 16-bit offset; these allow accessing the full 1MB of address space.

The Wonderful toolchain currently offers two memory models:

* *small* memory model - one segment (up to 64KB) of code (*near* code pointers),
* *medium* memory model - many segments (more than 64KB) of code (*far* code pointers).

In both cases, *data* is stored using *near* pointers by default, and thus points to RAM. This means that *data* stored in ROM
must be explicitly declared as *far* - see [Caveats](#caveats) for an explanation on how to achieve this.

## Caveats

### C compiler

While gcc-ia16 offers [competitive code generation and optimization](https://wiki.asie.pl/doku.php?id=notes:homebrew:8086_cc_performance) for the architecture, it is a little hackier than dedicated 8086 C compiler solutions.

* In the "small" and "medium" memory models, data is passed around using near pointers, which are limited to RAM. This means that *near* read-only data
  will end up being copied from ROM to RAM, using up limited heap memory, unless they are declared as *far*.

Note, though, that *far* read-only data is not supported on the wwitch target. As such, Wonderful provides a *__wf_rom* define that ensures correct
behaviour on both targets.

Here's an example:

```c
const int8_t neighbor_delta_x[4] = {0, 0, -1, 1}; // stored in RAM
const int8_t __wf_rom neighbor_delta_y[4] = {-1, 1, 0, 0}; // stored in ROM
// __wf_rom is defined to __far on wswan, and nothing on wwitch

// pointer address space modifiers affect functions as well
void write_text(uint8_t x, uint8_t y, const char *text); // only accepts near data pointers (IRAM on wswan, SRAM on wwitch)
void write_text(uint8_t x, uint8_t y, const char __seg_ss *text); // wwitch only: accepts near stack pointers (IRAM)
void write_text(uint8_t x, uint8_t y, const char __wf_iram *text); // only accepts near IRAM pointers
void write_text(uint8_t x, uint8_t y, const char __far *text); // accepts all pointers (ROM, IRAM, SRAM)
```

* In some cases, when calling pointers from arrays of far function pointers in optimization modes >= `-O1`, the code will be miscompiled. This is [a known issue](https://github.com/tkchia/gcc-ia16/issues/120), with no ETA for a fix. One can work around this by annotating the affected function to be compiled without optimizations:

```c
__attribute__((optimize("-O0"))) // https://github.com/tkchia/gcc-ia16/issues/120
static void call_from_my_function_table(uint8_t index) {
	my_function_table[index]();
}
```

* Counter-intuitively, using `-fno-function-sections` in the "medium" memory model can generate *smaller* and *faster* code:
    * In `-ffunction-sections` mode, each *function* is put in its own segment (16-byte alignment), necessiating the use of far calls (10 CPU cycles).
     In `-fno-function-sections` mode, each *compilation unit* is put in its own segment, allowing for more tightly packed code. While all functions are still using "far" calls, an optimization can be made reducing the cost of calls within the same compilation unit to 7 CPU cycles.
* ELF debug symbols are currently not supported, but will be eventually - see [gcc-ia16/#127](https://github.com/tkchia/gcc-ia16/pull/127).

### Assembler

## WonderWitch

The WonderWitch is an official homebrew development environment provided by [Qute Corporation](http://wonderwitch.qute.co.jp/).
Because of the awkward licensing terms and legacy nature of the original software, a decision has been made to try and implement
the target from scratch, using gcc-ia16's modern compiler like in the WonderSwan target. Unfortunately, this poses some challenges:

* While most of the WonderWitch's programming interface is abstracted away via FreyaBIOS (WonderSwan-side system software) interrupt calls,
  some (such as WonderSwan Color routines, dynamic libraries, and file access) are not.
* Under the WonderWitch environment, the data segment points to SRAM (segment 0x1000), while the stack segment points to console RAM
  (segment 0x0000). DS != SS is a somewhat less common thorn in 16-bit 8086 C development and, as such, has limited support. Importantly, unlike
  many of the compilers that came before it, gcc-ia16 is capable of emitting errors when a near stack-originating pointer is mistakenly used as
  a near data-originating pointer, and vice versa.

An important advantage of `libww` (Wonderful's reimplementation of the WonderWitch libraries) is that it is capable of inlining ASM calls
to such FreyaBIOS interrupts, saving the cost of `call` and `ret` instructions on them as a result, as well as allowing the compiler
to allocate and reorder register usage accordingly.

The current status of the WonderWitch target is experimental. It is capable of compiling some non-trivial programs (such as [WWTerm](https://github.com/WonderfulToolchain/WWTerm/)),
but not all components of the original libraries are appropriately supported, and miscompilations/compiler ICEs may occasionally happen.
