# alpine qemu emulation on a computer

`qemu` its a emulation system, complete alpine documentation 
for Qemu is on [../documents/alpine-newbie-qemu-virtualization.md](../documents/alpine-newbie-qemu-virtualization.md)

* **Host** where the qemu is runinng and emulated another machine
* **Guest** the emulated result product inside the virtual machine

Fortunately, since versions 2 of qemu all unspecified options if not specified 
will be auto configured according to the available environment.

Hardware virtualization only could be possible when host and guest are same or 
similar architecture.

## Emulate i386 COMPUTER ON ANY MACHINE without hardware virtualization

The i386 is the ancient name for 32bit computers, that are really started 
since i286+DX math co-procesor. The amd64 is the 64bit and is not same as ia64.

* added alpine repositories
* update database repositories and install packages for 32bit x86 and modules
* load and setup the tun module
* allow user of qemu group to manage briged devices
* change permissions of the configurations
* added/create our user to run the virtual machines (prevents lack of security)
* added the user to qemu group to property create virtualmachines with access

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update && apk add qemu qemu-img qemu-system-i386 qemu-modules wget

grep tun /etc/modules|| echo tun >> /etc/modules

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf

chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

adduser -S -D -g '' -s /bin/bash -h /home/general general

adduser general qemu
```

* change (or init X11 graphical session) for the user
* create virtual disk: faster RAW format, 4 gigs of size, for 32bit i386
* download the iso file to boot for install a 32bit x86 operating system
* run the vitual machine with but no matter host hardware

```
su -l  general

/usr/bin/qemu-img create -f raw vm1x86alpine-disk1.raw 4G

wget https://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86/alpine-standard-3.10.0-x86.iso

/usr/bin/qemu-system-i386 \
  -m 1024 \
  -cpu n270 -machine pc \
  -accel tcg -bios /usr/share/qemu/bios.bin \
  -boot order=dc \
  -cdrom alpine-standard-3.10.0-x86.iso \
  -hda vm1x86alpine-disk1.raw \
  -display curses
```

Here we pass `-accel tcg` to be able to run it on any host that is huge different 
hardware, example running this i386 over an arm host device, the `-display curses` 
is because we dont have X11 session initialized, but only works with older linux 
image isos, under X11 session use `-display none -serial mon:stdio -echr 2` because 
the x86/x64 linux kernels always request VGA framebuffers so we just redirect 
console ouput to the qemu monitor but not linux error; then the iso will boot the 
alpine system into the virtual machine, **you can perform all the steps of a real 
hardware machine installation** into such virtual machine, **after is finished you 
can just boot again but with no iso/cdrom** boot:

```
/usr/bin/qemu-system-i386 \
  -m 1024 \
  -cpu n270 -machine pc -bios /usr/share/qemu/bios.bin \
  -drive file=vm1x86alpine-disk1.raw,format=raw \
  -display curses
```

Once tested you can then power off (inside virtual machine) and re run with a bit 
more ram (example, 3072 here, you cannot assing more than half or real, and in 
the case of 32bit x86 please dont address more than 4 gigs of RAM);
run **again but with minimal set of devices hardware like sound and usb** as:

```
/usr/bin/qemu-system-i386 \
  -m 3072 \
  -name "vm1x86alpine310" \
  -cpu n270 -machine pc \
  -bios /usr/share/qemu/bios-256k.bin \
  -device ide-hd,drive=hd0 -drive file=vm1x86alpine-disk1.raw,id=hd0,format=raw,if=none \
  -device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3122-:22 \
  -device pci-ohci -device nec-usb-xhci \
  -device virtio-keyboard -device virtio-mouse -device virtio-tablet \
  -device vga,vgamem_mb=32 -device AC97 \
  -display curses
