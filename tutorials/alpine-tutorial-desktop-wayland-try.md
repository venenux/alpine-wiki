# Alpine WAYLAND desktop setup: FF version
===========================================================

Alpine must be previously installed. This will install a new fashioned desktop, for more traditional check [../../newbie/alpine-newbie-xfce-desktop.md](../../newbie/alpine-newbie-xfce-desktop.md)

* [How to use this guide](#how-to-use-this-guide)
* [Preparation](#preparation)
    * [setup OS configuration](#setup-os-configuration)
    * [setup system users](#setup-system-users)
    * [setup hardware support](#setup-hardware-support)
    * [setup audio and video](#setup-audio-and-video)
* [Instalacion Desktop WAYLAND Alpine](#instalacion-desktop-wayland-apine)
    * [Login manager and user configurations](#login-manager-and-user-configurations)
    * [multimedia and device enhanced](#multimedia-and-device-enhanced)
* [Licensing clarifications](#licensing-clarifications)
* [See also](#see-also)

## preparation Alpine

You must have already installed alpine, and wayland only works well in few GPU cards, for stable desktop use XFCE check [alpine-tutorial-desktop-xfce4-fast-forward.md](alpine-tutorial-desktop-xfce4-fast-forward.md)

> **Warning** **YOU MUST HAVE DIRECT WIRED INTERNET, if not ask for an ISO from VenenuX:** [https://t.me/alpine_linux/762](https://t.me/s/alpine_linux/762)
or configure a network connection check [alpine-tutorial-wifi-routering.md](alpine-tutorial-wifi-routering.md)

Main problem with alpine is that most old images of installation doe snot have the tools 
installed to setup the wifi.. so you must have wired connection or setup a wifi manually!

#### setup OS configuration

Feels lost here? check [How to use this guide](#how-to-use-this-guide) section of this document

> **Warning** For **didactic** processes, **the root password will be "toor"**, you can change it after

Runs following commands as root user:

```
sed -i -r 's|#PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config
rc-service sshd restart;rc-update add sshd default

cat > /root/.cshrc << EOF
unsetenv DISPLAY || true
HISTCONTROL=ignoreboth
EOF

cp /root/.cshrc  /root/.bashrc
 echo "root:toor" | chpasswd

hostname venenux-desktop
echo 'hostname="venenux-desktop"' > /etc/conf.d/hostname 
echo "venenux-desktop" > /etc/hostname
cat > /etc/hosts << EOF
127.0.0.1 venenux-desktop localhost.localdomain localhost
::1 localhost localhost.localdomain
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

apk add man-db man-pages nano binutils coreutils readline \
 sed attr dialog lsof less groff wget curl \
 file lz4 arch-install-scripts gawk tree pciutils usbutils lshw \
 zip p7zip xz tar cabextract cpio binutils lha acpi musl-locales musl-locales-lang \
 e2fsprogs e2fsprogs-doc btrfs-progs btrfs-progs-doc exfat-utils exfat-utils-doc \
 f2fs-tools f2fs-tools-doc dosfstools dosfstools-doc xfsprogs xfsprogs-doc jfsutils jfsutils-doc \
 arch-install-scripts util-linux zram-init tzdata tzdata-utils

apk add font-terminus

setfont /usr/share/consolefonts/ter-132n.psf.gz

sed -i "s#.*consolefont.*=.*#consolefont="ter-132n.psf.gz"#g" /etc/conf.d/consolefont

rc-update add consolefont boot
```

> **Warning**: `font-terminus` is ony since alpine v3.18, for older versions use `terminus-font`

For more extended info check [../../newbie/alpine-newbie-xfce-desktop.md](../../newbie/alpine-newbie-xfce-desktop.md#setup-os-configuration)

#### setup system users

```
apk add shadow shadow-uidmap doas musl-locales musl-locales-lang

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
```

> **Warning** your user name must be `general`, you can put a "human name" as you wish, later.

For more details check  [../../documents/alpine-newbie-xfce-desktop.md](../../documents/alpine-newbie-xfce-desktop.md#setup-system-users)

#### setup hardware support

```
apk add acpi acpid acpid-openrc alpine-conf eudev eudev-doc eudev-netifnames eudev-openrc \
 pciutils util-linux arch-install-scripts zram-init acpi-utils \
 fuse fuse-exfat-utils fuse-exfat avfs pcre2 cpufreqd bluez bluez-deprecated bluez-openrc \
 wpa_supplicant dhcpcd chrony macchanger wireless-tools iputils linux-firmware \
 networkmanager networkmanager-lang networkmanager-openvpn networkmanager-openvpn-lang

modprobe btusb && echo "btusb" >> /etc/modprobe
setup-devd udev

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

```

For more details check  [../../documents/alpine-newbie-xfce-desktop.md](../../documents/alpine-newbie-xfce-desktop.md)

#### setup audio and video

> **Note** on alpine 3.14 gtk3 will force xorg dependencies.. for 3.16 will use gtk4 and SDL2 with wayland

```
apk add mesa mesa-gl mesa-utils mesa-osmesa mesa-egl mesa-gles mesa-dri-gallium mesa-va-gallium libva-intel-driver intel-media-driver \
 xf86-video-amdgpu xf86-video-nouveau xf86-video-intel xf86-video-vmware xf86-video-ati \
 xf86-video-dummy xf86-input-evdev xf86-video-modesetting xf86-input-libinput \
 linux-firmware-amdgpu linux-firmware-radeon linux-firmware-nvidia linux-firmware-i915 linux-firmware-intel

apk add libxinerama xrandr kbd setxkbmap bluez bluez-openrc \
 dbus dbus-x11 udisks2 udisks2-lang \
 gvfs gvfs-fuse gvfs-archive gvfs-dav gvfs-nfs gvfs-lang

modprobe fbcon && echo "fbcon" >> /etc/modprobe

dbus-uuidgen > /var/lib/dbus/machine-id

rc-update add dbus

apk add font-noto-all ttf-dejavu ttf-linux-libertine ttf-liberation \
 font-bitstream-type1 font-bitstream-100dpi font-bitstream-75dpi \
 font-adobe-utopia-type1 font-adobe-utopia-75dpi font-adobe-utopia-100dpi \
 font-isas-misc

apk add alsa-lib alsa-utils alsa-plugins alsa-tools alsaconf sndio \
 pipewire pipewire-pulse pipewire-alsa pipewire-spa-bluez wireplumber-logind

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

> **Note** pure wayland will work only in modern OpenGL/VaPI capable gpu, that's why you never see here other models rather than intel, amd and nvidia

## Instalacion Desktop WAYLAND Alpine

At the moment of alpine 3.16 and wayland development, this mayor shit is 
that only works with OpenGL and 3D acceleration, its pretty ilogical that 
a so reduced environment need 3D for mostly eye candy.. ironically thing!

```
apk add gtk-update-icon-cache hicolor-icon-theme paper-gtk-theme adwaita-icon-theme \
 numix-icon-theme numix-themes numix-themes-gtk2 numix-themes-gtk3 numix-themes-metacity numix-themes-openbox numix-themes-xfce4-notifyd numix-themes-xfwm4

apk add xwayland \
 wayland wlroots foot sway sway-doc bemenu swaylock swaylockd swaybg swayidle \
 weston weston-backend-wayland weston-backend-x11 weston-backend-drm weston-backend-wayland weston-backend-headless \
 weston-doc weston-shell-desktop weston-desktop-x11 weston-clients weston-terminal \
 weston-xwayland weston-shell-desktop weston-shell-fullscreen weston-cms-static
```

At this point you already has a waylan environment and can choose beetween weston and sway, 
just login into and start your desktop, weston is just the first implementation, can be run 
inside and X11 or another wayland session, sway is a window manager and compositor.

#### Login manager and user configurations

If (after runs wayland) module auto-selection does not work, e.g. no mouse cursor under Sway, 
manually selection might be needed to declare explilcy variables for current session:

* `MESA_LOADER_DRIVER_OVERRIDE=crocus` for Intel's cards, all Intel Graphics up to Haswellsteam
* `MESA_LOADER_DRIVER_OVERRIDE=r300` for AMD's Radeon R300, R400, and R500 GPUs.
* `MESA_LOADER_DRIVER_OVERRIDE=r600` for AMD's Radeon R600 GPUs up to Northern Islands. Officially supported by AMD.
* `MESA_LOADER_DRIVER_OVERRIDE=radeonsi` for AMD's Southern Island GPUs and later. Officially supported by AMD.
* `MESA_LOADER_DRIVER_OVERRIDE=iris` for Intel's Iris modern GPUs and later.

So we need to isntall the required login manager, lightdm does not officially support wayland 
so you need to choose between GDM (for gtk) or SDDM (for QT) but you can use elogind to manage it:

```
apk elogind elogind-openrc lightdm lightdm-lang lightdm-gtk-greeter \
 polkit polkit-openrc polkit-elogind  networkmanager-elogind linux-pam \
 network-manager-applet network-manager-applet-lang vte3

rc-service networking restart

rc-service networkmanager restart

rc-service lightdm restart
```

> **Warning** : for alpine 3.14, 3.15 just works the login sesion for sway, maybe 3.16 will 
result in a blank screen, check https://github.com/swaywm/sway/pull/3634#issuecomment-462779163
furter releases now supports session manager correctly!

The wayland weston and sway configurations depens on your preferences, 
the above commands just provide defaults to made those compositors able to run for users.

If want autologin with TTY use this script to your system users:

```
for u in $(ls /home); do mkdir -p /home/$u/.config/sway/ && cp /etc/sway/config /home/$u/.config/sway/config ;done

for u in $(ls /home); do touch /home/$u/.config/weston.ini;done

cat > /etc/skel/.profile << EOF
    if test -z "\${XDG_RUNTIME_DIR}"; then
      export XDG_RUNTIME_DIR=/tmp/\$(id -u)-runtime-dir
      if ! test -d "\${XDG_RUNTIME_DIR}"; then
        mkdir "\${XDG_RUNTIME_DIR}"
        chmod 0700 "\${XDG_RUNTIME_DIR}"
      fi
    fi
EOF
for u in $(ls /home); do cp /etc/skel/.profile /home/$u/ ;done

cat > /tmp/.xinitrc << EOF
if [ -z "\${DISPLAY}" ] && [ "\${XDG_VTNR}" -eq 1 ]; then
  if [ "\$(fgconsole 2>/dev/null || echo -1)" -eq 1 ]; then
    dbus-run-session -- sway
    export SWAYSOCK=/run/user/$(id -u)/sway-ipc.$(id -u).$(pgrep -x sway).sock
  fi
fi
EOF
for u in $(ls /home); do cp /tmp/.xinitrc /home/$u/.xinitrc;done
```

On older versions (Alpine 3.12 or less) the xx-openrc packages dont exists!

#### desktop integration and device media

```
apk add xdg-desktop-portal xdg-desktop-portal-wlr xdg-desktop-portal-lang xdg-desktop-portal-gtk xdg-desktop-portal-gtk-lang
```

#### multimedia and hardware media device access for the users

```
apk add gst-plugins-base gst-plugins-bad gst-plugins-ugly gst-plugins-good gst-plugins-good-gtk gst-plugin-pipewire \
 libcanberra-gtk2 libcanberra-gtk3 libcanberra-gstreamer \
 mediainfo ffmpeg ffmpeg-doc ffmpeg-libs lame lame-doc rtkit rtkit-doc \
 mpv mpv-doc deadbeef deadbeef-lang libxinerama xrandr cairo pango pixman

apk add gvfs-fuse ntfs-3g gvfs-cdda gvfs-afp gvfs-mtp gvfs-smb gvfs-lang \
 gvfs-afc gvfs-nfs gvfs-archive gvfs-dav gvfs-gphoto2 gvfs-avahi

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
2. at the other OS just connect: `ssh -l root <ip>` change "`<ip>`" with the address of your device.
3. copy each separated by empty line, block of command, copy only blocks separate by empty line
4. and paste each separated by empty line block in the remnote (ssh), do not paste all the blocks at same time!

> **Warning** Some Linux or/and Mac terminals have security cut/paste locks, so 
if you paste, the first line will be preceded by garbage, check always the first char of your paste.

> **Warning** after finish, rerun: `sed -i -r 's|.*PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config`
and restart ssh `service sshd restart` becouse security implications.

Done? return to [Preparation](#preparation-alpine) section of this document.

#### hardware used

| item             | minimal feature   | Extra recommendations              |
| ---------------- | ----------------- | ---------------------------------- |
| RAM MB           | 1Gb DDR1          | 6Gb DDR3, web browsers consumes so much |
| CPU              | intel Dual Core   | Not necesary                       |
| RAM CPU          | 2Mb (L2) 4kb/L1   |  |
| GPU              | intel G41         | Radeon X1200 For web browsers and modern apps will be need |
| RAM GPU          | 256Mb             | 1Gb For web browsers and modern apps will be need |
| Storage          | 120Gb HDD WD      | 256Gb SSD are mandatory for speed |
| ARCH             | 32bits (i386/arm6)| 64bits (amd64) mandatory for most modern apps unfortunatelly |
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
* [alpine-tutorial-desktop-xfce4-fast-forward.md](alpine-tutorial-desktop-xfce4-fast-forward.md)
