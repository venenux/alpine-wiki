Alpine linux glosary

The Glossary is an extensive list of **terms of linux and technology**, 
but **focused on Alpine Linux definitions and considerable explanations**.


# A 
<!--
{{Define |Topic |Description}}
-->

{{Define|abuild|The tool to build packages from sources using APKBUILD is [}}

{{Define|ACF|'''A'''lpine Linux '''C'''onfiguration '''F'''ramework is an [[Glossary#MVC |mvc-style]([Abuild|abuild]].)] application for configuring an Alpine Linux device. The primary focus is for a web interface - ACF's main goal is to be a light-weight MVC "[webmin](https://en.wikipedia.org/wiki/Webmin)".
}}

{{Define|apk|'''A'''lpine Linux '''P'''ackage '''K'''eeper - A) The [Manager |package manager]([:category:Package)] for Alpine, used to install, query and remove software packages on a running Alpine system. Also the suffix of the binary packages, even if those basically are gzipped tar files.
}}

{{Define|APKBUILD|A build recipe that is used to build Alpine packages for ''apk''. It holds information of package name, version, license, dependencies, sources etc and how to compile the sources and package the binaries.
}}

{{Define |apkovl |[local backup#Committing_your_changes  |'''A'''pkovl]([Alpine)] is a file storing configuration files that have changed from the default ones.  It is used when running from ram.  The contents are overlaid on top of the contents of the apks that are loaded on boot.  The filename is <hostname>.apkovl.tar.gz and is stored on removable media whose path is defined in /etc/lbu/lbu.conf.
}}

# B 
{{Define|Busybox| [Busybox](https://www.busybox.net/) is a utility that combines many common Linux tools into a single program.  Most of the command-line tools in the core Alpine distribution are part of Busybox.
}}
<!--
{{Define|BSD| Berkeley Software Distribution (BSD) is a Unix operating system derivative developed and distributed from 1977 to 1995 by members of the University of California, Berkeley.
}}-->

{{Define|bash| [bash](https://www.gnu.org/software/bash/) is a command-line interpreter or "shell" that provides a command line user interface.
}}

# C 
{{Define|cgit|[cgit](https://git.zx2c4.com/cgit/) provides easy access to all [git repositories](https://git.alpinelinux.org/) hosted on the Alpine infrastructure.
}}

<!--
# D 
-->

# E 
{{Define|edge|[is the name of the development tree of Alpine Linux.
}}

# F 
{{Define |Flatpak |[[Flatpak]([Repositories#Edge|edge]])] is a technology for distributing [desktop |desktop]([:category:)] applications on GNU/Linux. 
}}

# G 
{{Define|git|The distributed version control system that Alpine uses. ([Git]([:category:)])}}

<!--
# H 
-->

# I 
{{Define|IRC| Internet Relay Chat (IRC) is a protocol for  Internet text messaging in real-time. Alpine-specific details can be found [here]([IRC|)].}}

<!--
# J == K # --> L ==
{{Define|lbu|[local backup |Local Backup Utility]([Alpine)]. A tool to make backups of user configuration. Since the system typically runs from RAM, '''lbu''' is used to save the state of the system to a file that is restored to bring the system back to its previous state.
}}

{{Define|LEAF|Linux Embedded Appliance Framework. Alpine Linux started as a fork of the LEAF project. A secure, feature-rich, customizable embedded Linux network appliance for use in a variety of network topologies. Although it can be used in other ways; it's primarily used as a Internet gateway, router, firewall, and wireless access point.
}}

# M 
{{Define|main|The so-called [<tt>main</tt>](https://nl.alpinelinux.org/alpine/v3.10/main/) repository contains software. Those packages are mature.
}}

{{Define|MVC|The [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) design pattern is used in ACF to separate presentation information from control logic. By MVC we mean:
:*'''M'''odel - code that reads / writes a config file, starts / stops daemons, or does other work modifying the router.
:*'''V'''iew - code that formats data for output
:*'''C'''ontroller - code that glues the two together
}}

{{Define|mkinitfs|Tool to generate the [initramfs image](https://en.wikipedia.org/wiki/Initial_ramdisk) for the kernel.
}}

{{Define|modloop|Loopback [cramfs](https://en.wikipedia.org/wiki/Cramfs) image where kernel modules are stored for tmpfs installs.
}}

{{define |musl |[musl](https://www.etalabs.net/compare_libcs.html) is a C standard library implementation for GNU/Linux. Musl libc is not [ABI](https://en.wikipedia.org/wiki/Application_binary_interface) compatible with [| uClibc]([Running_glibc_programs)].
}}

# N 
{{Define|NTP|Network Time Protocol ([NTP](https://en.wikipedia.org/wiki/Network_Time_Protocol)) is a protocol for synchronizing the clocks of computer systems. Alpine provides <tt>[setup scripts|setup-ntp]([Alpine)]</tt> for setting up.</br>''([What's the easiest way to make Busybox keep correct time?](http://lists.busybox.net/pipermail/busybox/2014-September/081668.html) 2014)''}}{{insecure url|Server presents invalid certificate}}

# O 
{{Define|OpenRC|[OpenRC](https://wiki.gentoo.org/wiki/Project:OpenRC) is a dependency based universal init system that works with the system provided [Init Scripts | init program]([Writing)].
}}

<!--
# P == Q # == R 
-->

# S 
{{Define|setup-*|Alpine contains a lot of scripts to configure a system. All those scripts start with <tt>setup-*</tt>. The most important one is <tt>[/ -name setup* -print &#124; sort</tt>)
}}

# T 
{{Define|testing|The so-called <tt>testing</tt> repository contains software packages which are new/untested/experimental.
}}

{{Define|tmpfs|[https://en.wikipedia.org/wiki/Tmpfs tmpfs]([Setup-alpine|setup-alpine]]</tt>.</br>(<tt>find) is a filesystem that exists in (virtual) memory only, like a [RAM disk](https://en.wikipedia.org/wiki/RAM_disk).}}

# U 
{{Define|uClibc| [uClibc](https://en.wikipedia.org/wiki/UClibc) (aka µClibc/pronounced yew-see-lib-see) is a C library for developing embedded Linux systems. It is much smaller than the [GNU C Library](https://www.gnu.org/software/libc/libc.html), but nearly all applications supported by glibc also work perfectly with uClibc.
}}

# V 
{{Define|vServer|[Linux-VServer](https://en.wikipedia.org/wiki/Linux-VServer) provides virtualization for GNU/Linux systems by kernel level isolation. This way it's possible to run multiple virtual units at once.}}

<!--
# W 
-->

# X 
{{Define|Xfce| [Xfce](https://www.xfce.org/) is a lightweight [desktop |desktop environment]([:category:)] that is available in the Alpine Linux repositories. 
}}

<!--
# Y 
{{Define |Topic |Description}}

# Z 
-->

[Newbie]([category:)]
