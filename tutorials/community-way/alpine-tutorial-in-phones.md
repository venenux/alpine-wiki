# Installing ALPINE LINUX on PHONEs

You must take into consideration that there is really no standar way to replace 
the phone's operating system, what is really done is to use the base of 
the operating system and use a parallel operating system 
(this in principle to explain it more simply).

**What happened then?** you can boot inside yor phone (whitout remove the phone's OS), 
the Alpine operating system, just like a kind of virtual machine but its not!

If you want more help directly [ask for help here](../README.md#help-online-directly).

## Introduction

TERMUX or ISH are applications providing linux like environment.

Using Alpine-linux is beneficial for programmers,computer science students, 

## Requirements

1. **IMPORTANT REQUIREMENT** Let's remember that you must have a minimum understanding 
of Linux terminology, and this does not mean that you know it by using Mint or Ubuntu or WSL2.
2. Download a terminal subsystem, in the following table we will provide the respectives 
in each phone cases and feature requirements:

| OS      | Version | Software to download | Minimal storage to use     | Quick link to download                                |
| ------- | ------- | -------------------- | -------------------------- | ----------------------------------------------------- |
| iOS     | 12.0+   | Download iSH Shell   | 100Mb (its custom repo)    | https://apps.apple.com/us/app/ish-shell/id1436902243  |
| Androit | 7.0+    | Download Termux      | 240Mb (depends of version) |  https://f-droid.org/repo/com.termux_118.apk           |

#### Issues when find apps for the phones

* Its best to "Copy-paste" all commands in `Termux` or `iSH` to avoid errors.
* Its pretty important to check the [Limitations and issues](#limitations-and-issues) section before assume anything.
* On iOS maybe you must search over App store for the keyword "iSH" if the links provided does not work.
* On Androit maybe you must enable the **[Allow App Installations from Unknown Sources](phones-androit-allow-external-apps-install.md)**

## Instalation

#### On iOS and iSH

The `iSH` uses the Alpine already embebed, so you may have to run apk update to fetch the Alpine repository list.

The `iSH` has its own repositories so the app is entirely self-contained and iSH with apk can pass app review. 
The repositories are a pseudo apk filesystem mounted on `/ish/apk` that when read, will actually download 
from App Store as on-demand resources. It also means that Apple can review all packages in iSH's repositories.

* open `iSH` app
* After open it, alpine its already started and runing in a terminal mode
* As optional get sure to get internet network by run `echo "nameserver 8.8.8.8" > /etc/resolv.conf`

#### On Androit and Termux

To install Alpine-Linux:

* open `Termux` app 
* run following command:

```
pkg install git

curl -LO https://raw.githubusercontent.com/Hax4us/TermuxAlpine/master/TermuxAlpine.sh && bash TermuxAlpine.sh

startalpine
```

* To logout/exit Alpine-Linux, just run `exit` until ends
* To start Alpine-Linux again open `Termux` and run `startalpine`
* As optional get sure to get internet network by run `echo "nameserver 8.8.8.8" > /etc/resolv.conf`


## Limitations and issues

Sometimes you will receive an error `error relocating ... symbol not found` when running any installed tool. 
This is because you installed some alpine packages too fresh but not upgraded dependences. 
A simple `apk upgrade -a` will fix it.

**IMPORTANT** to know:

* For `Androit` and `Termux` you must read: https://wiki.termux.com/wiki/Differences_from_Linux
* For `iOS` and `iSH` you must read: https://github.com/ish-app/ish/wiki/What-works%3F

##  Refrence Sources

* [iSH Github](https://github.com/ish-app/ish)
* [Termux Github](https://github.com/termux)  
* [TermuxAlpine Github](https://github.com/Hax4us/TermuxAlpine)

## See also

* [Ask for help here](../README.md#help-online-directly)
* [About Alpine linux](../../alpine/about.md)

# LICENSE

**CC BY-NC-SA**: this project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [../../alpine/copyright.md](../../alpine/copyright.md)