```

* the `-cpu n270 -machine pc` forces 32bit most compatible SSE2 cpu on defualt i440fx and PIIX3
* the `-bios /usr/share/qemu/bios-256k.bin` is optional featured, only for 32bit machines without EFI implementation
* the `-device ide-hd,drive=hd0 -drive file=vm1x86alpine-disk1.raw,id=hd0,format=raw,if=none` storage using SATA most compatible
* the `-device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3122-:22` allows sync network access and ssh on 3222
* the `-device pci-ohci` enables USB 1.1 that allows to use older USB devices or emulate such bus, but only one device
* the `-device nec-usb-xhci` enables USB 3.0 that allows to use also USB 2.0 in fact, but only few connected devices
* the `-device virtio-keyboard -device virtio-mouse -device virtio-tablet` allows keyboard and mouse touchpad
* the `-device vga,vgamem_mb=32 -device AC97` allows direct graphics video and audio for older 32bit pc
* the `-display curses` will only work if there is no graphic or modesetting or no X sessiom running

Now fron any of the machines of the network you can connect using ssh, by example 
if you run qemu in your own machine using `ssh root@localhost -p 3122`, if the 
console hangs with no output try `-display none -serial mon:stdio -echr 2` when 
running the qemu command.

## Emulate ARM64 COMPUTER ON ANY MACHINE without hardware virtualization

The aarch64 is the final name for 64bit ARM computers since ARMv8, for 32bit they 
are from ARMv1 to ARMv7. Cortex CPUs are mostly ARM and most recents are 64bit only.

> **WARNING** alpine packagers made stupid error, the `aavmf` package is hardmade \
only to `aarch64` architecture a nonsense problem cos is just a file that can \
be used by any other operating system in any architecture (Check debian packages), \
so you must download manually and installed forced.

* added alpine repositories
* update database repositories and install packages for 64bit x86 and modules
* download and install aarch64 bios files incorrected packaged in only one architecture
* load and setup the tun module
* allow user of qemu group to manage briged devices
* change permissions of the configurations
* added/create our user to run the virtual machines (prevents lack of security)
* added the user to qemu group to property create virtualmachines with access

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

wget https://dl-cdn.alpinelinux.org/alpine/v3.18/community/aarch64/aavmf-0.0.202302-r0.apk

apk add --allow-untrusted aavmf-0.0.202302-r0.apk

apk update && apk add qemu qemu-img qemu-system-aarch64 qemu-modules wget aavmf

grep tun /etc/modules|| echo tun >> /etc/modules

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf

chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

adduser -S -D -g '' -s /bin/bash -h /home/general general

adduser general qemu
```

* change (or init X11 graphical session) for the user
* create virtual disk: faster RAW format, 4 gigs of size, for 64bit ARM
* download the iso file to boot for install a 32bit x86 operating system
* run the vitual machine but no matter host hardware

```
su -l  general

/usr/bin/qemu-img create -f raw vm2arm64alpine-vitualdisk1-file.raw 4G

wget https://dl-cdn.alpinelinux.org/alpine/v3.19/releases/aarch64/alpine-standard-3.19.0-aarch64.iso

/usr/bin/qemu-system-aarch64 \
  -m 1024 \
  -cpu cortex-a35 -machine virt \
  -accel tcg -bios /usr/share/AAVMF/QEMU_EFI.fd \
  -boot order=dc \
  -cdrom alpine-standard-3.19.0-aarch64.iso \
  -hda vm2arm64alpine-vitualdisk1-file.raw \
  -display curses -nographic
```

Here we pass `-accel tcg` to be able to run it on any host that is huge different 
hardware, for example running this i386 over an arm host device also 
pass `-display curses -nographic` because we dont have X11 session initalized, 
the arm linux kernels never have an active VGA framebuffers so we just redirect 
console ouput to the serial tty active console; then the iso will boot the 
alpine system into the virtual machine, **you can perform all the steps of a real 
hardware machine installation** into such virtual machine, **after is finished you 
can just boot again but with no iso/cdrom** boot:

```
/usr/bin/qemu-system-aarch64 \
  -m 1024 \
  -cpu cortex-a35 -machine virt \
  -accel tcg -bios /usr/share/AAVMF/QEMU_EFI.fd \
  -drive file=vm2arm64alpine-vitualdisk1-file.raw,id=hd0,format=raw \
  -display curses -nographic
```

Once tested you can then power off (inside virtual machine) and re run with a bit 
more ram (example, 2048 here, you cannot assing more than half or real RAM of system 
host where you run the ARM64 virtual, also dont put more than 4096 for best compat),
run **again but with minimal set of devices hardware like sound and usb** as:

```
/usr/bin/qemu-system-aarch64 \
  -m 2048 \
  -name "vm2arm64alpine319" \
  -cpu cortex-a35 -machine virt 
  -bios /usr/share/AAVMF/QEMU_EFI.fd \
  -device virtio-scsi -device scsi-hd,drive=hd0 -drive file=vm2arm64alpine-vitualdisk1-file.raw,id=hd0,format=raw,if=none \
  -device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3222-:22 \
  -device pci-ohci -device nec-usb-xhci \
  -device virtio-keyboard -device virtio-mouse -device virtio-tablet \
  -device virtio-gpu,max_outputs=1 -device AC97 \
  -display none
```

* the `-cpu cortex-a35 -machine virt` are the most compatible ARM 64bit cpu and machine, for best compatibility to any ARM activty
* the `-bios /usr/share/AAVMF/QEMU_EFI.fd` is mandatory, all ARM machines has their own bios/way so we use most common for 64bit
* the `-device virtio-scsi` just add new base backend, teh scsi is the most compatible but with degraded performance
* the `-device scsi-hd,drive=hd0 -drive file=vm2arm64alpine-vitualdisk1-file.raw,id=hd0,format=raw,if=none` improved SATA
* the `-device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3222-:22` allows sync network access and ssh on 3222
* the `-device pci-ohci` enables USB 1.1 that allows to use older USB devices or emulate such bus, but only one device
* the `-device nec-usb-xhci` enables USB 3.0 that allows to use also USB 2.0 in fact, but only few connected devices
* the `-device virtio-keyboard -device virtio-mouse -device virtio-tablet` allows keyboard and mouse touchpad
* the `-device virtio-gpu,max_outputs=1 -device AC97` allows direct graphics video but compatible ac97 audio input/output

