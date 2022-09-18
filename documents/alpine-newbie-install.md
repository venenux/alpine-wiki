Alpine newbies install
======================

**Alpine Linux can be installed via many ways**, the most common ones
are listed here, for more information see last section of this document.

- **Computer device** like PC, laptop, or Raspberry, are forms or a
name for machines that uses the software named "Operating System"
like Alpine Linux, the target of a "install process" to property put
the Alpine system.
- **Image file** means it is a type of file that is downloaded and
*burned to a CD/DVD* or *dumped to a USB* with popular programs
(such as Nero, CloneCD, Brasero), although in the modern era it's
a deprecated way due nes kind of devices (Raspberri's, Phone's).
- **Boot** or **[Booting](alpine-bios-uefi-info.md)** means "Boot"
(started from a media source) a "image" recorded on DVD/CD or USB from
the disc drive or from a USB port respectively, the computer boots
this device and loads the Alpine operating system. Today that means
involved with [BIOS and UEFI](alpine-bios-uefi-info.md) systems.

**[Setup\_modes of Alpine
operation](Alpine_setup_scripts#Setup_modes "wikilink")**: Alpine Linux
is designed to run from RAM directly, which means the download image is
already a fully installed, but a minimally configured system. Review the
**[Setup\_modes of Alpine
operation](Alpine_setup_scripts#Setup_modes "wikilink")** for permanent
installation options on most devices.

## Additional Details

## Requirements

Alpine Linux has low system requirements. Nowadays almost hardware is
supported, More requirements documentation can be found on the
[Requirements](alpine-requirements.md) wiki page:

### Memory

  - At least 128 MB of RAM for a server without a GUI.
  - At least 1.6 GB for graphical desktop
  - At least 4 GB for heavy Firefox or Chromium usage\!

More documentation is available on the [Memory requirements
section](Requirements#Memory "wikilink") wiki page.

### Storage

  - At least 1 GB storage device for a server without a GUI
  - At least 10 GB for graphical desktop, or 80 GB overall

More documentation regarding requirements is available on the [Storage
requirements section](Requirements#Space "wikilink") wiki page

### GPU

The majority of GPUs are supported, but for advanced features, 3D
acceleration is managed by the MESA project:

  - Intel: mostly any Intel with one exception, intel i810/i815 will
    lack features because of its 4Mb memory, Mesa and Linux have dropped
    their support.
  - ATI/AMD: Only Radeon series with the exception of the last two years
    with respect to the Alpine release, Rage r128/match64 series has
    limited support.
  - Nvidia: Limited. Only a few are completly supported. Not all
    features are enabled.
  - Matrox: Not all features are supported. Just because they are
    shipped on most servers.
  - Sis: Limited features are supported. Code not updated on Xorg and
    Linux kernel.
  - Via: Limited features are supported. Openchrome code not updated on
    Xorg and Linux kernel.

More documentation available on the [Peripheral requirements
section](Requirements#Peripherals "wikilink") wiki page.

# Ways to install Alpine into machines listed by user cases

We have here many ways listed how you can put Alpine Linux in your
computer device, **PLEASE CHOOSE A USE CASE MOST CLOSE TO YOUR SETUP:**

## by booting a source downloaded file ISO on USB or CD/DVD/BR

1.  [Alpine Install: from a disc to a virtualbox machine single
    only](Alpine_Install:_from_a_disc_to_a_virtualbox_machine_single_only "wikilink"),
    install Alpine into VirtualBox virtual machine, use if you have a
    **VirtualBox virtual machine and only want to test it out**
2.  [Alpine Install: from a disc to a any computer single only
    boot](Alpine_Install:_from_a_disc_to_a_any_computer_single_only_boot "wikilink"),
    install Alpine into a real modern machine by burning a disc that
    will boot if you have

**UEFI or BIOS hardware and will be installing Alpine via a CD/DVD
drive**.

1.  [Alpine Install: from a usb to a any computer single only
    boot](Alpine_Install:_from_a_usb_to_a_any_computer_single_only_boot "wikilink"),
    install Alpine into real modern machine by creating a USB drive unit
    that will boot if you have **UEFI or BIOS hardware and will be
    installing Alpine via a USB drive**.
2.  [Alpine Install: from a usb to any computer dual boot linux
    Debian](Alpine_Install:_from_a_usb_to_any_computer_dual_boot_linux_Debian "wikilink"),
    install Alpine into most machines by creating a USB drive unit that
    will boot if you have **common hardware and want another Linux
    distro as your main OS via USB boot**.
3.  [Alpine Install: from a usb to any computer dual boot linux
    Alpine](Alpine_Install:_from_a_usb_to_any_computer_dual_boot_linux_Alpine "wikilink"),
    install Alpine into most machines by creating a USB drive unit that
    will boot if you have **common hardware and want Alpine Linux as
    your main OS via USB boot**.
4.  [Alpine Install: from a disc to a old computer single only
    boot](Alpine_Install:_from_a_disc_to_a_old_computer_single_only_boot "wikilink"),
    (special case for very very old PC or laptop hardware) by burning a
    disc that will boot if you have **BIOS only hardware and will be
    installing Alpine on it via CD/DVD drive**.
5.  [Alpine Install: from a disc to PC Engines
    APU](Alpine_Install:_from_a_disc_to_PC_Engines_APU "wikilink"): to
    install Alpine onto a second generation PC Engines APU system.
    Tested with an apu2d4 using latest alpine.

## by using from linux already started to new partition

1.  [Alpine Install: from alpine mirror to a new computer by
    chroot](Alpine_Install:_from_alpine_mirror_to_a_new_computer_by_chroot "wikilink"),
    install Alpine on a real (i.e. not virtual) modern machine directly
    using the Alpine mirror sources **if you will be using Alpine inside
    another Linux installation via chroot**.
2.  [Alpine Install: from alpine mirror to an external disc by
    chroot](Alpine_Install:_from_alpine_mirror_to_an_external_disc_by_chroot "wikilink"),
    install Alpine on a real (i.e. not virtual) modern machine directly
    using the Alpine mirror sources **if you extracted the disc for use
    with an existing instance of Linux**.
3.  [Alpine Install: from a iso to a virtualbox machine with external
    disc](Alpine_Install:_from_a_iso_to_a_virtualbox_machine_with_external_disc "wikilink"),
    install Alpine on a VirtualBox external disc machine. Use if you
    have an **older computer that doesn't boot, but need to extract the
    disc to prepare it for use**.

## by booting through network install media

1.  [Alpine Install: from a tarball to a bootable ARM
    device](Alpine_Install:_from_a_tarball_to_a_bootable_ARM_device "wikilink"),
    install Alpine on an ARM based device. Use if you have a **ARM based
    network capable install device**.  
      
2.  [Alpine on ARM](Alpine_on_ARM "wikilink") for those who need to dump
    to ARM based hardware

## by booting from external devices

If the computer does not automatically boot from the desired device, one
needs to bring up the boot menu selection for choosing the media to boot
from. Depending on the computer the menu may be accessed by quickly
pressing pressing a key repeatedly when booting starts. Sometimes you
need to press the button before starting the computer and hold it down
during bootup. Typical keys are: \`F9\`-\`F12\`, sometimes \`F7\` or
\`F8\`. If these don't bring up the boot menu, it may be necessary to
enter the BIOS configuration and adjust the boot settings. Typical keys
are: \`Del\` \`F1\` \`F2\` \`F6\` or \`Esc.\`

  -   - [Alpine Install: from a disc to PC Engines
        APU](Alpine_Install:_from_a_disc_to_PC_Engines_APU "wikilink"):
        to install Alpine into second generation PC Engines APU systems.
        Tested with an apu2d4 using latest Alpine.
      - [Bootstrapping Alpine on PC Engines
        ALIX.3](Bootstrapping_Alpine_on_PC_Engines_ALIX.3 "wikilink")
      - [Alpine on ARM](Alpine_on_ARM "wikilink") fisrt main reference
        to any ARM device

# Ways to install Alpine listed by architectures

## x86\_64 x86\_32 x86

The all popular 32 bit intel (i386 pc 32bit) and x86\_64 (i686 pc 64bit
and amd64)compatible (both)

  -   - [Alpine Install: from a disc to a virtualbox machine single
        only](Alpine_Install:_from_a_disc_to_a_virtualbox_machine_single_only "wikilink"),
        install Alpine into VirtualBox virtual machine, use if you have
        a **VirtualBox virtual machine and only wants to take a shoot
        into it**.
      - [Alpine Install: from a disc to a any computer single only
        boot](Alpine_Install:_from_a_disc_to_a_any_computer_single_only_boot "wikilink"),
        install Alpine into a real modern machine by burning a disc that
        will boot if you have a **UEFI or BIOS hardware and will be only
        Alpine into it through CD/DVD drive**.
      - [Alpine Install: from a usb to a any computer single only
        boot](Alpine_Install:_from_a_usb_to_a_any_computer_single_only_boot "wikilink"),
        install Alpine into real modern machine by creating a USB drive
        unit that will boot if you have a **UEFI or BIOS hardware and
        will be only Alpine into it through USB drive**.
      - [Alpine Install: from a usb to any computer dual boot linux
        Debian](Alpine_Install:_from_a_usb_to_any_computer_dual_boot_linux_Debian "wikilink"),
        install Alpine into most common machine by creating a USB drive
        unit that will boot if you have **common hardware and want
        another Linux as main OS through USB boot**.
      - [Alpine Install: from a usb to any computer dual boot linux
        Alpine](Alpine_Install:_from_a_usb_to_any_computer_dual_boot_linux_Alpine "wikilink"),
        install Alpine into most common machine by creating a USB drive
        unit that will boot if you have **common hardware and want
        Alpine Linux as main OS through USB boot**.
      - [Alpine Install: from a disc to a old computer single only
        boot](Alpine_Install:_from_a_disc_to_a_old_computer_single_only_boot "wikilink"),
        especial case for very very older hardware computers PC or
        laptops by burning a disc that will boot if you have a **BIOS
        only older hardware and will be only Alpine into it through
        CD/DVD drive**.
      - [Bootstrapping Alpine on PC Engines
        ALIX.3](Bootstrapping_Alpine_on_PC_Engines_ALIX.3 "wikilink")
      - [Alpine Install: from a disc to PC Engines
        APU](Alpine_Install:_from_a_disc_to_PC_Engines_APU "wikilink"):
        to install Alpine into second generation PC Engines APU systems,
        it were tested with an apu2d4 using alpine lasted.

## ppc64le

For the PowerPC devices with pure little-endian mode, mostly for POWER8
and POWER9

  -   - [Alpine Install: from a disc to a any computer single only
        boot](Alpine_Install:_from_a_disc_to_a_any_computer_single_only_boot "wikilink"),
        install Alpine on a real (i.e. ot virtual) modern machine by
        burning a disc that will boot if you have **UEFI or BIOS
        hardware and will be installing Alpine on it via a CD/DVD
        drive**.  
          
      - [Alpine Install: from a usb to a any computer single only
        boot](Alpine_Install:_from_a_usb_to_a_any_computer_single_only_boot "wikilink"),
        install Alpine on real (i.e. ot virtual) modern machine by
        creating a USB drive that will boot if you have **UEFI or BIOS
        hardware and will be installing Alpine on it via a USB
        drive**.  
          

## armhf armv7

ARM based hardware that does not have CD/DVD/BR boot support, only
execution state of the ARMv7 devices machines. Including video game
consoles; the newer ARM hard-float for newer, more powerful, 32-bit as
well as 64-bit devices.

  -   - [Alpine on ARM](Alpine_on_ARM "wikilink") Main reference for ARM
        devices

## aarch64

The 64-bit ARM only execution state of the ARMv8 device machines.

  -   - [Alpine Install: from a usb to a any computer single only
        boot](Alpine_Install:_from_a_usb_to_a_any_computer_single_only_boot "wikilink"),
        install Alpine on a real (i.e. not virtual) modern machine by
        creating a USB drive unit that will boot if you have **UEFI or
        BIOS hardware and will be installing Alpine on it via a USB
        drive**.  
          
      - [Alpine on ARM](Alpine_on_ARM "wikilink") Main reference for ARM
        devices

## s390x

For the Super powered IBM mainframes, especially IBM Z and IBM LinuxONE
servers

  -   - [Alpine Install: from a disc to a any computer single only
        boot](Alpine_Install:_from_a_disc_to_a_any_computer_single_only_boot "wikilink"),
        install Alpine on real (i.e.not virtual) modern machine by
        burning a disc that will boot if you have **UEFI or BIOS
        hardware and will be installing Alpine on it via a CD/DVD
        drive**.  
          
      - [Alpine Install: from a usb to a any computer single only
        boot](Alpine_Install:_from_a_usb_to_a_any_computer_single_only_boot "wikilink"),
        install Alpine into real (i.e.not virtual) modern machine by
        creating a USB drive unit that will boot if you have **UEFI or
        BIOS hardware and will be installing Alpine on it via a USB
        drive**.  
          

# Ways to use Alpine Linux without install

1.  [Alpine Install: from a usb disc to a machine single
    only](Alpine_Install:_from_a_usb_disc_to_a_machine_single_only "wikilink"),
    dump Alpine onto a usb/mmc card and live boot it on your machine
    without modifying any of your installed files or operating system.

# Documents series

| Previous required                         | What's next to read                                                                                                                           |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| [Alpine newbie](Alpine_newbie "wikilink") | [Alpine Install: from a disc to a virtualbox machine single only](Alpine_Install:_from_a_disc_to_a_virtualbox_machine_single_only "wikilink") |

# See Also

1.  [Newbie\_Alpine\_Ecosystem](Newbie_Alpine_Ecosystem "wikilink")
2.  [Alpine newbie apk packages](Alpine_newbie_apk_packages "wikilink")
3.  [Alpine newbie desktops](Alpine_newbie_desktops "wikilink")
4.  [Alpine newbie developer](Alpine_newbie_developer "wikilink")
5.  [Alpine newbie lammers](Alpine_newbie_lammers "wikilink")

[Category:Newbie](Category:Newbie "wikilink")
[Category:Installation](Category:Installation "wikilink")
