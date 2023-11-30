# alpine server apc UPS detecion service

`apcupsd` its a small program that communicates to the UPS devices, specifically the APC devices.

For more advenced check master-slave multi ups monitor and control powerloss network: [alpine-howto-acpupsd-service-multimonitor.md](alpine-howto-acpupsd-service-multimonitor.md)

## APC UPS single monitoring selft server

This tutorial will setup the server connected to and UPS APC, so 
will be selft monitoring their energy availability status.

This tutorial will permit that the machine selft shutdown after a power fail.

### preparation

* setup a hostname, here we use "develupscheck" string as hostname
* added and up to date normal repositories
* update repositories


```
hostname develupscheck

echo 'hostname="develupscheck"' > /etc/conf.d/hostname 

echo "develupscheck" > /etc/hostname

cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update
```

### Installation of apcupsd service

* install basic need packages and apcupsd modules
* add the service to the starup

```
apk add apcupsd eudev sed

rc-update add apcupsd
```

> **Warning**: the `eudev` package is highly necesry to autodetect the device, the `sed` is need becouse the extended regex


After installation:

* configuration files
    * `/etc/apcupsd.conf` the MAIN configuration file
    * `/etc/hosts.conf` other computers supported by the same UPS (NIS network driver for slaves)
    * `/etc/multimon.conf` parameters to display in web interface
* events files
    * `/etc/apccontrol` here you will find the various events
    * `/etc/changeme` sends email to change the UPS battery
    * `/etc/commfailure` sends email when connection with the UPS is lost
    * `/etc/commok` sends email when connection with the UPS is established
    * `/etc/offbattery` sends email when your computer runs on power
    * `/etc/onbattery` sends email when your computer runs on battery (UPS)
* programs files
    * `/sbin/apcaccess`
    * `/sbin/apctest`
    * `/sbin/apcupsd`
* service files
    * /etc/init.d/apcupsd` common daemon service to the APC ups software
    * /etc/init.d/apcupsd.powerfail` Same as Debian creates the symbolic link `/etc/init.d/ups-monitor` → /etc/apcupsd/ups-monitor. So, Debian halt script (/etc/init.d/halt) executes it. In this way, it kills power in the UPS with `apccontrol -killpower` after the shutdown of the system (if there is the file /etc/apcupsd/powerfail, which is then deleted)
* temporary files
    * `powerfail` “Flag file” created by apcupsd before shutdown the system, in order to inform the halt script that the shutdown is due to power outage (power fail)


### Configurations - basics

The apcupsd has internal defaults, the default values are for pure USB device types, 
that is USB cable only, there are some models that uses a internal serial by a RJ45 connector those 
can be manage also by USB or network, but you must take care of such configuration.

The most important part is the DEVICE/UPSTYPE combination, the most problematic are the SmartUPS 
with SERIAL or RJ45 connectors, specially the last one will need a special cable named 
as "smart signalling cable" that can be SERIAL: http://www.apcupsd.org/manual/manual.html#smart-custom-cable-for-smartupses 
or can be USBSERIAL: http://www.apcupsd.org/manual/manual.html#custom-rj45-smart-signalling-cable-for-backups-cs-models

Here the commands to configure the device using model ES 600M1 that is usb pure device:

* set UPSCABLE to the detected device, **Warning** beware of the previous paragraph advertisement
* set UPSTYPE to the configured device, **Warning** beware of the previous paragraph advertisement
* set POLLTIME to a minimun amout, so we will check more frecuently
* set ONBATTERYDELAY to a maximun of 12 so false positives will not shutdown the machine
* set BATTERYLEVEL to 40 percent so wel will shutdown the machine to do not take risk on empty source power
* set MINUTES to 10 as the same reason of the previous topic
* set ANNOY to 100 seconds we dont care the current logged users, the software and data inside is so important
* set ANNOYDELAY to 20 seconds so we will anunce to loged users about a fast shutdown
* enable the service after configured
* start (or restart) the service

```
sed -Ei "s|^[[:space:]]?UPSCABLE.*|UPSCABLE usb|g" /etc/apcupsd/apcupsd.conf

sed -Ei "s|^[[:space:]]?UPSTYPE.*|UPSTYPE usb|g" /etc/apcupsd/apcupsd.conf

sed -Ei "s|^[[:space:]]?POLLTIME.*|POLLTIME 40|g" /etc/apcupsd/apcupsd.conf

sed -Ei "s|^[[:space:]]?ONBATTERYDELAY.*|ONBATTERYDELAY 12|g" /etc/apcupsd/apcupsd.conf

sed -Ei "s|^[[:space:]]?BATTERYLEVEL.*|BATTERYLEVEL 40|g" /etc/apcupsd/apcupsd.conf

sed -Ei "s|^[[:space:]]?MINUTES.*|MINUTES 10|g" /etc/apcupsd/apcupsd.conf

sed -Ei "s|^[[:space:]]?ANNOY.*|ANNOY 100|g" /etc/apcupsd/apcupsd.conf

sed -Ei "s|^[[:space:]]?ANNOYDELAY.*|ANNOYDELAY 20|g" /etc/apcupsd/apcupsd.conf

rc-update add apcupsd

rc-service apcupsd restart
```

