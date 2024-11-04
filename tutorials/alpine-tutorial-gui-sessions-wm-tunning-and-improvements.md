# Alpine Window managers Desktop Tips and tunning improvements
===========================================================

# GUI tips 

There is two type of GUI env in linux now, X11 and Wayland the first one is 
the widelly compatible with any gpu, the second one only for modern GPU capable 
of 3D, this last is NOT LIGHT just is faster cos it uses all the advanced 
features of GPUs.

## Desktop features and improvements

### desktop integration and device media improvement

```
apk add shadow shadow-uidmap shadow-login doas \
 gst-plugins-base gst-plugins-bad gst-plugins-ugly \
 gst-plugins-good gst-plugins-good-gtk gst-plugin-pipewire \
 libcanberra-gtk2 libcanberra-gtk3 libcanberra-gstreamer \
 mediainfo ffmpeg ffmpeg-doc ffmpeg-libs lame lame-doc rtkit rtkit-doc \
 mpv mpv-doc deadbeef deadbeef-lang libxinerama xrandr cairo pango pixman
 gvfs gvfs-fuse gvfs-archive gvfs-afp gvfs-afp gvfs-afc gvfs-cdda \
 ntfs-3g gvfs-afc gvfs-nfs gvfs-archive gvfs-dav gvfs-gphoto2 gvfs-avahi
 gvfs-gphoto2 gvfs-mtp libreoffice libreoffice-gnome \
 zathura zathura-ps zathura-pdf-poppler zathura-djvu zathura-cb

for u in $(ls /home); do for g in plugdev audio cdrom dialout video netdev; do addgroup $u $g; done;done

cat > /etc/network/interfaces << EOF
auto lo
iface lo inet loopback
EOF

service networking restart && service wpa_supplicant restart && service networkmanager restart
```

### flakpack desktop integration

```
apk add xdg-desktop-portal xdg-desktop-portal-wlr xdg-desktop-portal-lang \
 xdg-desktop-portal-gtk xdg-desktop-portal-gtk-lang xdg-desktop-portal-gnome \
 xdg-desktop-portal-hyprland xdg-desktop-portal-kde xdg-desktop-portal-lxqt \
 xdg-desktop-portal-phosh xdg-desktop-portal-xapp
```

### X11 NOTES : mga matrox modules

The mga module since 2018 is only created for server boards, this section is 
not for the old cards that has limited GPU/KMS for the nowadays task.

This setting may work with older cards too, by the `Option "AccelMethod" "none"` 
make it possible to work under demanding 3D required envs:

```
Section "Device"
  Identifier "Matrox"
  Driver "modesetting"
  Option "AccelMethod" "none"
EndSection
```

# OPENBOX tunning improvements

Pre-configure with improved GTK applications using MATE such 
as `caja` for desktop handler and file manager, 
the `caja-extensions` for file manager actions, 
also `mate-control-center` for desktop configuration, 
`mate-polkit` for segurity elevation, 
the `mate-settings-daemon` for sync all the configs beetween GTK 2/3/4, 
the `engrampa` for archiving and compression tool cos 
are forced in junction with `caja` and provides better integration 
with propietary archived/compresed files.

This document assume you used [alpine-tutorial-desktops-openbox-fast-forward.md](alpine-tutorial-desktops-openbox-fast-forward.md) 
guide, so this just continue and change the settings no matter what are present. 

This document will assume you already has installed `openbox`, `elogind`, 
`lightdm` as default, `xorg-server`, `network-manager-applet`, `jgmenu`, and 
the `pipewire` framework using `wireplumber-logind` package, if not just run: 
`apk add openbox elogind lightdm xorg-server network-manager-applet jgmenu pipewire wireplumber-logind`

```
apk add mate-desktop mate-session-manager mate-panel sakura engrampa pluma \
 mate-themes mate-polkit mate-power-manager mate-settings-daemon \
 clipper mate-notification-daemon mate-screensaver mate-utils mate-system-monitor mate-menus \
 mate-control-center mate-control-center-lang caja caja-lang caja-extensions \
 mate-panel-lang mate-media mate-media-lang mate-screensaver-lang mate-utils-lang \
 mate-system-monitor-lang mate-applets mate-applets-lang mate-power-manager-lang mate-settings-daemon-lang \
 gvfs gvfs-fuse gvfs-archive gvfs-afp gvfs-afp gvfs-afc gvfs-cdda gvfs-gphoto2 gvfs-mtp \
 libreoffice libreoffice-gnome atril atril-lang atril-doc

sed -i -r 's|.*xfce-mcs-manager \&.*|/usr/libexec/mate-settings-daemon \&|g' /etc/xdg/openbox/autostart
```

