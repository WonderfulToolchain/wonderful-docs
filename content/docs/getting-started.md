---
title: "Getting Started"
weight: 1
---

# Getting Started

## Linux

### Requirements

To install Wonderful on your computer, you will need:

* a Linux distribution released sometime in the last decade (maybe a bit more),
* a CPU architecture compatible with x86_64 or AArch64.

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

Once you're done, feel free to install the necessary tools for the target of your choice by following the target-specific instructions.
You may also want to visit [our Discord server](https://discord.gg/CR7MCZNurp) to chat with fellow Wonderful users - an IRC bridge may also be provided, if user demand exists.

<!--    * [Bootstrap (ARMv6)](/bootstrap/wf-bootstrap-arm32v6.tar.gz) -->

### Troubleshooting

* If you run into `error setting certificate file: /etc/ssl/certs/ca-certificates.crt` while trying to use `wf-pacman`, you may need to install your distribution's SSL certificates package (f.e. `ca-certificates` on Debian).

## Windows

Native Windows support is currently not available - a third-party solution may come at a later date. For the time being, it is recommended to use a Linux virtual machine or the [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install).

### WSL2 troubleshooting

* If you run into `error: could not open file: /etc/mtab: No such file or directory` while trying to use `wf-pacman`, the following command can create the missing file: `sudo ln -s /proc/self/mounts /etc/mtab`

## macOS

Native macOS support is currently not available.

For the time being, it is recommended to use a Linux virtual machine.
