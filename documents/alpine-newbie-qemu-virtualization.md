# alpine qemu emulation

`qemu` its a emulation system that uses KVM (kernel virual machine) and also are capable of hypervision!

#### modes of emulation:

* Full: the qemu emulates all the machine, like have a pc inside other pc
* USer: the qemu uses binary translation so prograsm runs like into the native orignal machine

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
| CPU and motherboard | `cortex-a7 -machine virt` | `cortex-a35 -machine virt` | Note that ITS is not modeled in TCG mode |
| Defaults on chipset | GICv2M PCI GPIO | MSI GICv2M PCI/PCIe PL011 UART | no defaults, must select a machine profile |
| Hardware on virtual | it depends on machine | it depends on machine | Unfortunately Arm boards are currently undocumented |
| Use cases           | boot older software | basic testing only | migration of modern machines can be done with "q35" |
| it required KVM?    | cannot be determine | not tested | if you dont use kvm you cannot translate real devices in general |

##### emulation of devices and compatibility

| device type  | command line                 | i386  | amd64 | arm  | aarch64 |  observations |
| ------------- | --------------------------- | --- |  --- |  --- |  --- | ----------- |
| USB 1.x       | `-device pci-ohci`          | [x] | [x] | [x] | [x] | can be dificult with arm machines |
| USB 2.x       | `-device usb-ehci`          | [x] | [x] | [x] | [x] | is not the best option, consumes CPU |
| USB 3.x+2.x   | `-device nec-usb-xhci`      | [x] | [x] | [x] | [x] | best option, recent version has `qemu-xhci` |
| keyboard      | ``      | [x] | [x] | [x] | [x] | best option, recent version has `qemu-xhci` |
| Storage SCSI  | `-device virtio-scsi-device -device scsi-hd,drive=id1` | [x] | [x] | [x] | [x] | use with `-drive file=diskimg.qcow,if=none,id=d1` |

#### terminology to understand mechanish

The host is the plaform and architecture which QEMU is running on.

The guest is the architecture which is emulated by QEMU.

Initially QEMU was an emulation engine, with a Just-In-Time compiler (TCG) 
to dynamically translate target instruction set architecture (ISA) to host ISA.

There exists scenario where target and host architectures are the same. 
The terminology is usually Host and Guest (target).

Nowadays, QEMU offers virtualization through different accelerators. 
Virtualization is considered an accelerator because it prevents unneeded 
emulation of instructions when host and target share the same architecture. 

Only system level (aka supervisor/ring0) instructions might be emulated/intercepted.

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
* CPU model `host requires KVM.
* IOMMU must require KVM support, so combinations cannot be made for some cases
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
