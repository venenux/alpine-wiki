## alpine hugepages

Setup for large pages of memory mapping for applications, 
but first please check our [../alpine/copyright.md](../alpine/copyright.md) before taking this information to copy it.

### Introduction to hugepages

When a **process uses RAM, the CPU marks it as used by that process** in fact 
when you check the memory in proc filesystem a big portion is reserved as used;
for efficiency, the CPU **allocates such RAM in chunks** (4K bytes as most common option)
and **those are named pages that are swapped to disk that are used to maps which 
resources are used and to witch process belongs**.

Operating system supports "larger-than-default pages": HugePages on Linux, SuperPages on BSD, 
more pages you have, the more time it takes to find where memory is mapped, so 
if you use hugepages in ram, will improve performance due the RAM velocity.

> **Warning**: this technique does not make sense in little amounts of RAM machines mostly less than 64Gigs

Linux supports both normal-sized memory pages and large memory pages (Huge Page). 

#### Importance of configuring hugepages

A page size that is too small introduces a larger page table entry increasing 
the lookup time and additional overhead of the TLB (Translation lookaside buffer) 
when addressing.

Excessive page size wastes memory space, causes memory fragmentation, 
and reduces memory utilization. That is the reason because large pages 
only has meaning in large amounts of memory.

#### Standar Default hugepages and page sizes

The operation system do not have a official interface or configuration for hugepages 
and there is no standard for the hugepage filesystem or sizes

> **Warning**: as not make sense in little amounts of RAM because each application will need their own amount of memory

You **should configure the amount of memory an user can lock, so an application cannot crash 
your operating system by locking all the memory**. Note that any page can be locked in RAM, 
not just huge pages. You should allow the process to lock a little bit more memory than 
just the the space for hugepages. This means that if you configure a portion of hugepages 
for one application, if you need more applications with same technique so will need more hugepages 
configured, mostly separated (You can setup more than "one amount" of hugepages).

The default size of memory pages on most processors is 4KB, and although some processors 
use 8KB, 16KB, or 64KB as the default page size, 4KB pages are still the mainstream of 
the default memory page configuration of the operating system; in addition to the normal 
memory page size, different processors also contain large pages of different sizes.

#### Checking the hardware support for hugepages and page sizes

This support is built on top of multiple page size support
that is **provided by most modern architectures, this is a very scarce resource on processor**:

| Architecture | default page size | extra page size   | observations |
| ------------ | ----------------- | ----------------- | ------------ |
| arm64        | 2M                | 4k, 1G, 64K, 512M | 64k and 512M kernel with CONFIG_ARM64_64K_PAGES=y |
| i386         | 2M                | 4k, 4M            | 4k in older cpus, 4M only for pae cpu |
| amd64        | 2M                | 1G                | 1G only in most modern or high end cpu |
| ia64         | 4K, 4M            | 8K, 64K, 256K, 1M, 16M, 256M | it depends of cpu |
| ppc64        | 4K                | 16M               |  |

**Because this is a very scarce resource on processor** On alpine this 
**depends of the kernel and the CPU combination**:

* For arm64 kernel if was build with config of a 4KB standard PAGE_SIZE supports `2Megs` and `1Gigs` 
  of HugeTLB page sizes BUT with CONFIG_ARM64_64K_PAGES=y then only `512Megs` HugeTLB (and THP) pages 
  are available. These are available at run time and must be specified at kernel boot parameters to set
  on Linux, 2MiB with ARMv7 LPAE / ARMv8 or 1MiB on ARMv7 without LPAE are the defaults.
* For x86/ia64 based depending on the processor, there are at least two different huge page sizes on 
  processors: `2Megs` and `1Gigs` and last depends of the cpu flags detection with `grep pse /proc/cpuinfo | uniq` 
  command, fi returns the `pse` flag it supports `2Megs` pages and if also returns `pdpe1gp` it 
  supports `1Gigs` pases also, they may have to be activated at boot time.

