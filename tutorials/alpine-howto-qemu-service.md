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

```
apk add qemu-img qemu-system-$(uname -m) libvirt-qemu  libvirt-daemon qemu-modules
```

## Configurations

The qemu has internal defaults, most of the parameters are just fo rinteraction, 
but some ones are important for first tries:

```
sed -i 's|.*nographics_allow_host_audio.*=.*|nographics_allow_host_audio = 1|g' /etc/libvirt/qemu.conf
sed -i 's|.*vnc_listen.*=.*|vnc_listen = 0.0.0.0|g' /etc/libvirt/qemu.conf
sed -i 's|.*vnc_tls.*=.*|vnc_tls = 0|g' /etc/libvirt/qemu.conf
sed -i 's|.*spice_sasl.*=.*|spice_sasl = 0|g' /etc/libvirt/qemu.conf
sed -i 's|.*chardev_tls.*=.*|chardev_tls = 0|g' /etc/libvirt/qemu.conf
sed -i 's|.*vxhs_tls.*=.*|vxhs_tls = 0|g' /etc/libvirt/qemu.conf
sed -i 's|.*nbd_tls.*=.*|nbd_tls = 0|g' /etc/libvirt/qemu.conf
sed -i 's|^.user =.*|user = "qemu"|g' /etc/libvirt/qemu.conf
sed -i 's|^.group =.*|group = "qemu"|g' /etc/libvirt/qemu.conf
sed -i 's|.*dynamic_ownership =.*|dynamic_ownership = 1|g' /etc/libvirt/qemu.conf
sed -i 's|.*hugetlbs_mount =.*|hugetlbs_mount = "/dev/hugepages"|g' /etc/libvirt/qemu.conf
sed -i 's|.*bridge-helper =.*|bridge-helper = "/usr/lib/qemu/qemu-bridge-helper"|g' /etc/libvirt/qemu.conf

sed -i 's|.*listen_tls =.*|listen_tls = 0|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*unix_sock_group =.*|unix_sock_group = "libvirt"|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*unix_sock_ro_perms =.*|unix_sock_ro_perms = "0750"|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*unix_sock_rw_perms =.*|unix_sock_rw_perms = "0770"|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*auth_unix_ro =.*|auth_unix_ro = "none"|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*auth_unix_rw =.*|auth_unix_rw = "none"|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*tls_no_sanity_certificate =.*|tls_no_sanity_certificate = 1|g' /etc/libvirt/libvirtd.conf
sed -i 's|.*tls_no_verify_certificate =.*|tls_no_verify_certificate = 1|g' /etc/libvirt/libvirtd.conf
```

##### running as unique web service

* setup the user to run the machines
* add to the groups with privilegies
* add the services of the virtual library
* add the services of the virtual guest libraries


```
addgroup -S libvirt

adduser -S -h /var/lib/libvirt qemu qemu

rc-update add libvirtd

rc-update add libvirt-guests
```

> Warning: here we previously configure the program, 

##### Configure the hugepages


```
cat > /etc/default/grub  << EOF
GRUB_TIMEOUT=2
GRUB_DISABLE_SUBMENU=y
GRUB_DISABLE_RECOVERY=true
GRUB_CMDLINE_LINUX_DEFAULT="modules=sd-mod,usb-storage,ext4,nvme quiet rootfstype=ext4 default_hugepagesz=1G hugepagesz=1G hugepages=8 hugepagesz=2M hugepages=4096 "
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
