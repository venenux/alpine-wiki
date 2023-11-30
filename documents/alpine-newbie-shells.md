## alpine shell and terminal

> **Note**: You must have alpine already installed [alpine-newbie-install.md](alpine-newbie-install.md)

Short answer:

* **terminal** = text input/output environment, your computer is at same time a terminal
* **console** = physical terminal, anything you use as terminal but that does not means a complete computer
* **shell** = command line interpreter, is the software displayed inside a console terminal

In alpine the default terminal always will give you `ash` throught `busybox` pretty limited but functional enought.


## Default console shell

The operation system do not have a oficial interface, could be 
a console one or graphical one.

| Shell name | package(s) name                     | since | main program(S) |
| ---------- | ----------------------------------- | ----- | --------------- |
| bash       | bash bash-doc                       | 3.0   | `/bin/bash`, `/usr/lib/bash/` |
| dash       | dash dash-doc                       | 3.17  | `/bin/dash` |
| csh        | tcsh tcsh-doc                       | 3.4   | `/bin/csh`, `/bin/tcsh`, `/etc/tcsh.cshrc` |
| zsh        | zsh zsh-vcs zsh-zftp zsh-doc        | 3.4   | `/bin/zsh`, `/usr/lib/zsh/` |
| fish       | fish fish-tools fish-dev fish-doc   | 3.10  | `/usr/bin/fish`, `/etc/fish/config.fish`, `/usr/share/fish/` |
| nushell    | nushell nushell-plugins nushell-doc | edge  | `/usr/bin/nushell` |

#### change manually the default console shell

> **Note**: no shell id installed by default, only busybox for `ash`

* Install the dessired shell
* Check dessired shell is system valid
* seds file definition to Change default shell of user `general` to `bash`


```
apk add bash bash-doc sed

grep bash /etc/shells

sed -e '/general/ s#\:[^\:]*$#\:/bin/bash#g' /etc/passwd
```

> **Warning**: if the thirth step does not show something, you cannot use such shell (in this example `bash`)

This can be executed in one command 
as `apk add bash sed && sed -e '/general/ s#\:[^\:]*$#\:"$(grep bash /etc/shells)"#g' /etc/passwd`

#### Change managed the default shell with libuser

> **Note**: `libuser` is only since alpine v3.14

* Install the dessired shell
* Install the manager shell program
* Create need files
* Check dessired shell is system valid
* Change default shell of user `general` to `bash`

```
apk add bash bash-doc

apk add libuser libuser-doc

touch /etc/login.defs
mkdir /etc/default/ && touch /etc/default/useradd

grep bash /etc/shells

lchsh general
```

> **Warning**: if the thirth step does not show something, you cannot use such shell (in this example `bash`)

This program is interactive, will ask for the full path to the 
executable shell, and can be executed in one command 
as `apk add bash grep libuser && lchsh general`

#### Change managed the default shell with shadow

> **Note**: `shadow` in not installed by default, only busybox

* Install the dessired shell
* Install the manager shell program
* Create need files
* Check dessired shell is system valid
* Change default shell of user `general` to `bash`

```
apk add bash bash-doc

apk add shadow shadow-doc

touch /etc/login.defs
mkdir /etc/default/ && touch /etc/default/useradd

grep bash /etc/shells

chsh -s /bin/bash general
```

> **Warning**: if the thirth step does not show something, you cannot use such shell (in this example `bash`)

This can be executed in one command 
as `apk add bash grep shadow && chsh -s $(grep bash /etc/shells) general`

#### error of not valid shel or the shell program does not exist

This **was a big bug in alpine versions: https://gitlab.alpinelinux.org/alpine/aports/-/issues/11164**

In each of the options of this document whe put a stepp that said: "Check dessired shell is system valid", 
well each shell must be present in the file `/etc/shells` before apply.

> **Note**: If the shell is compiled manually and is not packaged, you can add into the file!

#### how to add a more recent version

This depends, check various examples:

* if you are using Alpine 3.8 and wants a more recent shell like `fish`, **you can!**
* if you are using Alpine 3.8 and wants a more recent shell like `nushell`: **you cannot cos is so far away from 3.8**
* if you are using Alpine 3.12 and wants a more recent shell like `fish`: **you can!**
* if you are using Alpine 3.17 or 3.18 and wants a more recent shell like `nushell`: **you can!**

This means you will:

* update current version repositories
* install required dependencies (check the pacakge name in pkgs.alpinelinux.org
* add the inmediate next version repository
* install the new version more up to date
* check for the required files
* change the shell for the desired user
* remove the inmediate extra repository, by setting the normal ones again
* test if work, if not you will need to upgrade to next alpine or use apk static files to repair world db

By example this show you how to install nushell into alpine 3.17 or 3.18 that is close to edge (as 2023):

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update && apk add shadow shadow-doc grep busybox-binsh libcrypto3 libgcc libssl3 sqlite-libs

cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
http://dl-cdn.alpinelinux.org/alpine/edge/main
http://dl-cdn.alpinelinux.org/alpine/edge/community
EOF

apk update && apk add nushell nushell-plugins nushell-doc

touch /etc/login.defs && mkdir /etc/default/ && touch /etc/default/useradd

chsh -s $(grep nushell /etc/shells) general

cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update
```

### About copyright material

**CC BY-NC-SA**: If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

Please check our [../alpine/copyright.md](../alpine/copyright.md).

