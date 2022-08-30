# Alpine XFCE4 quick FF
===========================================================

Real machine Pentium Dual Core E5500

### hardware used

| item             | minimal feature | we need more?                      |
| ---------------- | --------------- | ---------------------------------- |
| CPU              | intel Dual Core | Not necesary                       |
| GPU              | intel G41       | Radeon X1200 For web browsers and modern apps will be need |
| RAM CPU          | 2Mb (L2) 4kb/L1 |  |
| RAM GPU          | 256Mb           | 1Gb For web browsers and modern apps will be need |
| Sotrage          | 500Gb HDD WD    | 256Gb SSD are mandatory for speed |
| ARCH             | 32bits (i386)   | 64bits (i386) mandatory for most modern apps unfortunatelly |
| Audio            | AC 97           | HD audio and HDMI audio are a mess |

### services

| item      | port     | software | expuesto-ip                   | objetivo      |
| --------- | -------- | -------- | ----------------------------- | ------------- |
| vnc       | 15000    | vnc      | **SI**, to use remote desktop | unfortunatelly trhere's no anydesk for alpine working |
| ssh       | 19226    | openssh  | **NO**, only to be used once  | for admin management only |

### usernames

| item      | name                | password |
| --------- | ------------------- | -------- |
| remote    | daru                | daru     |
| admin     | root                | toor     |
| user      | general             | general  |


# XFCE4 over alpine linux
================================================

