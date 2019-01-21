# PEL - a lazy modifed MLL, as a toy OS

* [Overview](#overview)
* [Current development state](#current-development-state)
* [Future improvements](#future-improvements)
* [How to build](#how-to-build)
* [Overlay bundles](#overlay-bundles)
* [GraalVM](#graalvm)
* [BIOS and UEFI](#bios-and-uefi)
* [Installation](#installation)
* [Other projects](#other-projects)
* [Thank you!](#thank-you)

---

### Overview

WIP

### Current development state

see source code

### Future improvements

see nothing

### How to build

The section below is for Ubuntu and other Debian based distros.

```
# Resove build dependencies
sudo apt install wget make gawk gcc bc bison flex xorriso libelf-dev libssl-dev

# Build everything and produce ISO image.
./build_minimal_linux_live.sh
```

The default build process uses some custom provided ``CFLAGS``. They can be found in the ``.config`` file. Some of these additional flags were introduced in order to fix different issues which were reported during the development phase. However, there is no guarantee that the build process will run smoothly on your system with these particular flags. If you get compilation issues (please note that I'm talking about compilation issues, not about general shell script issues), you can try to disable these flags and then start the build process again. It may turn out that on your particular host system you don't need these flags.

### Overlay bundles

**Important note!** Most of the overlay bundles come without support since the build process for almost all of them is host specific and can vary significantly between different machines. Some overlay bundles have no dependencies to the host machine, e.g. the bundles which provide the DHCP functionality and the MLL source code. These bundles are enabled by default.

PlainEmbedLinux has the concept of ``overlay bundles``. During the boot process the ``OverlayFS`` driver merges the initramfs with the content of these bundles. This is the mechanism which allows you to provide additional software on top of MLL without touching the core build process. In fact the overlay bundle system has been designed to be completely independent from the MLL build process. You can build one or more overlay bundles without building MLL at all. However, some of the overlay bundles have dependencies on the software pieces provided by the MLL build process, so it is recommended to use the overlay build subsystem after you have produced the 'initramfs' area.

The overlay bundle system provides dependency management. If bundle 'b' depends on bundle 'a' you don't need to build bundle 'a' manually in advance. The bundle dependencies are described in special metadata file ``bundle_deps`` and all such dependencies are prepared automatically.

```
# How to build all overlay bundles.

cd minimal_overlay
./overlay_build.sh
```

```
# How to build specific overlay bundle. The example is for 'Open JDK'
# which depends on many GNU C libraries and on ZLIB. All dependencies
# are handled automatically by the overlay bundle system.

cd minimal_overlay
./overlay_build.sh openjdk
```

### GraalVM

The current development version of MLL partially supports [GraalVM](http://graalvm.org) (provided as overlay bundle). Note that the version of GraalVM in MLL is _release candidate_ which means it may not be very stable. Moreover, GraalVM has runtime dependencies on ``GCC`` and ``Bash`` and therefore some GraalVM feature are not supported in MLL, e.g. the ``gu`` updater and almost all GVM language wrapper scripts, including the ``R`` wrappers. Nevertheless, the core GVM features work fine. Java, Python, Ruby, Node and JavaScript work in MLL/GraalVM environment. Great, isn't it! :)

### BIOS and UEFI

Minimal Linux Live can be used on UEFI systems (as of version ``28-Jan-2018`` of MLL) thanks to the [systemd-boot](https://github.com/ivandavidov/systemd-boot) project. There are three build flavors that you can choose from:

* ``bios`` - MLL will be bootable only on legacy BIOS based systems. This is the default build flavor.
* ``uefi`` - MLL will be bootable only on UEFI based systems. (requires root)
* ``both`` - MLL will be bootable on both legacy BIOS and modern UEFI based systems. (requires root)

The generated MLL iso image is 'hybrid' which means that if it is 'burned' on external hard drive, this external hard drive will be bootable. You can use this behavior to install MLL on your USB flash device (read the next section).

Minimal Linux Live version ``20-Jan-2017`` provides experimental UEFI support and the MLL ISO image can be used on legacy BIOS based systems and on UEFI based systems with enabled UEFI shell (level support 1 or higher, see section ``3.1 - Levels Of Support`` of the [UEFI Shell specification](http://www.uefi.org/sites/default/files/resources/UEFI_Shell_2_2.pdf)).

### Installation

The build process produces ISO image which you can use in virtual machine or you can burn it on real CD/DVD. Installing MLL on USB flash drive currently is not supported but it can be easily achieved by using ``syslinux`` or  ``extlinux`` since MLL requires just two files (one kernel file and another initramfs file). This applies for legacy BIOS based systems.

Another way to install MLL on USB flash drive is by using [YUMI](http://pendrivelinux.com/yumi-multiboot-usb-creator) or other similar tools. This applies for legacy BIOS based systems.

Yet another way to install MLL on USB flash drive is by using the ``dd`` tool:

```
# Directly write the ISO image to your USB flash device (e.g. /dev/xxx)
dd if=minimal_linux_live.iso of=/dev/xxx
```

The USB flash device will be recognized as bootable device and you should be able to boot MLL successfully from it. If you have chosen the 'combined' build flavor (i.e. value ``both`` for the corresponding configuration property), then your USB flash device will be bootable on both legacy BIOS and modern UEFI based systems.

The build process also generates image the file ``mll_image.tgz``. This image contains everything from the initramfs area and everything from the overlay area, i.e. all overlay bundles that have been installed during the MLL build process. You can import and use the image in Docker like this:

```
# Import the MLL image in Docker.
docker import mll_image.tgz minimal-linux-live:latest

# Run MLL shell in Docker:
docker run -it minimal-linux-live /bin/sh
```

### Other projects

* [Minimal Linux Live](https://github.com/ivandavidov/minimal) - The base project.

### Thank you!

Yeah? Donate to MLL, I do not want money.
