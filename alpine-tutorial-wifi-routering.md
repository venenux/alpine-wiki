# AlpineLinux with wlan settings.

The alpine wiki is a crap .. information is "less" so much people is just losing the patiente..

## Introduction to Wireless devices

A "wifi" is a term to short name the Wireless most famous devices, 
but there are so many devices that are "wireless", like bluetooh 
as the most moderns.. (in the past there are others like IR, etc).

The wifi devices are from two kind, the only pure network ones and the hybrid ones, 
this means that a pure network device just are a device to use linked to a network, 
but currently bluetooth is a network device but for paired devices, just like point to point, 
so nomadays curerntly almost any laptop wifi is also a bluetooth device too, we called 
those for convenience hybrid ones.

Also important, many laptops and special computers, have a hardware button (or switch) 
to turn off wireless card, however, sometimes this are a software switch by the vendor, 
so we can also be blocked by kernel if the card its well supported. This is only 
by the two important requirements, first by using `rfkill` (util-linux) and also 
if the module of the device is currently well and complete supported hardware.

## Setup wireless on alpine Alpine Linux

AS any tech piece, we have hardware and software, this document only will cover 
the software part, cos hardware support depends of the available modules and 
reverse ingeniering that the linux community can made to the hardware.

So then a "wifi setup" is made by two parts:

* the module manager from kernel .. (like compiling rtl8192eu-linux or r8169)
* and the software interface.. (like installing wpa_supplicant or iwd programs)

Due the huge problem that represent the hardware, we only will covert software interface, 
and we will assume you already have the modules already compiled into the kernel.

**IMPORTANT** if in our telegram channel you called "driver" to a module kernel, you 
will be punished.. **a driver is a person that drive a car! and a folder it to put papers in it, not a directory, ok?**

### 1 - you dont have networking

If you dont have network how you can grab the packages?

#### Option 1 grab the packages manually

If you cannot setup wired network, you can use direct download of the packages 
and later put into your alpine computer, using USB.. this is easy but tedious, 
cos you need to grab the file and compute the dependeces by searching each package 
in the web search interface (yew only in the web cos in your alpine you dont have index generated).

1. Go to the package search web at https://pkgs.alpinelinux.org/packages
2. Fill the field **Package name** with `wpa_supplicant` with carte.
3. Change **edge** branch by your alpine version, this tutorial will assume alpine 3.12 only.
4. Fill field **arch** with your architecture, use `uname -m` as reference but not exact value.
5. Now hit enter, will show you one or two results, hit into the package name
6. After package shows, at the right side you can view the dependencies, lest make a list:
7. On a text file, made list of those names, later hit and click each then and do the same.
8. You will finish list when all the dependences will be already present in your list.
9. Go to http://dl-cdn.alpinelinux.org/alpine/**version**/**flavor**/**arch**/ for the files

This method is most common to cases like tvboxes, raspberrys, or small devices that 
dont have wired network devices and the ISO/IMG of installation media does not provide 
all the packages.

#### Option 2 using a local mirror

This means you already have a local copy of the main repository, or a local copy of all the 
hole repository of alpine packages, but the x86 (i386) and x86_64 (amd64) installation media 
provides extended instalation isos that have in the media all the necesary packages 
and only few are missing.

1. Go to http://dl-cdn.alpinelinux.org/alpine/
2. now visit the **version** you need to install, for x86 older 32bit use 3.12 as max
3. go to /releases/**arch**/ and download alpine-extended-**version**-**arch**.iso

When you perform the installation, you can grap the package cos the ISO media is already 
listed as a repository. If not, you can copy the entire repository locally and each need 
package taken from it manually.

#### Option 3 easyle use a wired connetion first

The most easy option is to first use a wired connetion that provides internet, so you can 
easyle install the necesary packages to perform the wifi setup.

### 2 - Install the packages

Wireless need a special packages, the pacakge `iwd` is available since 3.10 but 
ther are two problems: 
1. first! that crap package needs dbus, so how stupid is the linux community, 
if you in such stage dont have connection, so, how you can grab a complex set 
of packages due dbus dependencies that apart still have a short (almost married) 
relation with shitstemd?, yeah.. thanks for being stupids!
2. second! `iwd` its pretty BAD very bad packaged in alpine linux, due the simplicity 
itselft, the mayor feature of alpine is the mayor problem, minimalist and simplicity 
make the package almost unusuable, apart of the fact that such package just dont work 
without the need dependences that are not present without setup a network repository, 
like the dbus+glib interfaces libraries, yeah!!! thanks again!.

So yeah.. we will use the old and fiable methods, **later we will provide another tutorial with iwd**:

```
apk add wireless-tools wpa_supplicant linux-firmware util-linux
```

Those are the minimal packages to work, if you already have wired method internet, 
just install all the need packages as:

```
apk add wireless-tools wpa_supplicant dbus-libs libnl3 pcsc-lite-libs linux-firmware util-linux
```

**IMPORTANT** since 3.15 the `rfkill` program is at `util-linux-misc` package, and not 
in the `util-linux` package cos was splited so you must install it in recent alpine versions.

### 3 - configure wireless devices

#### check the devices availables

First of all check wich is your interface network device for wireless and is active:

```
rfkill list
```

This will show you the current names of the devices only if are wireless, also will show 
you bluetooh mixed ones, so be care with the name of witch device will you use.

* This document will assume the device named `wlan0` from such command.
* At the output if the line `Soft blocked` says `no` the device can be used or activated.
* At the output if the line `Hard blocked` says `no` the device can be used.
* If you receive `yes`, run `rfkill unblock wifi` but **this depends of kernel module support**
* If the module has not complete support of the hardware, this will be limited or impossible.

