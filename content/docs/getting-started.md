---
title: "Getting Started"
weight: 1
---

# Getting Started

The Wonderful toolchain can currently be installed on:

* Linux
* Windows (via MSYS2)

Once you're done following this tutorial, feel free to install the necessary tools for the target of your choice by following the target-specific instructions.
You may also want to visit [our Discord server](https://discord.gg/CR7MCZNurp) to chat with fellow Wonderful users - an IRC bridge may also be provided, if user demand exists.

## Linux

### Requirements

To install Wonderful on your computer, you will need:

* a Linux distribution released sometime in the last decade (maybe a bit more),
* a CPU architecture compatible with x86_64 or AArch64,
* the following command-line tools provided by your distribution: `bash`, `git`, `make`.

### Downloads

 * [Bootstrap (x86_64)](/bootstrap/wf-bootstrap-x86_64.tar.gz)
 * [Bootstrap (AArch64)](/bootstrap/wf-bootstrap-aarch64.tar.gz)

### Installation instructions

1. Create the `/opt/wonderful` directory: `mkdir /opt/wonderful`. Other installation locations are not supported at this time.
2. Give permissions to `/opt/wonderful` to the correct user (f.e. `chown -R [user] /opt/wonderful`).
3. Extract the bootstrap to `/opt/wonderful` (f.e. `cd /opt/wonderful/ && tar xzvf [path_to_bootstrap_tar_gz]`).
4. Add `/opt/wonderful/bin` to *PATH* (f.e. `export PATH=/opt/wonderful/bin:$PATH`).
5. Export *WONDERFUL_TOOLCHAIN* to point to `/opt/wonderful` (f.e. `export WONDERFUL_TOOLCHAIN=/opt/wonderful`).
6. Run `wf-pacman -Syu` (no sudo - you don't need root) to synchronize and update the toolchain's package manager.

<!--    * [Bootstrap (ARMv6)](/bootstrap/wf-bootstrap-armv6h.tar.gz) -->

### Troubleshooting

* If you run into `error setting certificate file: /etc/ssl/certs/ca-certificates.crt` while trying to use `wf-pacman`, you may need to install your distribution's SSL certificates package (f.e. `ca-certificates` on Debian).

## Windows (via MSYS2)

### Requirements

To install Wonderful on your computer, you will need:

* a recent version of Windows - Windows 10 and above are supported,
* a CPU architecture compatible with x86_64.

### Downloads

 * [Bootstrap (x86_64)](/bootstrap/wf-bootstrap-windows-x86_64.tar.gz)

### Installation instructions

1. Install [the MSYS2 environment](https://www.msys2.org/).
2. Install some useful packages: `pacman -S base-devel ca-certificates mingw-w64-ucrt-x86_64-ca-certificates`.
3. Create the `/opt/wonderful` directory: `mkdir /opt/wonderful`. Other installation locations are not supported at this time.
4. Extract the bootstrap to `/opt/wonderful` (f.e. `cd /opt/wonderful/ && tar xzvf [path_to_bootstrap_tar_gz]`).
5. Add `/opt/wonderful/bin` to *PATH* (f.e. `export PATH=/opt/wonderful/bin:$PATH`).
6. Export *WONDERFUL_TOOLCHAIN* to point to `/opt/wonderful` (f.e. `export WONDERFUL_TOOLCHAIN=/opt/wonderful`).
7. Run `wf-pacman -Syu` to synchronize and update the toolchain's package manager.

<!--    * [Bootstrap (ARMv6)](/bootstrap/wf-bootstrap-arm32v6.tar.gz) -->

### Alternative: Windows (via WSL2)

Since WSL2 is compatible with Linux, one can follow the Linux installation instructions to get Wonderful running on top of Windows this way.

* If you run into `error: could not open file: /etc/mtab: No such file or directory` while trying to use `wf-pacman`, the following command can create the missing file: `sudo ln -s /proc/self/mounts /etc/mtab`

## macOS

Native macOS support is currently not available. For the time being, it is recommended to use a Linux virtual machine.