Now fron any of the machines of the network you can connect using ssh, by example 
if you run qemu in your own machine using `ssh root@localhost -p 3222`

## Emulate AMD64 COMPUTER ON ANY MACHINE without hardware virtualization and two disks

The amd64 is the current codename of modern 64bit computers, that are really started 
since AMD phenon CPU. The amd64 is the 64bit and is not same as ia64. This time we 
will attach two disk and also external hardware to use internally. But there will 
no hardware direct access due the KVM missing feature (means this will work in any 
computer no matter if you use or not similar architecture)

* added alpine repositories
* update database repositories and install packages for 64bit x86 and modules
* load and setup the tun module
* allow user of qemu group to manage briged devices
* change permissions of the configurations
* added/create our user to run the virtual machines (prevents lack of security)
* added the user to qemu group to property create virtualmachines with access

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update && apk add qemu qemu-img qemu-system-x86_64 qemu-modules wget

grep tun /etc/modules|| echo tun >> /etc/modules

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf

chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

adduser -S -D -g '' -s /bin/bash -h /home/general general

adduser general qemu
```

* change (or init X11 graphical session) for the user
* create virtual disk: faster RAW format, 2 gigs of size, for 64bit amd64
* create slave disk: slower (but improved) QCOW2 format, 2 gigs of size
* download the iso file to boot for install a 32bit x86 operating system
* run the vitual machine with but no matter host hardware

```
su -l  general

/usr/bin/qemu-img create -f raw -o preallocation=full vm0x64alpine-vitualdisk1-file.raw 2G

/usr/bin/qemu-img create -f qcow2 -o preallocation=full,cluster_size=512k, vm0x64alpine-vitualdisk2-file.raw 2G

wget https://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86/alpine-standard-3.10.0-x86.iso

/usr/bin/qemu-system-x86_64 \
  -m 1024 \
  -cpu n270 -machine pc \
  -accel tcg -bios /usr/share/qemu/bios.bin \
  -boot order=dc 
  -cdrom alpine-standard-3.10.0-x86.iso  \
  -hda vm0x64alpine-vitualdisk1-file.raw \
  -hdb vm0x64alpine-vitualdisk2-file.raw \
  -display none -serial mon:stdio
```

Here we pass `-accel tcg` to be able to run it on any host that is huge different 
hardware, for example running this i386 over an arm host device also 
pass `-display none -serial mon:stdio` because we dont have X11 session initalized, 
the x86/x64 linux kernels always request VGA framebuffers so we just redirect 
console ouput to the qemu monitor but not linux error; then the iso will boot the 
alpine system into the virtual machine, **you can perform all the steps of a real 
hardware machine installation** into such virtual machine, **after is finished you 
can just boot again but with no iso/cdrom** boot:

```
/usr/bin/qemu-system-i386 \
  -m 1024 \
  -cpu n270 -machine pc -bios /usr/share/qemu/bios.bin \
  -drive file=vm0x64alpine-vitualdisk1-file.raw,id=hd0,format=raw,index=0
  -drive file=vm0x64alpine-vitualdisk2-file.raw,id=hd1,format=qcow2,index=1
  -display none -serial mon:stdio
```

Once tested you can then power off (inside virtual machine) and re run with a bit 
more ram (example, 3072 here, you cannot assing more than half or real, and in 
the case of 32bit x86 please dont address more than 4 gigs of RAM);
run **again but with minimal set of devices hardware like sound and usb** as:

```
/usr/bin/qemu-system-i386 \
  -m 3072 \
  -name "vm1x86alpine310" \
  -cpu n270 -machine pc \
  -bios /usr/share/qemu/bios-256k.bin
  -device virtio-blk -drive file=vm1x86alpine-disk1.raw,id=hd0,format=raw,if=none \
  -device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3122-:22 \
  -device pci-ohci -device nec-usb-xhci \
  -device virtio-keyboard -device virtio-mouse -device virtio-tablet \
  -device vga,vgamem_mb=32 -device AC97 \
  -display none
