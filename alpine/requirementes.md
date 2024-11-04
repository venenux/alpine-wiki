
This page will tell you what requirements you will need to use the
[Alpine Linux](about.md) operating system.

* [Hardware requirements](requirementes#hardware-requirements)
* [Software requirements](requirementes#software-requirements)

# Hardware requirements

For installation and usage consider that Alpine can run on several kinds
of devices, from the popular PC machine to video game consoles like the
Game Boy Advance and the 3DS, and as such you must check the following
of your machine:

- [Architecture](requirementes#architecture)
- [Memory](requirementes#memory)
- [Storage](requirementes#storage)
- [Peripherals](requirementes#peripherals)

## Architectures

**Architecture means kind of computer. The most popular architecture is
the misnamed "Intel IBM PC"** or "i386" which is actually in fact the
x86 or x64. There are other supported computer architectures that are
not "x86", like mainframes, servers, and embedded devices (such as
routers like Sonicwall and Cisco ones). Here are the architectures
supported by Alpine:

| Supported Arch | since   | until   | Meaning of installation and target architecture                                                            |
| -------------- | ------- | ------- | ---------------------------------------------------------------------------------------------------------- |
| x86\_64        | all     | current | The popular AMD64 compatible 64-bit x86 based machines, i386 is not recommended for newer/latest hardware. |
| x86            | all     | v3.17   | The all popular 32 bit intel (i386 pc 32bit) and x86\_64 with 32bit compatible (i686 pc 64bit amd64)       |
| ppc64le        | v3.6    | current | For the PowerPC devices with pure little-endian mode, mostly for POWER8 and POWER9                         |
| armhf          | v3.0    | current | The newer ARM hard-float for newer, more powerful 32-bit devices alongside 64-bit. Including video games\! |
| armv7          | v3.9    | current | The 32-bit ARM only execution state of the ARMv7 devices machines. Including video game consoles\!         |
| aarch64        | v3.5    | current | The 64-bit ARM only execution state of the ARMv8 device machines. Like Rasberri's                          |
| ppc64le        | v3.6    | current | for 64-bit big-endian PowerPC and Power ISA processors like some MAC computers.                            |
| s390x          | v3.6    | v3.15   | For the Super powered IBM mainframes, especially IBM Z and IBM LinuxONE servers.                           |

#### CPU

* Intel
    * Core i3 8121U are supported from alpine v3.6 to v3.14 only.
    * From 486 to pentium/pentiumII/pentium3 supported up to v3.16 only
* AMD
    * Semprom are pretty slower with recent kernels, so its practically not supported, use older versions of alpine or own build kernel
    * DX*SX 486 supported up to alpine v3.16 only
    * K5 and K6 supported up to alpine v3.16 only

## Memory

**Means minimum amount of RAM memory. Need of RAM it depends of the
meaning of the installation**, any hardware are supported and there is
minimum sizes for:

| Target Arch | Mim RAM to start | Min RAM to install | Min RAM for GUI | Best for GUI work |
| ----------- | ---------------- | ------------------ | --------------- | ----------------- |
| x86\_64     | 512 Megs         | 512 Megs           | 3 Gigs          | 8 Gigs            |
| x86\_32     | 128 Megs         | 256 Megs           | 1 Gigs          | 4 Gigs            |
| ppc64le     | 128 Megs         | 256 Megs           | 1 Gigs          | 8 Gigs            |
| armhf       | 256 Megs         | 512 Megs           | 2 Gigs          | 6 Gigs            |
| armv7       | 256 Megs         | 512 Megs           | 2 Gigs          | 6 Gigs            |
| aarch64     | 256 Megs         | 512 Megs           | 2 Gigs          | 8 Gigs            |
| ppc64le     | 256 Megs         | 512 Megs           | 1 Gigs          | 6 Gigs            |
| s390x       | 128 Megs         | 256 Megs           | 1 Gigs          | N/A               |

## Storage

**Means any external or internal storage device that can be added after
or before install to use** by the Alpine Linux system. Currently depends
of the current linux kernel supported.

**All the PATA and SATA hard disk drives are supported, also any USB or
SD** card that can be detected by USB BUS by the linux kernel subsystem
during install.

## Peripherals

**Means any external or internal device that can be added after or
before install to detectd** by the Alpine Linux system. Currently
depends of the current linux kernel supported.

##### ISA devices

**ISA devices** are not supported since 3.8 because kernel drops support.
those pc machines must use an older alpine linux, our wiki has good recipes 
for such cases. To support those devices use v3.8.0 alpine version as maximun.

##### GPU devices

**GPU devices** are supported, but for advanced features, 3D acceleration
are manager by MESA project:

- Intel: mostly any Intel by one exception, intel i810/i815 will lack
    of features cos only has 4Mb memory, Mesa and Linux drop theit
    support. Recent devs wants to deprecated intel support on mesa.
- ATI/AMD, only radeon series with exception of recent two last years
    respect Alpine release, Rage r128/match64 series has limited
    support. Recently AMD "Next" gen are only basic supported.
- Nvidia: limited; only few are complete supported\! not all features
    are allowed\! Needs 5.10 ad up kernels and have issues with 
    power management. Recommended to disable Display power management.
- Matrox: not all features are supported, this is shipped on most
    servers. Those GPU has 3D support, but with newer kernels due the 
    increment of requirementes that is practically not usefully.
- Sis: limited features are supported, since code are not updated on
    Xorg and Linux kernel
- Via: limited features are supported, since openchrome code are not
    updated on Xorg and Linux kernel

##### WIFI devices

**WIFI devices** are supported well if are not hybrid ones, nomadays 
currently wifi devices are now hybrids that are mostly mix of bluetooth and wifi.

Note that bluetooth adapter, while on the same card as your wifi will 
have a seperate hardware ID but both will be reconiced always as USB devices.

Mostly mayor of those are not well suported unless you use kernel 5.10 and up, 
so the recommendations for recent hybrits devices are Alpine v3.16 and up. 
the only problem are few modules like Broadcom (that some not matter if 
are older or newer will require compilation and firmware) and the 
Realtek Semiconductor only if your device are so so recent.

* prism54 FullMAC PCI / Cardbus devices used to be supported only by the
  prism54 wireless driver its not supported by alpine, only debian 4, 5, and 6 supports


# Software requirements

- [Media](requirementes#media)
- [Booting](requirementes#booting)
- [Storage](requirementes#storage)
- [Firmware](requirementes#firmware)

## Media

**Means the files need for dump the install media, and later boot from
the target install** machine, of course downloaded from
http://dl-cdn.alpinelinux.org/alpine/latest-stable/releases or main Download page.

| Available for | ISO (for USB, CD/DVD) | IMG (for Netboot) | TAR (for ROOTFS img) | Download links recommended                                             |
| ------------- | --------------------- | ----------------- | -------------------- | ---------------------------------------------------------------------- |
| x86\_64       | YES                   | YES               | N/A                  | http://dl-cdn.alpinelinux.org/alpine/latest-stable/releases/x86_64/  |
| x86           | YES (best is v3.12.0) | YES               | N/A                  | http://dl-cdn.alpinelinux.org/alpine/v3.12/releases/x86/     |
| ppc64le       | NO                    | YES               | YES                  | http://dl-cdn.alpinelinux.org/alpine/latest-stable/releases/ppc64le/ |
| armhf         | NO                    | YES               | YES                  | http://dl-cdn.alpinelinux.org/alpine/latest-stable/releases/armhf/   |
| armv7         | NO                    | YES               | YES                  | http://dl-cdn.alpinelinux.org/alpine/latest-stable/releases/armv7/   |
| aarch64       | YES                   | YES               | YES                  | http://dl-cdn.alpinelinux.org/alpine/latest-stable/releases/aarch64/ |
| mips64        | YES (until v3.14.0)   | YES               | N/A                  | https://dl-cdn.alpinelinux.org/alpine/v3.13/releases/mips64/  |
| s390x         | YES                   | YES               | N/A                  | http://dl-cdn.alpinelinux.org/alpine/v3.15/releases/s390x/   |

For some architectures, those pc machines must use an older alpine linux, 
our wiki has good recipes for such cases, like the `x86` knowed widely 
as `i386` or `32bit pc`, its better to use the recomended download link, 

Contrary to everything that stupids devs will say about using up to date, fashioned 
and latest versions, **the age of these devices will require that you do 
not use software so modern things** that it always increases the requirements 
to do the same task as any recent version of same.

## Booting

**Means support for kind of BIOS/UEFI/OEM setup of machine, and where can be
media downloaded will be boot**.

| Supported Arch | Supported BIOS         | Supported Types | Media Boot Recommended       |
| -------------- | ---------------------- | --------------- | ---------------------------- |
| x86\_64        | Coreboot, Vendor/OEM   | BIOS, UEFI      | **USB**, CD/DVD (ISO)        |
| x86            | Coreboot, Vendor/OEM   | BIOS, UEFI      | **USB**, CD/DVD (ISO)        |
| ppc64le        | Coreboot, Vendor/OEM   | BIOS, UEFI      | **USB**, CD/DVD (ISO)        |
| armhf          | Uboot, Vendor/OEM      | BIOS            | **NET**, MINIROOTFS (TAR.GZ) |
| armv7          | Uboot, Vendor/OEM      | BIOS, UEFI      | **NET**, MINIROOTFS (TAR.GZ) |
| aarch64        | ?Coreboot?, Vendor/OEM | BIOS, ?UEFI?    | **USB**, CD/DVD (ISO)        |
| mips64         | Vendor/OEM             | ?               | v3.14.0 end of support       |
| s390x          | Vendor/OEM             | BIOS, ?UEFI?    | **USB**, CD/DVD              |

#### Boot process

The boot process for most common computer are described at 
the [alpine-boot-uefi-bios.md](alpine-boot-uefi-bios.md) document.

The Uboot process for most common devices are described at 
the [apine-boot-uboot.md](alpine-boot-uboot.md) except for Odroid-C2 devices..

If the computer does not automatically boot from the desired device, one
needs to bring up the boot menu selection for choosing the media to boot
from. Depending on the computer the menu may be accessed by quickly
(repeatedly) pressing a key when booting starts, or sometimes it is
needed to press the button before starting the computer and keep holding
it when it boots. Typical keys are: `F9`-`F12`, sometimes `F7` or
`F8`. If these don't bring up the boot menu, it may be necessary to
enter the BIOS configuration and adjust the boot settings, for which
typical keys are: `Del.` `F1` `F2` `F6` or `Esc.`

## Space

**This means amount of available space in disk partitions to perform a
kind of install** and of course will depends of type and meaning of your
desired install, this are the recommended sizes but depends of the
[BIOS/UEFI support and disk layout](alpine-and-uefi.md) wiki page.

| Minimum sizes   | Partition for BOOT (`/boot`) | Partition for ROOT (`/`) | Partition for HOME (`/home`) | Partition for SWAP (`N/A`) |
| --------------- | ---------------------------- | ------------------------ | ---------------------------- | -------------------------- |
| base only       | 100 Megs                     | 500 Megs                 | 1+ Gigs                      | Optional                   |
| default server  | 200 Megs                     | 2 Gigs                   | 2 Gigs                       | 4 Gigs                     |
| default desktop | 250 Megs                     | 120 Gigs                 | 320 Gigs                     | 8 Gigs                     |
| mail server     | 200 Megs                     | 80 Gigs                  | 120+ Gigs                    | 8 Gigs                     |
| web server      | 200 Megs                     | 10 Gigs                  | 20+ Gigs                     | 8 Gigs                     |

## LICENSE

**CC BY-NC-SA**: the project allows users to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [alpine/copyright.md](copyright.md)

# See Also

* [about.md](about.md)
* [contribution.md](contribution.md)
* [README (index)](README.md)
* [README (main)](../README.md)

* [Hardware requirements](requirementes#hardware-requirements)
    - [Architecture](requirementes#architecture)
    - [Memory](requirementes#memory)
    - [Storage](requirementes#storage)
    - [Peripherals](requirementes#peripherals)
* [Software requirements](requirementes#software-requirements)
    - [Media](requirementes#media)
    - [Booting](requirementes#booting)
    - [Storage](requirementes#storage)
    - [Firmware](requirementes#firmware)

