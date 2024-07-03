# Alpine development guidelines

This is only for those that dont want to download bunch of things and needs a ready to use setup for any case!

The most important tool for development is the VCS (version control system).
The most usefull too for GUI development is the IDE (intregrated development environment)

From this, there's the RAD, means Rapid Application Development and the 
prefered way model for starting newbies.. 

In any case the base for development will be need, that means:

1. base compilers and libs (GCC, LLVM, make, cmake, etc)
2. base editors (nano, hexedit)
3. standart libs/tools for other development env! (Docker, qemu etc)
4. base GUI components ( IDE, RAD, qemu, etc)

You must have a [valid desktop configured  like alpine-newbie-xfce-desktop.md](alpine-newbie-xfce-desktop.md) or some of the others in tutorials!

## Base development setup

The base development only install console only packages and setup user to do things without gui, IDE or RAD, just to being able to reproduce development environments!

PRE-REQUISITE: [valid desktop configured  like alpine-newbie-xfce-desktop.md](alpine-newbie-xfce-desktop.md) or some of the others in tutorials!

#### base development packages to install

1. install base compilers, tools and libs like make, cmake, pkg-config was superset by pkgconf
2. install tools to build of basic programs and packages
3. install set of tools to merge, compare, and manage files
4. install the most used version control service: git (GIT) VCS
5. install the second most used version control service optionally: subversion (SVN) VCS
6. install the so famous version control service: mercurial (HG) VCS
7. install session terminal handler, so can save terminal sessions
8. install doas and sudo alternative for doas privilegie scalation!

```
apk add pkgconf make make-doc cmake cmake-doc cmake-bash-completion build-base arch-install-scripts abuild abuild-rootbld abuild-doc

apk add gcc gcc-gdc gcc-go g++ gcc-objc gcc-doc clang18 clang18-dev

apk add patch patch-doc patchutils patchutils-doc diffutils diffutils-doc zip zip-doc p7zip p7zip-doc xz xz-doc tar tar-doc file file-doc  sed sed-doc lsof lsof-doc less less-doc groff groff-doc gawk gawk-doc

apk add git git-bash-completion git-zsh-completion git-cvs git-svn github-cli git-diff-highlight git-doc

apk add subversion subversion-bash-completion subversion-zsh-completion subversion-yash-completion subversion-doc

apk add mercurial mercurial-bash-completion mercurial-zsh-completion mercurial-doc

apk add tmux screen byobu font-terminus

apk add doas doas-sudo-shim
```

> **Warning**: `font-terminus` is ony since alpine v3.18, for older versions use `terminus-font`

From this point you can work in console terminal all development task, 
of course, each development will need specific need, but this ones installed 
are always basic and mandatory (almost in all cases).

This firts set of package are only for console or most standart only.. after setup the user environment you can go forward to more GUI related setup.

#### user development setup environment

USer must be in some special groups what will be only available if you follow this guide from beggining!

1. Create the `general` user, please dont bypass this!
2. Add the users to the special new groups for development
3. Include the user `general` in the doas list for privilegie scalation!
4. Create the development directory, you must use only this place!
5. configure console fonts for space allowed TTY!

```bash
useradd -m -U -c "" -G abuild,wheel,input,disk,floppy,cdrom,dialout,audio,video,lp,netdev,games,users general

for u in $(ls /home); do for g in disk abuild wheel adm netdev kvm; do addgroup $u $g; done;done

cat >  /etc/doas.d/general.conf << EOF
permit keepenv :general
EOF

mkdir -m 775 -p /home/general/Devel
chown -R general:abuild /home/general/Devel

setfont /usr/share/consolefonts/ter-132n.psf.gz
sed -i "s#.*consolefont.*=.*#consolefont="ter-132n.psf.gz"#g" /etc/conf.d/consolefont
rc-update add consolefont boot
```

> **Warning**: those commands will not work if you dont install packages in first section above!

Only the user general is allowed to run root commans, the `doas-sudo-shim` packages allows to do that with any "sudo" or "doas" no matter you use for!

#### Login and setup new user environment

This must be run after install console tools and before setup gui development setup, user will setup the variables of environment:

1. Create the development directory
2. setup the variables to clone and use git command so free
3. create and config the user email for git usage
4. create and config the user name for git usage
5. create and config the user email for abuild
6. create and config the user name for abuild
7. generate a keygen for abuild

```
mkdir -m 775 -p /home/general/Devel
chown general:abuild /home/general/Devel

git config --global pull.rebase=true
git config --global ssh.postBuffer 2000000000
git config --global http.postBuffer 2000000000
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
git config --global https.postBuffer 2000000000
git config --global https.lowSpeedLimit 0
git config --global https.lowSpeedTime 999999

git config --global user.email "general@venenux.xxx"

git config --global user.name "generalvenenux"

doas sed -i 's|.*PACKAGER\s*=.*|PACKAGER="generalvenenux <general@venenux.xxx>"|g' /etc/abuild.conf

doas sed -i 's|.*MAINTAINER\s*=.*|MAINTAINER="\$PACKAGER"|g' /etc/abuild.conf

abuild-keygen -a -i -n
```

## base GUI development

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
