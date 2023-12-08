# alpine server qemu

`qemu` its a emulation system that uses KVM (kernel virual machine) and also are capable of hypervision!

#### modes of emulation:

* Full: the qemu emulates all the machine, like have a pc inside other pc
* USer: the qemu uses binary translation so prograsm runs like into the native orignal machine

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


#### Configurations of qemu with libvirt on hardware

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
* load the tun and vhost_net modules
* add the services of the virtual library
* add the services of the virtual guest libraries


```
adduser -S -D -g '' -h /var/lib/libvirt qemuvirt

adduser qemuvirt qemu

adduser qemuvirt kvm

modprobe tun && modprobe vhost_net

rc-update add libvirtd

rc-update add libvirt-guests
```

> Warning: here we previously configure the program, 

##### Auto Configure the hugepages for qemu and libvirt-qemu

If you will have a VM with 4096 Mb of RAM (only one)so then (4096/2)+1024 where 
the 2048 will be the amount of pages for.. this number will be the amouh of huges pages 
if your hugepagez configured is 2M, this will leave almost a quarter for other vm of hugepages, 
so the need hugepages with 2M hugepagez will be 3092

If you will have a VM with 4096 Mb of RAM (only one)so then (4096/1024)+8 where 
the 8 will be the amount of pages for.. this number will be the amouh of huges pages 
if your hugepagez configured is 1G, this will leave almost a quarter for other vm of hugepages, 
so the need hugepages for 1G hugepagez will be 16

Here we will configure hugepages for default system setup, for more deep use [../documents/alpine-newbie-hugepages.md](../documents/alpine-newbie-hugepages.md)

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

sed -i 's|.*hugetlbs_mount =.*|hugetlbs_mount = "/dev/hugepages"|g' /etc/libvirt/qemu.conf

service libvirt-guests restart
```

The mode 17770 will allow to users modify only their own resources

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
