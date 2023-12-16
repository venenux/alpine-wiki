# alpine qemu emulation

`qemu` its a emulation system that uses KVM (kernel virual machine) and also are capable of hypervision!

Initially QEMU was an emulation engine, with a Just-In-Time compiler (TCG) 
to dynamically translate target instruction set architecture (ISA) to host ISA.

Nowadays, QEMU offers virtualization through different accelerators. 
Virtualization is considered an accelerator because it prevents unneeded 
emulation of instructions when host and target share the same architecture. 

Only system level (aka supervisor/ring0) instructions might be emulated/intercepted.

#### modes of emulation:

* Full: the qemu emulates all the machine, like have a pc inside other pc
* USer: the qemu uses binary translation so prograsm runs like into the native orignal machine

#### terminology to understand mechanish

The **host** is the plaform and architecture which QEMU is running on, is the native 
machine where the qemu will run the virtual machine

The **guest** is the architecture which is emulated by QEMU, is the target machine 
that will be emulated from the host or native machine.

The **device front** end is how a device is presented to the guest. The type of 
device presented should match the hardware that the guest operating system is 
expecting to see. All devices can be specified with the `--device` command line option. 
Running QEMU with the command line options `--device help` will list all devices.
Running QEMU with command line `--device foo,help` will list optons for "foo" device.
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
| CPU and motherboard | `-cpu n270 -machine pc` | `-cpu Conroe -machine q35` | each provides defaults based on natural features |
| Defaults on chipset | i440FX+PIIX3 PATA PS2 | G31+ICH9 SATA USB | no matter choosed features can be combined but not the best |
| Hardware on virtual | ISA, PATA, BIOS | PCIe, MMCFG, vIOMMU, AHCI, UEFI | For MSDOS or WinXP you must set "pc" machine |
| Use cases           | boot older software | P2V, OVMF, vIOMMU | migration of modern machines can be done with "q35" |
| it required KVM?    | no, if you use defaults | yes if you map mor devices | if you dont use kvm you cannot translate real devices |

#### emulation of arm based machines and best configurations

| machine option      | 32bit arm32 armv6  | 64bit aarch64 armv8 |  observations |
| ------------------- | --------------------- | ------------------- | ------------ |
| CPU and motherboard | `cortex-a7 -machine virt` | `cortex-a35 -machine virt` | no defaults, must select a machine profile |
| Defaults on chipset | GICv2M PCI GPIO | MSI GICv2M PCI/PCIe PL011 UART | Note that ITS is not modeled in TCG mode |
| Hardware on virtual | it depends on machine | it depends on machine | Unfortunately Arm boards are currently undocumented |
| Use cases           | boot older software | basic testing only | you must setup specific machine board |
| it required KVM?    | cannot be determine | not tested | if you dont use kvm you cannot translate real devices in general |

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

|  Type         |  Qemu builin | block feats | migration | extra required | observations |
| ------------- | ------------ | ----------- | --------- | -------------- | ----------------------- |
| Qemu emulated | IDE          | Yes         | Yes       | No, build in   | slow or mid performance |
| Qemu emulated | NVMe         | Yes         | Yes       | command line   | mid performance |
| Qemu emulated | virtio-blk   | Yes         | Yes       | command line   | relative best performance, few disks |
| Qemu emulated | virtio-scsi  | Yes         | Yes       | command line   | mid or slow performance, many disks |
| vhost         | vhost-scsi   | no          | no        | command line   | relative or mid performance |
| vhost SPDK    | vhost-user   | no          | no        | command line   | relative or mid performance |
| Device assignment | vfio-pci | no          | no        | Exclusive use  | Hihg performance, device is blocked by qemu |

Sometimes higher performance means less flexibility

### QEMU setup for simple virtualization

The program can be used no matter if you have hardware support or not in its 
simples way, of couse is the most slow way to use emulation but runs on any system. 
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

apk add qemu qemu-img qemu-system-i386 qemu-modules

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

We can just boot a machine from a simple boot device:

```
/usr/bin/qemu-system-i386  -name "alpinebootqemu1"
```