```

* the `-cpu n270 -machine pc` forces 32bit most compatible SSE2 cpu on defualt i440fx and PIIX3
* the `-bios /usr/share/qemu/bios-256k.bin` is optional featured, only for 32bit machines without EFI implementation
* the `-device virtio-scsi` just add new base backend, teh scsi is the most compatible but with degraded performance
* the `-device scsi-hd,drive=hd0 -drive file=vm1x86alpine-disk1.raw,id=hd0,format=raw,if=none` improved SATA
* the `-device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3122-:22` allows sync network access and ssh on 3222
* the `-device pci-ohci` enables USB 1.1 that allows to use older USB devices or emulate such bus, but only one device
* the `-device nec-usb-xhci` enables USB 3.0 that allows to use also USB 2.0 in fact, but only few connected devices
* the `-device virtio-keyboard -device virtio-mouse -device virtio-tablet` allows keyboard and mouse touchpad
* the `-device vga,vgamem_mb=32 -device AC97` allows direct graphics video and audio for older 32bit pc

Now fron any of the machines of the network you can connect using ssh, by example 
if you run qemu in your own machine using `ssh root@localhost -p 3122`

## Emulation in simple mode on ANY MACHINE OF AN ARM32 COMPUTER without hardware virtualization

The "arm" simple only were 32bit names for ARMv1 to ARMv7. Qemu 
only can emulate those from ARMv4 to the ARMv7, ARMv1 to ARMv4 seems not documented. 
Cortex CPUs are mostly 64bit only except ARMv6 and ARMv7. For those use aarch64.

* added alpine repositories
* update database repositories and install packages for 32bit ARM, modules and bios
* load and setup the tun module
* allow user of qemu group to manage briged devices
* change permissions of the configurations
* added/create to our user to run the virtual machines (prevents lack of security)
* added the user to qemu group to property create virtualmachines with access

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update && apk add qemu qemu-img qemu-system-arm qemu-modules wget ovmf

grep tun /etc/modules|| echo tun >> /etc/modules

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf

chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

adduser -S -D -g '' -s /bin/bash -h /home/general general

adduser general qemu
```

* change (or init X11 graphical session) for the user
* create virtual disk: faster RAW format, 4 gigs of size, for the 32bit system
* download the iso file to boot and install the 32bit ARM operating system
* run the vitual machine, but **with OVMF bios firmware and specific cpu core**

```
su -l  general

/usr/bin/qemu-img create -f raw vm2arm7alpine-disk1.raw 4G

wget https://dl-cdn.alpinelinux.org/alpine/v3.19/releases/armv7/alpine-standard-3.19.0-armv7.iso

/usr/bin/qemu-system-aarch64 \
  -m 1024 \
  -cpu cortex-a7 -machine virt -bios /usr/share/OVMF/QEMU_EFI.fd \
  -accel tcg \
  -boot order=dc \
  -cdrom alpine-standard-3.19.0-armv7.iso \
  -hda vm2arm7alpine-disk1.raw \
  -display curses -nographic
```

Here we pass `-accel tcg` because we could run in any host machine not only same ARM, 
also we pass `-bios /usr/share/OVMF/QEMU_EFI.fd -cpu cortex-a7 -machine virt` as mandatory 
cos aarch64 will not run if you dont specify a bios boot and cpu core, also 
and `-display curses` becouse we dont have X11 session initalized, but also
add `-nographic` because inital redirection in ARM devices are not happened to any display 
and we will assume you dont have any x11 session running fi were the case.

But this will have two little details that could relly into problems:

* the image format is not specified, but qemu will limited the writting
* you cannot interact with the machine (no keyboard response) due lack of usb devices

Change the drive parameter and boot again with supprot for drive and keyboard using usb as:

```
/usr/bin/qemu-system-arm \
  -m 1024 \
  -cpu cortex-a7 -machine virt -bios /usr/share/OVMF/QEMU_EFI.fd \
  -accel tcg \
  -boot order=cd \
  -cdrom alpine-standard-3.19.0-armv7.iso \
  -device usb-ehci -device usb-kbd -device usb-mouse \
  -drive file=vm2arm7alpine-disk1.raw,format=raw \
  -display curses -nographic
```

The iso will boot the alpine system into the virtual machine, **you can perform 
all the steps of a real hardware machine installation** into such virtual machine, 
**after is finished you can just boot again but with no iso/cdrom** boot:

```
/usr/bin/qemu-system-arm \
  -m 1024 \
  -cpu cortex-a7 -machine virt -bios /usr/share/OVMF/QEMU_EFI.fd \
  -hda vm1x86alpine-disk1.raw \
  -display curses -nographic
```

Once tested you can then power off (inside virtual machine) and re run 
with a little of more ram (2048 here, you cannot assing more than half or real),
run **again but with minimal set of devices hardware like sound and usb** as:

