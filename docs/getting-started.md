# Getting Started/Installation

## Linux

### Requirements

To install Wonderful on your machine, you will need:

* a Linux distribution released sometime in the last decade, maybe a bit more,
* a CPU architecture compatible with x86_64, ARMv6, or AArch64.

x86_64 is best supported; ARMv6 and AArch64 receive slower/less frequent updates due to the lack of an ARM build server.

### Installation instructions

1. Download the Wonderful bootstrap:
    * [Bootstrap (x86_64)](/bootstrap/wf-bootstrap-x86_64.tar.gz)
    * [Bootstrap (AArch64)](/bootstrap/wf-bootstrap-aarch64.tar.gz)
    * [Bootstrap (ARMv6)](/bootstrap/wf-bootstrap-arm32v6.tar.gz)
2. Create the `/opt/wonderful` directory and give permissions to the user who will make use of the toolchain (f.e. `chown -R [user] /opt/wonderful`)
3. Extract the bootstrap to `/opt/wonderful`. Other installation locations are not supported.
4. Export *WONDERFUL_TOOLCHAIN* to point to `/opt/wonderful` (f.e. `export WONDERFUL_TOOLCHAIN=/opt/wonderful`).
5. Add `/opt/wonderful/bin` to *PATH* (f.e. `export PATH=/opt/wonderful/bin:$PATH`).
6. Run `wf-pacman -Syu` (no sudo - you don't need root) to synchronize and update the toolchain's package manager.

For further installation instructions, refer to the specific target(s) you'd like to work with.

## Windows

Native Windows support is currently not available - a third-party solution may come at a later date.

For the time being, it is recommended to use a Linux virtual machine or the [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install).

## macOS

Native macOS support is currently not available.

For the time being, it is recommended to use a Linux virtual machine.