#### check the availables wifis

Use the interface to scan for wireless access points, we need to check and make sure 
the ESSID you want to connect to appears:

```
iwlist scanning | grep ESSID
```

The output will show the available "wifi's" (SSID's), lest assume your one is named `mywifi` 
this document will use the SSID named `mywifi` to make the practice.

#### create the coifiguration wpa

Next, create a "wpa_supplicant" configuration stanza for the wireless access point:

```
echo "ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev" > /etc/wpa_supplicant/wpa_supplicant.conf
wpa_passphrase 'mywifi' 'thepasswordwifi' >> /etc/wpa_supplicant/wpa_supplicant.conf
```

The commands just help to initialize the configuration file.

This will be a minimal file, but this may need some extra configurarions depending of 
the network setup or the device using, by example some USB network devices change 
the MAC address ramdownly that produces problems for routers.. so a complete recommended 
setup is then:

```
# recommended, for idetification check wikipedia ISO/IEC country code list
country=VE

# this depends of the alpine package configuration, just use it as is:
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev

# wpa_supplicant initiates scanning of the wifi's (SSID AP) and 
# if no APs matching to the currently enabled in configuration are found, 
# a new network (IBSS or AP mode operation) may be initialized if configured
# the option 0 is only for special wired and option 2 for specifics purposes
# e.g., with ndiswrapper to enable operation with hidden SSIDs and optimized roaming
ap_scan=1

# Automatic scan for SSID and chosse one if routers are not 100% online
# autoscan is like bgscan but on disconnected or inactive state
# but ignored if If sched_scan_plans are configured and supported by the driver
# here a delay of 60 seconds will be used on each scan.. if you only have one SSID
# and if you only have good signal (over 80%) just put this to 300
autoscan=periodic:120

# Disable automatic offloading of scan requests (sched_scan) by the module kernel
disable_scan_offload=1

# general MAC address policy default 0 = permanet
mac_addr=0

# 0 = use permanent MAC address for pre-association operations (scanning, ANQP)
preassoc_mac_addr=0

# 0 = use permanent MAC address for GAS operations
gas_rand_mac_addr=0

# now lest improve the SSID wifi setup to connect:

network={
    # name of the wifi of the router to connect, the SSID AP
        ssid="mywifi"
    #psk="thepasswordwifi"
        psk=eec720af4bf770f83e9cd7d425d1ced46e0ca7e2df9be8745e4a16f810677ce4
    # this tell to wpa to use this before the other that have priority=2 or priority=1
        priority=3
    # this is an indentification, can be a str4ing name short but no symbols allowed
        id_str="mywifi"
}
```

So then the configuration for a password WPA/WPA2 wifi will be:

```
country=VE
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
ap_scan=1
autoscan=periodic:120
disable_scan_offload=1
mac_addr=0
preassoc_mac_addr=0
gas_rand_mac_addr=0
network={
        ssid="mywifi"
        psk=eec720af4bf770f83e9cd7d425d1ced46e0ca7e2df9be8745e4a16f810677ce4
        priority=3
        id_str="mywifi"
}
```

So then the configuration for a password less or no-password free wifi will be:

```
country=VE
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
ap_scan=1
autoscan=periodic:120
disable_scan_offload=1
mac_addr=0
preassoc_mac_addr=0
gas_rand_mac_addr=0
network={
        ssid="mywifi"
        key_mgmt=NONE
        priority=3
        id_str="mywifi"
}
```

#### 4 - testing the configuration

Start wpa_supplicant in the foreground to make sure the connection succeeds.

```
wpa_supplicant -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
```

This will take the console output when is running, this is just for testing. 
just stop by pressing `CRTL` and `C` key at the same time.

#### 5 - Automatic Configuration on System Boot

Alpine already has a configuration tool, but is just script that writes to 
the `/etc/network/interfaces` with pretty limited feature, so again minimalist shit, 
lest take own and made master usage of the onfigurations:

1. check if your `lo` interface is present and working
2. check if you machine already have other devices such `eth0` and maybe `eth1` or both
3. check if both or at least `lo` is already present in the `/etc/network/interfaces`
4. open with editor (`vi` busybox by default) the file `/etc/network/interfaces`
5. get sure `auto lo` is first and followed by `iface lo inet loopback` or add it
6. then only put similar for `eth0` and such kind of devices as `iface eth0 inet dhcp`
7. now add the `wlan0` part of the network wifi interface but this one must have the SSID
8. use the `metric` keyword to determine the device that will be use for internet as primary

So the file will be:

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
	metric 2

auto wlan0
iface wlan0 inet dhcp
	wireless-essid mywifi
	wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
	metric 1
```

So the bring the interface down to reconnect to the network, and later then 
start the service of the wireless conecctions, later then confiugure at boot:

```
ifconfig wlan0 down

/etc/init.d/wpa_supplicant start

rc-update add wpa_supplicant boot
```

> __Note__: If this errors with `ioctl 0x8914 failed: No error information`, 
that's `busybox ip`'s way of saying your wireless radio is rfkill'd, for information 
on how to unblock your wireless radio; the base installation should 
have `busybox rfkill` available, check the section [check the devices availables](#check-the-devices-availables).

**IMPORTANT** Hardware buttons to toggle wireless cards are handled by vendor specific 
kernel modules. Frequently, these are [WMI](https://lwn.net/Articles/391230/) modules. 
Particularly for very new hardware models, it happens that the model is not fully supported 
in the latest stable kernel yet. In this case, it often helps to search the kernel bug 
tracker for information and report the model to the maintainer of the respective 
vendor kernel module, if it has not happened already.


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