```
/usr/bin/qemu-system-arm \
  -m 1024 \
  -name "vm2arm7alpine319" \
  -cpu cortex-a7 -machine virt -bios /usr/share/OVMF/QEMU_EFI.fd \
  -device virtio-scsi -device scsi-hd,drive=hd0 -drive file=vm1x86alpine-disk1.raw,id=hd0,format=raw,if=none \
  -device rtl8139,netdev=nd1 -netdev user,id=nd1 \
  -device pci-ohci -device nec-usb-xhci \
  -device virtio-keyboard -device virtio-mouse -device virtio-tablet \
  -device cirrus-vga -device AC97 \
  -display curses -nographic
```

* the `-cpu n270 -machine pc` forces 32bit most compatible SSE2 cpu on defualt i440fx and PIIX3
* the `-device virtio-scsi` just add new base backend, this just feature kernel integration
* the `-device scsi-hd,drive=hd0 -drive file=vm1x86alpine-disk1.raw,id=hd0,format=raw,if=none` improved hda
* the `-device rtl8139,netdev=nd1 -netdev user,id=nd1` allows sync network access for internet browsing
* the `-device pci-ohci` enables USB 1.1 that allows to use older USB devices or emulate such bus
* the `-device nec-usb-xhci` enables USB 3.0 that allows to use also USB 2.0 in fact
* the `-device virtio-keyboard -device virtio-mouse -device virtio-tablet` allows keyboard and mouse touchpad
* the `-device cirrus-vga -device AC97` allows direct graphics video and audio for older 32bit pc

## Emulation on a x86_64 of an i386 or amd64 computer but hardware virtualization

you must check if your CPU support emulation by the command:
`apk add arch-install-scripts && LC_ALL=C lscpu | grep Virtualization`, 
this is necesary for `kvm` implementation, if the above command does 
not show nothing you cannot do such emulation in optimized way.

Hardware virtualization only could be possible when host and guest are same or 
similar architecture. By example run aarch64 over ARM machine or i386 over amd64 machine.

#### Prepare a i386 or amd64 virtual machine on a amd64 machine

* added alpine repositories
* update database repositories
* install basic need packages for emulate an x86 based and qemu modules
* load and setup the tun module
* installing for loading vhost_net module
* detecting intel cpu and load their nested kvm module
* detecting amd cpu and if true loading nested kvm module
* loading kvm module if still not loaded
* allow user of qemu group to manage briged devices
* change permissions of the configurations
* added/create to our user to run the virtual machines (prevents lack of security)
* added the user to the qemu group to property create virtualmachines with devices access
* added the user to the kvm group to property access hardware virtualization support

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add qemu-img qemu-system-i386 qemu-modules wget

grep tun /etc/modules|| echo tun >> /etc/modules
grep vhost_net /etc/modules|| echo vhost_net >> /etc/modules

/bin/bash -c [  -z '$(grep -i intel /proc/cpuinfo|head -n1)' ] && echo AMD  || modprobe kvm_intel nested=1 && echo "options kvm_intel nested=Y">/etc/modprobe.d/kvm_intel.conf

/bin/bash -c [  -z '$(grep -i amd /proc/cpuinfo|head -n1)' ] && echo INTEL  || modprobe kvm_amd nested=1 && echo "options kvm_amd nested=Y">/etc/modprobe.d/kvm_intel.conf

modprobe kvm

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf
chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

adduser -S -D -g '' -s /bin/bash -h /home/general general

adduser general qemu
adduser general kvm
```

##### running qemu 32bit i386 virtual with kvm support

* change (or initate session) on your user
* create the virtual disk to install the 64bit ARM system with faster RAW format
* download the iso file to boot and isntall a 64bit ARM operating system
* run the vitual machine with the prepared components, **but with** virtualization hardware

```
su -l  general

/usr/bin/qemu-img create -f raw computerint2alpine-vitualdisk1-file.raw 4G

wget https://dl-cdn.alpinelinux.org/alpine/v3.10/releases/x86/alpine-standard-3.10.3-x86.iso

/usr/bin/qemu-system-i386 \
  -m 256 \
  -hda computerint2alpine-vitualdisk1-file.raw \
  -cdrom alpine-standard-3.10.3-x86.iso \
  -boot once=d \
  -name "computerint2alpine310" \
  -enable-kvm -machine accel=kvm \
  -display curses
```

Here we pass `-enable-kvm -machine accel=kvm` for the hardware emulation 
and `-display curses` becouse we dont have X11 session initalized, if 
you already has xorg/X11 session you can just use `-display gtk`.

The iso will boot the alpine system into the virtual machine, you can perform all the steps 
of a real hardware machine installation into such virtual machine started, 
after is finished you can just **boot again but with no iso** boot:

```
/usr/bin/qemu-system-i386 \
  -m 1024 \
  -hda computerint2alpine-vitualdisk1-file.raw \
  -boot once=c \
  -name "computerint2alpine310" \
  -enable-kvm -machine accel=kvm \
  -device virtio-gpu -device usb-ehci -device intel-hda -device hda-output
  -display curses
