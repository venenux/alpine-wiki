# UEFI and BIOS support on Alpine

UEFI replaces the BIOS firmware interface originally present in all IBM
PC-compatible personal computers, early modern computer's UEFI firmware
implementations provide legacy support for BIOS services.

UBOOT are a boot process for embebed devices and minidevices, pretty 
mostly present at the single board computers and some phones.

The complete information is at the [alpine/requirementes.md booting](../alpine/requirementes.md#booting) section.

This document is the most up to date, the oficial wiki page from Alpine 
is currently outdated, please check the [Licensing clarifications](#licensing-clarifications) 
section of this document for any copyright issue.

## About BIOS and UEFI

In the old days, **BIOS**(for **B**asic **I**nput **O**utput **S**ystem)
was how computers booted from the 1980s onwards. But now in newer
hardware for devices, servers, laptops and desktops computers the 
**UEFI**(for **U**nified **E**xtensible **F**irmware **I**nterface) defines a
software interface between an operating system and platform firmware
into the vendor hardware.

> **Note** Consult more at [../alpine/alpine-boot-uefi-bios.md](../alpine/alpine-boot-uefi-bios.md)

## Where i will find BIOS based devices

Any computer until 2010 or older will only have a BIOS system to boot. Any IBM pc 
computer wil have also BIOS and any coputer until 2014 will have almost UEFI legacy 
support to boot like BIOS.

On those system marked as older harware you will need a MBR based partition 
table only, and cannot use GTP partitions.

## Where i will find UEFI based devices

Any computer since 2014 will have UEFI for sure, but 2013/2014 ones wil have 
buggy UEFI implementations. Its recommended to avoid any computer from 2013 
or 2014, and if you have a BIOS+UEFI then enabled the legacy mode.

Also any Vendor marked computer bqased on x86 or 64bit, or any 64bit 
computer will have UEFI, specially those that are parnership with Mocosoft.

# Alpine UEFI support

Currently are enought for boot most systems, not all the architectures are complete
supported. Since 3.16 Alpine is able to setup UEFI only with grub install flavor, syslinux 
can able to install UEFI but only with few devices. Some users need to setup non grub to work.

> **Warning** Mayor information is at [alpine/requirementes.md booting](../alpine/requirementes.md#booting) section document.

## Minimum Alpine partition scheme

Alpine Linux requires a root partition, but on UEFI systems an EFI, a 
"System Partition" is also required. So a minimun of 3 partitions will be required.

**UEFI booting does not involve any "boot" flag, that's it's a need only
for BIOS booting**. 

**A BIOS "boot partition for EFI" is only required when using GRUB for BIOS 
booting from a GPT disk**. The partition has nothing to do and it must not be
formatted with a file system or mounted.

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

## Secure Boot Support

Support **for [secure boot are since Alpine 3.15](https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.15.0#UEFI_Secure_Boot)
realized by package `secureboot-hook` and `efi-mkkeys`, this means 
that you must load a own signed kernel and put a own certificate** to the UEFI/BIOS.

Due the "Unsigned code curse", Alpine linux [EFI System Partition](#uefi-gpt-minimal-layout)
**are not the complete overall of the [Secure Boot](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface),
it's just the need minimal infrastructure to property boot** it!

Is **recommended to disable Secure Boot. Alpine has no own certificate, 
the process only permit to load your own certificate to your UEFI BIOS, 
it does not have a certificate** which some other Linux distributions
(mostly enterprise-related) have.

> **Warning** for more information about please check [alpine/alpine-boot-uefi-bios.md Secure Boot](../alpine/alpine-boot-uefi-bios.md#secure-boot) section document.

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
* [alpine-setup-install-script.md](../alpine/alpine-setup-install-script.md)
* [alpine/alpine-boot-uefi-bios.md](../alpine/alpine-boot-uefi-bios.md)