**Documentation nowadays are too focused on X86 arches, check https://gitlab.com/fusilsystem/arm_tlb_huge_pages 
for more information about ARM arches .. is a very extensive information!**

This must be in coordination with the kernel, that must also have already configured 
the "CONFIG"s hugepage related settings enabled to property set the required parameters 
at boot time, check the next section:

#### Checking if the kernel supports hugepages respect amount of RAM

First check if your operating system of Alpine kernel it supports with:

```
grep HUGETLB /boot/config-$(uname -r | cut -d'-' -f3)
```

You must check the following output settings enabled with `y` as:

* CONFIG_ARM64_64K_PAGES=y this means you can use 1Ggs of pages in ARM64
* CONFIG_CGROUP_HUGETLB=y
* CONFIG_ARCH_WANT_GENERAL_HUGETLB=y
* CONFIG_HUGETLBFS=y this means you can mount the hugetlb filesystem
* CONFIG_HUGETLB_PAGE=y
* CONFIG_ARCH_WANT_HUGETLB_PAGE_OPTIMIZE_VMEMMAP=y
* CONFIG_HUGETLB_PAGE_OPTIMIZE_VMEMMAP=y

> **Warning**: Alpine kernels were bad configured https://gitlab.alpinelinux.org/alpine/aports/-/issues/11337 until mckaygerhard reports such problem

Secondly you must check for system environment support with command:

```
grep -R "" /sys/kernel/mm/hugepages/ /proc/sys/vm/*huge*
```

You must check the following output settings enabled, and with attention 
  where `<size>` we just described before depending of architecture

* `/sys/kernel/mm/hugepages/hugepages-<size>kB/nr_hugepages_mempolicy`
* `/sys/kernel/mm/hugepages/hugepages-<size>kB/nr_hugepages`
* `/proc/sys/vm/hugetlb_shm_group`
* `/proc/sys/vm/nr_hugepages`
* `/proc/sys/vm/nr_hugepages_mempolicy`

Those are all documented in https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt

### Alpine configuration of hugepages

