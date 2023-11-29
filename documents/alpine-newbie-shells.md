
you must have already alpine installed:

## Default console shell

The operation system do not have a oficial interface, could be 
a console one or graphical one.

| Shell name | package name  | since | main program(S) |
| ---------- | ------------- | ----- | --------------- |
| bash       | bash          | 3.0   | `/bin/bash`, `/usr/lib/bash/` |
| dash       | dash          | 3.17  | `/bin/dash` |
| csh        | tcsh          | 3.4   | `/bin/csh`, `/bin/tcsh`, `/etc/tcsh.cshrc` |
| zsh        | zsh           | 3.4   | `/bin/zsh`, `/usr/lib/zsh/` |
| fish       | fish          | 3.10  | `/usr/bin/fish`, `/etc/fish/config.fish`, `/usr/share/fish/` |

### change manually the default console shell

* Install the dessird shell
* Check dessired shell is present
* seds file definition to Change default shell of user `general` to `bash`


```
apk add bash bash-doc sed

grep bash /etc/shells

sed -e '/general/ s#\:[^\:]*$#\:/bin/bash#g' /etc/passwd
```

> Warning: if the thirth step does not show something, you cannot use such shell (in this example `bash`)

This can be executed in one command 
as `apk add bash sed && sed -e '/general/ s#\:[^\:]*$#\:"$(grep bash /etc/shells)"#g' /etc/passwd`

### Change managed the default shell with libuser

> Warning: `libuser` in only since alpine v3.14

* Install the dessired shell
* Install the manager shell program
* Create need files
* Check dessired shell is present
* Change default shell of user `general` to `bash`

```
apk add bash bash-doc

apk add libuser libuser-doc

touch /etc/login.defs
mkdir /etc/default/ && touch /etc/default/useradd

grep bash /etc/shells

lchsh general
```

> Warning: if the thirth step does not show something, you cannot use such shell (in this example `bash`)

This program is interactive, will ask for the full path to the 
executable shell, and can be executed in one command 
as `apk add bash grep libuser && lchsh general`

### Change managed the default shell with shadow

> Warning: `shadow` in not installed by default, only busybox

* Install the dessired shell
* Install the manager shell program
* Create need files
* Check dessired shell is present
* Change default shell of user `general` to `bash`

```
apk add bash bash-doc

apk add shadow shadow-doc

touch /etc/login.defs
mkdir /etc/default/ && touch /etc/default/useradd

grep bash /etc/shells

chsh -s /bin/bash general
```

> Warning: if the thirth step does not show something, you cannot use such shell (in this example `bash`)

This can be executed in one command 
as `apk add bash grep shadow && chsh -s $(grep bash /etc/shells) general`

### error of not valid shel or the shell program does not exist

This was a bg in older alpine versions: https://gitlab.alpinelinux.org/alpine/aports/-/issues/11164

### About copyright material

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

Please check our [../alpine/copyright.md](../alpine/copyright.md).

