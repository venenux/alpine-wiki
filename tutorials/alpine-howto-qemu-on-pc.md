# alpine qemu emulation on a computer

`qemu` its a emulation system, in a computer it can use 
the KVM for kernel virual machine, only in x86 and arm based coputers

Complete alpine documentation for Qemu is on [../documents/alpine-newbie-qemu-virtualization.md](../documents/alpine-newbie-qemu-virtualization.md)

This how to is only for qemu withour libvirt management, for that 
you could visit [alpine-howto-libvirt-qemu-kvm-service.md](alpine-howto-libvirt-qemu-kvm-service.md)

## Emulation in simple mode on a x86_64 of a i386 computer

Fortunately, since versions 2 of qemu all unspecified options is not specified 
will try to be configured according to the available environment.

First preparation of the things and later try to running the virtualization:

#### Prepare a 1386 virtual machine on a amd64 machine

* added alpine repositories
* update database repositories
* install basic need packages for emulate a 32bit x86 and qemu modules
* load and setup the tun module
* allow user of qemu group to manage briged devices
* change permissions of the configurations
* added/create to our user to run the virtual machines (prevents lack of security)

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add qemu-img qemu-system-i386 qemu-modules wget

grep tun /etc/modules|| echo tun >> /etc/modules

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf

chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

adduser -S -D -g '' -s /bin/bash -h /home/general general

adduser general qemu
```

#### Run a 1386 virtual machine on a amd64 machine without virtualization

* change (or initate session) on your user
* create the virtual disk to install the 32bit x86 system with faster RAW format
* download the iso file to boot and isntall a a 32bit x86 operating system
* run the vitual machine with the prepared components, but witouyt virtualization hardware

```
su -l  general

/usr/bin/qemu-img create -f raw computer1alpine-vitualdisk1-file.raw 4G

wget https://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86/alpine-standard-3.10.0-x86.iso

/usr/bin/qemu-system-i386 \
  -m 256 \
  -hda computer1alpine-vitualdisk1-file.raw
  -cdrom alpine-standard-3.10.0-x86.iso 
  -boot once=d
  -name "computer1alpine312"
  -no-kvm
  -display curses
```

Here we pass `-no-kvm` becouse we do not prepared kvm virtual environment, 
and `-display curses` becouse we dont have X11 session initalized, if 
you try to run a virtual machine will show you error about 

#### Emulation on a x86_64 of an ARM computer

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
adduser -S -D -g '' -s /bin/bash -h /var/lib/libvirt qemuvirt

adduser qemuvirt qemu

adduser qemuvirt kvm

modprobe tun && modprobe vhost_net

/usr/bin/qemu-system-$(uname -m) \
  -m 256 \
  -net none 
  -name "alpinebootqemu1"
  -enable-kvm
```

This will boot the alpine iso of the same architecture of the host machine, so 
you must have already configured and the iso in the same directory of the 
place where you execute the command, if not just give full path!
The adition this time is the `-enable-kvm` parameter that will bring 
improved performance, becouse will use direc hardware of the host machine.

> Warning: here we cannot see much more cos we dont pass enought arguments, check usage in sections below

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

> Warning: here we cannot see much more cos we dont pass enought arguments, check usage in sections below

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
grep hugetlbfs /etc/fstab || echo \
"hugetlbfs /dev/hugepages hugetlbfs rw,pagesize=$(grep Hugepagesize /proc/meminfo|tr -s ' '|cut -d' ' -f 2)k,mode=1770,relatime,gid=$(getent group qemu | cut -d':' -f3) 0 0" \
 >> /etc/fstab
```

#### running qemu with hugepages memory support

* setup the user to run the machines
* add to the groups with privilegies
* load the tun and vhost_net modules
* download and iso to boot
* run and pass the parameter to qemu command that uses the hugepages feature


```
adduser -S -D -g '' -s /bin/bash -h /var/lib/libvirt qemuvirt

adduser qemuvirt qemu

adduser qemuvirt kvm

modprobe tun && modprobe vhost_net

wget https://dl-cdn.alpinelinux.org/alpine/v3.12/releases/$(uname -m)/alpine-standard-3.12.0-$(uname -m).iso

/usr/bin/qemu-system-$(uname -m) \
  -m 256 \
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

> Warning: here we cannot see much more cos we dont pass enought arguments, check usage in sections below


## Qemu usage

We learned how to setup property the environment to run qemu 
but now we must to setup the virtual machines, taking into consideration:

* qemu is command line only, no saved configurations are possible, 
  for that you must [use libvirt framework in combination](alpine-howto-qemu-libvirt-service.md)
* virsh can configure and save virtual machines and later just lauch
  but this will need the [use libvirt framework in combination](alpine-howto-qemu-libvirt-service.md)


### Starting a clean empty virtual machine

Will start a simples default machine with 256 Megs of RAM

```
apk add qemu-system-$(uname -m)

/usr/bin/qemu-system-$(uname -m) \
  -m 256 \
  -net none 
  -name "alpinebootqemu1"
```

If you want to start again such machine you must to re run same command.


### Error initialization

If you dont have a X11 sesions, or if you dont have allowed to share your devices, by 
example when you run linux inside shit operating systems, will raise some errors like:

* `Error GTK initialization failed` means you dont have any X11 sesion running 
or you dont have allowe to connect to the x11 sesions (by example if you run from non 
linux host or inside a crap operating system)
* `MESA'LOADER: failed to open ...` means you cannot see the display running 
or you dont install X11/mesa environment/packages yet (by example running server only 
linux host or inside a crap operating system)

Solution is to run healess and setup non graphics output screen, for that we
will connect using vnc only so the screen will be server over network/localhost 
only, see next section for examples.


### Starting a clean empty virtual machine but no X11

Will start a simples default machine with 256 Megs of RAM, but 
if you are on a server the screen will be text only

```
apk add qemu-system-$(uname -m)

/usr/bin/qemu-system-$(uname -m) \
  -m 256 \
  -net none 
  -name "alpinebootqemu2"
  -display curses
```

No graphic devices will be on the virtual machine if not expecified (see more examples for)

This will start the virtual machine but will only work for text only operation 
by example FREEDOS or TTY, becouse there is no graphics defined only text mode 
becouse we used the `-display curses` that will only output text to the console 
and there is no connection for communications due we used `-net none` parameter.

If you wants to get out of the text machine you must to kill the command.

### Starting a clean empty virtual machine but no grapchics neither text or X11

Will start a simples default machine with 256 Megs of RAM, but 
if you are on a server the screen will be text only

```
apk add qemu-system-$(uname -m)

/usr/bin/qemu-system-$(uname -m) \
  -m 256 \
  -net none 
  -name "alpinebootqemu3"
  -display none
```

No graphic devices will be on the virtual machine if not expecified (see more examples for)

This will start the virtual machine but will not be any form of communication 
with the machine, becouse there is no graphics defined neither a display to see 
what is happened inside the virutal machine becouse we used the `-display none` 
parameter and ther is no connection due we used `-net none` parameter.

If you wants to get out of the text machine you must to kill the command.

### Starting a clean empty virtual machine but from different architecture

This will start a virtual machine for aarch64 architecture with 356Megs of RAM 
but no network card and no hardisk configured, neither cdrom boot device.

```
apk add qemu-system-aarch64

/usr/bin/qemu-system-aarch64 \
  -m 256 \
  -net none 
  -name "alpinebootqemu4"
  -display curses
```

If you wants to get out of the text machine you must to kill the command.
If you want to start again such machine you must to re run same command.

## Tutorials for qemu




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
