Alpine Linux tutorias and howtos
=================================

**Where to start?** you will need check if you have enought Linux terminology knowledge, but 
_**if you are impatient and want an Alpine ready touse in 1 hour check **_ [.. this link: alpine XFCE desktop](alpine-tutorial-desktop-xfce4-fast-forward.md) 
if you want to avoid the explanations and just go all the way. For the wayland fashioned 
desktop try the guide [alpine tutorial desktop wayland](alpine-tutorial-desktop-wayland-try.md) 
but **Wayland is not recommended due several issues and X11 its more widelly working**.

## TUTORIALS AND HOWTOS

The **tutorials are hands-on docs** expected to try and achieve the goals described 
in each step, possibly with the help of a good example. The output in one step 
is the starting point for the following steps.

The **Howtos are smaller articles** explaining how to perform a particular task 
with Alpine Linux, that expects a minimal knowledge from reader to perform actions. 

The **Guides are fast documents** with only direct ways to do a particular task
with Alpine Linux, that expects a minimal knowledge from reader to perform actions. 

## INSTALL

* Common methods for computers:
    * [Install from CD to HDD/SDD PC single boot only](alpine-install-from-cd-to-disk-pc-single-boot-only.md)
    * [Install from USB to HDD/SDD PC single boot only](alpine-install-from-usb-to-disk-pc-single-boot-only.md)
* Serial console and sepcial devices:
    * [Install from USB to Serial PCENGINE APU single boot](alpine-install-from-cd-to-pcengine-apu-single-boot.md)
* Networking setup, wifi or PXE boots:
    * [alpine-tutorial-wifi-routering.md](alpine-tutorial-wifi-routering.md)
    * [servers-howto-setup-PXE-service-for-others-linuxes-ES.md](servers-howto-setup-PXE-service-for-others-linuxes-ES.md)

## PHONES

* Alpine in your phone: [alpine-tutorial-in-phones.md](alpine-tutorial-in-phones.md)
    * [phones-androit-allow-external-apps-install.md](phones-androit-allow-external-apps-install.md)

## DESKTOPS

* Complete desktops, means the programs are integrated and sync using XDG desktop compliant environment:
    * XFCE4 desktop guide: [alpine-tutorial-desktop-xfce4-fast-forward.md](alpine-tutorial-desktop-xfce4-fast-forward.md)
    * WAYLAND desktop guide: [alpine-tutorial-desktop-wayland-try.md](alpine-tutorial-desktop-wayland-try.md)
* Window managers means the desktop its not integrated, each program has their own environment but can tuned:
    * Openbox desktop guide: [alpine-tutorial-desktops-openbox-fast-forward.md](alpine-tutorial-desktops-openbox-fast-forward.md)
* Issues:
    * At the point of 2022/v3.16 **[all the gvfs handlers are broken in Alpine](https://gitlab.alpinelinux.org/alpine/aports/-/issues/14183)**
    * Alpine its not stable to use as desktop, developers [only solve issues to current or mayor version upgrades, not LTS due "community" nature of packages](https://gitlab.alpinelinux.org/alpine/aports/-/issues/14182#note_262134)
* Power management
    * APC UPS configuration [alpine-howto-apcupsd-service.md](alpine-howto-apcupsd-service.md)

## SERVERS

* Boot services
    * [servers-howto-setup-PXE-service-for-others-linuxes-ES.md](servers-howto-setup-PXE-service-for-others-linuxes-ES.md)
* WEB services
    * LAMP setup [server-alpine-LAMP-professional-fast-forward.md](server-alpine-LAMP-professional-fast-forward.md) (this one includes apache2+ssl+php+odbc+postgres+mysql)
    * Deploy gitea [alpine-howto-gitea-package.md](alpine-howto-gitea-package.md)
    * Rails app [alpine-how-to-rails.md (WIP)](tutorials/alpine-how-to-rails.md)
    * Filebrowser [alpine-howto-filebrowser-service.md](tutorials/alpine-howto-filebrowser-service.md)
* Power management
    * APC UPS configuration [alpine-howto-apcupsd-service.md](alpine-howto-apcupsd-service.md)

## Convention for naming the files:

1. each one must started with the word "alpine" unless are just a extra document like the androit external apps
2. must be followed "tutorial" or "guide" or "howto" 
3. rest of the name can be whatever and do not have spaces.. each word must be separated with "-"
4. the name must not contains any simbol or space, only numbers and letters and "-" are allowed
5. the extension of the file must be ".md"
6. you can upload extra support files like scripts shell or photos
7. the name of each extra files must be same as the document and extra word to idetify


## Acknowledges

**CONSIDERATIONS**: Please check the [Feature Differences OF ALPINE LINUX](../documents/README.md#feature-differences) 
but those are the main ones you should considering before usage of Alpine linux:

1. Alpine Linux **is designed for power users, with security and simplicity in mind**.
2. Alpine Linux **is not for newbies most knowed as new baby users or windosers**.
3. It is built around musl libc, not glibc, which means there might be incompatibilites with some packages
4. Its main utilities (coreutils) are derived from busybox and suckless, so all commands are pretty simplistic

**PREREQUISITES**: Please check the [alpine preparation requirements for you](alpine-newbie-prepare.md) 
but you can use linux using [termux](tutorial-alpine-in-phone.md) or [ish](tutorial-alpine-in-phone.md) on your phone.

* **Manners**, you will must have a vocation for reading and analysis.
* **Time**, you will need time, cos you will need to read a lot!
* **Computer**, you should **have a pc, or phone**, and must have a UNIX like operating system, like MAC/IOS or Androit/Linux
* **Internet**, and minimal bandwitch of internet network.. 

## Help online directly

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

