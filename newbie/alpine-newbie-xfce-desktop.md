# Alpine XFCE4 desktop setup
===========================================================

* [preparation](#preparation-xfce4-aline)
    * [instalation](#instalation)
    * [setup OS configuration](#setup-os-configuration)
    * [configuration programs and repositories](#configuration-programs-and-repositories)
    * [setup system users](#setup-system-users)
    * [setup hardware media support and graphical subsystem](#setup-hardware-media-support-and-graphical-subsystem)
    * [setup software graphical fonts and languajes](#setup-software-graphical-fonts-and-languajes)
    * [setup hardware media support and sound subsystem](#setup-hardware-media-support-and-sound-subsystem)
* [Instalacion XFCE4 Alpine](#instalacion-xfce4-apine)
* [Desktop multimedia and media devices](#desktop-multimedia-and-media-devices)
* [Desktop Office suite](#desktop-office-suite)
* [Development](#development)
    * [base console development](#base-console-development)
    * [base gui development](#base-gui-development)
* [Licensing clarifications](#licensing-clarifications)
* [See also](#see-also)

## preparation Xfce4 Alpine

To have a usable desktop without having to wonder why it doesn't sound, 
or why the video file doesn't display, **you should avoid minimalisms if 
you don't already have advanced knowledge** of alpine and even more so of linux.

For desktop you should have at least 100G of space on the ROOT disk, 
and at least 2G of memory on the RAM.

The document **assumes that you already have Alpine Linux installed 
otherwise check the best option at [alpine-newbie-install.md](alpine-newbie-install.md),** 
we recommend 3.10 to 3.12 for 32bit PC and 3.12 for older RPi, 
for modern machines we recommend 3.16 64bit versions.

#### Internet required

This document will work only if you have direct wired connection to internet.

**If you dont have direct network connection, please use our direct VenenuX Alpine ISOS**
[CURRENT LINK https://t.me/alpine_linux/762, but ask in telegram alpine network for newer one](https://t.me/s/alpine_linux/762)

#### setup OS configuration

1. deny access to the ssh root user, or well, get sure to deny such access
2. setup and start the ssh service, this always be present in Alpine installs
3. set the name of the computer, here we used `venenux-desktop`, please avoid symbols
4. hardcoded and set the network connection, to first wired network device
5. setup services of network to init at boot
6. configure root account
7. create a remote connection limited account `daru`
8. configure a default remote connection account `daru` as standard

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

1. Setup main and community repositories of sources of programs
2. update the local references of such repositories
3. install main command line utilities man page manager and easy to use editor
4. install string manipulation tools for the commands on the console
5. install file downloaders and url handlers for command line administration
6. install backend support for backend of archivers, will be need in any futher kind of install
7. setup the pager for large outputs over the console
8. install the support for languaje environment


``` bash
cat > /etc/apk/repositories << EOF
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add man-db man-pages nano nano-doc binutils binutils-doc coreutils coreutils-doc readline readline-doc

apk add sed sed-doc attr attr-doc dialog dialog-doc lsof less less-doc groff groff-doc

apk add wget wget-doc curl curl-doc bash bash-doc bash-completion terminus-font

apk add zip p7zip xz tar cabextract cpio binutils lha acpi 

export PAGER=less

apk add musl-locales musl-locales-lang

```

The packages that have a "-doc" part handles the manpages.. you can 
avoid those packages, if Alpine usage will be for GUI only.

The packages of locales will be need as base for multi-lang enviroment.

#### setup system users

We will follow a protocol for better identification of all the commands 
in further documents, the user will be called "general"; this is important 
that it has does matter, what does not matter is for you cos you only "use".

The other important part here is the groups, user will not have porper access 
to resources becouse the OS manages the access using levels by groups. Of course 
all of this only works using the proper policy kit software.

1. install and setup advanced management of users and administration tool privilegies
2. setup the shell csh for all present users and future ones
3. setup the shell bash for all present users and future ones
4. setup the shell for skel directory, that will server as template
5. setup the Xresources configuration for skel directory, that will server as template
6. configure defaults for each new creation of new users with bash as default shell
7. configure the login environement for each user when login at the console
8. create the default new general usage user `general` for your desktop or console daily work
9. include this `general` user in the most used need groups to have proper access to resources

```
apk add shadow shadow-doc shadow-uidmap bash bash-doc bash-completion bash-dev doas doas-doc

cat > /tmp/tmpcs.tmp << EOF
set history = 10000
set prompt = "$ "
EOF

for i in $(ls /home);do cat /tmp/tmpcs.tmp > /home/$i/.cshrc;done

cat > /tmp/tmpbs.tmp << EOF
set prompt = "$ "
set history = 10000
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
EOF

for i in $(ls /home);do cat /tmp/tmpbs.tmp > /home/$i/.bashrc;done

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


useradd -m -U -c "" -G wheel,input,disk,floppy,cdrom,dialout,audio,video,lp,netdev,games,users,ping general

for u in $(ls /home); do for g in disk lp floppy audio cdrom dialout video lp netdev games users ping; do addgroup $u $g; done;done
```

In the last part the `general` user is created since for purposes of usage 
the name its hard coded for the commands that can be sense and important, 
because of the references in further documents.

At last, the groups are cofigured for the user, each one will define a 
resource access, specially those for `disk`, `audio`, `video` and `netdev`.

#### setup hardware media support and graphical subsystem

Until this point, we have a user that can login to the system, 
and bunch of programs to interact with the OS, but still do not 
have any programs to make the computer or device work for us.. 
this target depends of the set of programs, this section will 
configure the graphical environment to property setup any desktop.

1. install programs to manage devices dinamically by the OS and not admin user
2. setup service for dynamic device manager at boot, energy management and cpu frecuency management
3. install and configure the graphical subsystem and modesetting multi GPU video card support
4. install set of need modules for 2D/3D specific GPU or video card first generation support
5. install set of need modules for 2D/3D common GPU or video card almost modern still supported
6. install support for multi resolution and keyboard language configuration over GUI programs
7. install the bus communitacion support, policy management and login backend and frontend
8. generate the machine id identification hack for stupid shistemd linux standards
9. activate the service of the bus cominucations, the policy rules and graphical login backend
10. start the service of the bus cominucations, the policy rules and graphical login backend
11. activate the service of the graphical frontend login manager
12. start the service of the graphical frontend login manager
13. install support for abstract device filesystem representation using FUSE user space
14. activate the service of abstract device filesystem representation using FUSE user space
15. install software backend for usage of abstract device filesystem representation using FUSE user space

```
apk add acpi eudev eudev-doc eudev-rule-generator eudev-openrc linux-firmware cpufreqd pciutils pciutil-doc util-linux util-linux-doc

rc-update add udev
rc-update add acpid
rc-update add cpufreqd

setup-xorg-base xinit mesa-dri-gallium xf86-video-dummy xf86-video-modesetting xf86-video-qxl xf86-video-vmware xf86-input-evdev xf86-input-synaptics

apk add xf86-video-r128 xf86-video-s3virge xf86-video-apm xf86-video-i128 xf86-video-glint xf86-video-tdfx xf86-video-openchrome

apk add xf86-video-intel xf86-video-amdgpu xf86-video-ati xf86-video-nouveau xf86-video-nv xf86-video-vesa

apk add libxinerama xrandr kbd setxkbmap

apk add dbus dbus-x11 elogind elogind-openrc elogind-lang polkit polkit-openrc polkit-elogind lightdm lightdm-lang lightdm-gtk-greeter 

dbus-uuidgen > /var/lib/dbus/machine-id

rc-update add dbus
rc-update add elogind
rc-update add polkit

rc-service dbus restart

rc-service elogind restart

rc-service polkit restart

rc-update add lightdm

rc-service lightdm restart

apk add fuse fuse-exfat-utils archivemount fuse-exfat avfs vte3 pcre2

rc-service fuse start

rc-update add fuse

apk add gvfs udisks2 udisks2-lang udisks2-doc gvfs-fuse gvfs-archive gvfs-dav gvfs-nfs gvfs-lang

```

#### setup software graphical fonts and languajes

Now we have support for graphics in the operating system, 
but still do not have any graphical program, so we will prepare
minimal set of software for the right working of the programs 
such like the fonts and the support of multi languajes.

1. install console support fonts for tty terminal and console terminal programs
2. setup and configure the default tty terminal fonts
3. set as service the configuration of the font at terminal
4. install basic set of default fonts for graphical programs, dejavu and bitstream ones
5. install fonts for japanese symbos and sans/comic like fonts
6. install fonts for cyrillic langs to display russian words
7. install fonts for mid-orient langs like arabic, hebrew, armenian, hindi / sanskrit
8. install fonts for langs like tailan and  malayalam
9. install fonts for east-orient like chinese symbols,hangul and kana ones, includign japanese

```
apk add terminus-font

setfont /usr/share/consolefonts/ter-132n.psf.gz

sed -i "s#.*consolefont.*=.*#consolefont="ter-132n.psf.gz"#g" /etc/conf.d/consolefont

rc-update add consolefont boot

apk add ttf-dejavu font-bitstream-type1 font-bitstream-100dpi font-bitstream-75dpi font-adobe-utopia-type1 font-adobe-utopia-75dpi font-adobe-utopia-100dpi font-noto-georgian

apk add font-noto font-noto-extra font-noto-cjk font-noto-cjk-extra font-arabic-misc ttf-liberation ttf-linux-libertine 

apk add font-misc-cyrillic font-mutt-misc font-screen-cyrillic font-winitzki-cyrillic font-cronyx-cyrillic

apk add font-noto-hebrew font-noto-arabic font-noto-armenian font-noto-cherokee font-noto-devanagari font-noto-ethiopic

apk add font-noto-lao font-noto-malayalam font-noto-tamil font-noto-thaana font-noto-thai

apk add font-noto-cjk font-noto-cjk-extra font-isas-misc
```

If you want to bypass and dont know anything, run steps from 1 to 4 adn then 
after step 4 just install `font-noto-all` that provides support for all 
langs symbols words.

#### setup hardware media support and sound subsystem

Audio support these days depends on video support, although it can still be 
configured separately, nowadays it is strongly tied to video, that's why its 
placed here this section, just below of the hardware video setup.

It should be noted that in Linux there is the system audio layer and the user audio layer, 
formerly the first was and can used directly, nomadays, by using the second 
we are in transition to the wonderful pipewire framework.

1. install the base audio subsystem software
2. install the base audio framework user support
3. restart bus communicacion, due we already have the software video subsystem support bus communication
4. setup the access prority for the audio communication
5. install optional bluetooth software support if you have a bluez audio card or phones
6. setup the bluetooth service
7. start the bluetooth service so the audio devices can be stablished

```
apk add alsa-utils alsa-utils-doc alsa-plugins alsa-plugins-doc alsa-tools alsa-tools-doc alsaconf

apk add pipewire pipewire-doc pipewire-pulse pipewire-alsa sndio sndio-doc

rc-service dbus restart

cat > /etc/security/limits.d/audio-limits.conf << EOF
@audio - memlock 256
@audio - nice -11
@audio - rtprio 88
EOF

apk add bluez bluez-openrc pipewire-spa-bluez

rc-update add bluetooth

rc-service bluetooth start
```

Now at this point we have audio and video configured, additionally we have 
basic support for managing devices dynamically when connecting or disconnecting, 
the only kind of devices not covered yet until this point are printers.

For any desktop this is the point where you choose which graphical environment 
or desktop to install. Next section will prefers the XFCE4 desktop setup, 
currently ws the first and its the most complete desktop packaged in the repos.

## instalacion Xfce4 Alpine

Since Alpine 3.13 the XFCE4 desktop its full modern, this means its pretty more heavy, 
cos is build using GTK3 only.

On older or cheap devices, mostly for 32bit devices its better to use alpine 3.10 or 3.12 
that uses GTK2 for almost all the programs.

1. install base desktop software packages
2. install base XFCE4 desktop packages, at this point, the system its ready for graphicall session
3. install and setup the login manager programs and policy manager software
4. setup the display login manager to start at boot
5. start the display manager, now you can use a minimal XFCE4 desktop, continue for complete experience
6. install cliboard manager, keyboard configurator, screensaver, screenshooter, and baterry manager
7. install packages for multi lang environment of the XFCE4 desktop if need, if login, logout and relogin

```
apk add gtk-update-icon-cache hicolor-icon-theme paper-gtk-theme adwaita-icon-theme mate-themes

apk add xfce4 xfce4-session xfce4-panel xfce4-terminal xarchiver mousepad network-manager-applet xfwm4-themes xfce-polkit xfce4-skel xfce4-power-manager xfce4-settings

apk add lightdm elogind elogind-openrc elogind-lang polkit polkit-openrc polkit-elogind lightdm lightdm-lang lightdm-gtk-greeter 

rc-update add lightdm

rc-service lightdm restart

apk add xfce4-clipman-plugin xfce4-xkb-plugin xfce4-screensaver xfce4-screenshooter xfce4-taskmanager xfce4-whiskermenu-plugin xfce4-battery-plugin

apk add xfce4-panel-lang xfce4-clipman-plugin-lang xfce4-xkb-plugin-lang xfce4-screenshooter-lang xfce4-taskmanager-lang xfce4-whiskermenu-plugin-lang xfce4-battery-plugin-lang xfce4-power-manager-lang xfce4-settings-lang
 
```

## Desktop multimedia and media devices


The media in linux its per se reduced, and in alpine so then more limited, 
with this lines you will have all the need suported, for converting and playing, 
and minimal support for editing.

1. install support for media management muti format over the programs
2. install support for sound and output of notifications over the programas
3. install support for media format reading and management backend (convertiion and reading)
4. install viewers for video and audio files
5. install management backend for media devices
6. install video resoulution flexibility and network backend support
7. install network frontend support and frontend management of network
8. install service task manager on boot
9. setup service of wifi backend management
10. setup general networking backend management service
11. setup all the users to property manage the audio, video and network devices
12. reconfigure the interfaces so the network gui can then manage it
13. start the network manager backend services, so the gui can manager then
14. at this point logout and relogin from your current sesion to start to work

```
apk add gst-plugins-base gst-plugins-bad gst-plugins-bad-lang gst-plugins-ugly gst-plugins-ugly-lang gst-plugins-good gst-plugins-good-gtk

apk add libcanberra-gtk2 libcanberra-gtk3 libcanberra-gstreamer wxgtk-media wxgtk3-media wxgtk-lang

apk add mediainfo ffmpeg ffmpeg-doc ffmpeg-libs lame lame-doc rtkit rtkit-doc 

apk add mpv mpv-doc deadbeef deadbeef-lang deadbeef-doc

apk add gvfs gvfs-fuse gvfs-archive gvfs-afp gvfs-afp gvfs-afc gvfs-cdda gvfs-gphoto2 gvfs-mtp pcmanfm

apk add libxinerama xrandr wpa_supplicant dhcpcd chrony macchanger wireless-tools iputils

apk add network-manager-applet network-manager-applet-lang networkmanager networkmanager-lang networkmanager-elogind networkmanager-elogind-lang networkmanager-elogind-openrc networkmanager-openvpn networkmanager-openvpn-lang

rc-update add chrony

rc-update add wpa_supplicant

rc-update add networkmanager

for u in $(ls /home); do for g in plugdev audio cdrom dialout video netdev; do addgroup $u $g; done;done

cat > /etc/network/interfaces << EOF
auto lo
iface lo inet loopback
EOF

service networking restart

service wpa_supplicant restart

service networkmanager restart

```

## Desktop Office suite

In linux world there's no mayor suite or programs in such topic, 
just we need a reader (pdf, ebooks, cbr, zbr, etc) and office 
suite for word/calc processing (doc, xls, odt, ods, etc).


```
apk add libreoffice libreoffice-gnome evince evince-lang evince-doc
```

The programs will appear in the Office menu category.

## development

This is only for those that dont want to download bunch of thing 
when install some programs from sources. In any cae, modding and plugin hacks 
will need this for minecraft or minetest hard hacker players.

The most important tool for development is the VCS:  version control system.
The most usefull too for GUI development is the IDE: intregrated development environment

From this, there's the RAD, means Rapid Application Development and the 
prefered way model for starting newbies.. 

#### base console development

1. install the minimal standard needs like make, cmake and gcc, pkg-config was superset by pkgconf
2. install set of tools to merge or compare files
3. install the most used version control service: git (GIT) VCS
4. install the second most used version control service optionally: subversion (SVN) VCS
5. install the so famous version control service: mercurial (HG) VCS
6. install session terminal handler, so can save terminal sessions

```
apk add pkgconf make cmake cmake-bash-completion gcc gcc-gdc gcc-go g++ gcc-objc gcc-doc

apk add patch patch-doc patchutils patchutils-doc diffutils diffutils-doc

apk add git git-bash-completion git-zsh-completion git-cvs git-svn github-cli git-diff-highlight git-doc

apk add subversion subversion-bash-completion subversion-zsh-completion subversion-yash-completion subversion-doc

apk add mercurial mercurial-bash-completion mercurial-zsh-completion mercurial-doc

apk add tmux screen
```

From this point you can work in console terminal all development task, 
of course, each development will need specific need, but this ones installed 
are always basic and mandatory (almost in all cases).

Next section, will cover some usefully artifact like an IDE (Intregrated Development Environment):

#### base gui development

The most compatible and all availabe IDE is geany, **this powerful tool 
is very poorly valued, since like everything in linux it must be configured 
according to your interest**, unlike other java or python craps that comes
already configured.

In such case **Geany is as close to the Unix spirit** it featured: overview, 
htmlpreview, compile, diff, CVS, colorpicker, htmlpicker, tabletools, codecleanup, 
and much more.

For terminal insteach of the usage of tmux or screen inside the already GUI, 
we recommended the yet ready for similar purposes Terminator.

```
apk add geany geany-plugins-lang geany-plugins-addons geany-plugins-geanyextrasel geany-plugins-overview geany-plugins-geanyvc geany-plugins-treebrowser geany-plugins-tableconvert geany-plugins-spellcheck geany-plugins-shiftcolumn geany-plugins-utils geany-lang meld meld-lang

apk add terminator terminator-lang
```

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
