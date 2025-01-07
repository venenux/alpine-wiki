# alpine + Webmin

Webmin is a complete tool focused on a gui for lazy and stupid users, 
it helps to manage easyle user, services and inclusivelly remote machines!

The problem with webmin is that does not support officially the Alpine Linux, 
so basically the workaround is to trick the system into thinking it's Geento!

This material is copyright, check [LICENSE](#license) at the end of the document!
and you can also watch the mckaygerhard's video also at https://t.me/alpine_linux/1402

* [Install alpine linux](#install-alpine-linux)
* [1 - Environment](#1---setup-environment)
* [2 - Download and setup webmin](#2---download-and-setup-webmin)
* [3 - Configuration for modules](#3---configuration-for-modules)
    * [Problems and fails](#problems-and-fails)
* [4 - Full automated way](#4---full-automated-way)
* [How to use this guide](#how-to-use-this-guide)
* [LICENSE](#LICENSE)

This document will not explain anything; you must to obey, as must be cos just works 
and works very well, please if you dont know check [How to use this guide](#how-to-use-this-guide) 
section before starts:

## Install alpine linux

> **Warning**: if you already have alpine running just foward to [0 - Environment](#0---setup-environment) part!

Those commands are for any distro, it will create a disk and runs a virtual 
machine to install Alpine Linux 3.20 as base system OS for webmin setup.

You can use alpine 3.13, to 3.21, or edge... any alpine will work since 3.13 
to install webmin system. In this part we use 3.20 but any other version still work!

```
mkdir -p /home/general/VM/alpine320 && cd /home/general/VM/alpine320

qemu-img create -f raw computerint1alpine-vitualdisk1-file.raw 6G

wget -c -t8 --no-check-certificate http://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-extended-3.20.0-x86_64.iso

/usr/bin/qemu-system-x86_64  -m 2048 -name "computerint1alpine320" \
 -cpu host -machine q35 \
 -device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3222-:22,hostfwd=tcp::9080-:80,hostfwd=tcp::9443-:443 \
 -device virtio-keyboard -device virtio-mouse -device virtio-tablet -device virtio-vga,max_outputs=1 \
 -drive file=computerint1alpine-vitualdisk1-file.raw,format=raw \
 -cdrom alpine-extended-3.20.0-x86_64.iso -boot d
```
* When start it, will ask for root just write "root" and enter to start the command `setup-alpine`

#### the setup-alpine command procedure

* keyboard and variant, example for Latin is es and after then es-winkeys
* hostname: just hit enter, it's the name of the computer, must be only strings.
* Network: select the eth0 one that is the network cable and answer dhcp.
* Network (again): only happends if you have wifi or second card.. must ignore it
* DNS Options: It is recommended to use 8.8.8.8 and none for the domain
* Root: password for the administrative account, take care and dont forgive it
* Timezone: use UTC only for one OS, otherwise America/Panama or something similar
* Proxy Options: Use none if you are connecting directly to the Internet.
* NTP Options: Use chrony the packet already in the medium (extended).
* APK mirror: if you are over slow or no interent, type Skip or none
* User: modern alpine releases allows user creation, skip by typing no
* SSH Options: Use openssh the package that already comes in the medium (extended).
* Root allow: here you must type yes because we do not setup user yet!
* SSH key: just type here none
* Disk Options: Use sda as the entire hard drive present will be used.
* Mode: Select sys to install the system on disk.

Then reboot and if you are using a virtual machine change the line `-boot d` to ` -boot c`

## 1 - Setup environment

Do not miss or bypass any package or command, or webmin will silend fails, 
this first part will setup main and community repositories, later install 
dependencies for the installation and also core running webmin software, 
and at last will setup console font, please if you dont know check [How to use this guide](#how-to-use-this-guide) 
section, otherwise runs all of the following commands as root user:

```
cat > /etc/apk/repositories << EOF
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add openssl perl perl-net-ssleay perl-io-socket-ssl perl-io-tty \
 perl-datetime perl-datetime-timezone perl-datetime-locale attr diffutils \
 at dcron man-pages nano binutils coreutils readline shared-mime-info \
 grep gawk sed attr dialog lsof less groff procps wget curl terminus-font \
 file findutils gawk tree pciutils usbutils lshw tzdata tzdata-utils \
 zip unzip p7zip xz tar cabextract cpio binutils lha gzip lz4 \
 ethtool musl-locales musl-locales-lang  arch-install-scripts util-linux \
 docs iproute2-minimal psmisc net-tools lsof curl wget apkbuild-cpan

rc-update add consolefont boot
```

> **Warning**: execute all the commands before, if you dont know how to, check [How to use this guide](#how-to-use-this-guide)

## 2 - Download and setup webmin

Webmin does not have a way to automate the installation, but at the end 
of the document we have a full automation way.

This download, extracted the files, invoke setup and answer the questions; 
please if you dont know how check [How to use this guide](#how-to-use-this-guide) 
section, otherwise runs all of the following commands as root user:
**this part will install minimal WEBMIN, in the [4 - Full automated way](#4---full-automated-way) section 
section there are ionstructions for FULL WEBMIN install with all the need tools**, 
this section si the mosft faster, and you can watch the video https://t.me/alpine_linux/1402

```
apk add aria2

cd /tmp

aria2c https://github.com/webmin/webmin/releases/download/2.202/webmin-2.202-minimal.tar.gz

gunzip webmin-2.202-minimal.tar.gz

tar xf webmin-2.202-minimal.tar

webmin-2.202/setup.sh /usr/share/webapps/webmin
```

After runs last command, a question will raise, follow those directions:

* "Config file directory ", just hit enter for `/etc/webmin`
* "Log file directory ", just hit enter for `/var/log/webmin`
* "Full path to perl (default /usr/bin/perl):", just hit enter
* "Operating system:", write/choose `84` (Gentoo), or `102` (standart)
* "Version:", ex 5.10, the value of the comand `uname -r | cut -d. -f1,2`
* "Web server port (default 10000):", just hit enter for `10000`
* "Login name (default admin):", just hit enter for `admin`
* "Login password:", write your password, please be care for!
* "Password again:", re-write your password, please be care for!
* "Use SSL (y/n):", **WARNING**, if you use "y" proxy reverse will fails!
* "Start Webmin at boot time (y/n):", write/choose "y" and hit enter

After answered all the previous questions listed as indicated, then can 
open a browser and go to `http://<webserveripaddres>:10000` where the 
`<webserveripaddres>` is the ip address of your computer, that you can 
check it using the command `ip add | grep inet` (use the second lines)

## 4 - Configuration for modules

The installation only provides core functionality of webmin, but for full power 
or not have problems before installing the most basic modules we still need to 
install some additional packages, this avoid problems when you try to install 
extra modules of the webmin system, so please if you dont know check [How to use this guide](#how-to-use-this-guide) 
section, otherwise runs all of the following commands as root user:

```
apk add doas bash shadow shadow-uidmap musl-locales musl-locales-lang \
 e2fsprogs btrfs-progs exfat-utils f2fs-tools dosfstools xfsprogs jfsutils zfs \
 acpi patch coreutils mdadm e2fsprogs-extra attr smartmontools doas-sudo-shim \
 iproute2 netpbm poppler-utils libjpeg-turbo-utils

cat > /etc/doas.d/apkgeneral.conf << EOF
permit nopass general as root cmd apk
permit nopass general as root cmd service
EOF

useradd -m -U -c "" -s /bin/bash -G wheel,input,disk,floppy,cdrom,dialout,audio,video,lp,netdev,games,users,ping,wheel general

for u in $(ls /home); do for g in disk lp floppy audio cdrom dialout video lp netdev games users ping wheel; do addgroup $u $g; done;done

echo "general:general" | chpasswd

sed -i -r 's|#PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config

echo AllowUsers general >> /etc/ssh/sshd_config

service sshd restart
```

Now everithing will be using the "general" user, so RUNS "su -l general" 
or LOGIN WITH general USER, this user will have the power to install packages 
and also restart services as super user. Also will be from now the only user 
to get access using ssh to the machine.

#### Problems and fails

Following packages does not have any alternative in Alpine repositories

* libauthen-pam-perl : used to sync webmin users with system users, by the use of PAM.
* libdigest-sha-perl, libdigest-md5-perl : used to calcular other SHA/MD5 checksums
* libtime-piece-perl, libtime-hires-perl : uses to format and sync the time settings
* libencode-detect-perl, libsocket6-perl, lynx, qrencode : enconde and decode qr

## 4 - Full automated way

This is a ful automated install using expect tk script, please if you dont know 
how to runs the bach of the commands here check [How to use this guide](#how-to-use-this-guide) 
cos each empty line means "wait the outpot of the command just pre executed". 
This part will install the FULL WEBMIN rerlease with all need tools for future modules:

```
cat > /etc/apk/repositories << EOF
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add doas bash shadow shadow-uidmap musl-locales musl-locales-lang \
 openssl perl perl-net-ssleay perl-io-socket-ssl perl-io-tty \
 perl-datetime perl-datetime-timezone perl-datetime-locale attr diffutils \
 at dcron man-pages nano binutils coreutils readline shared-mime-info \
 grep gawk sed attr dialog lsof less groff wget curl terminus-font \
 file findutils gawk tree pciutils usbutils lshw tzdata tzdata-utils \
 zip unzip p7zip xz tar cabextract cpio binutils lha gzip lz4 \
 ethtool musl-locales musl-locales-lang  arch-install-scripts util-linux \
 docs iproute2-minimal psmisc net-tools lsof curl wget apkbuild-cpan \
 e2fsprogs btrfs-progs exfat-utils f2fs-tools dosfstools xfsprogs jfsutils zfs \
 acpi patch coreutils mdadm e2fsprogs-extra attr smartmontools doas-sudo-shim \
 iproute2 netpbm poppler-utils libjpeg-turbo-utils aria2 expect

cd /tmp && aria2c https://github.com/webmin/webmin/releases/download/2.202/webmin-2.202.tar.gz && tar xzf webmin-2.202.tar.gz

cat > /tmp/webmin.exp << EOF
#!/usr/bin/expect
set timeout -1
spawn "/tmp/webmin-2.202/setup.sh /usr/share/webapps/webmin"
expect "Config file directory " {send "\r"}
expect "Log file directory " {send "\r"}
expect "Full path to perl (default /usr/bin/perl):" {send "\r"}
expect "Operating system:" {send "87\r"}
expect "Version:" {send "3.20\r"}
expect "Web server port (default 10000):" {send "\r"}
expect "Login name (default admin):" {send "admin\r"}
expect "Login password:" {send "admin\r"}
expect "Password again:" {send "admin\r"}
expect "Use SSL (y/n):" {send  "n\r"}
expect "Start Webmin at boot time (y/n):" {send  "n\r"}
EOF

expect /tmp/webmin.exp && rm /tmp/webmin.exp

cat > /etc/doas.d/apkgeneral.conf << EOF
permit nopass general as root cmd apk
EOF
useradd -m -U -c "" -s /bin/bash -G wheel,input,disk,floppy,cdrom,dialout,audio,video,lp,netdev,games,users,ping,wheel general
echo "general:general" | chpasswd

sed -i -r 's|#PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config
echo AllowUsers general >> /etc/ssh/sshd_config
service sshd restart
```

## How to use this guide

This guide **structure all the commands in blocks, each block its separated by a line spaced**, 
so you must **type each line as is.. and hit enter**, so you noted that then you 
typed each separated clocks of commands, copy/type only blocks separated by an empty line, 
all new(next) lines are made by just enter. the terminal will detect if must execute or not.

This guide is for install process, many parts will need you understand minimal 
knowledge of linux.

This guide assumed you have a serial port allowed in the targeted computer, also 
its important you shuold understand the way of the configuration in this guide.

> **Warning**  Some Linux or/and Mac terminals have security cut/paste locks, so 
if you paste, the first line will be preceded by garbage, check always the first char of your paste.

## see also

- 🗯 IRC
  - 💬 `##alpine_telegram_english`
  - 💬 `#alpine_linux_english`
- 📱 Telegram https://t.me/alpine_linux
  - 🇬🇧 https://t.me/alpine_linux_english
  - 🇷🇺 https://t.me/alpine_linux_pycckuu (dual english russian, low activity)
  - 🇨🇴 https://t.me/alpine_linux_espanol
  - 🇧🇬 https://t.me/alpine_linux_bulgarian (dual english bulgarian, low activity)
  - 🇨🇳 https://t.me/alpine_linux_chinese (dual english chinese, low activity)
  - 📡 https://t.me/opentechnologies (open languajes but english as main)
- Matrix
  - 👥 https://matrix.to/#/#alpine-linux-english:matrix.org

# LICENSE

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [[alpine/copyright.md](../../alpine/copyright.md)](https://venenux.github.io/alpine-wiki/#/alpine/copyright)
