# Alpine XFCE4 desktop setup: FF version
===========================================================

This was using real machine Pentium Dual Core E5500.
For more extended info check [../../newbie/alpine-newbie-xfce-desktop.md](../../newbie/alpine-newbie-xfce-desktop.md)

### hardware used

| item             | minimal feature   | Extra recommendations              |
| ---------------- | ----------------- | ---------------------------------- |
| RAM MB           | 1Gb DDR1          | 6Gb DDR3, web browsers consumes so much |
| CPU              | intel Dual Core   | Not necesary                       |
| RAM CPU          | 2Mb (L2) 4kb/L1   |  |
| GPU              | intel G41         | Radeon X1200 For web browsers and modern apps will be need |
| RAM GPU          | 256Mb             | 1Gb For web browsers and modern apps will be need |
| Storage          | 120Gb HDD WD      | 256Gb SSD are mandatory for speed |
| ARCH             | 32bits (i386/arm6)| 64bits (i386) mandatory for most modern apps unfortunatelly |
| Audio            | AC 97             | HD audio and HDMI audio are a mess |

### usernames

| item      | name                | password |
| --------- | ------------------- | -------- |
| remote    | daru                | daru     |
| admin     | root                | toor     |
| user      | general             | general  |


* [Preparation](#preparation-xfce4-aline)
* [Instalation](#instalation)
    * [setup OS configuration](#setup-os-configuration)
    * [setup system users](#setup-system-users)
    * [setup hardware support](#setup-hardware-support)
    * [setup audio and video](#setup-audio-and-video)
* [Instalacion XFCE4 Alpine](#instalacion-xfce4-apine)
* [Desktop multimedia and media devices](#desktop-multimedia-and-media-devices)
* [Development](#development)
* [How to use this guide](#how-to-use-this-guide)
* [Licensing clarifications](#licensing-clarifications)
* [See also](#see-also)

## preparation Xfce4 Alpine

Alpine must be already installed, check [../../newbie/alpine-newbie-install.md](../../newbie/alpine-newbie-install.md),** 

**If you dont have direct network connection, please use our direct VenenuX Alpine ISOS**
[CURRENT LINK https://t.me/alpine_linux/762, but ask in telegram alpine network for newer one or other arches](https://t.me/s/alpine_linux/762)

#### setup OS configuration

For more extended info check [../../newbie/alpine-newbie-xfce-desktop.md](../../newbie/alpine-newbie-xfce-desktop.md#setup-os-configuration)

```
sed -i -r 's|#PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config

rc-service sshd restart;rc-update add sshd default

hostname venenux-desktop
echo 'hostname="venenux-desktop"' > /etc/conf.d/hostname 
echo "venenux-desktop" > /etc/hostname

cat > /etc/hosts << EOF
127.0.0.1 venenux-desktop localhost.localdomain localhost
::1 localhost localhost.localdomain
EOF

cat > /etc/network/interfaces << EOF
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

iface eth0 inet6 dhcp
    pre-up echo 0 > /proc/sys/net/ipv6/conf/eth0/accept_ra
EOF

rc-service networking restart;rc-update add networking boot

cat > /root/.cshrc << EOF
unsetenv DISPLAY || true
HISTCONTROL=ignoreboth
EOF

cp /root/.cshrc  /root/.bashrc  /root/.profile

 echo "root:toor" | chpasswd

apk add tcsh

add-shell '/bin/csh'

adduser -D -g "" -u 998 -h /opt/daru -s /bin/csh daru

 echo "daru:daru" | chpasswd

rm -f /opt/daru/*
mkdir /opt/daru
cat > /opt/daru/.cshrc << EOF
unsetenv DISPLAY
set autologout = 6
set prompt = "$ "
set history = 0
set ignoreeof
EOF
cp /opt/daru/.cshrc /opt/daru/.bashrc
chown -R daru:daru /opt/daru

cat > /etc/apk/repositories << EOF
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add man-db man-pages nano binutils coreutils readline \
 sed attr dialog lsof less groff wget curl terminus-font \
 zip p7zip xz tar cabextract cpio binutils lha acpi musl-locales musl-locales-lang

export PAGER=less

```

#### setup system users

```
apk add shadow shadow-uidmap doas musl-locales musl-locales-lang

cat > /tmp/tmpcs.tmp << EOF
set history = 10000
set prompt = "$ "
EOF

mkdir /etc/skel
cat /tmp/tmpcs.tmp > /etc/skel/.cshrc
cat /tmp/tmpbs.tmp > /etc/skel/.bashrc

cat > /etc/skel/.Xresources << EOF
Xft.antialias: 0
Xft.rgba:      rgb
Xft.autohint:  0
Xft.hinting:   1
Xft.hintstyle: hintslight
EOF

cat > /etc/default/useradd << EOF
# useradd defaults file
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
EOF

cat > /etc/login.defs << EOF
USERGROUPS_ENAB yes
SYSLOG_SU_ENAB		yes
SYSLOG_SG_ENAB		yes
SULOG_FILE	/var/log/sulog
SU_NAME		su
EOF

useradd -m -U -c "" -G wheel,input,disk,floppy,cdrom,dialout,audio,video,lp,netdev,games,users,ping general

for u in $(ls /home); do for g in disk lp floppy audio cdrom dialout video lp netdev games users ping; do addgroup $u $g; done;done
```

**WARNING** your user name must be `general`, you can put a "human name" as you wish, later.

For more details check  [../../newbie/alpine-newbie-xfce-desktop.md](../../newbie/alpine-newbie-xfce-desktop.md#setup-system-users)

#### setup hardware support

```
apk add acpi eudev eudev-doc eudev-rule-generator eudev-openrc linux-firmware \
 fuse fuse-exfat-utils fuse-exfat avfs pcre2 cpufreqd bluez bluez-openrc \
 wpa_supplicant dhcpcd chrony macchanger wireless-tools iputils \
 networkmanager networkmanager-lang networkmanager-openvpn networkmanager-openvpn-lang

rc-update add udev
rc-update add acpid
rc-update add cpufreqd
rc-update add fuse
rc-update add bluetooth
rc-update add chrony
rc-update add wpa_supplicant
rc-update add networkmanager

rc-service networking restart

rc-service wpa_supplicant restart

rc-service bluetooth restart

rc-service udev restart 

rc-service fuse restart

rc-service cpufreqd restart

```

For more details check  [../../newbie/alpine-newbie-xfce-desktop.md](../../newbie/alpine-newbie-xfce-desktop.md#setup-software-graphical-fonts-and-languajes)

#### setup audio and video

```
setup-xorg-base xinit mesa-dri-gallium xf86-video-dummy xf86-video-modesetting xf86-video-vesa

apk add libxinerama xrandr kbd setxkbmap bluez bluez-openrc \
 dbus dbus-x11 elogind elogind-openrc lightdm lightdm-lang lightdm-gtk-greeter \
 polkit polkit-openrc polkit-elogind udisks2 udisks2-lang \
 gvfs gvfs-fuse gvfs-archive gvfs-dav gvfs-nfs gvfs-lang \
 networkmanager-elogind networkmanager-elogind-lang networkmanager-elogind-openrc

dbus-uuidgen > /var/lib/dbus/machine-id

rc-update add dbus
rc-update add elogind
rc-update add polkit
rc-update add lightdm

apk add font-noto-all ttf-dejavu ttf-linux-libertine ttf-liberation \
 font-bitstream-type1 font-bitstream-100dpi font-bitstream-75dpi \
 font-adobe-utopia-type1 font-adobe-utopia-75dpi font-adobe-utopia-100dpi \
 font-isas-misc

apk add alsa-utils alsa-plugins alsa-tools alsaconf \
 pipewire pipewire-pulse pipewire-alsa pipewire-spa-bluez

cat > /etc/security/limits.d/audio-limits.conf << EOF
@audio - memlock 256
@audio - nice -11
@audio - rtprio 88
EOF

rc-service dbus restart

rc-service elogind restart

rc-service polkit restart

rc-service lightdm restart

```

For more details check  [../../newbie/alpine-newbie-xfce-desktop.md](../../newbie/alpine-newbie-xfce-desktop.md#setup-software-graphical-fonts-and-languajes)

## instalacion Xfce4 Alpine

Since Alpine 3.13 the XFCE4 desktop its GTK3 for 32bit devices its better to use alpine 3.10 
or 3.12 that uses GTK2 for almost all the programs.

```
apk add gtk-update-icon-cache hicolor-icon-theme paper-gtk-theme adwaita-icon-theme

apk add xfce4 xfce4-session xfce4-panel xfce4-terminal xarchiver mousepad \
 xfwm4-themes xfce-polkit xfce4-skel xfce4-power-manager xfce4-settings \
 xfce4-clipman-plugin xfce4-xkb-plugin xfce4-screensaver xfce4-screenshooter xfce4-taskmanager \
 xfce4-panel-lang xfce4-clipman-plugin-lang xfce4-xkb-plugin-lang xfce4-screenshooter-lang \
 xfce4-taskmanager-lang xfce4-battery-plugin-lang xfce4-power-manager-lang xfce4-settings-lang \
 gvfs gvfs-fuse gvfs-archive gvfs-afp gvfs-afp gvfs-afc gvfs-cdda gvfs-gphoto2 gvfs-mtp \
 network-manager-applet network-manager-applet-lang vte3 \
 libreoffice libreoffice-gnome evince evince-lang evince-doc

rc-service networking restart

rc-service wpa_supplicant restart

rc-service networkmanager restart

rc-service lightdm restart

```

## Desktop multimedia and media devices

```
apk add gst-plugins-base gst-plugins-bad gst-plugins-ugly gst-plugins-good gst-plugins-good-gtk \
 libcanberra-gtk2 libcanberra-gtk3 libcanberra-gstreamer wxgtk-media wxgtk3-media wxgtk-lang \
 mediainfo ffmpeg ffmpeg-doc ffmpeg-libs lame lame-doc rtkit rtkit-doc \
 mpv mpv-doc deadbeef deadbeef-lang libxinerama xrandr 

for u in $(ls /home); do for g in plugdev audio cdrom dialout video netdev; do addgroup $u $g; done;done

cat > /etc/network/interfaces << EOF
auto lo
iface lo inet loopback
EOF

```

## development

```
apk add pkgconf make cmake gcc gcc-gdc gcc-go g++ gcc-objc gcc-doc \
 patch patch-doc patchutils patchutils-doc diffutils diffutils-doc \
 git git-cvs git-svn github-cli git-diff-highlight git-doc \
 subversion subversion-doc mercurial mercurial-doc \
 geany geany-plugins-lang geany-plugins-addons geany-plugins-geanyextrasel \
 geany-plugins-overview geany-plugins-geanyvc geany-plugins-treebrowser \
 geany-plugins-tableconvert geany-plugins-spellcheck geany-plugins-shiftcolumn \
 geany-plugins-utils geany-lang \
 terminator terminator-lang tmux screen meld meld-lang
```

## How to use this guide

Just install alpine, and try to login as root using that command:

1. at the Alpine installation: `sed -i 's|.*PermitRootLogin.*|PermitRootLogin yes|g' /etc/ssh/sshd_config;service sshd restart`
2. at the other OS just connect: `ssh -l root <ip>` change "`<ip>`" with the address of your device.
3. copy each separated by empty line, block of command, copy only blocks separate by empty line
4. and paste each separated by empty line block in the remnote (ssh), do not paste all the blocks at same time!

**CAUTION** Some Linux or/and Mac terminals have security cut/paste locks, so 
if you paste, the first line will be preceded by garbage, check always the first char of your paste.

**WARNING** after finish, rerun: `sed -i -r 's|.*PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config`
and restart ssh `service sshd restart` becouse security implications.

## Licensing clarifications

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

## See also

* [README.md](README.md)
* [alpine-newbie-install.md](alpine-newbie-install.md)
