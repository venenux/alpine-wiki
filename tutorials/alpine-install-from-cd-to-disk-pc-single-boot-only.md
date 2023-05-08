# Alpine Install: from a disc to any computer single only boot

**Overall description:** Alpine Installation from a official disc/iso
burned into DVD/CD to a new computer with or without
[UEFI](Alpine_and_UEFI.md) and will be single only boot, 
means that any thing in the computer and their disc will be erased to
put Alpine Linux as main system

This document will guide you to **install Alpine into a new empty or
just fresh PC or Laptop hardware computer, use if you have a [BIOS or
UEFI based hardware](Alpine_and_UEFI.md) and only wants Alpine**
Linux into it.

> **Warning** This method is in disuse today in favor of [usbstiks and imgs](alpine-install-from-usb-to-disk-pc-single-boot-only.md).

Feels lost here? check [How to use this guide](#how-to-use-this-guide) section of this document

## Terminology

-   **[UEFI](Alpine_and_UEFI.md)**: it\'s a new system included
    in every new hardware machine laptop or desktops, that will manage
    the early boot process as a little operating system, see more in the
    [Alpine and UEFI](Alpine_and_UEFI.md) page.
-   **New machine**: will be your real machine fresh and ready to
    install your new Alpine operating system, with a installed CD/DVD
    Rom optical drive where to put the burned downloaded disc media
    installation.
-   **Optical drive**: will be your hardware drive input to put the
    burned downloaded iso media with the operating system Alpine to
    install as source media; this drive are commonly named [DVD/CD
    Rom](https://en.wikipedia.org/wiki/CD-ROM) unit.
-   **Source media**: will be the just burned/ disc from the downloaded
    iso file of Alpine operating system. Will be put into the optical
    drive or named [DVD/CD Rom](https://en.wikipedia.org/wiki/CD-ROM) to
    property boot the source disc as media installation.
-   **Target media**: will be the storage medium device into the new
    computer target where the Alpine files for operating system will be
    installed, its one partition from the
    [HardDisk](https://en.wikipedia.org/wiki/Hard_disk_drive) of the new
    computer.

## Requirements

-   A blank disc (CD blank or DVD blank or BR blank) to just burn/record
    the source media file downloaded
-   In the new machine we need optical drive as input source media
-   In the new machine we need at least 512Mb of RAM, but required 2Gb
    of RAM for desktop/graphical applications
-   In the new machine we need target media with at least 2G of hard
    disk, but required 10G for desktops
-   Will need to previously downloaded and burned the Source media ISO
    file from <https://alpinelinux.org/downloads/>

## Preparing the source medium to install

Download the source medium to install and put into your home documents
in a modern computer. There are more hardware medium sources to
download, like the arm and i386, but ISO CD/DVD images are only to
PC/Laptops that are i386 and amd64, so by downloading the x86 (32bit)
flavor will be same for both cases, but UEFI need 64bit, so change to
the x86_64 (amd64) if your computer is the most modern and lasted
hardware today.

The source medium to install for [UEFI or modern hardware](Alpine_and_UEFI.md) 
**are just 64-bit only**, the download URL will be as following format:
`http://dl-cdn.alpinelinux.org/alpine/v<VERSION>/releases/<ARCH>/alpine-standard-<VERSION>.0-<ARCH>.iso`
where `ARCH` and `VERSION` could be:

-   `<ARCH>` could be one of:
    -   **x86**: the most used i386 32-bit x86 based machines, if your
        computer are too older use this only.
    -   **x86_64**: The popular AMD64 compatible 64-bit x86 based
        machines, i386 are not recommended for newer/lasted hardware.
    -   **s390x**: For the Super powered IBM mainframes, especially IBM
        Z and IBM LinuxONE servers.
    -   **ppc64le**: For the PowerPC devices with pure little-endian
        mode, mostly for POWER8 and POWER9
-   `<VERSION>` could be one of:
    -   **latest-stable** for a more up to date without taking care of
        numbered
    -   **3.12** the most recommended for machines between 2016 to 2018

EXAMPLE if you plan **to using 3.17 version the available links to download will be:**

-   for **x86_64** computers:
    `http://dl-cdn.alpinelinux.org/alpine/v3.17/releases/x86_64/alpine-standard-3.17.0-x86_64.iso`
-   for **x86** older BIOS computers we recommend:
    `http://dl-cdn.alpinelinux.org/alpine/v3.12/releases/ppc64le/alpine-standard-3.12.1-x86.iso`
-   for **ppc64le** machines
    `http://dl-cdn.alpinelinux.org/alpine/v3.10/releases/ppc64le/alpine-standard-3.10.1-ppc64le.iso`

**Usin Graphical download way**: Just point the web browser to that url and the
download of the iso file will start. A file with **.iso** extension
type, with name like `"alpine-standard-3.17.0-x86_64.iso"` (if amd64) or
like `alpine-standard-3.12.1-x86.iso` (if i386); will be downloaded
commonly into the Download directory of your home documents filesystem.

**Usin Command line method way**: in unix-like terminal execute:
`wget -c -t8 --no-check-certificate http://dl-cdn.alpinelinux.org/alpine/v3.17/releases/x86_64/alpine-standard-3.17.0-x86_64.iso`,
and where you run the command, in that place/dir will be downloaded the file.

## Burning the source medium to install

After downloading the source media file from [Alpine download page](https://alpinelinux.org/downloads/) 
**put the blank disc into the input optical drive** named [DVD/CD Rom](https://en.wikipedia.org/wiki/CD-ROM) 
and **open your CD/DVD recording program, choose to "burn from iso file"** and wait the
process will end.

In detail if you downloaded with **Graphical download** (using a web
browser), the source media file will be into the Download directory. If
you downloaded with **Command line method** your source file probably
will be in your root document home (or just `$HOME` of your Linux
install or MAC install filesystem).

In Linux, assuming the blank disc is in the optical drive, the command
to record/burn the downloaded source media file is :

`$ umount /dev/sr0;cdrecord -v -sao dev=/dev/sr0 alpine-standard-3.10.0-x86_64.iso`

If your blank media is a DVD or BD disc the command will be then :

`$ umount /dev/sr0;growisofs -dvd-compat -Z /dev/sr0=alpine-standard-3.10.0-x86_64.iso`

> **Note** `growisofs` has a small bug with blank BD-R media. It issues an error message after the burning is complete. Programs like k3b then believe the whole burn run failed.}}

## Booting the Alpine ISO disc

When the machine start, you must be sure to choose the optical drive
(commonly named CD/DVD Rom drive), so the disc/iso will boot and after a
while a command line shell will show you:

> **Note** When starting Alpine it will ask for the login, just typing root and pressing enter allows you to start:

![](https://venenux.github.io/alpine-espanol/instalar/instalar-desde-virtualbox-a-discoreal-dualboot-screenshot-01.png)

> **Warning** Tip: If your system is not configured to boot from a USB drive, it must be configured in the BIOS, '''ask/search to your vendor or technical support''', Toshiba computers need to hit F1 to choose boot medium, DELL must hit F11 to choose medium for example, and so and so}}

#### after start to setup the script

For this use case, unlike other installation systems, Alpine's is automatic on the disk, 
and mounts several partitions depending on the case, as in this document the procedure 
is automatic, at least four partitons will be created by the installer and these will be 
configured as needed, if it is UEFI or BIOS, always the automatic setup will do that.

> **Warning** The drive will be assumed to be fully used since alpine setup in this guide 
will be on its own and single setup only, it will not be necessary to partition or format 
as the installer will do it. Assumes a minimum 4 gig disk where swap will be 2 gigs in sizes.

> **Warning** if you do not download the extended ISO it may require internet.!!!

#### runing the setup script

After entering the root environment and gets the console prompt installation media, 
you must perform these commands, that will:

* prepare disk sizes, boot partiton to 500 megs
* perpare swap sices, swap partition to 2  Gigs
* setup boot loader to grup (if wants lilo just change to lilo)
* and run setup script to configure and process instalation:

```
export BOOT_SIZE=500

export SWAP_SIZE=2048

export BOOTLOADER=grub

setup-alpine
```

This will start some questions, these are in the following order:

* keyboard and variant, example for Latin is esand afteres-winkeys
* hostname: just hit enter, it's the name of the computer.
* Network options: select the `eth0` one that is the network cable and answer `dhcp`.
* DNS Options: It is recommended to use `8.8.8.8 ` and `none` for the domain
* Time zone options: Just use the suggested defaults.
* Proxy Options: Use noneif you are connecting directly to the Internet.
* SSH Options: Use opensshthe package that already comes in the middle.
* NTP Options: Use chronythe packet already in the middle.
* Mode: Select `sys` to install the system on disk.
* Disk Options: Use `sda` as the entire hard drive present will be used.

After answering `sys` to the questions about the drive, and since there will only 
be one drive, answering `sda` on which drive to use, this will create and leave 
your hard drive as follows:

* `/dev/sda1` as BOOT in 500Mb in `/boot`
* `/dev/sda2` as SWAP in 4Gb
* `/dev/sda3` as ROOT in 200Gb in `/` (approximately or rest of space available)

In a few minutes everything will be ready to use ofering a console when boot new system.

![](https://venenux.github.io/alpine-espanol/instalar/install-alpine-alpine-setup-3-setup-scripts.png)

#### custom setup boot loader

If the new local system was configured to run in `diskless` or `data` mode, or if you 
choose do not install boot loader, and you do not want keep booting from the initial 
(and possibly read-only) installation media, the boot system needs to be copied to 
another device or partition or setup manually.

The target partition may be identified using lsblk (after installing it with apk add lsblk) 
and/or blkid, similar to previously identifying the initial installation media device.

The procedure to copy the boot system is using setup-bootable

Once everything is in place, but is you use `diskless` or `data` save your customized 
configuration with `lbu commit` before rebooting.

## Finishing the installation

After all of the scripts in the setup end, a "reboot" will be offered,
just type "reboot" and press enter, remove the boot media and newly
installed system will be booted.

**You cannot see a graphical window system? take it easy** and get
calmed down.. in Alpine all are made by the right way.. so **if user
need a desktop.. user can install a desktop**


## How to use this guide

This guide **structure all the commands in blocks, each block its separated by a line spaced**, 
so you must **type each line as is.. and hit enter**, so you noted that then you 
typed each separated clocks of commands, copy/type only blocks separated by an empty line, 
all new(next) lines are made by just enter. the terminal will detect if must execute or not.

This guide is for install process, many parts will need you understand minimal 
knowledge of linux.

This guide assumed you have a serial port allowed in the targeted computer, also 
its important you shuold understand the way of the configuration in this guide.

> **Warning**  Some Linux or/and Mac terminals have security cut/paste locks, so 
if you paste, the first line will be preceded by garbage, check always the first char of your paste.

## Licensing clarifications

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

https://codeberg.org/alpine/alpine-wiki/src/branch/main#license

## See also

* [README.md](README.md)
* [alpine-newbie-install.md](../../newbie/alpine-newbie-install.md)
* [alpine-tutorial-desktop-wayland-try.md](alpine-tutorial-desktop-wayland-try.md)
* [alpine-tutorial-desktop-xfce4-fast-forward.md](alpine-tutorial-desktop-xfce4-fast-forward.md)
