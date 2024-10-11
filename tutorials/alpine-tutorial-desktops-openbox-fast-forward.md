# Alpine OPENBOX desktop setup: FF version
===========================================================

Alpine must be previously installed. This will install a light simplistic desktop, 
for more traditional check [../../newbie/alpine-newbie-xfce-desktop.md](../../newbie/alpine-newbie-xfce-desktop.md)

* [How to use this guide](#how-to-use-this-guide)
* [Preparation](#preparation-alpine)
    * [setup OS configuration](#setup-os-configuration)
    * [setup system users](#setup-system-users)
    * [setup hardware support](#setup-hardware-support)
    * [setup audio and video](#setup-audio-and-video)
* [Licensing clarifications](#licensing-clarifications)
* [See also](#see-also)

## preparation Alpine

> **Warning** **YOU MUST HAVE DIRECT WIRED INTERNET, if not** configure 
a wireless network connection as [alpine-tutorial-wifi-routering.md](alpine-tutorial-wifi-routering.md)

Feels lost here? check [How to use this guide](#how-to-use-this-guide) section of this document

#### setup OS configuration

```
sed -i -r 's|#PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config
rc-service sshd restart;rc-update add sshd default

cat > /root/.cshrc << EOF
unsetenv DISPLAY || true
HISTCONTROL=ignoreboth
EOF

cat > /etc/apk/repositories << EOF
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add tcsh && add-shell '/bin/csh'

adduser -D -g "" -u 998 -h /opt/daru -s /bin/csh daru
 echo "daru:daru" | chpasswd

rm -f /opt/daru/*
mkdir /opt/daru
cat > /opt/daru/.cshrc << EOF
unsetenv DISPLAY
export PAGER=less
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

apk add mandoc man-pages nano binutils coreutils readline \
 sed attr dialog lsof less groff wget curl terminus-font \
 file lz4 gawk tree pciutils usbutils lshw tzdata tzdata-utils \
 zip p7zip xz tar cabextract cpio binutils lha acpi musl-locales musl-locales-lang \
 e2fsprogs btrfs-progs exfat-utils f2fs-tools dosfstools xfsprogs jfsutils \
 arch-install-scripts util-linux docs

apk add font-terminus

setfont /usr/share/consolefonts/ter-120n.psf.gz

sed -i "s#.*consolefont.*=.*#consolefont="ter-120n.psf.gz"#g" /etc/conf.d/consolefont

rc-update add consolefont boot
```

#### setup system users

```
apk add bash shadow shadow-uidmap shadow-login doas lang musl-locales

cat > /etc/doas.d/apkgeneral.conf << EOF
permit nopass general as root cmd apk
permit keepenv daru as root
EOF

cat > /tmp/tmp.tmp << EOF
set history = 10000
set prompt = "$ "
EOF

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
SYSLOG_SU_ENAB		yes
SYSLOG_SG_ENAB		yes
SULOG_FILE	/var/log/sulog
SU_NAME		su
EOF

useradd -m -U -c "" -G wheel,input,disk,floppy,cdrom,dialout,audio,video,lp,netdev,games,users,ping general

for u in $(ls /home); do for g in disk lp floppy audio cdrom dialout video lp netdev games users ping; do addgroup $u $g; done;done

for u in $(ls /home); do chown -R $u:$u /home/$u; done

echo "general:general" | chpasswd
```

> **Warning** your user name must be `general` here password is `general`, you can put a "human name" as you wish, later and change password later.

For more details check  [../../documents/alpine-newbie-xfce-desktop.md](../../documents/alpine-newbie-xfce-desktop.md#setup-system-users)

#### setup hardware support

```
apk add acpi acpid acpid-openrc alpine-conf \
 eudev eudev-doc eudev-rule-generator eudev-openrc \
 pciutils util-linux arch-install-scripts zram-init acpi-utils rsyslog \
 fuse fuse-exfat-utils fuse-exfat avfs pcre2 cpufreqd bluez bluez-deprecated bluez-openrc \
 wpa_supplicant dhcpcd chrony macchanger wireless-tools iputils linux-firmware \
 networkmanager networkmanager-lang networkmanager-openvpn networkmanager-openvpn-lang

modprobe btusb && echo "btusb" >> /etc/modprobe
setup-devd udev

rc-update add rsyslog
rc-update add udev
rc-update add acpid
rc-update add cpufreqd
rc-update add fuse
rc-update add bluetooth
rc-update add chronyd
rc-update add wpa_supplicant
rc-update add networkmanager

rc-service networking restart

rc-service wpa_supplicant restart

rc-service bluetooth restart

rc-service udev restart 

rc-service fuse restart

rc-service cpufreqd restart

rc-service rsyslog restart
```

For more details check  [../../documents/alpine-newbie-xfce-desktop.md](../../documents/alpine-newbie-xfce-desktop.md)

#### setup audio and video

> **Note** on olders alpine gtk3 will force xorg dependencies.. for 3.16+ will use gtk4 and SDL2

```
apk add mesa mesa-gl mesa-utils mesa-osmesa mesa-egl mesa-gles \
 mesa-dri-gallium mesa-va-gallium libva-intel-driver intel-media-driver \
 xinit xorg-server xorg-server-xnest xorg-server-xnest xorg-server-doc \
 xf86-video-vesa xf86-video-amdgpu xf86-video-nouveau xf86-video-intel \
 xf86-video-apm xf86-video-vmware xf86-video-ati xf86-video-nv xf86-video-openchrome \
 xf86-video-r128 xf86-video-qxl xf86-video-sis xf86-video-i128 xf86-video-i740 \
 xf86-video-savage xf86-video-s3virge xf86-video-chips xf86-video-tdfx xf86-video-ast \
 xf86-video-rendition xf86-video-ark xf86-video-siliconmotion xf86-video-fbdev \
 xf86-video-dummy xf86-input-evdev xf86-video-modesetting xf86-input-libinput \
 linux-firmware-amdgpu linux-firmware-radeon linux-firmware-nvidia linux-firmware-i915 linux-firmware-intel
 dbus dbus-x11 udisks2 udisks2-lang

dbus-uuidgen > /var/lib/dbus/machine-id

rc-update add dbus

apk add font-noto-all ttf-dejavu ttf-linux-libertine ttf-liberation \
 font-bitstream-type1 font-bitstream-100dpi font-bitstream-75dpi \
 font-adobe-utopia-type1 font-adobe-utopia-75dpi font-adobe-utopia-100dpi \
 font-isas-misc

apk add alsa-lib alsa-utils alsa-plugins alsa-tools alsaconf sndio \
 pipewire pipewire-pulse pipewire-alsa pipewire-spa-bluez wireplumber-logind

modprobe snd-pcm-oss
modprobe snd-mixer-oss
sed -i '/snd-pcm-oss/d' /etc/modules
sed -i '/snd-mixer-oss/d' /etc/modules
echo -e "snd-pcm-oss\nsnd-mixer-oss" >> /etc/modules
rc-service alsa restart
amixer sset Master unmute;  amixer sset PCM unmute;  amixer set Master 100%;  amixer set PCM 100%

cat > /etc/security/limits.d/audio-limits.conf << EOF
@audio - memlock 4096
@audio - nice -11
@audio - rtprio 88
@pipewire - memlock 4194304
@pipewire - nice -19
@pipewire - rtprio 95
EOF

rc-update add alsa

rc-service dbus restart

rc-service alsa restart

```

> **Warning** your user name must be `general`, you can put a "human name" as you wish, later.

## Instalation of XORG graphical base

```
apk add gtk-update-icon-cache xdg-user-dirs \
 xdg-desktop-portal xdg-desktop-portal-gtk \
 hicolor-icon-theme paper-gtk-theme adwaita-icon-theme \
 numix-icon-theme numix-themes numix-themes-gtk2 numix-themes-gtk3 \
 numix-themes-openbox
```

At this point you have graphics support but no desktop or session installed.

#### Login manager and session configurations

WE have LightDM and GREETD:

```
apk add elogind elogind-openrc lightdm lightdm-lang lightdm-gtk-greeter \
 polkit polkit-openrc polkit-elogind  networkmanager-elogind linux-pam \
 network-manager-applet network-manager-applet-lang vte3 shadow-login

rc-update add elogind
rc-update add polkit
rc-update add greetd

rc-service networking restart

rc-service networkmanager restart

rc-service elogind restart

rc-service polkit restart

rc-service lightdm restart
```

#### Installing OPENBOX as desktop adn configure it

```
apk add openbox openbox-doc dunst redshift scrot parcellite
 arandr xrandr gpicview zathura zathura-ps zathura-pdf-poppler \
 lxsession  libxinerama kbd setxkbmap \
 tint2  font-jetbrains-mono font-jetbrains-mono-nl wezterm-fonts \
 jgmenu xsel jgmenu-doc pcmanfm lxsession terminator xarchiver mousepad \
 terminator

mkdir -p /etc/xdg/openbox
touch /etc/xdg/openbox/autostart
sed -i '/pcmanfm/d' /etc/xdg/openbox/autostart
echo -e "pcmanfm --desktop &\n" >> /etc/xdg/openbox/autostart
sed -i '/tint2/d' /etc/xdg/openbox/autostart
echo -e "tint2 &\n" >> /etc/xdg/openbox/autostart
sed -i '/caja\.desktop\.wm/d' /etc/xdg/openbox/autostart 
echo -e 'gsettings set org.caja.desktop show-desktop-icons true >/dev/null 2>&1&\n' >> /etc/xdg/openbox/autostart
sed -i '/gnome\.desktop\.wm/d' /etc/xdg/openbox/autostart 
echo -e 'gsettings set org.gnome.desktop.wm.preferences button-layout "menu:minimize,maximize,close" >/dev/null 2>&1&\n' >> /etc/xdg/openbox/autostart
cat > /etc/xdg/openbox/environment << EOF
QT_QPA_PLATFORMTHEME=gtk2
XDG_SESSION_TYPE=x11
XDG_SESSION_DESKTOP=openbox
XDG_CURRENT_DESKTOP="mate:LXDE:openbox"
QT_QPA_PLATFORM=xcb
GDK_BACKEND="wayland,x11"
SDL_VIDEODRIVER="wayland,x11"
_JAVA_AWT_WM_NONREPARENTING=1
EOF

sed -i -r 's|.*titleLayout.*|<titleLayout>NDLSIMC</titleLayout>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|.*keepBorder.*|<keepBorder>yes</keepBorder>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|.*animateIconify.*|<animateIconify>no</animateIconify>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|[1-9]</size>|12</size>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|[1-9]</number>|1</number>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|.*drawContents.*|<drawContents>no</drawContents>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|<command>kfmclient.*|<command>pcmanfm</command>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|Clearlooks|Bear2|g' /etc/xdg/openbox/rc.mxl
sed -i -r 's|.*root-menu.*|<action name="Execute"><command>jgmenu_run</command></action>|g' /etc/xdg/openbox/rc.xml

cat > /etc/skel/.config/jgmenu/append.csv << EOF
Exit Session,openbox --exit,exit
^sep()
EOF
for u in $(ls /home); do mkdir -p /home/$u/.config/jgmenu && cp /etc/skel/.config/jgmenu/append.csv /home/$u/.config/jgmenu/append.csv; done
cat > /etc/skel/.config/Trolltech.conf << EOF
[Qt]
style=GTK+
EOF
for u in $(ls /home); do mkdir -p /home/$u/.config && cp /etc/skel/.config/Trolltech.conf /home/$u/.config/Trolltech.conf; done
for u in $(ls /home); do mkdir -p /home/$u/.config/openbox; done
for u in $(ls /home); do mkdir -p /home/$u/.config/jgmenu && cp /etc/skel/.config/jgmenu/jgmenurc /home/$u/.config/jgmenu/jgmenurc; done
for u in $(ls /home); do chown -R $u:$u /home/$u; done

```

#### OPENBOX menu configuration

```
mkdir -p /etc/xdg/openbox
cat > /etc/xdg/openbox/menu.xml << EOF
<?xml version="1.0" encoding="UTF-8"?>
<openbox_menu xmlns="http://openbox.org" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://openbox.org/ file:///usr/share/openbox/menu.xsd">
<menu id="root-menu" label="Openbox 3">
  <item label="Terminal">
    <action name="Execute"><execute>terminator</execute></action>
  </item>
  <item label="Applicacions">
    <action name="Execute"><execute>jgmenu-run --at-pointer</execute></action>
  </item>
  <item label="Exit">
    <action name="Exit" />
  </item>
</menu>
</openbox_menu>
EOF

```

#### multimedia and hardware media device access for the users

```
apk add gst-plugins-base gst-plugins-bad gst-plugins-ugly gst-plugins-good gst-plugins-good-gtk gst-plugin-pipewire \
 apk-gtk3 libcanberra-gtk3 libcanberra-gtk2 libcanberra-gstreamer libcanberra-pulse \
 qt5-qtbase-x11 qt6-qtbase-x11 gtk+3.0-demo  gtk4.0-demo \
 mediainfo ffmpeg ffmpeg-doc ffmpeg-libs lame lame-doc rtkit rtkit-doc \
 mpv mpv-doc deadbeef deadbeef-lang libxinerama xrandr cairo pango pixman

apk add gvfs gvfs-fuse ntfs-3g gvfs-archive gvfs-mtp gvfs-lang gvfs-nfs \
 gvfs-smb gvfs-cdda gvfs-afp gvfs-afc gvfs-dav gvfs-gphoto2 gvfs-avahi

for u in $(ls /home); do for g in plugdev audio cdrom dialout video netdev; do addgroup $u $g; done;done

cat > /etc/network/interfaces << EOF
auto lo
iface lo inet loopback
EOF

service networking restart

service wpa_supplicant restart

service networkmanager restart
```

## How to use this guide

This guide **structure all the commands in blocks, each block its separated by a line spaced**, 
so you must **type each line as is.. and hit enter**, so you noted that then you 
typed each separated clocks of commands, copy/type only blocks separated by an empty line, 
all new(next) lines are made by just enter. the terminal will detect if must execute or not.

**If you have another computer or gui**, try to use SSH client like putty or just in terminal (MAC or Linux) do:

1. at the Alpine installation: `sed -i 's|.*PermitRootLogin.*|PermitRootLogin yes|g' /etc/ssh/sshd_config;service sshd restart`
2. then from other OS just connect: `ssh -l root <ip>` change "`<ip>`" with the address of your device.
3. copy each separated by empty line, block of command, copy only blocks separate by empty line
4. and paste each separated by empty line block in the remnote (ssh), do not paste all the blocks at same time!

> **Warning** Some Linux or/and Mac terminals have security cut/paste locks, so 
if you paste, the first line will be preceded by garbage, check always the first char of your paste.

> **Warning** after finish, rerun: `sed -i -r 's|.*PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config`
and restart ssh `service sshd restart` becouse security implications.

**If you runs locally on same computer** all commmands must be run as root unless pointed in the guide

1. type string by string each line of command and runs each line one by one
2. because you still dont have graphical interfaces you must type by hand each line and runs by hit enter key

Done? return to [Preparation](#preparation-alpine) section of this document.

#### hardware used

| item             | minimal feature   | Extra recommendations              |
| ---------------- | ----------------- | ---------------------------------- |
| RAM MB           | 1Gb DDR1          | 6Gb DDR3, web browsers consumes so much |
| CPU              | intel Dual Core   | Not necesary                       |
| RAM CPU          | 2Mb (L2) 4kb/L1   |                                    |
| GPU              | intel G41         | Radeon X1200 For web browsers and modern apps will be need |
| RAM GPU          | 256Mb             | 1Gb For web browsers and modern apps will be need |
| Storage          | 120Gb HDD WD      | 256Gb SSD are mandatory for speed  |
| ARCH             | 32bits (i386/arm6)| 64bits (amd64) mandatory for most  |
| Audio            | AC 97             | HD audio and HDMI audio are a mess |

#### usernames

| item      | name                | password |
| --------- | ------------------- | -------- |
| remote    | daru                | daru     |
| admin     | root                | toor     |
| user      | general             | general  |

Done? return to [Preparation](#preparation-alpine) section of this document.

## Licensing clarifications

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

https://codeberg.org/alpine/alpine-wiki/src/branch/main#license

## See also

* [README.md](README.md)
* [alpine-newbie-install.md](../../newbie/alpine-newbie-install.md)
* [alpine-tutorial-desktop-wayland-try.md](alpine-tutorial-desktop-wayland-try.md)