After configuraton and restarting the service you can test it by the `apcaccess` command line:

```
$ apcaccess 
APC      : 001,036,0856
DATE     : 2023-11-30 16:56:57 -0400  
HOSTNAME : develupscheck
VERSION  : 3.14.14 (31 May 2016) debian
UPSNAME  : develupscheck
CABLE    : USB Cable
DRIVER   : USB UPS Driver
UPSMODE  : Stand Alone
STARTTIME: 2023-11-30 16:56:55 -0400  
MODEL    : Back-UPS ES 600M1 
STATUS   : ONLINE 
LINEV    : 119.0 Volts
LOADPCT  : 7.0 Percent
BCHARGE  : 100.0 Percent
TIMELEFT : 93.3 Minutes
MBATTCHG : 5 Percent
MINTIMEL : 3 Minutes
MAXTIME  : 0 Seconds
SENSE    : Medium
LOTRANS  : 92.0 Volts
HITRANS  : 139.0 Volts
ALARMDEL : 30 Seconds
BATTV    : 13.5 Volts
LASTXFER : Low line voltage
NUMXFERS : 0
TONBATT  : 0 Seconds
CUMONBATT: 0 Seconds
XOFFBATT : N/A
SELFTEST : NO
STATFLAG : 0x05000008
SERIALNO : 4B2220P00773  
BATTDATE : 2022-05-17
NOMINV   : 120 Volts
NOMBATTV : 12.0 Volts
NOMPOWER : 330 Watts
FIRMWARE : 928.a9 .D U
END APC  : 2023-11-30 16:56:57 -0400 
```


#### Installation of apcupsd-webif network and web monitor servide

* install basic need packages and apcupsd web modules

```
apk add apcupsd apcupsd-webif sed lighttpd
```

> **Warning**: the apcupsd service must be configure, inclusivelly if the UPS is not in the machine connected

Due a bug in packaging becouse of lazy developers `apcupsd` must be previously configured https://t.me/alpine_linux_english/71210

### Configuration of apcupsd-webif - web service monitoring


Here the configuration to allow check from any host and enable network monitoring

* Enable the NETSERVER parameter
* Allow any host to check this UPS, to filter you must use the ip of the net card, does not support network segments
* The port must be the same of the cgi program if you want to use the cgi web monitor, dont change it from 3551!
* enable alias, cgi, accesslog, dirlisting module in lighttpd
* create the cgi directory, the localhost www directory and cache directory
* mount the shared cgi program
* set property permissions
* set service automatically
* mount directory for the cgi security using aliasing (do not copy in, please be profesional)
* restart services


```
sed -Ei "s|^[[:space:]]?NETSERVER.*|NETSERVER on|g" /etc/apcupsd/apcupsd.conf

sed -Ei "s|^[[:space:]]?NISIP.*|NISIP 0.0.0.0|g" /etc/apcupsd/apcupsd.conf

sed -Ei "s|^[[:space:]]?NISPORT.*|NISPORT 3551|g" /etc/apcupsd/apcupsd.conf

sed -i -r 's#\#.*mod_alias.*,.*#    "mod_alias",#g' /etc/lighttpd/lighttpd.conf
sed -i -r 's#.*include "mod_cgi.conf".*#   include "mod_cgi.conf"#g' /etc/lighttpd/lighttpd.conf
sed -i -r 's#\#.*mod_accesslog.*,.*#    "mod_accesslog",#g' /etc/lighttpd/lighttpd.conf
sed -i -r 's#\#.*dir-listing.activate*=.*,.*#dir-listing.activate   = "enable"#g' /etc/lighttpd/lighttpd.conf

mkdir -p /var/www/localhost/cgi-bin/apcupsd && mkdir -p /var/www/localhost/htdocs && mkdir -p /var/lib/lighttpd

chown -R lighttpd:lighttpd /var/www/localhost/ && chown -R lighttpd:lighttpd /var/lib/lighttpd && chown -R lighttpd:lighttpd /var/log/lighttpd

mount /usr/share/webapps/apcupsd /var/www/localhost/cgi-bin/apcupsd --bind

rc-update add lighttpd default

rc-service lighttpd restart
```

After that you can visit the URI http://localhost/cgi-bin/acpupsd/multimon.cgi to check the web monitor service


#### Configurations - advanced with events and scripts

To configure custom event you must see for `apccontrol`, which apcupsd can manage (power outage etc.); 
if you want o make changes, DO NOT modify this file directly, but create a script with 
the name of an event and place it in `/etc/apcupsd` (eg a custom script `/etc/apcupsd/doshutdown` 
will be executed first when the `doshutdown` event enabled inside the `apccontrol` definition).


## see also

* Apline master-slave multi ups monitor and control powerloss network: [alpine-howto-acpupsd-service-multimonitor.md](alpine-howto-acpupsd-service-multimonitor.md)
* Recalibrating UPS for exhausted batteries http://www.apcupsd.org/manual/manual.html#recalibrating-the-ups-runtime

# LICENSE

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
