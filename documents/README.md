Alpine Linux newbie category
============

This category section will try to guide new ones with enought knowledge of Linux.

1. Alpine Linux **is designed for power users, with security and simplicity in mind**.
2. Alpine Linux **is not for newbies most knowed as new baby users or windosers**.

Alpine is the [OS (Operating System)](linux-alpine-terminology.md#o), that runs on your **machine**. 
**Programs** such as a web browser **runs** on the **OS**, and web pages like "wiki.alpinelinux.org" 
are handled by the web browser.

Alpine is the [Program combo](linux-alpine-terminology.md#p) that really runs in **dockers**, 
due to the great popularity of Docker, Alpine Linux is one of the most deployed operating systems 
currently in use, because within every Docker deploy the docker image it uses is almost always Alpine Linux.

Linux is just the **kernel** that handles and manages the **hardware** to the operating system.

* [Feature Differences](#feature-differences)
* [Installation](#installation)
    * [alpine-newbie-xfce-desktop.md](alpine-newbie-xfce-desktop.md)
* [Post install and Software Packages](#post-install-and-software-packages)
    * [About the repositories of programs](#about-the-repositories-of-programs)
    * [APK and package formats](#apk-and-package-formats)
    * [Developer](#developer)
        * [Developers: compilers, IDE's and tools](#developer)
    * [Servers: deploy in production](#servers--deploy-in-production)
        * [fail2ban: server protection](server-alpine-fail2ban-professional.md)
        * [MySQL/Mariadb professional](server-alpine-mysql-professional.md)
* [Community and contact](#community)

## Feature Differences

Alpine Linux is different from most other Linux distros in a few ways:

* It is built around musl libc, not glibc, which means there might be incompatibilites with some packages
* Its main utilities (coreutils) are derived from busybox and suckless, but GNU coreutils can be installed
* It uses a hardened Linux kernel by default, but releases offers LTS Linux kernel packages
* It compiles all userspace binaries as position-independent executables with stack-smashing protection

In next sections you will find general information necessary to start in the Alpine powered world for a new user.

| Link                                              | Purpose of the section                      |
| ------------------------------------------------- | ------------------------------------------- |
| [FAQ](FAQ.md)                                     | Some Frequently Asked Questions that might be useful to you |
| [Alpine Newbie Prepare](alpine-newbie-prepare.md) | Some post installation steps you might want to take |
| [Alpine Newbie Install](alpine-newbie-install.md) | Those writings are more focused on "Follow these steps blindly" for beginners, those pages are specific general cases, by example on virtual-box ones. |
| [Alpine Newbie Configs](alpine-newbie-configs.md) | Some post installation steps you might want to take |
| [Alpine newbie Desktop](alpine-newbie-desktop.md) | As a minimal distribution, Alpine follows the rule of "upstream provided", this means that Alpine doesn't ship with any graphical environments neither specific integrated configurations for. So means, but, you can installed some Desktops and Window Managers but must configured by yourself. |
| [Alpine newbie Develop](alpine-newbie-develop.md) | Alpine development stack: Alpine Linux is the most used Linux for deploying software, making it a good choice if you are a developer. |

## Installation

Alpine newbie install wiki pages are **focused on the idea to cover popular quick cases** 
that only offer the ready-to-use installation, **that is, cases where only alpine will be 
the OS to install**, in order to understand it faster, once understood, you can play and 
deep more granular over your preferred install.

For more granular or more specific cases you will have to read the hand book installation wiki page 
at [../alpine/Installation.md](../alpine/Installation.md). Its more recommended you first 
use an alpine virtual-box install and understand the system before try more deep install process, 
that is why we offered in newbie category the [Alpine Newbie Install](alpine-newbie-install.md) pages.

1. [Alpine Newbie Prepare](alpine-newbie-prepare.md)
2. [Alpine Newbie Install](alpine-newbie-install.md)
    * [alpine-newbie-shells.md](alpine-newbie-shells.md)
3. [alpine-newbie-xfce-desktop.md](alpine-newbie-xfce-desktop.md)

## Post install and Software Packages

The **programs**, the software installed to Alpine comes from two places: 
**repositories** (those managed by Alpine) 
and original **upstream sources** (those compiled as Unix-like traditional way).

If you are so hurry just read and go to the [Alpine Newbie Configs](alpine-newbie-configs.md) page, 
but here we will provide a minimal information about the nature of the post install artifacts.

#### About the repositories of programs

Alpine software repositories are managed by the repositories and uses packages. 
Each Alpine release have two branch of repositories. The **/community** repository 
of each Alpine release contains community supported packages that were accepted 
from the **/testing** repository. Only **/main** repository of each version of Alpine 
release are supported for Main Alpine Developers and Man Powers and received 
official support by almost few years until new releases happened.

* **About the main packages**: Main packages are the Alpine package software that have 
direct support and updates from the Alpine core and main team, also have official 
special documentation. Are always available for all releases and will have almost 
substitutions if some are not continued from upstream. Commonly those packages 
are selected due their responsibility and stability respect upstream availability. 
Those are in testing and when performs well or are mature goes to main branch.
* **About the contribution ones**: User package contribution repositories are 
those made by users in team with the official developers and well near integrated 
to the Alpine packages. Those have supported by those user contributions and could 
end if the user also ends respect with Alpine work, by example may not have 
substitution in next release due lack of support by upstream author or Alpine mantainer. 
Those packages mostly comes from testing and are included when are accepted.
* **About the testing ones**: New packages or new versions come into testing repositories 
of edge Alpine version and are those made by any contributor or man power 
on Alpine, the edge is unstable current development, this branch of repository 
has no release linked or related of Alpine. Those are in testing and when accepted 
goes to community.

Please check [Alpine Newbie Configs](alpine-newbie-configs.md) for guidelines

#### APK and package formats

Software packages for Alpine Linux are digitally signed tar.gz archives containing 
programs, configuration files, and dependency metadata. They have the extension .apk 
(yes, please don't confused with Androit ones), and are often called "a-packs".

Are managed with the apk command, located at `/sbin/apk`, it uses `/etc/apk/` place for 
the configurations files, and stores all downloaded "a-packs" files in `/etc/apk/cache` 
from the repositories before unpacks and put the package files compiled into the installed system.

As new user those technical tips are not necessary now, you can read the [Alpine Newbie apk packages](alpine-newbie-apk-packages.md) 
page to just read all about packages install only.. for more deep in advanced: 
those technical topic are in the apk wiki official page.

#### Developer

In earlier days, Alpine used a separate Gentoo build environment. 
Nowadays we can build in Alpine environment itself.

There's many kind of developers.. more are Distro targeted (like Alpine package development), 
others web oriented as Front-end web development or a Back-End Web Developer 
(like webpage design or applications services) , and others DevOps 
(backend programming and/or software development (Dev) and information-technology operations (Ops))

In DevOps and/or Web Development, Whatever will be, installation 
of the devel tools are the next step: the alpine-sdk is a metapackage 
that pulls in the most essential packages used to development environments; 
Also the crosstool-ng if you will setup different architectures or cross-compiling.

For development of packages.. there's two branchs: using the Alpine edge branch 
or using the Alpine stable, the only difference are the target, edge used the most up 
to date but not well tested software packages and the results are for the next Alpine releases. 
The recommendation to develop packages for newbies, are using Alpine Linux in a chroot, 
later when users got more experience.. must move to others way to develop packages.. at 
the actual status of a newbie it's the most easy and faster way.

All the Development process are detailed for newbie users in the Alpine newbie developer page.

##### Developers: compilers, IDE's and tools

1. [Alpine newbie Console](alpine-newbie-console.md)
    * [alpine-newbie-shells.md](alpine-newbie-shells.md)
2. [Alpine newbie Desktop](alpine-newbie-desktop.md)
3. [Alpine newbie Develop](alpine-newbie-develop.md)
4. [Alpine newbie developer: full stack web](alpine-newbie-develop-full-stack-web.md)
5. [Alpine newbie developer: backend devel](alpine-newbie-develop-backend-developer.md)

##### Servers: deploy in production

1. [Alpine newbie Console](alpine-newbie-console.md)
2. [Alpine production deploy](alpine-production-deploy.md)
3. [Alpine advanced hugepages](alpine-newbie-hugepages.md)
4. [Alpine newbie qemu virtualization](alpine-newbie-qemu-virtualization.md)

* Special documentation:
    * How to protect professinally servers [server-alpine-fail2ban-professional.md](server-alpine-fail2ban-professional.md)
    * How to setup apache professional at [server-alpine-apache2-professional.md](server-alpine-apache2-professional.md)
    * How to setup gitea professional at [server-alpine-gitea-professional.md](server-alpine-gitea-professional.md)
    * How to setup MySQL/MariaDB professional at [server-alpine-mysql-professional.md](server-alpine-mysql-professional.md)
    * Implementation of a certificate [guide-only-dehydrated.md](guide-only-dehydrated.md)
    * Configuring the hugepage sizes at [alpine-newbie-hugepages.md](alpine-newbie-hugepages.md)
    * Virtualization [Alpine newbie qemu virtualization](alpine-newbie-qemu-virtualization.md)
* Specific doucmentation cases:
    * ARM tvbox TODO
    * Alpine huge pages and applications enabled for [alpine-newbie-hugepages.md](alpine-newbie-hugepages.md)

## Community

We know that new users are not so xperts, so then here are the networks you can check for newbies:

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
- ⛓ Matrix
  - 👥 https://matrix.to/#/#alpine-linux-english:matrix.org

# LICENSE

**CC BY-NC-SA**: this project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [../alpine/copyright.md](../alpine/copyright.md)

