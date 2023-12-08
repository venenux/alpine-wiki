# alpine server qemu

`qemu` its a emulation system that uses KVM (kernel virual machine) and also are capable of hypervision!

#### modes of emulation:

* Full: the qemu emulates all the machine, like have a pc inside other pc
* USer: the qemu uses binary translation so prograsm runs like into the native orignal machine

## QEMU alone for direc usage

The program can be used no mmater if you have hardware support or not in its simples way, 
of couse is the most slow way to use emulation but runs on any system. For advanced usage 
with KVM and hardware support forward to the next mayor section [QEMU with KVM and hardware virtualization](#qemu-with-kvm-and-hardware-virtualization).

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

apk add qemu-img qemu-system-$(uname -m) qemu-modules

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
/usr/bin/qemu-system-$(uname -m) \
  -m 256 \
  -boot once=d -cdrom alpine-standard-3.12.0-$(uname -m).iso 
  -net none 
  -name "alpinebootqemu1"
```

This will boot the alpine iso of the same architecture of the host machine, so 
you must have already configured and the iso in the same directory of the 
place where you execute the command, if not just give full path!


## QEMU with KVM and hardware virtualization

#### Checking Virtualization Hardware support

you must check if your CPU support emulation by the command:
`grep "vmx|svm" /proc/cpuinfo | uniq`, this is necesary for `kvm` implementation, 
if the above command does not show nothing you cannot do such emulation.

#### Installation qemu with kvm

* install basic need packages and qemu modules
* installing to path
* installing for loading tun module
* installing for loading vhost_net module
* detecting intel cpu and load their nested kvm module
* detecting amd cpu and if true loading nested kvm module

```
apk add bash qemu-img qemu-system-$(uname -m) qemu-modules libvirt-qemu  libvirt-daemon eudev-netifnames

grep tun /etc/modules|| echo tun >> /etc/modules
grep vhost_net /etc/modules|| echo vhost_net >> /etc/modules

/bin/bash -c [  -z '$(grep -i intel /proc/cpuinfo|head -n1)' ] && echo AMD  || modprobe kvm_intel nested=1 && echo "options kvm_intel nested=Y">/etc/modprobe.d/kvm_intel.conf

/bin/bash -c [  -z '$(grep -i amd /proc/cpuinfo|head -n1)' ] && echo INTEL  || modprobe kvm_amd nested=1 && echo "options kvm_amd nested=Y">/etc/modprobe.d/kvm_intel.conf
```

> Warning: the `eudev-netifnames` is only need for compatibility with SHITstemd related script if you will use it

##### running qemu with kvm support

* setup the user to run the machines
* add to the groups with privilegies
* load the tun and vhost_net modules
* add the services of the virtual library
* add the services of the virtual guest libraries


```
adduser -S -D -g '' -h /var/lib/libvirt qemuvirt

adduser qemuvirt qemu

adduser qemuvirt kvm

modprobe tun && modprobe vhost_net

/usr/bin/qemu-system-$(uname -m) \
  -m 256 \
  -boot once=d -cdrom alpine-standard-3.12.0-$(uname -m).iso 
  -net none 
  -name "alpinebootqemu1"
  -enable-kvm
```

This will boot the alpine iso of the same architecture of the host machine, so 
you must have already configured and the iso in the same directory of the 
place where you execute the command, if not just give full path!
The adition this time is the `-enable-kvm` parameter that will bring 
improved performance, becouse will use direc hardware of the host machine.

> Warning: here we previously configure the program, 

#### running the qemu without the kvm support

If your computer that will act as host does not support the KVM 
infraestructure you can deactivate it explicit in command line:

```
/usr/bin/qemu-system-$(uname -m) \
  -m 256 \
  -net none 
  -name "alpinebootqemu1"
  -no-kvm
```

Obviously you will not need to confiugure any KVM or qemu group or user, 
the qemu system will just interpreted all the things but will runs more slow also.

This time we parsed `-no-kvm` but you can also add `-machine accel=tcg` to try 
others ways of optimization.

## QEMU with HugePages memory

Here we will configure hugepages for default system setup, 
for more deep use [../documents/alpine-newbie-hugepages.md](../documents/alpine-newbie-hugepages.md)

Foir this you must previously enabled KVM on your setup, 
as [QEMU with KVM and hardware virtualization](#qemu-with-kvm-and-hardware-virtualization) 
section described.

##### Auto Configure the hugepages for qemu

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

rmmod tun && rmmod vhost_net && rmmod vhost && modprobe vhost_net && modprobe tun

mount -t hugetlbfs -o rw,pagesize=$(grep Hugepagesize /proc/meminfo|tr -s ' '|cut -d' ' -f 2)k,mode=1770,relatime,gid=$(getent group qemu | cut -d':' -f3) hugetlbfs /dev/hugepages
```

The mode 17770 will allow to users modify only their own resources, 
the mount of the **hugetlbfs can be automatized by added this to fstab** with:

```
echo \
"hugetlbfs /dev/hugepages hugetlbfs rw,pagesize=$(grep Hugepagesize /proc/meminfo|tr -s ' '|cut -d' ' -f 2)k,mode=1770,relatime,gid=$(getent group qemu | cut -d':' -f3) 0 0" \
 >> /etc/fstab
```

#### running qemu with hugepages memory support

* setup the user to run the machines
* add to the groups with privilegies
* load the tun and vhost_net modules
* download and iso to boot


```
adduser -S -D -g '' -h /var/lib/libvirt qemuvirt

adduser qemuvirt qemu

adduser qemuvirt kvm

modprobe tun && modprobe vhost_net

wget https://dl-cdn.alpinelinux.org/alpine/v3.12/releases/$(uname -m)/alpine-standard-3.12.0-$(uname -m).iso

/usr/bin/qemu-system-$(uname -m) \
  -m 256 \
  -boot once=d -cdrom alpine-standard-3.12.0-$(uname -m).iso 
  -net none 
  -name "alpinebootqemu1"
  -enable-kvm
  -mem-path /dev/hugepages
```

This will boot the alpine iso of the same architecture of the host machine, so 
you must have already configured and the iso in the same directory of the 
place where you execute the command, if not just give full path!
The adition this time is the `-enable-kvm` and `-mem-path /dev/hugepages` parameters 
that will bring improved performance, becouse will use direc hardware of the host machine.

> Warning: here we previously configure the program, 

## Qemu usage

We learned how to setup property the environment to run qemu 
but now we must to setup the virtual machines, taking into consideration:

* qemu is command line only, no saved configurations are possible, 
  for that you must [use libvirt framework in combination](alpine-howto-qemu-libvirt-service.md)
* virsh can configure and save virtual machines and later just lauch
  but this will need the [use libvirt framework in combination](alpine-howto-qemu-libvirt-service.md)


## Starting a clean empty virtual machine

```
apk add qemu-system-$(uname -m)

/usr/bin/qemu-system-$(uname -m) \
  -m 256 \
  -net none 
  -name "alpinebootqemu1"
```

If you want to start again such machine you must to re run same command.

## Starting a clean empty virtual machine but from different architecture

This will start a virtual machine for aarch64 architecture with 356Megs of RAM 
but no network card and no hardisk configured, neither cdrom boot device.

```
apk add qemu-system-aarch64

/usr/bin/qemu-system-aarch64 \
  -m 256 \
  -net none 
  -name "alpinebootqemu2"
```

If you want to start again such machine you must to re run same command.

## Starting a clean empty aarch64 virtual machine adn boot iso alpine aarch64

This will start a virtual machine for aarch64 architecture with 356Megs of RAM 
but no network card and no hardisk configured, but will boot the iso alpine 
as representation of a CDROM device with a CD disk inserted so will boot alpine linux 3.19

```
apk add qemu-system-aarch64

wget https://dl-cdn.alpinelinux.org/alpine/v3.19/releases/aarch64/alpine-standard-3.19.0-aarch64.iso

/usr/bin/qemu-system-aarch64 \
  -m 256 \
  -boot once=d,menu=off -cdrom alpine-standard-3.19.0-aarch64.iso 
  -net none 
  -name "alpinebootqemu3"
```

The `menu=off` adition is need if you dont have a direct output monitor to check the screen 
of the virtual machine.. cos will wait until user input choose!

## Creating a virtual hard disk and manage it

The most simple is the **RAW** format, is **the most faster and compatible but lest featured**:

```
apk add qemu-img

qemu-img create -f raw computer1-vitualdisk1-file.raw 4G
```

You can use a **QCOW2** format, is **the most popular but not so faster than raw, but featured**:

```
apk add qemu-img

qemu-img create -f qcow2 computer1-vitualdisk2-file.img 4G
```

QCOW does not support holes, so is less storage occupies, only the already 
used by the virtual environment, also supports snapshots of the image in some 
point of the time with AES encryption.

## Starting a i386 virtual machine using prevous disks created

```
apk add qemu-system-i386 qemu-img

qemu-img create -f raw computer4-vitualdisk1-file.raw 4G

qemu-img create -f qcow2 computer4-vitualdisk2-file.img 4G

wget https://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86/alpine-standard-3.10.0-x86.iso

/usr/bin/qemu-system-i386 \
  -m 256 \
  -hda computer4-vitualdisk1-file.raw \
  -hdb computer4-vitualdisk2-file.img \
  -boot once=c,menu=off \
  -net none 
  -name "computer4alpine"
```

The `menu=off` adition is need if you dont have a direct output monitor to check the screen 
of the virtual machine.. cos will wait until user input choose! This will boot the iso file 
as representation of a CDROM device with a CD disk inserted so will boot alpine linux 3.10 
but the machine is 32bit only.

If you want to start again such machine you must to re run same command.

## Starting a i386 virtual machine with iso boot and previous disks created

```
apk add qemu-system-i386 qemu-img

qemu-img create -f raw computer4-vitualdisk1-file.raw 4G

qemu-img create -f qcow2 computer4-vitualdisk2-file.img 4G

wget https://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86/alpine-standard-3.10.0-x86.iso

/usr/bin/qemu-system-i386 \
  -m 256 \
  -cdrom alpine-standard-3.10.0-x86.iso 
  -hda computer4-vitualdisk1-file.raw \
  -hdb computer4-vitualdisk2-file.img \
  -boot once=d,menu=off \
  -net none 
  -name "computer4alpine"
```

The `menu=off` adition is need if you dont have a direct output monitor to check the screen 
of the virtual machine.. cos will wait until user input choose! This will boot the iso file 
as representation of a CDROM device with a CD disk inserted so will boot alpine linux 3.10 
but the machine is 32bit only.

If you want to start again such machine you must to re run same command.


## Starting a virtual machine with hard disk and net support


## see also


# LICENSE

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