* [preparation](#preparation-xfce4-aline)
    * [instalation](#instalation)
    * [setup OS configuration](#setup-os-configuration)
    * [configuration programs and repositories](#configuration-programs-and-repositories)
    * [setup system users](#setup-system-users)
    * [setup hardware media support and xorg](#setup-hardware-media-support-and-xorg)
* [instalation xorg](#instalacion-xorg-apine)
* [instalacion xfce](#instalacion-xfce4-apine)
* [Configuracion](#configuracion-xfce4-alpine)

## preparation Xfce4 Alpine

#### booting alpine

1. **download the iso image**: any alpine iso from 3.9 to 3.16 are valid, for olders 3.10 are best:
https://dl-4.alpinelinux.org/alpine/v3.10/releases/x86_64/alpine-extended-3.10.5-x86_64.iso 
2. **burn iso to usb or DVD/CD disk** to load into the machine to boot or virtual machine, 
you must boot the CD/DVD or USB from BIOS/UEFI or from the virtual machine
2. **at boot the alpine will ask `login` just type `root`**, 
this will permit to run commands to install the operatinig system

#### instalation

```
export BOOT_SIZE=500

export SWAP_SIZE=8182

export BOOTLOADER=grub

setup-alpine
```

* teclado y variante, ejemplo para latino es `es` y depues `es-winkeys`
* hostname: escribir `venenux-desktop`, es el nombre de la computadora.
* Opciones de red: seleccione `eth0` porque asumimos una sola interfaz
* Opciones de red (ip): contestar `none` despues contestar `no` a manual
* Opciones de DNS: (dominio) escriba `fusilsystem.com` y enter
* Opciones de DNS: (nameserver) se recomienda usar `8.8.8.8`
* Opcion de clave root, escribir "root" las dos veces, despues se mejorara!
* Opciones de zona horaria: solo use UTC
* Opciones de proxy: use `none` y si uso dhcp en red ya tendra internet.
* Opciones de repo mirror: cuando pregunte escriba `done`
* Opciones de SSH: use `openssh` el paquete que ya viene en el medio.
* Opciones de NTP: use `chrony` el paquete que ya viene en el medio.
* Opciones de disco: use "sda" ya que se usara todo el disco duro presente.
* Modo: seleccione "sys" para instalar el sistema en el disco.
* Confirmacion de borrado: pedira confirme borrar el disco conteste `y`
* Confirmacion de particiones: solo sale si tiene previas, conteste `y`

#### setup OS configuration

```
sed -i -r 's|#PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config

service sshd restart

rc-update add sshd default

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

rc-service networking restart

rc-update add networking boot

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

```

#### configuration programs and repositories


``` bash
cat > /etc/apk/repositories << EOF
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add sed sed-doc attr attr-doc dialog dialog-doc lsof less less-doc groff groff-doc

apk add man-pages nano nano-doc binutils binutils-doc coreutils coreutils-doc readline readline-doc

apk add wget wget-doc curl curl-doc bash bash-doc bash-completion terminus-font

apk add zip p7zip xz tar cabextract cpio binutils lha acpi 

export PAGER=less

apk add musl-locales musl-locales-lang man-db

```

#### setup system users


```
apk add shadow shadow-doc shadow-uidmap bash bash-doc bash-completion bash-dev doas doas-doc

cat > /tmp/tmp.tmp << EOF
set history = 10000
if (\$?prompt) then
    set prompt = "$ "
    set history = 10000
endif
EOF

for i in $(ls /home);do cat /tmp/tmp.tmp > /home/$i/.cshrc;done
for i in $(ls /home);do cat /tmp/tmp.tmp > /home/$i/.bashrc;done

mkdir /etc/skel
cat /tmp/tmp.tmp > /etc/skel/.cshrc
cat /tmp/tmp.tmp > /etc/skel/.bashrc

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
#MAIL_DIR        /var/mail
#MAIL_FILE      .mail
#FAILLOG_ENAB		yes
LOG_OK_LOGINS		no
SYSLOG_SU_ENAB		yes
SYSLOG_SG_ENAB		yes
SULOG_FILE	/var/log/sulog
SU_NAME		su
ENV_SUPATH	PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV_PATH	PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
UMASK		022
UID_MIN			 1000
UID_MAX			60000
SYS_UID_MIN		  100
SYS_UID_MAX		  999
GID_MIN			 1000
GID_MAX			60000
SYS_GID_MIN		  100
SYS_GID_MAX		  999
LOGIN_RETRIES		3
LOGIN_TIMEOUT		60
CONSOLE_GROUPS		floppy:audio:cdrom:users
EOF


useradd -m -U -c "" -G wheel,input,disk,floppy,cdrom,dialout,netdev,audio,video,lp,usb,users,ping general

for u in $(ls /home); do for g in disk lp floppy audio cdrom dialout video netdev games users; do addgroup $u $g; done;done
```

#### setup hardware media support and xorg

```
apk add eudev eudev-doc eudev-rule-generator eudev-openrc

rc-update add udev

rc-update add acpid

setup-xorg-base xinit mesa-dri-gallium linux-firmware kbd xf86-input-evdev xf86-input-synaptics setxkbmap

apk add  libxinerama xrandr

apk add acpi dbus dbus-x11 elogind elogind-openrc elogind-lang polkit polkit-openrc polkit-elogind lightdm lightdm-lang lightdm-gtk-greeter 

dbus-uuidgen > /var/lib/dbus/machine-id

rc-update add dbus

rc-update add elogind

rc-update add polkit

apk add ttf-dejavu font-bitstream-type1 font-bitstream-100dpi font-bitstream-75dpi

apk add terminus-font font-noto font-noto-extra font-arabic-misc ttf-liberation ttf-linux-libertine 

apk add font-misc-cyrillic font-mutt-misc font-screen-cyrillic font-winitzki-cyrillic font-cronyx-cyrillic

apk add font-noto-arabic font-noto-armenian font-noto-cherokee font-noto-devanagari font-noto-ethiopic font-noto-georgian

apk add font-noto-hebrew font-noto-lao font-noto-malayalam font-noto-tamil font-noto-thaana font-noto-thai

setfont /usr/share/consolefonts/ter-132n.psf.gz

sed -i "s#.*consolefont.*=.*#consolefont="ter-132n.psf.gz"#g" /etc/conf.d/consolefont

rc-update add consolefont boot

apk add alsa-utils alsa-utils-doc alsa-plugins alsa-plugins-doc alsa-tools alsa-tools-doc alsaconf pipewire pipewire-doc pipewire-pulse pipewire-alsa sndio sndio-doc

rc-service dbus start

cat > /etc/security/limits.d/audio-limits.conf << EOF
@audio - memlock 256
@audio - nice -11
@audio - rtprio 88
EOF

apk add bluez bluez-openrc pipewire-spa-bluez

rc-update add bluetooth

rc-service bluetooth start

apk add cpufreqd

rc-update add cpufreqd

apk add gtk-update-icon-cache vte3 pcre2 udisks2 udisks2-lang udisks2-doc

apk add fuse fuse-exfat-utils archivemount fuse-exfat avfs

rc-service fuse start

rc-update add fuse

apk add gvfs gvfs-fuse gvfs-archive gvfs-dav gvfs-nfs gvfs-lang
```

## instalacion Xfce4 Alpine


```
apk add xfwm4-themes hicolor-icon-theme paper-gtk-theme network-manager-applet adwaita-icon-theme mate-themes

apk add xfce4 xfce4-terminal xfce4-screensaver xfce4-session xfce4-session-doc xarchiver mousepad

apk add xfce-polkit xfce4-skel xfce4-power-manager xfce4-power-manager-lang xfce4-settings xfce4-settings-lang

apk add xfce4-panel xfce4-panel-doc xfce4-panel-lang xfce4-clipman-plugin xfce4-clipman-plugin-lang xfce4-xkb-plugin xfce4-xkb-plugin-lang xfce4-xkb-plugin-doc

apk add xfce4-screenshooter xfce4-screenshooter-doc xfce4-screenshooter-lang xfce4-taskmanager xfce4-taskmanager-lang

apk add xfce4-whiskermenu-plugin xfce4-whiskermenu-plugin-lang xfce4-whiskermenu-plugin-doc xfce4-battery-plugin xfce4-battery-plugin-lang

rc-update add lightdm

rc-service lightdm restart
```

## multimedia


The media in linux its per se reduced, and in alpine so then more limited, 
with this lines you will have all the need suported, for converting and playing, 
for editing 


```
apk add gst-plugins-base gst-plugins-bad gst-plugins-bad-lang gst-plugins-ugly gst-plugins-ugly-lang gst-plugins-good gst-plugins-good-gtk

apk add libcanberra-gtk2 libcanberra-gtk3 libcanberra-gstreamer wxgtk-media wxgtk3-media wxgtk-lang

apk add mediainfo ffmpeg ffmpeg-doc ffmpeg-libs lame lame-doc rtkit rtkit-doc 

apk add mpv mpv-doc deadbeef deadbeef-lang deadbeef-doc

apk add gvfs gvfs-fuse gvfs-archive gvfs-afp gvfs-afp gvfs-afc gvfs-cdda gvfs-gphoto2 gvfs-mtp

apk add libxinerama xrandr wpa_supplicant dhcpcd chrony macchanger wireless-tools iputils

apk add network-manager-applet network-manager-applet-lang networkmanager networkmanager-lang networkmanager-elogind networkmanager-elogind-lang networkmanager-elogind-openrc networkmanager-openvpn networkmanager-openvpn-lang

rc-update add chrony

rc-update add wpa_supplicant

rc-update add networkmanager

for u in $(ls /home); do for g in plugdev; do addgroup $u $g; done;done

cat > /etc/network/interfaces << EOF
auto lo
iface lo inet loopback
EOF

service networking restart

service wpa_supplicant restart

service networkmanager restart

```

## office suite

In linux world there's no mayor suite or programs in such topic, 
just we need a reader (pdf, ebooks, cbr, zbr, etc) and office 
suite for word/calc processing (doc, xls, odt, ods, etc).


```
apk add libreoffice libreoffice-gnome evince evince-lang evince-doc
```

## development

This is only for those that dont want to download bunch of thing 
when install some programs from sources. In any cae, modding and plugin hacks 
will need this for minecraft or minetest hard hacker players.

#### base console only devel


```
apk add make cmake cmake-bash-completion gcc gcc-gdc gcc-go g++ gcc-objc gcc-doc

apk add patch patch-doc patchutils patchutils-doc diffutils diffutils-doc

apk add git git-bash-completion git-zsh-completion git-cvs git-svn github-cli git-diff-highlight git-doc

apk add subversion subversion-bash-completion subversion-zsh-completion subversion-yash-completion subversion-doc

apk add mercurial mercurial-bash-completion mercurial-zsh-completion mercurial-doc
```

#### base gui devel


```
apk add geany geany-plugins-lang geany-plugins-addons geany-plugins-geanyextrasel geany-plugins-overview geany-plugins-geanyvc geany-plugins-treebrowser geany-plugins-tableconvert geany-plugins-spellcheck geany-plugins-shiftcolumn geany-plugins-utils geany-lang meld meld-lang

apk add terminator terminator-lang
```