#### Why not just install MATE

Installing MATE will add a layer of complexity, **we just use the daemon sync 
for settings and the polkit for security elevation**, session handler is managed 
by the openbox itselft, the virtual filesystems are handled by the gvfs package 
software, and the **powermanager is the only thing you will be forced to use if 
your bare metal computer is a laptop** (that can be avoid if you configured couple 
of backends programs as fdpowerdown).

Also we do not use by default the panel, but here we will configure it for 
older non modern users, you just can avoid the panel configuration cos we 
already have the GJmenu handler for.

Unfortunatelly Alpine packagers do not have any GUI exec launcher.. so the panel 
will provide this, unless you like to have a lof of stupid terminal windows 
opened to just launch programs being a stupid geek old school and wasting 
lof of threats of execution and mem ram.

### Reasons and justifications

* Most of tools and integration here were using MATE ones, this cos are the 
only tools that does no use client side decorations (GTK-CSD) and integrates 
very well and provided advanced usage, most close to the GNOME ones that are 
currently well developed but too stupidity eye candy and does not integrated to 
most older or newer rest of programs.
* By the same reason we used polkit and settings daemon from MATE also 
if were used the ones from XFCE this will install more depends, the MATE ones 
already uses GTK/GIO packages from GNOME and FXCE will install the sames but 
plus install also others from itselft, so its nonsense.
* We used `engrampa` by the same previous reason, its based on `file-roller` 
that is the archiving tool with best support for non-free archivers, the 
light well knowed `xarchiver` is too poor in such support and the rest are 
made in QT framework that will force lot of dependencies and poor GTK integration.
* We used `caja` cos web browsers only understand `caja`, `thunar` and `nautilus` 
as filemanagers with pinting focus of downloaded file, you can change by `pcmanfm` 
but file pointing will be limited if you manage huge amount of files in same 
directory (forced to use find tool event just click on the web browser after 
file downloading).
* For image viewer and document viewer you can use `atril` and `eom` but those 
are the best cos supports virtual file systems, the `gpicview` package is 
very light but does not support any virtual file system, in the same line is 
the document viewer, there is `zathura` that two more packages to support ps/pdf 
files for extra dependencies.

> **Warning** the `openbox-doc` package must be installed, just run `apk add openbox-doc`

#### openbox session improved menu and desktop configuration

```
cat > /etc/xdg/openbox/menu.xml << EOF
<?xml version="1.0" encoding="UTF-8"?>
<openbox_menu xmlns="http://openbox.org" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://openbox.org/ file:///usr/share/openbox/menu.xsd">

<menu id="root-menu" label="Openbox 3">
  <item label="Terminal">
    <action name="Execute"><execute>terminator</execute></action>
  </item>
  <item label="Applicacions">
    <action name="Execute"><execute>jgmenu-run</execute></action>
  </item>
  <separator />
  <item label="Configure">
    <action name="Execute"><execute>mate-control-center</execute></action>
  </item>
  <item label="Reload">
    <action name="Reconfigure" />
  </item>
  <item label="Restart">
    <action name="Restart" />
  </item>
  <separator />
  <item label="Exit">
    <action name="Exit" />
  </item>
</menu>
</openbox_menu>
EOF

mkdir -p /etc/skel/.config/jgmenu/
cat > /etc/skel/.config/jgmenu/jgmenurc << EOF
stay_alive           = 0
tint2_look           = 0
position_mode        = pointer
terminal_exec        = terminator
terminal_args        = -e
menu_width           = 200
menu_padding_top     = 5
menu_padding_right   = 1
menu_padding_bottom  = 4
menu_padding_left    = 1
menu_radius          = 0
menu_border          = 1
menu_halign          = left
sub_hover_action     = 1
item_margin_y        = 2
item_height          = 20
item_padding_x       = 4
item_radius          = 0
item_border          = 0
sep_height           = 3
font                = Sans 12
icon_size            = 16
EOF

cat > /etc/skel/.config/jgmenu/append.csv << EOF
Exit Openbox,openbox --exit,exit
^sep()
EOF

for u in $(ls /home); do mkdir -p /home/$u/.config/jgmenu && cp /etc/skel/.config/jgmenu/jgmenurc /home/$u/.config/jgmenu/jgmenurc; done

sed -i -r 's|.*titleLayout.*|<titleLayout>NDLSIMC</titleLayout>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|.*keepBorder.*|<keepBorder>yes</keepBorder>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|.*animateIconify.*|<animateIconify>no</animateIconify>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|[1-9]</size>|12</size>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|[1-9]</number>|1</number>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|.*drawContents.*|<drawContents>no</drawContents>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|<command>kfmclient.*|<command>pcmanfm</command>|g' /etc/xdg/openbox/rc.xml
sed -i -r 's|Clearlooks|Bear2|g' /etc/xdg/openbox/rc.mxl
sed -i -r 's|.*root-menu.*|<action name="Execute"><command>jgmenu_run</command></action>|g' /etc/xdg/openbox/rc.xml

for u in $(ls /home); do mkdir -p /home/$u/.config/openbox; done

for u in $(ls /home); do chown -R $u:$u /home/$u; done
```

