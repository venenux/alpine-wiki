# alpine qemu emulation

`qemu` its a emulation system that uses KVM (kernel virtual machine) and also are capable of hypervision!

Initially QEMU was an emulation engine, with a Just-In-Time compiler (TCG) 
to dynamically translate target instruction set architecture (ISA) to host ISA.

Nowadays, QEMU offers virtualization through different accelerators. 
Virtualization is considered an accelerator because it prevents unneeded 
emulation of instructions when host and target share the same architecture. 

Only system level (aka supervisor/ring0) instructions might be emulated/intercepted.

#### modes of emulation:

* Full: the qemu emulates all the machine, like have a pc inside other pc
* USer: the qemu uses binary translation so programs runs like into original machine

#### terminology to understand mechanism

The **host** is the platform and architecture which QEMU is running on, is the native 
machine where the qemu will run the virtual machine

The **guest** is the architecture which is emulated by QEMU, is the target machine 
that will be emulated from the host or native machine.

The **device front** end is how a device is presented to the guest. The type of 
device presented should match the hardware that the guest operating system is 
expecting to see. All devices can be specified with the `--device` command line option. 
Running QEMU with the command line options `--device help` will list all devices.
Running QEMU with command line `--device foo,help` will list options for "foo" device.
The **device front* end is often paired with a back end, which describes how the 
hosts devices resources are used in the emulation.

The **device back** end describes how the data from the emulated device will be 
processed by QEMU. The configuration of the back end is usually specific to the 
class of device being emulated.
While the choice of back end is generally transparent to the guest, there are 
cases where features will not be reported to the guest if the back end is unable 
to support it.

The **Device pass through** is where the device is actually given access to the 
underlying hardware. This can be as simple as exposing a single USB device on the 
host system to the guest or dedicating a video card in a PCI slot to the exclusive.

#### packages in alpine

| name         | debian equivalent |  important files |
| -------------| -------------- | ----------|
| qemu         | seabios           | bios.bin |
| aavmf        | qemu-efi-aarch64  | QEMU_EFI.fd |
| qemu-modules | qemu-system-gui+qemu-system-common |  |

#### emulation of x86 based machines and best configurations

| machine option      |  32bit x86 i386  | 64bit x86_64 amd64 |  observations |
| ------------------- | --------------------- | ------------------- | ------------ |
| bios best option    | bios-256k.bin (qemu)  |  OVMF.fd  (ovmf)    |  both are under `/usr/share` paths, ovmf is EFI only |
| CPU and motherboard | `-cpu n270 -machine pc` | `-cpu Conroe -machine q35` | each provides defaults based on natural features |
| Defaults on chipset | i440FX+PIIX3 PATA PS2 | G31+ICH9 SATA USB | no matter choosed features can be combined but not the best |
| Hardware on virtual | ISA, PCI, PATA, BIOS | PCIe, MMCFG, vIOMMU, AHCI, UEFI | For MSDOS or WinXP you must set "pc" machine |
| Use cases           | boot older software | P2V, OVMF, vIOMMU | migration of modern machines can be done with "q35" |
| it required KVM?    | no, if you use defaults | yes if you map real devices | if you dont use kvm you cannot translate real devices |

#### emulation of arm based machines and best configurations

| machine option      | 32bit arm32 armv6  | 64bit aarch64 armv8 |  observations |
| ------------------- | --------------------- | ------------------- | ------------ |
| bios best option    | default               | QEMU_EFI.fd (aamf)  | under `/usr/share/` path using aamvf package from aarch64 arch |
| CPU and motherboard | `cortex-a7 -machine virt` | `cortex-a35 -machine virt` | no defaults, must select a machine profile |
| Defaults on chipset | GICv2M PCI GPIO | MSI GICv2M PCI/PCIe PL011 UART | Note that ITS is not modeled in TCG mode |
| Hardware on virtual | PCIe, PCI, it depends on machine | PCIe, it depends on machine | Unfortunately Arm boards are currently undocumented |
| Use cases           | boot older software | basic testing only | you must setup specific machine board |
| it required KVM?    | no, for basic emulation | not tested, need for real devices mapped | if you dont use kvm you cannot translate real devices in general |

#### emulation of devices and compatibility

