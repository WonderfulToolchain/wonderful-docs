---
title: "Frequently Asked Questions"
weight: 2
---

# Frequently Asked Questions

## Installation

### Can I use a system-provided version of the Pacman package manager?

No, and it is unlikely to ever be possible. `wf-pacman` is customized to (a) treat `/opt/wonderful` as the root directory and (b) allow running without root.

* This approach allows reducing concerns of safety and security over a package manager that can write to the entire system space.
* The Wonderful toolchain wishes to minimize system configuration divergence while still supporting diverse configurations. Standardizing components of the toolchain helps maintain that goal.