> **Warning**: QEMU should **never be run as root** use the `-runas` option to make QEMU drop root privileges.

#### Error initialization on qemu if no display

If you dont have a X11 sesions, or if you dont have allowed to share your devices, by 
example when you run linux inside shit operating systems, will raise some errors like:

* `Error GTK initialization failed` means you dont have any X11 sesion running 
or you dont have allowe to connect to the x11 sesions (by example if you run from non 
linux host or inside a crap operating system)
* `MESA-LOADER: failed to open ...` means you cannot see the display running 
or you dont install X11/mesa environment/packages yet (by example running server only 
linux host or inside a crap operating system)

Solution is to run healess and setup non graphics output screen, next setups will 
show two ways, first we just see the text only way with no X11 allowed:


#### Running qemu machines with simple emulation but no X11 output only text

We can just boot a machine but with arguments display to "ncurses" to run 
with no need of instalations of Xorg complete software:

```
/usr/bin/qemu-system-i386 -name "alpinebootqemu2" -display curses
```

The ncurses interface will take all the console output as a screen output!

To run again or terminate such command you will need to kill in another console!


### QEMU with KVM and hardware virtualization

You can improve the performance if your machine is PC based or ARM based (some) 
with the KVM (kernel virtual machine) and your vitualization support from hardware!

* The argument `accel=kvm` of the `-machine` option is equivalent to the `-enable-kvm`
* CPU model `host` requires KVM, but you must take care if you use PCI passthrough
* Using OVMF you can passthrough devices (a graphics card), offering the virtual machine native performance
* IOMMU must require KVM support, and `-device intel-iommu` should not be set if PCI passthrough is required
* IOMMU needs to adding the kernel parameter `intel_iommu=on` for remapping IO in guest
* If you start your virtual machine and experience very bad performance, should check for proper KVM support
* KVM needs to be enabled in order to start Win7 or Win8 properly without a blue screen.
* TPM must be enabled (throught q35 and UEFI+TPM fartures) in order to start Win10 without a blue screen.

#### Where can be loaded the KVM support

The KVM for Hardware virtualization **only could be possible when host and guest 
are same or similar architecture**, some examples:

| use case      | host    | guest   | KVM possible? |
| ------------- | ------- | ------- | ---------- |
| emulate i386  | i386    | i386    | Yes         |
| emulate amd64 | i386    | amd64   | Yes, only if host is also 64bit and kernel is amd64 |
| emulate i386  | amd64   | i386    | Yes         |
| emulate arm   | i386    | arm     | no, there is no common hardware for kvm |
| emulate i386  | aarch64 | i386    | no, there is no common hardware for kvm |
| emulate arm   | aarch64 | armv7   | Yes, but becouse host already has all the guest cpu feaures |
| emulate arm   | armv6   | armv7   | Yes, **but limited** host does not support all the guest features |
| emulate amd64 | amd64   | amd64   | Yes, only if host is a more recent cpu rather than guest |
| emulate arm   | aarch64 | aarch64 | Yes, only if host is a more recent cpu rather than guest | |

#### Checking Virtualization Hardware support

You must check if your CPU support emulation by the command:
`apk add arch-install-scripts && lscpu | grep Virtualization`, this is necesary 
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
/usr/bin/qemu-system-i386 -name "alpinebootqemu3" -enable-kvm -display curses
```

The adition of `-enable-kvm` can be combined with acceleration machine, 
if the host is amd64 then this i386 will run accelerated:

```
/usr/bin/qemu-system-i386 -name "alpinebootqemu4" -enable-kvm -machine accel=kvm -display curses
```

The adition this time is the `-machine accel=kvm` parameter that will give 
improved performance, because will use direc hardware of the host machine.
To run again or terminate such command you will need to kill in another console!

#### Running the qemu without the kvm support

If your computer that will act as host it does not support the KVM 
infraestructure you can deactivate it explicit in command line:

```
/usr/bin/qemu-system-$(uname -m) -name "alpinebootqemu5" -no-kvm -display curses
```

Obviously you will not need to confiugure any KVM or qemu group or user, 
the qemu system will just interpreted all the things but will runs more slow also.

This time we parsed `-no-kvm` but you can also add `-machine accel=tcg` to try 
others ways of optimization. To run again or terminate such command you will 
need to kill in another console!

### QEMU with HugePages memory

Here we will configure hugepages for default system setup, 
for more deep use [alpine-newbie-hugepages.md](alpine-newbie-hugepages.md)

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
/usr/bin/qemu-system-$(uname -m) -name "alpinebootqemu6" -enable-kvm -mem-path /dev/hugepages -display curses
```

