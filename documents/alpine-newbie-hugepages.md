## alpine hugepages

Short answer:

When a **process uses RAM, the CPU marks it as used by that process** in fact 
when you check the memory in proc filesystem a big portion is reserved as used;
for efficiency, the CPU **allocates such RAM in chunks-4K bytes** as most common option
and **those are named pages that are swapped to disk that are used to maps which 
resources are used and to wich process belongs**.

Operating system supports "larger-than-default pages": HugePages on Linux, SuperPages on BSD, 
more pages you have, the more time it takes to find where memory is mapped, so 
if you use hugepages in ram, will improve performance due the RAM velocity.

> **Warning**: this technique does not make sense in little amounts of RAM machines mostly less than 96Gigs

#### Standar Default hugepages

The operation system do not have a oficial interface or configuration for hugepages 
and there is no standar for the hugepage filesystem or sizes

You **should configure the amount of memory a user can lock, so an application cannot crash 
your operating system by locking all the memory**. Note that any page can be locked in RAM, 
not just huge pages. You should allow the process to lock a little bit more memory than 
just the the space for hugepages.

#### Cheking the hardware support for hugepages

This support is built on top of multiple page size support
that is **provided by most modern architectures, this is a very scarce resource on processor**:

* x86 architecture (i386/amd64) support 4K and 2M page sizes and 1G (amd64) for modern ones
* ia64 architecture supports multiple page sizes 4K, 8K, 64K, 256K, 1M, 4M, 16M, 256M 
* ppc64 supports 4K and 16M.

**Because this is a very scarce resource on processor** On alpine this 
**depends of the kernel and the CPU combination**:

* For arm64 kernel if was build with config of a 4KB standard PAGE_SIZE supports `2Megs` and `1Gigs` 
  of HugeTLB page sizes BUT with CONFIG_ARM64_64K_PAGES=y then only `512Megs` HugeTLB (and THP) pages 
  are available. These are available at run time and must be specified at kernel boot parameters to set.
* For x86 based depending on the processor, there are at least two different huge page sizes on 
  procesors: `2Megs` and `1Gigs` and last depends of the cpu flags detection with `grep pse /proc/cpuinfo | uniq` 
  command, fi returns the `pse` flag it supports `2Megs` pages and if also returns `pdpe1gp` it 
  supports `1Gigs` pases also, they may have to be activated at boot time.
* Kernel must also have already configure the "CONFIG"s hugepage realted settings 
  enabled to property set the required parameters at boot time, check the 
  section [Cheking if the kernel supports hugepages](#cheking-if-the-kernel-supports-hugepages)

#### Cheking if the kernel supports hugepages

First check if your operating system of Alpine kernel it supports with:

```
grep HUGETLB /boot/config-$(uname -r | cut -d'-' -f3)
```

You must check the following output settings enabled with `y` as:

* CONFIG_ARM64_64K_PAGES=y read about arm64 setting in previous sections
* CONFIG_CGROUP_HUGETLB=y
* CONFIG_ARCH_WANT_GENERAL_HUGETLB=y
* CONFIG_HUGETLBFS=y
* CONFIG_HUGETLB_PAGE=y
* CONFIG_ARCH_WANT_HUGETLB_PAGE_OPTIMIZE_VMEMMAP=y
* CONFIG_HUGETLB_PAGE_OPTIMIZE_VMEMMAP=y

> **Warning**: Alpine kernels were bad configured https://gitlab.alpinelinux.org/alpine/aports/-/issues/11337 until mckaygerhard reports such problem

Secondly you must check for system environment support with command:

```
grep -R "" /sys/kernel/mm/hugepages/ /proc/sys/vm/*huge*
```

You must check the following output settings enabled as:

* `/sys/kernel/mm/hugepages/hugepages-<size>kB/nr_hugepages_mempolicy`  where `<size>` can be `1048576` or `2048`
* `/sys/kernel/mm/hugepages/hugepages-<size>kB/nr_hugepages`  where `<size>` can be `1048576` or `2048`
* `/proc/sys/vm/hugetlb_shm_group`
* `/proc/sys/vm/nr_hugepages`
* `/proc/sys/vm/nr_hugepages_mempolicy`


#### Configure system hugepages with 2Megs size

This is for a machine with 64 gigs of phisical RAM, using the 
default hugepage **size of 2 megs for an amount of 16 gigs of hugepages** memory 
to be used by the user general throught the `kvm` group of system

* configure grub to boot kernel parameters with `default_hugepagesz` indication
* mount the hugepage filesystem depending of the KVM group usin same hugepage size parameters
* added the user `general` to the group `kvm`
* reboot cos the new parameters must be booted by the kernel (somethings in fly changes are not efficient)

```
cat > /etc/default/grub  << EOF
GRUB_TIMEOUT=2
GRUB_DISABLE_SUBMENU=y
GRUB_DISABLE_RECOVERY=true
GRUB_CMDLINE_LINUX_DEFAULT="modules=sd-mod,usb-storage,ext4,nvme rootfstype=ext4 default_hugepagesz=2M hugepagesz=2M hugepages=8192"
EOF

mount -t hugetlbfs -o rw,pagesize=2M,mode=1770,relatime,gid=$(getent group kvm | cut -d':' -f3) none /dev/hugepages

adduser general kvm

reboot
```

Of course you can include the mount on the `fstab` file.

#### Configure system hugepages with 1Gigs size

This is for a machine with 64 gigs of phisical RAM, using the 
default hugepage **size of 2 megs for an amount of 16 gigs** of hugepages memory 
to be used by the user general throught the `kvm` group of system

* configure grub to boot kernel parameters with `default_hugepagesz` indication
* mount the hugepage filesystem depending of the KVM group usin same hugepage size parameters
* added the user `general` to the group `kvm`
* reboot cos the new parameters must be booted by the kernel (somethings in fly changes are not efficient)

```
cat > /etc/default/grub  << EOF
GRUB_TIMEOUT=2
GRUB_DISABLE_SUBMENU=y
GRUB_DISABLE_RECOVERY=true
GRUB_CMDLINE_LINUX_DEFAULT="modules=sd-mod,usb-storage,ext4,nvme rootfstype=ext4 default_hugepagesz=1G hugepagesz=1G hugepages=16 "
EOF

mount -t hugetlbfs -o rw,pagesize=1024M,mode=1770,relatime,gid=$(getent group kvm | cut -d':' -f3) none /dev/hugepages

adduser general kvm

reboot
```

Of course you can include the mount on the `fstab` file.

> Warning: For this being supported the cpu must support 1G page sizes

### About copyright material

**CC BY-NC-SA**: If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

Please check our [../alpine/copyright.md](../alpine/copyright.md).