```

Now after restart we added som paremeters, with KVM activated more hardware can 
be emulated without impact in performance, with such common parameters we add
the `-device virtio-gpu -device usb-ehci -device intel-hda -device hda-output` 
for VGA, USB, HDA, and oputput of HDA sound.

##### Auto Configure the hugepages for qemu on an amd64 machine

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

##### running qemu ARM 64 virtual with kvm support and hugepages

* change (or initate session) on your user, after the kvm and hugepages configuration (previous sections)
* create the virtual disk to install the 32bit x86 system with faster RAW format
* download the iso file to boot and isntall a 64bit ARM operating system
* run the vitual machine with the prepared components, **but with** virtualization hardware

```
su -l  general

/usr/bin/qemu-img create -f raw vm2arm7alpine-disk1.raw 4G

wget https://dl-cdn.alpinelinux.org/alpine/v3.19/releases/aarch64/alpine-standard-3.19.0-aarch64.iso

/usr/bin/qemu-system-aarch64 \
  -m 256 \
  -hda vm2arm7alpine-disk1.raw \
  -cdrom alpine-standard-3.19.0-aarch64.iso \
  -boot once=d \
  -name "vm2arm7alpine312" \
  -enable-kvm -machine accel=kvm -mem-path /dev/hugepages \
  -display curses
```

Here we pass `-enable-kvm -machine accel=kvm` for the hardware emulation 
also `-mem-path /dev/hugepages` to have better RAM management respect host 
and `-display curses` becouse we dont have X11 session initalized, if 
you already has xorg/X11 session you can just use `-display gtk`.


## Emulation on a x86_64 of a i386 computer with Debian 9 and networking sharing vnc using kvm and hugepages

you must check if your CPU support emulation by the command:
`apk add arch-install-scripts && LC_ALL=C lscpu | grep Virtualization`, 
this is necesary for `kvm` implementation, if the above command does 
not show nothing you cannot do such emulation in optimized way.

* added alpine repositories
* update database repositories
* install basic need packages for emulate a 32bit i386 and qemu modules
* load and setup the tun module
* installing for loading vhost_net module
* detecting intel cpu and load their nested kvm module
* detecting amd cpu and if true loading nested kvm module
* loading kvm module if still not loaded
* allow user of qemu group to manage briged devices
* change permissions of the configurations
* added/create to our user to run the virtual machines (prevents lack of security)
* added the user to the qemu group to property create virtualmachines with devices access
* added the user to the kvm group to property access hardware virtualization support

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add qemu-img qemu-system-i386 qemu-modules wget

grep tun /etc/modules|| echo tun >> /etc/modules
grep vhost_net /etc/modules|| echo vhost_net >> /etc/modules

/bin/bash -c [  -z '$(grep -i intel /proc/cpuinfo|head -n1)' ] && echo AMD  || modprobe kvm_intel nested=1 && echo "options kvm_intel nested=Y">/etc/modprobe.d/kvm_intel.conf

/bin/bash -c [  -z '$(grep -i amd /proc/cpuinfo|head -n1)' ] && echo INTEL  || modprobe kvm_amd nested=1 && echo "options kvm_amd nested=Y">/etc/modprobe.d/kvm_intel.conf

modprobe kvm

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf
chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

adduser -S -D -g '' -s /bin/bash -h /home/general general

adduser general qemu
adduser general kvm
```

##### running qemu ARM 64 virtual with kvm support

* change (or initate session) on your user
* create the virtual disk to install the 64bit ARM system with faster RAW format
* download the iso file to boot and isntall a 64bit ARM operating system
* run the vitual machine with the prepared components, **but with** virtualization hardware
* Foir ARM you must specify board model to use with the `-machine` option; there is no default.

```
su -l  general

/usr/bin/qemu-img create -f raw vm2arm7alpine-disk1.raw 4G

wget https://cdimage.debian.org/mirror/cdimage/archive/9.13.0-live/i386/iso-hybrid/debian-live-9.13.0-i386-mate.iso

/usr/bin/qemu-system-i386 \
  -m 256 \
  -hda vm2arm7alpine-disk1.raw \
  -cdrom debian-live-9.13.0-i386-mate.iso \
  -boot once=d \
  -name "vm2arm7alpine312" \
  -enable-kvm -machine accel=kvm \
  -display curses
```

Here we pass `-enable-kvm -machine accel=kvm` for the hardware emulation 
and `-display curses` becouse we dont have X11 session initalized, if 
you already has xorg/X11 session you can just use `-display gtk`.

The iso will boot the alpine sistem into the virtual machine, you can perform all the steps 
of a real hardware machine installation into such virtual machine started, 
after is finished you can just **boot again but with no iso** boot:

```
/usr/bin/qemu-system-i386 \
  -m 1024 \
  -hda vm2arm7alpine-disk1.raw \
  -boot once=c \
  -name "vm2arm7alpine312" \
  -enable-kvm -machine accel=kvm \
  -display curses
```


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

