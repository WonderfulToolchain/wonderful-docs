---
title: "License"
weight: 4
---

# License

The Wonderful toolchain consists of many tools and libraries, which are provided under a variety of FSF/OSI-compliant open source licenses.

Everything listed under "Tools" applies only to the toolchain itself; the output of these tools is generally not copyrighted and may be used freely, unless otherwise specified.

Everything listed under "Libraries" applies additionally to the binaries and programs created with this toolchain, unless otherwise specified.

Example code (`/opt/wonderful/examples`) is typically provided under the Creative Commons 0 license, including the bundled assets, if any.

## Tools

The tools are distributed under a variety of licenses. To learn the specific information for each package, please use the following command:

    $ wf-pacman -Si

## Libraries

### wswan target

* libgcc: [GPLv3+ with GCC Runtime Library Exception](https://github.com/WonderfulToolchain/gcc-ia16/blob/wonderful-gcc-6.3.0-tkchia/COPYING.RUNTIME)
* libc: [Creative Commons 0](https://github.com/WonderfulToolchain/target-wswan-syslibs/blob/main/libc/LICENSE)
* libws: [zlib license](https://github.com/WonderfulToolchain/target-wswan-syslibs/blob/main/LICENSE)
* libwsx: As libwsx is an assortment of third-party code, each header (and its included code) comes with its own license header, including *but not limited to*:
  * `wsx/aplib.h`: zlib license
  * `wsx/lzsa.h`: zlib license
  * `wsx/planar_unpack.h`: zlib license
  * `wsx/zx0.h`: zlib license
* libww: [zlib license](https://github.com/WonderfulToolchain/target-wswan-syslibs/blob/main/LICENSE)

### wwitch target

All of the libraries used on the wswan target also apply to the wwitch target, in addition to:

* libww: [zlib license](https://github.com/WonderfulToolchain/target-wswan-syslibs/blob/main/LICENSE)

## Third-party toolchains

For third-party toolchains distributed as part of Wonderful (such as BlocksDS), please study their relevant documentation.

## Documentation

* wonderful.asie.pl: This website's text contents are available under the terms of the CC BY-SA 4.0 license.
  * [The Swan Book](https://wonderful.asie.pl/doc/wswan/guide/): CC BY-SA 4.0 license for content (text, images, etc.); Creative Commons 0 for example code.
