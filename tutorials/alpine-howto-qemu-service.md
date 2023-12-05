# alpine server qemu

`qemu` its a emulation system that uses KVM (kernel virual machine) and also are capable of hypervision!

#### modes of emulation:

* Full: the qemu emulates all the machine, like have a pc inside other pc
* USer: the qemu uses binary translation so prograsm runs like into the native orignal machine

## Preparations

* setup a hostname, here we use "qemu" string as hostname
* added and update normal repositories


```
hostname develvirtual

echo 'hostname="develvirtual"' > /etc/conf.d/hostname 

echo "develvirtual" > /etc/hostname

cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update
```

## Installation

* install basic need packages and qemu modules
* installing to path
* installing and loading tun module

```
apk add qemu-img qemu-system-$(uname -m) libvirt-qemu  libvirt-daemon qemu-modules eudev-netifnames

cat /etc/modules | grep tun || echo tun >> /etc/modules
```

> Warning: the `eudev-netifnames` is only need for compatibility with SHITstemd related script if you will use it

## Configurations

The qemu has internal defaults, most of the parameters are just for interaction, 
but some ones are important for first tries:

```
sed -i 's|.*vnc_listen.*=.*|vnc_listen = 0.0.0.0|g' /etc/libvirt/qemu.conf
sed -i 's|.*vnc_auto_unix_socket.*=.*|vnc_auto_unix_socket = 1|g' /etc/libvirt/qemu.conf
sed -i 's|.*vnc_tls.*=.*|vnc_tls = 0|g' /etc/libvirt/qemu.conf
sed -i 's|.*vnc_password.*=.*|#vnc_password = "0"|g' /etc/libvirt/qemu.conf
sed -i 's|.*vnc_sasl.*=.*|vnc_sasl = 0|g' /etc/libvirt/qemu.conf

sed -i 's|.*spice_listen.*=.*|spice_listen = 0.0.0.0|g' /etc/libvirt/qemu.conf
sed -i 's|.*spice_auto_unix_socket.*=.*|spice_auto_unix_socket = 1|g' /etc/libvirt/qemu.conf
sed -i 's|.*spice_tls.*=.*|spice_tls = 0|g' /etc/libvirt/qemu.conf
sed -i 's|.*spice_password.*=.*|#spice_password = "0"|g' /etc/libvirt/qemu.conf
sed -i 's|.*spice_sasl.*=.*|spice_sasl = 0|g' /etc/libvirt/qemu.conf

sed -i 's|^.security_driver.*=.*|security_driver = "none"|g' /etc/libvirt/qemu.conf
sed -i 's|.*chardev_tls.*=.*|chardev_tls = 0|g' /etc/libvirt/qemu.conf
sed -i 's|.*vxhs_tls.*=.*|vxhs_tls = 0|g' /etc/libvirt/qemu.conf
sed -i 's|.*nbd_tls.*=.*|nbd_tls = 0|g' /etc/libvirt/qemu.conf

sed -i 's|^.user =.*|user = "general"|g' /etc/libvirt/qemu.conf
sed -i 's|^.group =.*|group = "kvm"|g' /etc/libvirt/qemu.conf
sed -i 's|.*dynamic_ownership =.*|dynamic_ownership = 1|g' /etc/libvirt/qemu.conf

sed -i 's|.*hugetlbs_mount =.*|hugetlbs_mount = "/dev/hugepages"|g' /etc/libvirt/qemu.conf
sed -i 's|.*bridge-helper =.*|bridge-helper = "/usr/lib/qemu/qemu-bridge-helper"|g' /etc/libvirt/qemu.conf

sed -i 's|.*vnc_allow_host_audio.*=.*|vnc_allow_host_audio = 1|g' /etc/libvirt/qemu.conf
sed -i 's|.*nographics_allow_host_audio.*=.*|nographics_allow_host_audio = 1|g' /etc/libvirt/qemu.conf


sed -i 's|.*listen_tls =.*|listen_tls = 0|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*unix_sock_group =.*|unix_sock_group = "libvirt"|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*unix_sock_ro_perms =.*|unix_sock_ro_perms = "0750"|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*unix_sock_rw_perms =.*|unix_sock_rw_perms = "0770"|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*auth_unix_ro =.*|auth_unix_ro = "none"|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*auth_unix_rw =.*|auth_unix_rw = "none"|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*tls_no_sanity_certificate =.*|tls_no_sanity_certificate = 1|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*tls_no_verify_certificate =.*|tls_no_verify_certificate = 1|g' /etc/libvirt/libvirtd.conf

sed -i 's|.*allow br.*|allow br0|g' /etc/qemu/bridge.conf
chown -R root:qemu /etc/qemu
chmod 640 /etc/qemu/bridge.conf
```

##### running as unique web service

* setup the user to run the machines
* add to the groups with privilegies
* add the services of the virtual library
* add the services of the virtual guest libraries


```
adduser -S -D -g '' -h /var/lib/libvirt qemuvirt

adduser qemuvirt qemu

adduser qemuvirt kvm

rc-update add libvirtd

rc-update add libvirt-guests
```

> Warning: here we previously configure the program, 

##### Configure the hugepages

**Option 1 : 2Megs of huge pages**

```
cat > /etc/default/grub  << EOF
GRUB_TIMEOUT=2
GRUB_DISABLE_SUBMENU=y
GRUB_DISABLE_RECOVERY=true
GRUB_CMDLINE_LINUX_DEFAULT="modules=sd-mod,usb-storage,ext4,nvme rootfstype=ext4 default_hugepagesz=2M hugepagesz=1G hugepages=8 hugepagesz=2M hugepages=4096 "
EOF

mount -t hugetlbfs -o rw,pagesize=2M,mode=1770,relatime,gid=$(getent group qemu | cut -d':' -f3) none /dev/hugepages
```

**Option 2 : 1Gigs of huge pages**

```
cat > /etc/default/grub  << EOF
GRUB_TIMEOUT=2
GRUB_DISABLE_SUBMENU=y
GRUB_DISABLE_RECOVERY=true
GRUB_CMDLINE_LINUX_DEFAULT="modules=sd-mod,usb-storage,ext4,nvme rootfstype=ext4 default_hugepagesz=1G hugepagesz=1G hugepages=8 hugepagesz=2M hugepages=4096 "
EOF

mount -t hugetlbfs -o rw,pagesize=1024M,mode=1770,relatime,gid=$(getent group qemu | cut -d':' -f3) none /dev/hugepages
```


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