This will start a virtual machine for i386 architecture with 356Megs of RAM 
but no network card and no hardisk configured, neither cdrom boot device.

```
apk add qemu-system-i386

/usr/bin/qemu-system-i386 \
  -m 256 \
  -net none 
  -name "alpinebootqemu4"
  -display curses
```

If you wants to get out of the text machine you must to kill the command.
If you want to start again such machine you must to re run same command.

## Tutorials for qemu

#### emulation x86 machines over any x86 using KVM and hugepages and nested emulation

As super user, under an x86 computer only (emulation must be of x86 also, so kvm will work):

> **Warning** we assumed 2M hugepagesize and 16G system host RAM! use "16" instead of "4096" on `nr_hugepages` if not!

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add qemu qemu-img qemu-system-i386 qemu-modules wget

grep tun /etc/modules|| echo tun >> /etc/modules
grep vhost_net /etc/modules|| echo vhost_net >> /etc/modules

/bin/bash -c [  -z '$(grep -i intel /proc/cpuinfo|head -n1)' ] && echo AMD  || modprobe kvm_intel nested=1 && echo "options kvm_intel nested=Y">/etc/modprobe.d/kvm_intel.conf
/bin/bash -c [  -z '$(grep -i amd /proc/cpuinfo|head -n1)' ] && echo INTEL  || modprobe kvm_amd nested=1 && echo "options kvm_amd nested=Y">/etc/modprobe.d/kvm_intel.conf

modprobe kvm

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf
chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

adduser -S -D -g '' -s /bin/bash -h /home/general general && adduser general qemu && adduser general kvm

echo 4096  > /proc/sys/vm/nr_hugepages && echo 1 > /proc/sys/vm/compact_memory

echo "options vhost max_mem_regions=16" > /etc/modprobe.d/vhost.conf

rmmod tun && rmmod vhost_net && rmmod vhost && modprobe vhost_net && modprobe tun

mount -t hugetlbfs -o rw,pagesize=$(grep Hugepagesize /proc/meminfo|tr -s ' '|cut -d' ' -f 2)k,mode=1770,relatime,gid=$(getent group kvm | cut -d':' -f3) hugetlbfs /dev/hugepages

su -l general
```

As user general, emulation of i386 machine boot for alpine install

```
mkdir -p /home/general/VMs/vm1x86alpine318 && cd /home/general/VMs/vm1x86alpine318

wget https://dl-cdn.alpinelinux.org/alpine/v3.18/releases/x86/alpine-extended-3.18.5-x86.iso

qemu-img create -f raw -o preallocation=full vm1x86alpine318.raw 4G

/usr/bin/qemu-system-i386 \
 -m 2048 -mem-path /dev/hugepages \
 -name "computerint1alpine318" -rtc base=localtime \
 -cpu host -machine pc -accel kvm \
 -drive file=vm1x86alpine318.raw,id=hd0,format=raw \
 -device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3222-:22 \
 -device pci-ohci -device nec-usb-xhci \
 -device cirrus-vga,vgamem_mb=16 \
 -device AC97 \
 -display none -serial mon:stdio -echr 2 \
 -cdrom alpine-extended-3.18.5-x86.iso -boot order=d
```

Then install alpine and later still as general user after alpine installed, poweroff and again run:

```
/usr/bin/qemu-system-i386 \
 -m 2048 -mem-path /dev/hugepages \
 -name "computerint1alpine318" -rtc base=localtime \
 -cpu n270 -machine pc,hpet=true,acpi=on -accel kvm \
 -object iothread,id=iot0 \
 -drive file=vm1x86alpine318.raw,id=hd0,format=raw \
 -device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3222-:22 \
 -device pci-ohci -device nec-usb-xhci \
 -device virtio-keyboard -device virtio-mouse -device virtio-tablet \
 -device virtio-vga,max_outputs=1 \
 -device AC97 \
 -display none -nographic \