# Wayland shit session tips

Wayland is so crap becouse nothig great work under and forced 3d gpu usage

### Wayland environments and need variables

If (after runs wayland) module auto-selection does not work, e.g. no mouse cursor under Sway, 
manually selection might be needed to declare explilcy variables by example:

* `WLR_NO_HARDWARE_CURSORS=1` will be need for QT environment under wayland
* `MESA_LOADER_DRIVER_OVERRIDE=crocus` for Intel's GPUs up to Haswellsteam
* `MESA_LOADER_DRIVER_OVERRIDE=r300` for AMD's Radeon R300, R400, and R500 GPUs.
* `MESA_LOADER_DRIVER_OVERRIDE=r600` for AMD's Radeon R600 up to Nort Islands.
* `MESA_LOADER_DRIVER_OVERRIDE=radeonsi` for AMD's Sout Island GPUs and later.
* `MESA_LOADER_DRIVER_OVERRIDE=iris` for Intel's Iris/Arc modern GPUs and later.
* `_JAVA_AWT_WM_NONREPARENTING=1` for threath [all events as X11 requests](https://github.com/Smithay/smithay/issues/389#issuecomment-933202741).
* `SDL_VIDEODRIVER=wayland` may be used with caution, ex: Steam is buitl witout it.
* `QT_QPA_PLATFORM=wayland` used with caution, for qt6 must be, for qt5 can be null
* `MOZ_ENABLE_WAYLAND=1` only will work in lasted firefox package (3.20+)

### Setup Greetd wayland autologin with labwc

Only GDM, SDDM and GREETD fully supports wayland and only if you have intel or AMD gpu, so crap!

This guide will:

1. install required packages like greetd, labwc, elogind etc
2. setup the user that will login it automatically
3. setup the greetd service to autologin
4. setup environment to do not fail first session

```
apk add elogind elogind-openrc shadow-login greetd greetd-gtkgreet cage \
 mesa mesa-gl mesa-utils mesa-osmesa mesa-egl mesa-gles \
 mesa-dri-gallium mesa-va-gallium libva-intel-driver \
 xf86-video-intel xf86-video-amdgpu xf86-video-nouveau xf86-video-ati \
 xf86-input-evdev xf86-video-modesetting xf86-input-libinput \
 linux-firmware-amdgpu linux-firmware-radeon linux-firmware-nvidia \
 linux-firmware-i915 linux-firmware-intel font-noto-all ttf-dejavu \
 polkit polkit-openrc polkit-elogind  networkmanager-elogind linux-pam \
 labwc xwayland wayland-libs-server wayland wlroots weston-desktop-x11 \
 weston weston-backend-wayland weston-backend-x11 weston-backend-drm \
 weston-backend-wayland weston-backend-headless weston-shell-desktop \
 weston-clients wlogout font-jetbrains-mono wezterm-fonts \
 dbus dbus-x11 udisks2 shadow shadow-uidmap shadow-login doas


cat > /etc/doas.d/apkgeneral.conf << EOF
permit nopass general as root cmd apk
EOF
useradd -m -U -c "" -G wheel,input,disk,floppy,cdrom,dialout,audio,video,lp,netdev,games,users,ping general


cat > /etc/greetd/config.toml << EOF
[terminal]
vt = next
switch = true
[default_session]
command = "dbus-run-session labwc"
user = general
EOF
cat > /etc/conf.d/greetd << EOF
cfgfile="/etc/greetd/config.toml"
rc_need=elogind
EOF

rc-update add elogind
rc-update add polkit
rc-update add greetd

rc-service elogind restart

rc-service polkit restart

rc-service greetd restart
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
* [alpine-tutorial-desktop-wayland-try.md](alpine-tutorial-desktop-wayland-try.md)
