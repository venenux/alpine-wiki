
**Overall description:** Alpine Installation from a official disc/iso
burned into DVD/CD to a new computer with [BIOS](alpine-bios-uefi-info.md#bios) 
or [UEFI](alpine-bios-uefi-info.md#uefi) and will be single only boot.

**It means:** that any thing in the computer and their storage will be erased to
put Alpine Linux as main system, using a optical disk media as source install

This document will guide you to ***install Alpine into a new empty or
just fresh PC or Laptop hardware computer**, use if you have a [BIOS or UEFI](../alpine/alpine-and-uefi.md) 
and only wants Alpine Linux into the target computer. For other ways 
of install check the index installation page cases at [Alpine Newbie Install](alpine-newbie-install.md) page

## Terminology

- **[UEFI](alpine-bios-uefi-info.md#uefi) **: it's a new system included
    in every new hardware machine laptop or desktops, that will manage
    the early boot process as a little operating system, see more in the
    [alpine-and-uefi.md](alpine-and-uefi.md) page.
- **New machine**: will be your real machine fresh and ready to
    install your new Alpine operating system, with a installed CD/DVD
    Rom optical drive where to put the burned downloaded disc media
    installation.
- **Source media**: will be the just burned/ disc from the downloaded
    iso file of Alpine operating system. Will be put into the optical
    drive or named [DVD/CD Rom](https://en.wikipedia.org/wiki/CD-ROM) to
    property boot the source disc as media installation.
- **Target media**: will be the storage medium device into the new
    computer target where the Alpine files for operating system will be
    installed, its one partition from the
    [HardDisk](https://en.wikipedia.org/wiki/Hard_disk_drive) of the new
    computer.
- **Optical drive**: will be your hardware drive input to put the
    burned downloaded iso media with the operating system Alpine to
    install as source media; this drive are commonly named [DVD/CD
    Rom](https://en.wikipedia.org/wiki/CD-ROM) unit.

## Requirements

- A blank disc (CD blank or DVD blank or BR blank) to just burn/record
    the source media file downloaded
- In the new machine we need optical drive as input source media
- In the new machine we need at least 512Mb of RAM, but required 2Gb
    of RAM for desktop/graphical applications
- In the new machine we need target media with at least 2G of hard
    disk, but required 10G for desktops
- Will need to previously downloaded and burned the Source media ISO
    file from <https://alpinelinux.org/downloads/>

Detailed requirements are at [../alpine/requirementes.md](../alpine/requirementes.md)

## Preparing the source medium to install

Download the source medium to install and put into your home documents, 
this document will use the DISK IMAGE medium type ( ISO IMG formats)

If the source medium to install download URL will be as following format:
`http://dl-cdn.alpinelinux.org/alpine/v<VERSION>/releases/<ARCH>/alpine-standard-<VERSION>.0-<ARCH>.iso`
where `ARCH` and `VERSION` could be:

- `<ARCH>` will be
    - **x86**: the most used i386 32-bit x86 based machines, if your
        computer are too older use this only.
    - **x86\_64**: the popular AMD64 compatible 64-bit x86 based machines, for
        modern computers use this then.
    - **s390x**: For the Super powered IBM mainframes, use this for 
        especially IBM Z and IBM LinuxONE servers then.
    - **ppc64le**: For the PowerPC devices with pure little-endian
        mode, mostly for POWER8 and POWER9 machines.
- `<VERSION>` could be
    - **latest-stable** for a more up to date without taking care of stability
    - **3.10** the most recommended for machines between 2016 to 2018 mostly pc's
    - **3.14** the most recommended for rasberris and small devices (`x86_64`)
    - **3.16** the most recommended for machines from 2017 up to 2022 and servers

Example, so if we will use 3.10 alpine version the correct links to download will be:

- for **x86\_64** a link to download should be:
    `http://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86_64/alpine-standard-3.10.0-x86_64.iso`
- for **ppc64le** a link to download should be:
    `http://dl-cdn.alpinelinux.org/alpine/v3.10/releases/ppc64le/alpine-standard-3.10.1-ppc64le.iso`

**Graphical download**: Just point the web browser to that url and the
download of the iso file will start. A file with **.iso** extension
type, with name like `"alpine-standard-3.10.0-x86_64.iso"` (if amd64) or
like `alpine-standard-3.10.1-s390x.iso` (if s390x); will be downloaded
commonly into the Download directory of your home or documents filesystem.

**Command line method**: in unix-like terminal execute (by example to download the 3.10 for 64bit pc):
`wget -c -t8 --no-check-certificate http://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86_64/alpine-standard-3.10.0-x86_64.iso`,
and where you run the command, in that place/dir will be downloaded the file.

## Burning the source medium to install

After downloading the source media file from [Alpine download page](https://alpinelinux.org/downloads/) 
put the blank disc into the input optical drive
named [DVD/CDRom](https://en.wikipedia.org/wiki/CD-ROM) and **open your CD/DVD
recording program, choose to "burn from iso file"** and wait the process
will end.

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

## Booting the Alpine ISO disc

When the machine start, you must be sure to choose the optical drive
(commonly named CD/DVD Rom drive), so the disc/iso will boot and after a
while a command line shell will show you:

![Installation : setup-alpine : booting process until login prompt](install-alpine-boot-up-live-01.png)

## Runing the setup install

TODO: The steps here are common to all the normal metos of simple install

## Finishing the installation

After all of the scripts in the setup end, a "reboot" will be offered,
just type "reboot" and press enter, remove the boot media and newly
installed system will be booted.

![install-alpine-alpine-setup-9-setup-disk-3-7-end.png](install-alpine-alpine-setup-9-setup-disk-3-7-end.png)

**You cannot see a graphical window system? take it easy** and get calmed down.. 
in Alpine all are made by the right way.. so **if user need a desktop.. user can (must) install a desktop**

# Documents series

| Previous required                                 | What's next to read                                |
| ------------------------------------------------- | -------------------------------------------------- |
| [Alpine Newbie Prepare](alpine-newbie-prepare.md) | [Alpine Newbie Configs](alpine-newbie-configs.md)  |

# See Also

1.  [Alpine Newbie Install](alpine-newbie-install.md)
2.  [Alpine newbie Desktop](alpine-newbie-desktop.md)

