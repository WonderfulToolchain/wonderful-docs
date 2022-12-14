# Target: WonderSwan

## Hardware overview

The WonderSwan is a handheld console released by Bandai in March 1999, with an enhanced Color model released later that year. From a brief hardware perspective, it features:

* NEC V30MZ (80186-compatible) CPU @ 3 MHz
    * Note: The V30MZ performs many key operations in 1-3 cycles. It also features a prefetch queue and simple instruction pipeline. It is overall more powerful than a 3 MHz Intel 80186 chip would be.
* 16KB (WS) / 64KB of RAM - shared between software data, video data and audio wavetables
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
    * Color:
        * Sound DMA support
        * Hyper Voice: fifth "channel", stereo samples, headphone output only
* Cartridge:
    * Two 64KB ROM banks + one 768KB ROM bank
    * SRAM (one 64KB bank) or EEPROM save data
    * Optional real-time clock
* Other:
    * Eleven keypad inputs (X1-X4, Y1-Y4, A, B, Start)
    * 9600/38400 bps UART (for console, computer or peripheral connectivity)
    * Color:
        * General ROM->RAM DMA support

## Hardware documentation

 * **[STSWS](http://perfectkiosk.net/stsws.html)** - the most recent documentation source.
 * **[WSMan](http://daifukkat.su/docs/wsman/)** - contains some information not yet in STSWS.
 * [Everything You Never Wanted to Know about the WonderSwan RTC](https://forums.nesdev.org/viewtopic.php?t=21513)

Datasheets for some components are available:

 * [NEC V30MZ Preliminary User's Manual](https://www.renesas.com/us/en/document/lbr/v30mztm-hardware-preliminary)
 * [Seiko S-3511A RTC](http://perfectkiosk.net/S-3511A.pdf)

## Toolchain overview

The Wonderful toolchain for WonderSwan is based on [gcc-ia16](https://github.com/tkchia/build-ia16/), a fork of GCC 6.3.0 targetting 8086-class CPUs.
Additional patches have been developed to feature a V30MZ-aware cost calculator for more effective optimization.

Developing with C is well-supported up to C11. Developing with C++ (up to C++14) is theoretically viable, but currently not supported.

Note: The NEC V30MZ uses distinct opcode and register naming compared to its compatible Intel counterpart. However, for the purposes of this toolchain,
we will be exclusively using Intel names in documentation and code.

### Memory models

8086-class CPUs feature *segmentation* - a practice of using *segment registers* (CS - Code Segment, DS - Data Segment, SS - Stack Segment, ES) to extend the address space from 16 bits to 20 bits. This means that we can distinguish two class of pointers:

* *near* pointers - containing only a 16-bit offset; these allow accessing up to 64KB of data;
* *far* pointers - containing a 16-bit segment and a 16-bit offset; these allow accessing the full 1MB of address space.

The Wonderful toolchain currently offers two memory models:

* *small* memory model - one segment (up to 64KB) of code (*near* code pointers),
* *medium* memory model - many segments (more than 64KB) of code (*far* code pointers).

In both cases, *data* is stored using *near* pointers by default, and thus points to RAM. This means that *data* stored in ROM
must be explicitly declared as *far* - see [Caveats](#caveats) for an explanation on how to achieve this.

### C&lt;-&gt;ASM Calling convention

Wonderful currently uses the `20180813` version of the `regparmcall` calling convention, as defined by gcc-ia16.

#### Function parameter passing

For typical functions, the first three arguments or words, whichever comes first, are passed via the registers
`AX`, `DX` and `CX`, in this order. Bytes are passed via `AL`, `DL` and `CL`. The remaining arguments are pushed
onto the stack.

Arguments are not split between registers and stack. A far pointer, or 32-bit integer, will be passed via `DX:AX`, `CX:DX`, or entirely on the stack.
  
For functions with *variable arguments*, all arguments are pushed onto the stack. It is the callee's responsibility to remove arguments off the stack.

For example, the following function signature:

    void outportw(uint8_t port, uint16_t value);

results in the following calling convention:

 * `AL` = `port`,
 * `DX` = `value`.

The following function signature:

    void __far* memcpy(void __far* s1, const void __far* s2, size_t n);

results in the following calling convention:

 * `DX:AX` = `s1`,
 * stack (4 bytes allocated) = `s2`,
 * stack (2 bytes allocated) = `n`,
 * return in `DX:AX`.

 The following function signature:

    void __far* memcpy(void __far* s1, const void __far* s2, size_t n);

results in the following calling convention:

 * `DX:AX` = `s1`,
 * stack (4 bytes allocated) = `s2`,
 * stack (2 bytes allocated) = `n`,
 * return in `DX:AX`.

#### Function call register allocation

* `AX`, `BX`, `CX`, `DX` can be modified by the callee,
* `SI`, `DI`, `BP`, `DS`, `ES` must be saved by the callee - the value must not be changed after function return relative to the state at function entry.

### Library documentation

* [libws](/doc/libws) - WonderSwan hardware access library

## Installation instructions

TODO

## Testing your homebrew

### Emulation

As of writing, the following emulators are recommended for testing WonderSwan homebrew:

* Ares - higher accuracy, no built-in debugger
* [Mednafen](https://mednafen.github.io/) - lower accuracy, comes with limited debugger

At this time, neither is accurate enough to be considered indistinguishable from real hardware.

### Real hardware

To test WonderSwan homebrew on real hardware, you'll need a *flashcart* and, optionally, an *EXT port adapter* (for serial communications).

#### Flashcart

 * [WS Flash Masta USB](https://www.flashmasta.com/product/ws-flash-masta-usb-cartridge-for-wonderswan/) - ~$100, ships from USA; 16 x 8MB slots, 512KB SRAM, RTC; flashable via USB port.

Older flashcart solutions (WonderDog, WonderMagic Color, etc.) should also work, but have not been tested.

**Note that using a WonderWitch cartridge is not recommended** - they only have 512KB of flash and require overwriting the recovery software to run regular WonderSwan ROMs. (If you have an external WonderSwan cartridge flasher, this option may be permissible as a last resort.)

If you choose the WS Flash Masta, it is additionally recommended to install [CartFriend](https://github.com/WonderfulToolchain/ws-cartfriend/releases/) - an alternate, more compatible and featureful launcher developed by the Wonderful team.

#### EXT port adapter

* [ExtFriend](https://github.com/WonderfulToolchain/ws-extfriend) - DIY option using an RP2040/Raspberry Pi Pico
* [FTDI FT232RL DIY option](https://www.yaronet.com/topics/191502-cable-usb-wonderswan) (Warning: Requires a *genuine* FTDI FT232RL chip, not a clone)
* [RetroOnyx's USB Link Cable](https://www.retroonyx.com/product-page/wonderswan-usb-link-cable) - $85
* WonderWitch RS-232 adapter
* WonderWave IrDA adapter + USB IrDA adapter - allegedly 9600 bps only; if you're adventurous

## Caveats

### C compiler

While gcc-ia16 offers [great code generation and optimization](https://wiki.asie.pl/doku.php?id=notes:homebrew:8086_cc_performance), it is a little hackier than dedicated 8086 C compiler solutions.

* In the "small" and "medium" memory models, data is passed around using near pointers, which are limited to RAM. This means that read-only data
  will end up being copied to RAM, using up limited heap memory, unless they are declared as far. For example:

```c
const int8_t neighbor_delta_x[4] = {0, 0, -1, 1}; // stored in RAM
const int8_t __far neighbor_delta_y[4] = {-1, 1, 0, 0}; // stored in ROM

// this affects functions as well
void write_text(uint8_t x, uint8_t y, const char *text); // only accepts RAM pointers
void write_text(uint8_t x, uint8_t y, const char __far *text); // accepts RAM and ROM pointers
```

* Jump and switch tables in higher optimization modes are also stored in the near data segment. There is no known workaround for this, so jump/switch table generation must be disabled using the `-fno-jump-tables` argument to save RAM.
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

### Assembler

* Using the *segment* of a global symbol in code is currently only possible with a hack - see [binutils-ia16/#6](https://github.com/tkchia/binutils-ia16/issues/6) for a bug report.

## Target: WonderWitch

The WonderWitch is an official homebrew development environment provided by [Qute Corporation](http://wonderwitch.qute.co.jp/).
Because of the awkward licensing terms and dated restrictions of the original software, a decision has been made to try and implement
the target from scratch, using gcc-ia16's modern compiler like in the WonderSwan target. Unfortunately, this poses some challenges:

* While most of the WonderWitch's programming interface is abstracted away via FreyaBIOS (WonderSwan-side system software) interrupt calls,
  some (such as WonderSwan Color routines, dynamic libraries, and file access) are not.
* Under the WonderWitch environment, the data segment points to SRAM (segment 0x1000), while the stack segment points to console RAM
  (segment 0x0000). DS != SS is a somewhat uncommon thorn in 8086 C development and, as such, has limited support. Importantly, unlike
  many compilers that came before it, gcc-ia16 is capable of emitting errors when a near stack-originating pointer is mistakenly used as
  a near data-originating pointer, and vice versa.

An important advantage of `libww` (Wonderful's reimplementation of the WonderWitch libraries) is that it is capable of inlining ASM calls
to such FreyaBIOS interrupts, saving the cost of `call` and `ret` instructions on them as a result, as well as allowing the compiler
to allocate and reorder register usage accordingly.

The current status of the WonderWitch target is experimental. It is capable of compiling some non-trivial programs (such as [WWTerm](https://github.com/WonderfulToolchain/WWTerm/)),
but not all components of the original libraries are appropriately supported, and miscompilations/compiler ICEs may occasionally happen.
As a native WonderSwan target is available and acquiring legitimate WonderWitch cartridges is getting much more expensive than aftermarket
flashcarts, there is less interest in polishing the WonderWitch target - however, contributions are welcome!
