# Manage your phone Android from Alpine Linux

You must take into consideration that there is really no standar way to replace 
the phone's operating system, since android 14 you cannot manage the filesystem 
no matter if you are rooted the device, the new filesystem is read only for.

**What can i manage?** you can use your device from your computer, using Alpine 
linux, there are two mayor tools for, starting by the popular **scrscpy** 
that is available since Alpine 3.16 https://pkgs.alpinelinux.org/packages?name=scrcpy&branch=v3.16

If you want more help directly [ask for help here](../README.md#help-online-directly).

## Introduction

ADB is part of the androit sdk tools but the package is named **android-tools** 
on alpine https://pkgs.alpinelinux.org/packages?name=android-tools&branch=v3.16 
and is available since 3.14 (but not scrscpy so on older versions only commands)

## Requirements

1. **IMPORTANT REQUIREMENT** your device must have Androit version 5.0 or superior
2. Enable the ADB mode as check [phones-androit-allow-adb-usb-developer-mode.md](phones-androit-allow-adb-usb-developer-mode.md)
3. Alpine version must be 3.14 as minimun, or 3.26 if you want to use remote screen

#### Limitations

* You will not have audio on older devices, audio will need Android 11+ on your device.

## Instalation of adb and scrscpy

```
doas su

apk add android-tools scrcpy
```

## Preparing adb to manage your phone

Now plugs the device to manage using USB cable, when plugs this will use 
the USB charge mode, so tab to the notifications upper place swipe down and 
check for USB connected device notifiacion.

When tap over USB connected device notificacion, the Android OS will bring to 
you the USB mode options, change from "only charging" mode to "MTP mode" (if 
this does not appears use the "File transfer mode" then)

> **Warning** a pop up notice will appears, acept and check the checkbox!

If you wants to do this procedure manually go to the **Settings** and navigate 
to the **Connected Devices** and go to **Connection preferences** or to 
the **Connected devices** and find the **USB conneciton mode**, the problem is 
that this screen is hidden on most devices.

> **Warning** if the pop up notice dialog does not appears go **Settings** 
and/or **Developer mode** and hit **Revoke USB debbuging authorizations**!

If you’re bringing up a port and you require ADB access before the UI is 
available (no dialog with allow USB key of computer), you can disable this 
protection by editing `/etc/default/adbd` and change `ADBD_SECURE` to 0, 
this only work in most recent version of ADB tool.

## Using adb to manage your phone

First list the devices, this is priority to connect if you have multiple devices

```
adb devices -l
```

This will output this:

```
general@venenux-alpine:~$ adb devices -l
List of devices attached
0123456789ABCDEF       device usb:1-2 product:4G model:4G device:lentk6735m_65u_l1 transport_id:7
ART3S487103378         device usb:1-1 product:ART_3S model:ART_3S device:ART_3S transport_id:5
```

The important part here is the serial number, the first column, is needed to 
next command to work that will invoke the only tool that linux user needs:

```
adb -s ART3S487103378 shell
```

This will change the console to the shell internal phone/tv console of android, 
the outpout will looks like that:

```
general@venenux-dell1:~$ adb -s ART3S487103378 shell
ART_3S:/ $ uname -a
Linux localhost 5.4.161-ab872 #1 SMP PREEMPT Tue Dec 6 18:00:08 CST 2022 aarch64
ART_3S:/ $ exit
general@venenux-dell1:~$
```

There are a huge amount of combinations on ADB commands, for remote management 
using graphical tools lest use **scrscpy** tool:

## Using the SCRSCPY tool

> **Warning** you must doo all the previous procedures in this document until this point

To start to use just with the device connected run:

```
ADB=/usr/bin/adb /usr/bin/scrcpy -s ART3S487103378 -b 4M --keyboard=sdk --mouse=sdk --video-codec=h264 --video-bit-rate=1M
```

Noted that the `-s ART3S487103378` its so important, of course we previously 
listed the phone/tv android devices list with `adb devices` to select the 
correct device by the serial string (first column).

## Set adb vendor keys on older ADB androids tools

This happened **when do not receive the dialog key confirmation on your device**, 
also when you got in log output this:

> error: device unauthorized.
> This adb server's $ADB_VENDOR_KEYS is not set

1. get sure to unconnect the phone device
2. now kill the existing adb server
3. remove the existing keys, **this will revocate already paired devices**
4. re run the adb server to generate new keys
5. kill/stop the new server to free key files

```
adb kill-server

doas killall adb

rm -v /home/*/.android/adbkey /home/*/.android/adbkey.pub

adb start-server

adb kill-server
```

Now in the device to connect, go to the **Settings** and navigate to 
the **Developer mode** and hit **Revoke USB debbuging authorizations**

With those steps are enough to remade the procedur and connect the device 
but if you still do not see the dialog key confirmation repeat and also:

At this point copy `.android/adbkey.pub` (public key) from your home 
to device to pair on path `/data/misc/adb/adb_keys` and get sure to 
set permissions to `766/-rwxrw-rw-` at least before/when copy to device.

Then reboot the phone/tv android device and check with `adb devices`.

> **Warning**: this procedur only works for one device with one computer.

If you want to add multiple PC as authorized to a single device, just 
repeat the first five steps on the new computer to being paired and then 
append the public keys to the `/data/misc/adb/adb_keys` of the device phone.

## Limitations and issues

Sometimes you will receive an error `error relocating ... symbol not found` when running any installed tool. 
This is because you installed some alpine packages too fresh but not upgraded dependences. 
A simple `apk upgrade -a` will fix it.

##  Refrence Sources

* [manage adb keys if no dialog](https://stackoverflow.com/questions/32132434/set-adb-vendor-keys/42052776#42052776)
* [manage your phone with archsit linux](https://arch-deb.blogspot.com/2025/07/conectar-el-telefono-o-smarttv-y.html)

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

For more information check the [../alpine/copyright.md](../alpine/copyright.md)