All of this depends on the [support offered in the processor](#checking-the-hardware-support-for-hugepages-and-page-sizes) 
and the amount of [physical memory available with the kernel OS](#checking-if-the-kernel-supports-hugepages-respect-amount-of-RAM) 
as described above.

#### Configure system hugepages with 2Megs size

This is for a machine with 64 gigs of physical RAM, using the 
default hugepage **size of 2 megs for an amount of 8 gigs of hugepages** memory 
to be used by the user general through the `kvm` group of system

* configure grub to boot kernel parameters with `default_hugepagesz` indication
* mount the hugepage filesystem depending of the KVM group usin same hugepage size parameters
* added the user `general` to the group `kvm`
* reboot cos the new parameters must be booted by the kernel (somethings in fly changes are not efficient)

> **Warning**: here the grub will use ext4 root filesystem with nvme storage, you must change the settings depending your setup

```
cat > /etc/default/grub  << EOF
GRUB_TIMEOUT=2
GRUB_DISABLE_SUBMENU=y
GRUB_DISABLE_RECOVERY=true
GRUB_CMDLINE_LINUX_DEFAULT="modules=sd-mod,usb-storage,ext4,nvme rootfstype=ext4 default_hugepagesz=2M hugepagesz=2M hugepages=4096"
EOF

mount -t hugetlbfs -o rw,pagesize=2M,mode=1770,relatime,gid=$(getent group kvm | cut -d':' -f3) hugetlbfs /dev/hugepages

adduser general kvm

reboot
```

Of course you can include the mount on the `fstab` file with `hugetlbfs /dev/hugepages hugetlbfs rw,pagesize=2048k,size=8G,mode=1770,relatime 0 0`

#### Configure system hugepages with 1Gigs size

This is for a machine with 64 gigs of physical RAM, using the 
default hugepage **size of 2 megs for an amount of 16 gigs** of hugepages memory 
to be used by the user general through the `kvm` group of system

* configure grub to boot kernel parameters with `default_hugepagesz` indication
* mount the hugepage filesystem depending of the KVM group usin same hugepage size parameters
* added the user `general` to the group `kvm`
* reboot cos the new parameters must be booted by the kernel (somethings in fly changes are not efficient)

> **Warning**: here the grub will use ext4 root filesystem with nvme storage, you must change the settings depending your setup

```
cat > /etc/default/grub  << EOF
GRUB_TIMEOUT=2
GRUB_DISABLE_SUBMENU=y
GRUB_DISABLE_RECOVERY=true
GRUB_CMDLINE_LINUX_DEFAULT="modules=sd-mod,usb-storage,ext4,nvme rootfstype=ext4 default_hugepagesz=1G hugepagesz=1G hugepages=16 "
EOF

mount -t hugetlbfs -o rw,pagesize=1024M,mode=1770,relatime,gid=$(getent group kvm | cut -d':' -f3) hugetlbfs /dev/hugepages

adduser general kvm

reboot
```

Of course you can include the mount on the `fstab` file with `hugetlbfs /dev/hugepages hugetlbfs rw,pagesize=1048576k,size=16G,mode=1770,relatime 0 0`

> Warning: For this being supported the cpu must support 1G page sizes

### Hugepage enabled applications in alpine linux

An application can allocate/use HugeTlbPage through two different means:

* **hugetlb filesystem**: mmap system call require a mounted hugetlbfs, with appropriate permissions.
* **Shared memory segment** (shmat/shmget system calls or mmap with MAP_HUGETLB), must be member of a group, configured in /proc/sys/vm/hugetlb_shm_group.

| Application | hugetlbfs | shared memory | observations |
| ----------- | --------- | ------------- | ------------ |
| QEMU        | Yes       | No            | tested the package but seems packagers dont noted such feature |
| MySQL       | No        | Yes           | canot confirm alpine packagers take around this feature |
| Java        | No        | Yes           | there is not JEE experts in java realted packagers |
| Libvirt     | Yes       | No            | is just a framework for qemu so it relies on qemu |
| Postgresql  | Yes       | Yes           | is well packaged, seems  |
| xen         | Yes       | ??            | guest does not support hugepages https://wiki.xenproject.org/wiki/Huge_Page_Support |
| xmrig       | Yes       | Yes           | software it made by default, but alpine packagers dont know about it |
| apache2     | ??        | Yes           | can cause locks, alpine developers seems does not know nothing about it at 2022 |
| docker      | X         | X             | it permits that apps inside accesss to hugepages, this is so bad and dangerous |

For java best support of shmat/shmget use the packages at https://t.me/alpine_linux/1248, 
and seems you must lauch with `-XX:+UseLargePages`. The maximum size of the Java 
heap (-Xmx) should fit in your reserved Huge pages (note that if you already have 
an application using hugepages, you must check free pages); same for `ulimit -l` 
and/or memlock in `/etc/security/limits.conf`.

For QEMU best setup use the guide here: [../tutorials/alpine-howto-qemu-emulation.md](../tutorials/alpine-howto-qemu-emulation.md)

For xmrig you should be root and no other application must use the mounted or 
enabled hugepages, in other case you should be part of a group that can access 
the hugepages, unfortunatelly this is not so clear and alpine documentation is 
so vage around, so best is to use as root and not have another application acceding hugepages!

For apache2 if you run it with other applications that already uses hugepages, 
will cause locks so check if you have already an application using hugepages, 
you must check free pages); same for `ulimit -l` and/or memlock in `/etc/security/limits.conf`.

The docker documentation lists capabilities without `CAP_`. However, both seem to work fine 
(it does check whether the cap name is valid, inserting a deliberate typo gives an error). 
The documentation also says this capability is not granted by default.

### About copyright material

**CC BY-NC-SA**: If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

Please check our [../alpine/copyright.md](../alpine/copyright.md).