```

#### emulation amd64 machines over amd64 using KVM and hugepages and nested emulation

As super user, under an 64bit computer only (emulation must be of x86 also, so kvm will work):

> **Warning** we assumed 1G hugepagesize and 64G system host RAM! use "4096" instead of "16" on `nr_hugepages` if not!

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add qemu qemu-img qemu-system-x86_64 qemu-modules wget

grep tun /etc/modules|| echo tun >> /etc/modules
grep vhost_net /etc/modules|| echo vhost_net >> /etc/modules

/bin/bash -c [  -z '$(grep -i intel /proc/cpuinfo|head -n1)' ] && echo AMD  || modprobe kvm_intel nested=1 && echo "options kvm_intel nested=Y">/etc/modprobe.d/kvm_intel.conf
/bin/bash -c [  -z '$(grep -i amd /proc/cpuinfo|head -n1)' ] && echo INTEL  || modprobe kvm_amd nested=1 && echo "options kvm_amd nested=Y">/etc/modprobe.d/kvm_intel.conf

modprobe kvm

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf
chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

adduser -S -D -g '' -s /bin/bash -h /home/general general && adduser general qemu && adduser general kvm

echo 16  > /proc/sys/vm/nr_hugepages && echo 1 > /proc/sys/vm/compact_memory

echo "options vhost max_mem_regions=16" > /etc/modprobe.d/vhost.conf

rmmod tun && rmmod vhost_net && rmmod vhost && modprobe vhost_net && modprobe tun

mount -t hugetlbfs -o rw,pagesize=$(grep Hugepagesize /proc/meminfo|tr -s ' '|cut -d' ' -f 2)k,mode=1770,relatime,gid=$(getent group kvm | cut -d':' -f3) hugetlbfs /dev/hugepages

su -l general
```

As user general, emulation of amd64 machine boot for alpine install

```
mkdir -p /home/general/VMs/vm1x64alpine318 && cd /home/general/VMs/vm1x64alpine318

wget https://dl-cdn.alpinelinux.org/alpine/v3.18/releases/x86/alpine-extended-3.18.5-x86_64.iso

qemu-img create -f raw -o preallocation=full vm1x64alpine318.raw 4G

/usr/bin/qemu-system-x86_64 \
 -m 4096 -mem-path /dev/hugepages \
 -name "computerint2alpine318" -rtc base=localtime \
 -cpu qemu64 -machine q35 -accel kvm \
 -device virtio-blk-pci,drive=hd0 -drive file=vm1x64alpine318.raw,id=hd0,format=raw,if=none \
 -device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3223-:22 \
 -device pci-ohci -device nec-usb-xhci \
 -device virtio-keyboard -device virtio-mouse -device virtio-tablet \
 -device virtio-vga,max_outputs=1 \
 -device AC97 \
 -display none -serial mon:stdio -echr 2 \
 -cdrom alpine-extended-3.18.5-x86_64.iso -boot order=d
```

As user general, emulation of amd64 machine boot after alpine install:

```
/usr/bin/qemu-system-x86_64 \
 -m 2048 -mem-path /dev/hugepages \
 -name "computerint2alpine318" -rtc base=localtime \
 -cpu host -machine q35 -accel kvm \
 -object iothread,id=iot0 \
 -device virtio-blk-pci,iothread=iot0,drive=hd0 -drive file=vm1x86alpine318.raw,id=hd0,format=raw,if=none \
 -device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3222-:22 \
 -device pci-ohci -device nec-usb-xhci \
 -device virtio-keyboard -device virtio-mouse -device virtio-tablet \
 -device virtio-vga,max_outputs=3 \
 -device AC97 \
 -display none -nographic \
```


#### how to change cdrom inside qemu using monitor output console

Qemu provides a way to change the iso in the virtual cdrom device via the monitor 
interface (Ctrl+Alt+2 if you have display output active in your active window of qemu).

```
QEMU 0.10.5 monitor - type 'help' for more information
(qemu)
```

The commands you'll want to use are `info block`, `eject`, and `change`. First we 
need to determine which block device is the cdrom device you are interested in. 
Issue the info block command and look if you already have any cdrom device support:

```
(qemu) info block
hd0 (#block180): /home/general/VMs/vm1x64alpine318/vm1x86alpine318.raw (raw)
    Attached to:      /machine/peripheral-anon/device[0]
    Cache mode:       writeback

sd0: [not inserted]
    Removable device: not locked, tray closed
```

The `sd0` is the only cdrom device in this example and there isn't any media inserted. 

1. eject any possible disk from the current cdrom device using `eject`
2. change the cdrom device disk by usage of the `change` command supplying the device name 

We must add the device name (sd0) and the path to the new iso file (that can be relative 
to the current directory), last optional command could be the format, raw for iso files.


```
(qemu) eject sd0

(qemu) change sd0 alpine-virt-3.19.0-x86_64.iso raw
```

We now can check the result of the operation and see that there is a device at the cdrom:

```
(qemu) info block
hd0 (#block180): /home/general/VMs/vm1x64alpine318/vm1x86alpine318.raw (raw)
    Attached to:      /machine/peripheral-anon/device[0]
    Cache mode:       writeback

sd0 (#block580): /home/general/VMs/vm1x64alpine318/alpine-virt-3.19.0-x86_64.iso (raw)
    Removable device: not locked, tray closed
    Cache mode:       writeback
```

To free such cdrom from such disk you can then eject the iso file image:

```
(qemu) eject sd0
```


## see also

* https://mathiashueber.com/virtual-machine-audio-setup-get-pulse-audio-working/
* https://lists.gnu.org/archive/html/qemu-discuss/2020-10/msg00031.html

# LICENSE

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
