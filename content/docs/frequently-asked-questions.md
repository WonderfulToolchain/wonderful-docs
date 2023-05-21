---
title: "Frequently Asked Questions"
weight: 2
---

# Frequently Asked Questions

## Installation

### Why is the toolchain available only for Linux systems?

* The toolchain is primarily maintained by one person; even factoring in pull requests, this means the amount of time available to develop the project is rather low.
Personally, I believe there is more benefit to and I personally hold more interest in working on libraries, target platform support, and similar features than maintaining
support for different host operating systems.
* Some operating systems (like Windows or macOS) require additional spending to legally build, maintain and test their respective versions; at the same time,
free-of-charge solutions exist for running Linux on most other operating systems.

### Can I use a system-provided version of the Pacman package manager?

No, and it is unlikely to ever be possible. `wf-pacman` is customized to (a) treat `/opt/wonderful` as the root directory and (b) allow running without root.

* This approach allows reducing concerns of safety and security over a package manager that can write to the entire system space.
* The Wonderful toolchain wishes to minimize system configuration divergence while still supporting diverse configurations. Standardizing components of the toolchain helps maintain that goal.
