# UEFI and BIOS support on Alpine

UEFI replaces the BIOS firmware interface originally present in all IBM
PC-compatible personal computers, early modern computer's UEFI firmware
implementations provide legacy support for BIOS services.

This document is the most up to date, the oficial wiki page from Alpine 
is currently outdated, please check the [Licensing clarifications](#licensing-clarifications) 
section of this document for any copyright issue.

## Table of Contents

- [About BIOS and UEFI](#about-bios-and-uefi)
  - [The history so far](#the-history-so-far)
- [Alpine UEFI support](#alpine-uefi-support)
  - [Minimum Alpine partition scheme](#minimum-alpine-partition-scheme)
  - [Notes about the boot flags and boot partition](#notes-about-the-boot-flags-and-boot-partition)
  - [Alpine disk layout for UEFI](#alpine-disk-layout-for-uefi)
      - [UEFI/GPT minimal layout](#uefigpt-minimal-layout)
      - [BIOS/MBR minimal layout](#biosmbr-minimal-layout)
      - [BIOS/GPT minimal layout](#biosgpt-minimal-layout)
- [BIOS boot process for newbies](#bios-boot-process-for-newbies)
- [UEFI boot process explained](#uefi-boot-process-explained)
  - [UEFI mandatory partition mechanics](#uefi-mandatory-partition-mechanics)
  - [What's this infamous "Secure Boot"?](#whats-this-infamous-secure-boot)
  - [How to boot unsigned code?](#how-to-boot-unsigned-code)
- [Overall notes and conclusions](#overall-notes-and-conclusions)
  - [Licensing clarifications](#licensing-clarifications)
  - [See also](#see-also)

## About BIOS and UEFI

In the old days, **BIOS**(for **B**asic **I**nput **O**utput **S**ystem)
was how computers booted from the 1980s onwards. But now in newer
hardware for devices, servers, laptops and desktops computers the 
**UEFI**(for **U**nified **E**xtensible **F**irmware **I**nterface) defines a
software interface between an operating system and platform firmware
into the vendor hardware.

## The history so far

All this was driven by a problem in the most extensive and used
architecture: x86 32-bit, inclusivelly a new 2020's Skylake i7-6700k
still has an 80286 embedded in it **because all x86 BIOS strictly only
supports 16-bit 8088-derivative processors**.

Due newer incoming 64-bit incoming processors the older computers boot
process are not more possible. **It started life on Itanium (Intel's first
64-bit processor) systems. Itanium had no support for 32-bit, and
certainly no embedded 80286**, so they had to come up with a different
system.

So then Intel developed the original Extensible Firmware Interface (EFI)
specification. Some of the EFI's practices and data formats mirror those
from M$ Redmon's OS. In 2005, UEFI deprecated EFI 1.10 (the final
release of EFI). The Unified EFI Forum is the industry body that (seems)
"manages" the UEFI specification.

# Alpine UEFI support

Currently are enought for boot most systems, not all the architectures are complete
supported.

The **support for [EFI System Partition](https://en.wikipedia.org/wiki/EFI_system_partition) was
started in the [Alpine 3.7.0 new mayor release](https://alpinelinux.org/posts/Alpine-3.7.0-released.html)**,
preliminary support in that version does not create the [EFI Partition](https://en.wikipedia.org/wiki/EFI_system_partition), 
only was support for existing ones or manually created so you can integrate dual boot for Alpine.

Started **in the [Alpine 3.8.0 new mayor release](https://alpinelinux.org/posts/Alpine-3.8.0-released.html)
support in the installer for the GRUB boot loader was added** so now
Linux experimental users can play with combinations of solutions and
proper [UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface)
complete installations. Please refer to [UEFI_and_BIOS section of this page](#UEFI_and_BIOS_definitions_and_introduction)
first.

Started in [Alpine 3.15 is able to setup UEFI and Secure Boot](https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.15.0#UEFI_Secure_Boot) 
only with grub install flavor, syslinux can able to install UEFI but only with few devices. 
Some users need to setup non grub to work.

**[EFI System Partition](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface#EFI_system_partition)
are not the complete overall of the [UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface),
it's just the need minimal infrastructure to property boot by and [UEFI modern machine](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface#Implementation_and_adoption)..**

> **Warning** check at the [UEFI mandatory partition mechanics](#uefi-mandatory-partition-mechanics) section of this document.

## Minimum Alpine partition scheme

Alpine Linux requires a root partition, but on UEFI systems an EFI, a 
"System Partition" is also required. So a minimun of 3 partitions will be required.

The **EFI System Partition** will be the `/boot` one, it must contain a bootloader 
program in. The current status of that mechanics to boot **in Alpine Linux are still 
in development and has good basic support**. See [UEFI mandatory partition mechanics](#uefi-mandatory-partition-mechanics) 
and [UEFI/GPT minimal layout](#uefi-gpt-minimal-layout) for details.

## Notes about the boot flags and boot partition

**UEFI booting does not involve any "boot" flag, that's it's a need only
for BIOS booting**. The UEFI booting relies solely on the boot entries in
NVRAM. Parted and its front-ends use a "boot" flag on GPT to indicate
that a partition is an "EFI system partition".

**A BIOS "boot partition for EFI" is only required when using GRUB for BIOS 
booting from a GPT disk**. The partition has nothing to do and it must not be
formatted with a file system or mounted.

## Alpine disk layout for UEFI

You will need a disk layout that your system firmware is capable of
booting, you **will need a boot partition and a root partition**. Other
architectures may have different requirements and not all are supported,
please read next sections for details.

If you don't already know what filesystem format you want your boot
partition, choose **ext2**. The **root partition, and any additional
partitions or LVM volume groups, may be in any format that the kernel is
capable of reading**.

#### UEFI/GPT minimal layout

| Mount point   | Partition | Partition type Purpose        | Recommended minimum size | Formats |
|---------------|-----------|-------------------------------|--------------------------|---------|
| /boot or /efi | /dev/sda1 | GPT UEFI Boot partition       | 260 MiB                  | ext2/3/4 |
| /             | /dev/sda2 | Alpine Linux root system OS   | 1–32 GiB                 | btreefs,ext2/3/4,xfs |
| none          | /dev/sda3 | Linux swap memory             | 1-2Gb                    | swap |

#### BIOS/MBR minimal layout

| Mount point | Partition | Partition type Purpose         | Recommended minimum size | Formats |
|-------------|-----------|--------------------------------|--------------------------|---------|
| /boot       | /dev/sda1 | Boot partition **(optional)**  | 100 MiB                  | btreefs,ext2/3/4,xfs |
| /           | /dev/sda2 | Alpine Linux root system OS    | 1–32 GiB                 | btreefs,ext2/3/4,xfs |
| none        | /dev/sda3 | Linux swap memory              | 1-2Gb                    | swap |

#### BIOS/GPT minimal layout

| Mount point | Partition | Partition type Purpose      | Recommended minimum size | Formats |
|-------------|-----------|-----------------------------|--------------------------|---------|
| None        | /dev/sda1 | GPT BIOS boot partition     | 20 MiB                   | ext2/ext3 |
| /           | /dev/sda2 | Alpine Linux root system OS | 1–32 GiB                 | btreefs,ext2/3/4,xfs |
| none        | /dev/sda3 | Linux swap memory           | 1-2Gb                    | swap |

# BIOS boot process for newbies

BIOS mainly supports two methods of booting - loading approximately 448
bytes of 8088 machine code from the start of a floppy disk, or the same
from the start of a fixed IDE disk.

BIOS can only assume one boot loader occupying the start of hard drive.
So each OS overwrites it with its own boot loader. This is very messy.
There's also the 2 TiB issue with MBR.

In order to make your drive more useful, it's split up into partitions -
chunks of disk space which can be treated as independent drives from
inside your OS. Windows (following on from MS-DOS) only supports one
method for partitioning its boot drive on BIOS systems, which is MBR.

MBR cannot handle disks larger than 2 TiB (2<sup>32</sup> × 512 bytes).
Therefore, it is impossible to use any drive space beyond 2 TiB using
MBR layout. So if you're booting from it and use BIOS, you MUST use
MBR - and you simply can't use any space beyond that if your boot drive
is 2TB or bigger.

Modern motherboards (since approximately 2011 onwards) are using UEFI
natively, but most can emulate BIOS through the CSM (Compatibility
Support Module) to maintain support for BIOS-style booting.

# UEFI boot process explained

Well, let's start with installers. It'll read a UDF or FAT32-formatted
USB drive or DVD, and look for the file /efi/boot/bootx64.efi and run
it. An app, written in the UEFI "OS". It can be anything! Here's classic
text adventure Zork, as a UEFI app.

It's possible to make boot media which is valid for both UEFI and BIOS.
Unfortunately, in a slightly user-unfriendly twist, you (the user) need
to pick the right boot entry. For example, on the wife's PC, a USB stick
gets listed as both "UEFI: Sandisk Cruzer Edge" and "USB: Sandisk Cruzer
Edge". Just... make sure you pick the right entry. It's impossible to
change mode after this point.

It uses a different partitioning system called GPT instead of MBR, and
secondly it creates an extra \~100 meg partition called the "EFI System
Partition" - a FAT32 partition where the boot loader apps get installed
to (no more boot sectors).

Each OS will stick its boot loader somewhere in the ESP, then send a
signal to the firmware to write this new loader's location into the
CMOS. Each entry installed in this manner will get its own listing in
your "boot devices" list on the firmware - so if you installed MACOSX,
you'll have "MACOSX Boot Manager" as an entry next to your DVD drive and
hard drive after you reboot. This is why you don't do the old "unplug
drive A when installing a different OS to drive B" thing, or swap
cables, or anything like that. You should only have one ESP, the one on
drive A.

## UEFI mandatory partition mechanics

Regular UEFI boot has several lists of possible boot entries, stored in
UEFI config variables (normally in NVRAM), and boot order config
variables stored alongside them. Unfortunately, a lot of PC UEFI
implementations have got this wrong and so don't work properly.

The correct way for this to work when booting off local disk is for a
boot variable to point to a vendor-specific bootloader program in
`\EFI\$bootloader.efi` on the EFI System Partition (ESP), a specially
tagged partition (Some OS's formatted as Fat32.. that's are unnecessary
due it's just to able to poor OS's to boot like M$ Redmond OS's). The
current status of that mechanics to boot in Alpine Linux are still in
development and only basic support to existing made are provided.

## What's this infamous "Secure Boot"?

It's a way for your motherboard to prevent tampering of your OS (seems 
stupidity of boot-sector viruses??? please!). The UEFI/BIOS provide a 
list of certificates to trust that signed the OS kernels, then the firmware 
enforces that everything involved with the boot process (not just the boot
loader, but the OS kernel itself, and all your device firmware like your
GPU BIOS) are signed with a trusted key.

It works using cryptographic checksums and signatures. It **stops your
system from booting unsigned code**. You can sign your own, and trust
the certificate you used to do that signing. Or you can get the boot
code signed by Microsoft - every motherboard has a small list of
pre-trusted certificates which almost (always) includes Microsoft's
certificates, which they currently let anyone use for a small fee.

Most of the programs that are expected to run in the UEFI environment
are boot loaders, but others exist too. There are also programs to deal
with firmware updates before operating system startup (like fwupdate and
fwupd), and other utilities may live here too.

Support **for [secure boot are since Alpine 3.15](https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.15.0#UEFI_Secure_Boot)
realized by package `secureboot-hook` and `efi-mkkeys`, this means 
that you must load a own signed kernel and put a own certificate** to the UEFI/BIOS.

Due the "Unsigned code curse", Alpine linux [EFI System Partition](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface#EFI_system_partition)
**are not the complete overall of the [UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface),
it's just the need minimal infrastructure to property boot** it!

## How to boot unsigned code?

**Alpine users have to first disable Secure Boot to be able to install
Alpine Linux, cos since supported, it not handle their own certificate** 
and the methods for doing this vary massively from one system to another, 
making this potentially quite difficult for users.

This is due to Microsoft's actions as a Certification Authority (CA) for
Secure Boot. They sign programs/bootloaders on behalf of other trusted
organizations so that their programs will run, but at great cost.. and
there's nothing related to free software but affects to.. There's no
Alpine Linux Certification like are with other enterprise related Linux.

Support **for [secure boot are since Alpine 3.15](https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.15.0#UEFI_Secure_Boot)
realized by package `secureboot-hook` and `efi-mkkeys`, this means 
that you must load a own signed kernel and put a own certificate** to 
the UEFI/BIOS and **not a real direct boot from fresh UEFI/BIOS list.**.

# Overall notes and conclusions

Currently Alpine UEFI and Secure Boot are very basic and enought to work, 
but are just implementations and **not official UEFI listed so Secure Boot must be
disabled at first install**.

BIOS computers or **UEFI computers with Compatibility Support BIOS are
the easiest and most reliable way to install**, they do not need the 
new EFI partition to boot nor new special files.

## Licensing clarifications

This document were started at oficial Alpine wiki, but was over  22:22, 18 August 2019, 
so the wiki licence was pretty simple "are owned by creator" so cannot be redistribute 
without the following license:

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

Complete license at: https://codeberg.org/alpine/alpine-wiki/src/branch/main#license

Original started at: https://wiki.alpinelinux.org/w/index.php?title=Alpine_and_UEFI&oldid=16188

## See also

* [README.md](README.md)
* [alpine-setup-install-script.md](alpine-setup-install-script.md)