| device type    | command line                           | i386  | amd64 | arm  | aarch64 |  observations |
| -------------- | -------------------------------------- | --- |  --- |  --- |  --- | ----------- |
| USB 1.x        | `-device pci-ohci`                     | Yes | Yes | Yes | Yes | can be dificult with arm machines |
| USB 2.x        | `-device usb-ehci`                     | Yes | Yes | No | Yes | is not the best option, consumes CPU |
| USB 3.x+2.x    | `-device nec-usb-xhci`                 | Yes | Yes | Yes | Yes | best option, recent version has `qemu-xhci` |
| keyboard/mouse | `-device usb-mouse -device usb-tablet` | Yes | Yes | Not | Yes | best option, recent version has `usb-kbd` |
| Storage Virtio | `-device virtio-scsi -device scsi-hd,drive=hd1` | Not | Yes | Yes | Yes | slow, use with `-drive file=diskimg.qcow,if=none,id=hd1` |
| Storage IDE    |                                        | in | Yes | Not | Not | Built in on i440fx and q32 machines |
| HD audio       | `-device intel-hda -device hda-duplex` | Not | Yes | Yes | Yes | `-device AC97` is best for i386 32bit and older software |
| Hardware entropy | `-device virtio-rng-pci,rng=rng0`    | Yes | Yes | Yes | Yes | Use with `-object rng-random,id=rng0,filename=/dev/urandom` |
| Hardware HDD/SSD | `-drive file=/dev/sdX#,cache=none,if=virtio` | Yes | Yes | Not | Not | Use with caution, `X` is disk and `#` is partition |

#### Provisioning virtual disks

AS we said.. qemu can manage in many ways the storage, it can use emulated 
storage or inclusivelly real one, this will impact in performance, and also 
will deal with limitations:

|  Type         |  Qemu builin | block feats | migration | extra required | observations |
| ------------- | ------------ | ----------- | --------- | -------------- | ----------------------- |
| Qemu emulated | IDE          | Yes         | Yes       | No, build in   | slow or mid performance |
| Qemu emulated | NVMe         | Yes         | Yes       | command line   | basic or mid performance |
| Qemu emulated | virtio-blk   | Yes         | Yes       | command line   | relative best performance, few disks |
| Qemu emulated | virtio-scsi  | Yes         | Yes       | command line   | mid or slow performance, many disks |
| vhost         | vhost-scsi   | no          | no        | command line   | relative or mid performance |
| vhost SPDK    | vhost-user   | no          | no        | command line   | relative or mid performance |
| Device assignment | vfio-pci | no          | no        | Exclusive use  | Hihg performance, device is blocked by qemu |

Sometimes higher performance means less flexibility

### QEMU setup for simple virtualization