The adition this time is the `-enable-kvm` and `-mem-path /dev/hugepages` parameters 
that will bring improved performance, because will use direc hardware of the host 
but also including RAM access mapping,  To run again or terminate such command you will 
need to kill in another console!

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

* RAW format: `qemu-img create -f raw storagedisk0.raw 10G`
* Qcow2 format: `qemu-img create -f qcow2 -o cluster_size=512k storagedisk1.img 10G`

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

By example, for the 10G previously created images:

* QCOW format: `-drive file=storagedisk1.img,if=none,l2_cache-size=1572864,cache_size=2097152,refcount-cache-size=262144,cache-clean-interval=700`

The values here for the 10G QCOW2 disk1 were estimated from the calculated values, 
so will cover also disk of 15G or 20G inclusivelly, the `cache-clean-interval=700` 
is to clean the cache cos large cache sizes implicts large amount of memory, because 
QEMU has a separate L2 cache for each qcow2 file, that gets worse with snapshots.

### Qemu and network configurations

Network configurations si the first way to configure communication support 
with guess from host, network configuration is always autoconfigured.

TODO

### Qemu usage

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
of vgabios, uefivga, VGA ouput and any OS supported, allows max 16Mgs video 
and is the best choice compatibility wise, pretty much any guest should 
be able to bring up a working display on this device; on default use 4Mgs 
of video, it not gets full hd but is close to 1024p if use 16 megs, The 
**best option for older systems emulaton and full emulation compatibilty**
2. `-device virtio-vga,max_outputs=2` the most compatible with modern and 
up to date systems, has no dedicated video memory (except VGA compat.), 
if supports vgabios, uefivga, VGA ouput, gest os will need special module 
that is since 3.2 into linux; it gets full HD on it defaults, has (optional) 
hardware-assisted opengl acceleration which in turn needs opengl support 
enabled if `-display xxx,gl=on` is enabled in the qemu display (sdl/gtk), 
**recommended for both modern or older systems with support of opengl**
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
if supports vgabios, uefivga, VGA ouput, and any guest OS will support it; 
featured 2D acceletarion so is mostly great for remote connection, but 
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
for linux or mac modern system and moder platform desktop emulation**

#### Audio and Network device recommendations

For audio in order of best compatibilty to most improvement:

1. `-device AC97` is enought for older or newer devices, it comes with 
the mic and output at the same time, all the inputs and ouputs are auto 
mapped to the backend and provided without , this is the **best option 
for emulation of 32 or 64 bits of ARM or X86 based computers**
2. `-device intel-hda -device hda-duplex` the High definition audio, 
that must be provided with a specific device way, the `hda-duplex` 
**will provide only "line in" and "line out" sound, no mic allowed**.
3. `-device intel-hda -device hda-micro` the High definition audio, 
that must be provided with a specific device way, the `hda-micro` 
**will provide only "mic in" and "line out" sound, no "line in" allowed**.

For network devices this will rely n two options only:

* `-device rtl8139,netdev=nd1 -netdev user,id=nd1` is the most compatible 
for older or newer systems, performance is enought decent.
* `-device virtio-net,netdev=nd1 -netdev user,id=nd1` is usefully for 
newer guest operating systems, performance will need of KVM.

## see also

* [../tutorials/alpine-howto-qemu-on-pc.md](../tutorials/alpine-howto-qemu-on-pc.md)
* [../tutorials/alpine-howto-qemu-on-arm.md](../tutorials/alpine-howto-qemu-on-arm.md)
* [server-alpine-libvirt-qemu-virtualization.md](server-alpine-libvirt-qemu-virtualization.md)

# LICENSE

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