The program can be used no matter if you have hardware support or not in its 
simplest way, of couse is the most slow way to use emulation but runs on any system. 
For advanced usage with KVM and hardware support forward to the next 
mayor section [QEMU with KVM and hardware virtualization](#qemu-with-kvm-and-hardware-virtualization).

#### installing qemu for simple emulation

* (added and) update normal repositories
* update database repositories
* install basic need packages and qemu modules
* load and setup the tun module


```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add qemu qemu-img qemu-system-i386 qemu-system-arm qemu-modules

grep tun /etc/modules|| echo tun >> /etc/modules
```

#### Configuring qemu for simple emulation

* allow user of qemu group to manage briged devices
* change permissions of the configurations

```
sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf

chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf
```

#### Running qemu machines with simple emulation

We can just boot a machine from a simple boot device of 32bit intel, 
over any other, it will relative decent becouse is an older sistem to emulate:

```
/usr/bin/qemu-system-i386  -name "alpinebootqemu1"
```

> **Warning**: QEMU should **never be run as root** use the `-runas` option to make QEMU drop root privileges.

In any hardware you can also boot other machines, but emulation will be slow
if "guest" (emulated) and "host" (that runs emulator) are not "common":

```
/usr/bin/qemu-system-arm  -name "alpinebootqemu1"
```

> **Warning**: QEMU should **never be run as root** use the `-runas` option to make QEMU drop root privileges.

#### Error initialization on qemu if no display

If you dont have a X11 sessions, or if you dont have allowed to share your devices, by 
example when you run linux inside shit operating systems, will raise some errors like:

* `Error GTK initialization failed` means you dont have any X11 session running 
or you dont allow to connect to the x11 sessions (by example if you run from non 
linux host or inside a crap operating system)
* `MESA-LOADER: failed to open ...` means you cannot see the display running 
or you dont install X11/mesa environment/packages yet (by example running server only 
linux host or inside a crap operating system)

Solution is to run headless and setup non graphics output screen, next setups will 
show two ways, first we just see the text only way with no X11 allowed:


#### Running qemu machines with simple emulation but no X11 output only text

We can just boot a machine but with arguments display to "ncurses" to run 
with no need of installations of Xorg complete software, but caution, this 
means you will only see what the emulated software are allowed to show in:

```
/usr/bin/qemu-system-i386 -name "alpinebootqemu2" -display curses
```

The ncurses interface will take all the console output as a screen output! 
but when the OS emulated or software runs any graphical thing, blank output 
will happened and you can see anything then!

To run again or terminate such command you will need to kill in another console, 
or send shutdown action over the monitor console!


### QEMU with KVM and hardware virtualization

You can improve the performance if your machine is PC based or ARM based (some) 
with the KVM (kernel virtual machine) and your virtualization support from hardware!

* The argument `accel=kvm` of the `-machine` option is equivalent to the `-accel kvm`
* CPU model `host` requires KVM, but you must take care if you use PCI passthrough
* The passthrough options offers the virtual machine native performance with KVM
* IOMMU requires KVM, and `-device intel-iommu` should not be set if PCI passthrough
* IOMMU needs add kernel parameter `iommu=on` for remapping IO in guest
* On virtual machine with bad performance, should check for proper KVM support
* KVM is required in order to start Win7 or Win8 properly without a blue screen
* TPM is need (thought q35 and UEFI+TPM features) to start Win10 without errors

#### Where can be loaded the KVM support

The KVM for Hardware virtualization **only could be possible when host and guest 
are same or similar architecture**, some examples:

| host/guess | i386   | amd64  | armv6  | armv7  | aarch64 | observations |
| ---------- | ------ | ------ | ------ | ------ | ------- | ------------ |
| i386       | KVM(*) | KVM(*) | TCG    | TCG    | TCG     | only if common hardware is present, otherwise TCG |
| amd64      | KVM(*) | KVM(*) | TCG    | TCG    | TCG     | only if common hardware and host is more recent rather guess, otherwise will be TCG |
| armv6      | TCG    | TCG    | KVM(*) | KVM(*) | KVM(*)  | only if host is a more recent cpu rather than guest |
| armv7      | TCG    | TCG    | KVM    | KVM    | KVM(*)  | only if common hardware and host is more recent rather guess, otherwise will be TCG |
| aarch64    | TCG    | TCG    | KVM    | KVM    | KVM(*)  | only if common hardware is present, otherwise TCG |

In such matrix, we fount at the column of i386, that if host is armv7 the emulation 
only will be full emulated (TCG) and not accelerated (KVM), cos there no common 
hardware to accelerated, but in case of emulation of armv6 any host of aarch64 or armv7 
will do accelerated because the host is more recent and also has common hardware!

For marks (*) the underliying hardware must be equal or superior respect the emulated 
so by example we cannot emulate SSE4 from a host that does not have SSE4 flags on cpu, 
by example for emulation of i386 guess over i386 host, if the guess is a pentium4 the 
host cpu must be as minimun pentium4, if you already has a pentium3 and wants to emulate 
a pentium4 you will be very limited then!

#### Checking Virtualization Hardware support

You must check if your CPU support emulation by the command:
`apk add arch-install-scripts && lscpu | grep Virtualization`, this is necessary 
for `kvm` implementation, if the above command does not show nothing you cannot 
do such emulation.

#### Installation qemu with KVM

* install basic need packages and qemu modules
* installing for loading tun module
* installing for loading vhost_net module
* detecting intel cpu and load their nested kvm module
* detecting amd cpu and if true loading nested kvm module
* setup to allow users to use bridged network devices
* setup proper permissions for those devices
* setup the user to run the machines
* add to the groups with privilegies

```
apk add bash qemu-img qemu-system-* qemu-modules qemu-tools qemu-img

rmmod tun && rmmod vhost_net && rmmod vhost && modprobe vhost_net && modprobe tun
grep tun /etc/modules|| echo tun >> /etc/modules
grep vhost_net /etc/modules|| echo vhost_net >> /etc/modules

/bin/bash -c [  -z '$(grep -i intel /proc/cpuinfo|head -n1)' ] && echo AMD  || modprobe kvm_intel nested=1 && echo "options kvm_intel nested=Y">/etc/modprobe.d/kvm_intel.conf

/bin/bash -c [  -z '$(grep -i amd /proc/cpuinfo|head -n1)' ] && echo INTEL  || modprobe kvm_amd nested=1 && echo "options kvm_amd nested=Y">/etc/modprobe.d/kvm_intel.conf

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf

chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

adduser -S -D -g '' -s /bin/bash -h /home/general general

adduser general qemu && adduser general kvm
```

#### Running a simple machine with KVM support

We can just boot a machine but with arguments to enable KVM and 
display to "ncurses" to avoid X11 xorg software:

```
/usr/bin/qemu-system-i386 -name "alpinebootqemu3" -accel kvm -display curses
```

The addition of `-accel kvm` if the host is amd64 then this i386 virtual machine 
will run accelerated, will give improved performance, because will use direct 
hardware of the host machine.

To run again or terminate such command you will need to kill in another console!

#### Running the qemu without the kvm support

If your computer that will act as host it does not support the KVM 
infrastructure you can deactivate it explicit in command line:

```
/usr/bin/qemu-system-i386 -name "alpinebootqemu5" -accel tcg -display curses
```

Obviously you will not need to configure any KVM or qemu group or user, 
the qemu system will just interpreted all the things but will runs more slow also.

This time we parsed `-accel tcg` but you can also add `-machine accel=tcg` to try 
others ways of optimization. To run again or terminate such command you will 
need to kill in another console!

## QEMU and the memory - the IOMMU or DMAR and hugepages

IOMMU (Input/Output Memory Management Unit) is a feature of some CPUs that allows 
to map physical and virtual memory addresses to manage resources securely; this 
prevent buffer overflow attacks and other security threats by allowing devices 
to access only the memory regions that are explicitly allocated to them.

The IOMMU allows each VM to have its own virtual address space, preventing one VM 
from accessing the memory of another VM or the host system where multiple virtual 
machines (VMs) share the same physical hardware.

**Hugepages allows to map huge address of RAM more faster, so IOMMU is highly 
necessary to and for emulation of unknown or insecure operating systems.**, 

Here we will configure hugepages for default system setup, 
for more deep use [alpine-newbie-hugepages.md](alpine-newbie-hugepages.md)

#### Checking IOMMU or DMAR support

You must run a kernel with support for IOMMU, most of the desktop focused linux 
do not enable it, so you must check with two steps:

1. Check if the CPU already were factorized for that. (Any xeon is)
2. Check if BIOS/motherboard already set the feature and enable it!
    * the IOMMU technology is called "Intel VT-d" for intel CPUs
    * the IOMMU technology is called "AMD-Vi" or IOMMU for AMD CPUs
3. Add to the kernel parameters the `iommu=on` after `ro` part on grub or `cmdline` 
4. After reboot check with root command: `dmesg | grep -e DMAR -e IOMMU`

You should see a "DMAR" word (if intel) or "IOMMU" word (if AMD).

Specifically, `iommu` kernel parameter manages the use of this technology in the 
system, there are also `intel_iommu` is for intel and `amd_iommu` for amd.

**Manufacturers sometimes do not give the complete specifications but only those 
most popular ones** such as how many cores it has or the speed, while **the more 
internal specifications such as the Intel-Vt or the AMD-Vi do not mention whether 
they exist or not**

CPU manufacturers show them but **motherboard manufacturers often do not show 
them or even change their name.** By example almost **any XEON provides all necesary 
technology but if are mounted in a poor motherboard will not give you nothing.**

#### Auto Configure the hugepages for qemu

If you will have a VM with 4096 Mb of RAM (only one)so then (4096/2)+1024 where 
the 1024 will be the amount of pages for.. this number will be the amouh of huges pages 
if your huge pages size configured is 2M, this will leave almost a quarter for other vm of hugepages, 
so the need hugepages with 2M "hugepagez" will be 3092

If you will have a VM with 4096 Mb of RAM (only one)so then (4096/2)+8 where 
the 8 will be the amount of pages for.. this number will be the amouh of huges pages 
if your huge pages size configured is 1G, this will leave almost a quarter for other vm of hugepages, 
so the need hugepages for 1G "hugepagez" will be 16

* allow some more regions rather than 8 ony for mem workaround bug on qemu limits
* mount and setup the virtual host modules, specially for networking
* create the directory for mounting the hugepages if not exists
* mounting the hugepages with hugelgfs type into such directory
* configure the mount directory for libvirt daemon module of qemu
* restart the service

```
echo "options vhost max_mem_regions=16" > /etc/modprobe.d/vhost.conf

mount -t hugetlbfs -o rw,pagesize=$(grep Hugepagesize /proc/meminfo|tr -s ' '|cut -d' ' -f 2)k,mode=1770,relatime,gid=$(getent group qemu | cut -d':' -f3) hugetlbfs /dev/hugepages
```

The mode 17770 will allow to users modify only their own resources, 
the mount of the **hugetlbfs can be automatized by added this to fstab** with:

```
grep hugetlbfs /etc/fstab || echo \
"hugetlbfs /dev/hugepages hugetlbfs rw,pagesize=$(grep Hugepagesize /proc/meminfo|tr -s ' '|cut -d' ' -f 2)k,mode=1770,relatime,gid=$(getent group qemu | cut -d':' -f3) 0 0" \
 >> /etc/fstab
```

> **Warning**: such autodetect the group and hugepage size in one command and will add to fstab

Example of how could look the entry in fstab after run such command:

```
hugetlbfs /dev/hugepages hugetlbfs rw,pagesize=2048k,mode=1770,relatime,gid=33 0 0
```

Where the hugepasesize is 2M the most common, and the gid is 33 a number that is not fixed, 
will depends of your current group, here we are using the "qemu" group for access from user.

#### running qemu with hugepages memory support and KVM acceleration

```
/usr/bin/qemu-system-i386 -accel kvm -mem-path /dev/hugepages -display curses
```

The addition this time is the `-accel kvm` and `-mem-path /dev/hugepages` parameters 
that will bring improved performance, because will use direc hardware of the host 
but also including RAM access mapping,  To run again or terminate such command you will 
need to kill in another console!

### Qemu and network configurations

Network configurations are the first way to configure communication support 
with guess from host, network configuration is always auto configured.

Keep in mind that communicating with the virtual machine management is not the same 
as communicating with the virtualized system, for management and communication of 
the virtual machine see the connection section using sockets or ports below.

TODO

### Qemu and disk image storages

Best options for are RAW and COW, this last the Qcow2.

* Image file is more flexible, but slower rather than raw file format
* Both supports snapshots, but Qcow2 writes snapshot directly.
* Raw block device has better performance, but harder to manage, by example 
  snapshot is supported with raw block device, but need to create a 
  separate file image to write those snapshot states respect the image disk.
* Image file is organized in units of constant size called clusters, 
  raw image file is organized directly as the filesystem under.
* Image file does not take all the space on disk, because does not write zeroes, 
  but it grow faster, wasting more disk space and duplicating data if the
  cluster size is greater than default.

The RAW storage is the best option for people that has slow real hardware, 
but will require complete file size allocation.

#### Creation of disk images

* RAW format: `qemu-img create -f raw -o preallocation=full storagedisk0.raw 10G`
* Qcow2 format: `qemu-img create -f qcow2 -o cluster_size=512k storagedisk1.img 10G`

The RAW image file format is best for fast implementation, for best performance 
using a qcow2 image file, increase the cluster size when creating the qcow2 file, 
around 2 Megs and also perform a preallocation like we done with RAW before.

* `qemu-img create -f qcow2 -o cluster_size=2M,reallocation=full storagedisk1.img 10G`

**Warning** this will make that the storage gain performance but this defeats 
thin provisioning, also mayor cluster size will increase waste of storage.

#### Tune up disk storage for QCOW formats

A common option is L2 table cache size, that is related to the amount of mapped 
size of virtual disk as `disk_size = l2_cache_size * cluster_size / 8` 
so the based on disk size you could configure qemu drives (all in bytes):
`l2_cache_size = ( 8 * disk_size ) / cluster_size`, reasonable values are 128k 
or 512k for `cluster_size` for calculations, 2M will take space too quickly.

Default qemu option today is 32M of cache sizes, but **unfortunatelly large cache 
sizes implicts large amount of memory, because QEMU has a separate L2 cache for 
each qcow2 file**, that gets worse with snapshots. This will tune up.

* l2-cache-size: maximum size of the L2 table, can be use in `qemu-system-<arch>` with `-drive`
* refcount-cache-size: maximum size of refcount block, can be use in `qemu-system-<arch>` with `-drive`
* cache-size: maximum size of both caches combined, can be use in `qemu-system-<arch>` with `-drive`

By example, for the 10G previously created images add to the qemu command VM this:

* `-drive file=storagedisk1.img,if=none,l2_cache-size=1572864,cache_size=2097152,refcount-cache-size=262144,cache-clean-interval=700`

The values here for the 10G QCOW2 disk1 were estimated from the calculated values, 
so will cover also disk of 15G or 20G inclusivelly, the `cache-clean-interval=700` 
is to clean the cache cos large cache sizes implicts large amount of memory, because 
QEMU has a separate L2 cache for each qcow2 file, that gets worse with snapshots.

#### Storage device recommendations

This depends of the virtual machine, over x86 by default uses SATA `ich9-ahci` 
but for ARM you must specify based oon the machine to emulated, there is 
a option generic but guess (emulated OS) will need virtio modules on kernel.

Also it depends of the virtualization, if you wil use file storage virtualization 
(using a virtual hard disk and not real hard disk through passthrought) you must 
tune up the file of virtual disk and also the device of storage. On other case, 
if you use the real hardware is more complicated, you must know address and 
feature limits of the real hardware storage to use.

For file based virtual storage those are from best to most featured:

1. `-device virtio-blk,drive=hd0` is newer virtio block device for file storage 
emulation **best option for performance but dont support huge amount of "disks"**
is prefered for ARM and Amd64 emulation, can be used on any emulation if the 
kernel of operating system support it!
2. `-device ide-hd,drive=hd0` is the most compatible storage device front, use 
it for older emulations. Will be a SATA similar to `ich9-ahci` if not specify in 
the drive with `if=ide` so will be `-drive file=storagedisk1.img,if=ide,id=hd0` 
so for older emulation with older OSs this could be best option.
3. `-device virtio-scsi -device scsi-hd,drive=hd0` is newer virtio block device 
for file storage emulation **best option for huge amount of "disks" but less 
performance** is prefered for server cluster emulation, can be used on any emulation 
if the kernel of operating system support it! Dont use it on older OSs.

With the `aio=threads` option is the preferred option when storing the VM image 
file on an ext4 file system. With other file systems, `aio=native` must be used.

More complex combinations can be made, inclusive just usage of one partition, 
event a complete hard drive, or network mount drive also.

## Qemu usage

We learned how to setup property the environment to run qemu 
but now we must to setup the virtual machines, taking into consideration:

* qemu it has a huge list of configurations we are not the qemu project so 
  but you can see some howtos examples at [../tutorials/alpine-howto-qemu-on-pc.md](../tutorials/alpine-howto-qemu-on-pc.md)
* qemu is command line only, no saved configurations are possible, 
  for that you must [use libvirt framework in combination](server-alpine-libvirt-qemu-virtualization.md)
* virsh can configure and save virtual machines and later just lauch
  but this will need the [use libvirt framework in combination](server-alpine-libvirt-qemu-virtualization.md)

#### Display video and device recommendations

In order of best compatibilty to most improvement:

1. `-device cirrus-vga,vgamem_mb=16` the most compatible with older and 
modern systems, this has only most common features and enables the needs 
of vgabios, uefivga, VGA output and any OS supported, allows max 16Mgs video 
and is the best choice compatibility wise, pretty much any guest should 
be able to bring up a working display on this device; on default use 4Mgs 
of video, it not gets full hd but is close to 1024p if use 16 megs, The 
**best option for older systems emulation and full emulation compatibility**
2. `-device virtio-vga,max_outputs=2` the most compatible with modern and 
up to date systems, has no dedicated video memory (except VGA compat.), 
if supports vgabios, uefivga, VGA output, gest os will need special module 
that is since 3.2 into linux; it gets full HD on it defaults, has (optional) 
hardware-assisted opengl acceleration which in turn needs opengl support 
enabled if `-display xxx,gl=on` is enabled in the qemu display (sdl/gtk), 
**recommended for both modern or older systems with support of opengl**
but will require KVM support on host machine.
3. `-device ramfb` very simple display device; ases a framebuffer stored 
in guest memory; does not have vga, support vgabios and uefivga; the 
firmware initializes it and allows to use it as boot display (grub boot 
menu, efifb, ...) without needing complex legacy VGA emulation. **Only 
recommended for SOC devices like older ARM boars emulation**
4. `-device bochs-display` the simple linear framebuffer, with modesetting, 
does not have vga, supports vgabios, supports uefivga, and is linux focused; 
firmware will setup a linear framebuffer as GOP anyway and never use any 
legacy VGA features, so this device is **best option for UEFI related; 
also this is best option for server virtualized implementation**.
4. `-device xql-vga` mostly dated, it feature is multihead support to a 
second display to remote connection, mostly for crap operating systems; 
if supports vgabios, uefivga, VGA output and any guest OS will support it; 
featured 2D acceleration so is mostly great for remote connection, but 
relies on a special client, offloading 2D acceleration to the spice client
mostly virt-viewer; **its best option for modern and older combinations**.
5. `-device virtio-gpu,max_outputs=2` the most featured with modern and 
up to date systems, has no dedicated video memory, it lack of VGA layer, 
only has uefivga, will reduce the attack surface (no VGA emulation) and 
reduce the memory footprint by 8 MB (no pci memory bar for VGA compatibility);
that is since 4.0 into linux; it gets full HD on it defaults, has (optional) 
hardware-assisted opengl acceleration which in turn needs opengl support 
enabled if `-display xxx,gl=on` is enabled in the qemu display (sdl/gtk)
gpu data will be stored in main memory instead. **This option is the best 
for linux or mac modern system and modern platform desktop emulation** 
but will require of KVM on host machine.

**WARNING** opengl native support its broken in alpine qemu package: 
https://t.me/alpine_linux/1287

#### Audio device recommendations

**If the `audiodev` backend is not provided, QEMU looks up for it and adds 
it automatically, this only works for a single audiodev.**
For audio in order of best compatibilty to most improvement on modern hosted:

1. `-device AC97` is enough for older or newer devices, it comes with 
the mic and output at the same time, all the inputs and outputs are auto 
mapped to the backend and provided without, this is the **best option 
for emulation of 32 or 64 bits of ARM or X86 based computers and older OS**
2. `-device intel-hda -device hda-duplex` the High definition audio, 
that must be provided with a specific device way, the `hda-duplex` 
**will provide only "line in" and "line out" sound, no mic allowed**.
3. `-device intel-hda -device hda-micro` the High definition audio, 
that must be provided with a specific device way, the `hda-micro` 
**will provide only "mic in" and "line out" sound, no "line in" allowed**.
4. `-device ich9-intel-hda` is the only **recommended device to the 
emulation of modern propietary operating system** from Apple or M$.
5. `-device virtio-sound-pci` is the most recent feature of qemu, but 
only supported for modern guest OSs, its not recommended on most cases 
becouse is too recent and is complete virtualized (more slow but featured).

**IMPORTANT** On the host end (the VM not the host), available support include 
ALSA, OSS and WAV. The latter two being not so useful for practical audio on 
modern systems, ALSA is the only real option.
The problem is, **KVM does not support virtual ALSA devices, requires 
exclusive access to the hardware ALSA device**, which of course **leaves 
the host system (the machine runing qemu) without audio**. If you need the 
virtual machine for some audio applications, it would be hugely practical 
to be able to somehow mix guest audio with host audio.

This can be achieved using ALSA loopback or JAck/Pulseaudio/Pipewire.

**ALSA**: If you use `AC97` you should not get problems, using command 
as `-device ac97,audiodev=sd1 -audiodev alsa,id=sd1` but is not the 
same if you try to boot modern system, so must use `HDA` audio device 
as `-device intel-hda -device hda-micro,audiodev=sd1 -audiodev alsa,id=sd1,out.try-poll=off`
where audiodev will be the host system audio serve for the guess host, 
this need `modprobe snd-aloop;modprobe snd-pcm-oss;modprobe snd-mixer-oss` 
modules to be loaded, mostly firs one `snd-aloop` that is not so common.

**PULSE**: This case is not the same, you will need the address socket 
of the pulse server running (Each user runs their own instance), generally 
will be of `unix:/run/user/<ID>/pulse/native` where `<ID>` is the numeric 
id of the user that runs the command of qemu (with `id -u general` using 
the name `general` as username login you can get it); then you must use 
as `-device ac97,audiodev=sd1 -audiodev pa,id=sd1,server=unix:/run/user/<ID>/pulse/native`
if you try older guest OS using the older AC97, but for modern ones use 
as `-device intel-hda -device hda-micro,audiodev=sd1 -audiodev pa,id=sd1,server=unix:/run/user/<ID>/pulse/native`

#### Network device recommendations

**If the `netdev` backend is not provided, QEMU looks up for it and adds 
it automatically, this only works for a single netdev.**
For network devices this will rely on two options only as most recommended:

* `-device rtl8139,netdev=nd1 -netdev user,id=nd1` is the most compatible 
for older or newer systems, performance is enought decent.
* `-device virtio-net,netdev=nd1 -netdev user,id=nd1` is usefully for 
newer guest operating systems, performance will need of KVM.

#### Manage running qemu instance with qemu monitor and TCP connection

If you need to communicate with qemu remotely you can send network commands, by 
**using a TCP session over monitor and netcat**, for that the virtual machine instance 
must have a name with `-name` and declare a tcp/address with `-monitor` so we can 
use a unix connection to communicate with the virtual machine for commands.


1. Install need tools apart of qemu like `apk add netcat-openbsd`
2. Run qemu VM, add arguments with `-monitor tcp:0.0.0.0:<port>,server,nowait;`
3. No you can use netcat as: `echo info\ status |nc -N <ip> <port>`

Example: `echo info\ status |nc -N localhost 19101` if you run both in same machine 
and qemu was run with addition of `-monitor tcp:0.0.0.0:19101,server,nowait`.

This method is the most **insecure due lack of ssl, but the most flexible** because 
the connection can be made across network.

#### Manage running qemu instance with qemu monitor and Unix socket connection

If you need to communicate with qemu locally but secure you can send socket commands, 
**using unix socket and `sockat` utility**, for that the virtual machine instance 
must have a name with `-name` and declare a unix/socket with `-monitor` so we can 
use a unix connection to communicate with the virtual machine for commands.


1. Install need tools apart of qemu like `apk add sockat`
2. Run qemu VM, add arguments with `-monitor unix:<path/to/socketname>,server,nowait;`
3. No you can use netcat as: `echo info\ status |nc -N <ip> 19101`

Example: `echo "info status" | socat - unix-connect:qemuvm1socket` if you run 
both in same directory (qemu in one tty/console and socat in another tty/console) 
and qemu was run with addition of `-monitor unix:qemuvm1socket,server,nowait`

This method is the most **elegant and secure, but the most limited** because 
the unix socket is only local, or NFS made.

You can made a session monitor: `socat -,echo=0,icanon=0 unix-connect:qemuvm1socket`

#### Guest setup and combinations

* For 32-bit guests systems Intel 82801AA AC97 its recommended and could use http://www.linux-kvm.org/page/Sound .
* For 64-bit guests systems Intel HDA must be used most of those provided base usage already.
* USB 2.0 pass through can be configured from host to guest with variations of: `-usb -device usb-ehci,id=ehci -device usb-host,bus=ehci.0,vendorid=1452`

For Windows 8.1 USB tablet is available only with USB 2.0 pass through (QEMU option: `-device usb-ehci,id=ehci -device usb-tablet,bus=ehci.0`
The USB tablet device helps the Windows guest to accurately track mouse movements. Without it mouse movements will be jerky.
Another device that can be presented to the Windows guest is the random number generator. Add QEMU option: `-device virtio-rng-pci` . Now install the viorng driver from the driver image.
For Windows 10, to boot using UEFI the sys-firmware/edk2-ovmf is required on the host, then add QEMU option: `-bios /usr/share/edk2-ovmf/OVMF_CODE.fd`. to the qemu call. This option is essential for running Hyper-V guest images.

#### Spice share clipbard

using the SPICE guest agent you can shared the clipboard, you must install the SPICE guest tools from https://www.spice-space.org/download.html 
and configure your VM to enable the SPICE guest agent:

```
-device virtio-serial-pci \
-chardev spicevmc,id=vdagent,name=vdagent \
-device virtserialport,chardev=vdagent,name=com.redhat.spice.0
```

## see also

* [../tutorials/alpine-howto-qemu-on-pc.md](../tutorials/alpine-howto-qemu-on-pc.md)
* [../tutorials/alpine-howto-qemu-on-arm.md](../tutorials/alpine-howto-qemu-on-arm.md)
* [server-alpine-libvirt-qemu-virtualization.md](server-alpine-libvirt-qemu-virtualization.md)
* https://wiki.gentoo.org/wiki/QEMU/Windows_guest

# LICENSE

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
