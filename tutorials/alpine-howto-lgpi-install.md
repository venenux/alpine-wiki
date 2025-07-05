# alpine + lgpi

LGPI is the Information Resource-Manager with an additional GUI for admin task.
You can use it to build up a database with an inventory for your company by 
example computer, software, printers, etc... It has enhanced functions to make 
the daily life for the administrators easier, like a job-tracking-system with 
mail-notification and methods to build a database with basic information about 
your network-topology.

1. the precise inventory of all the technical resources stored in a database.
2. management and the history of the maintenance actions and bound procedures

This application is dynamic and is directly connected to the users who can post 
requests to the technicians. An interface thus authorizes the latter with if 
required preventing the service of maintenance and indexing a problem encountered 
with one of the technical resources to which they have access.

The installation here doesn't involve stealing the entire domain, is in 
flexible mode. Instead of taking over the entire root, it only uses a subpath 
of the server URL (i.e., in a subdirectory). Additionally, it uses an alias, 
so it's not placed in the web root directory, but in the Alpine webapps, 
as it should be.

This material is copyright, check [LICENSE](#license) at the end of the document!
and you can also watch the mckaygerhard's video also at https://t.me/alpine_linux/1402

* [Install alpine linux](#install-alpine-linux)
* [1 - Environment](#1---setup-environment)
* [2 - Download and setup webmin](#2---download-and-setup-webmin)
* [3 - Configuration for modules](#3---configuration-for-modules)
  * [Problems and fails](#problems-and-fails)
* [4 - Full automated way](#4---full-automated-way)
* [5 - basic webmin modules]()
* [How to use this guide](#how-to-use-this-guide)
* [LICENSE](#LICENSE)

This document will not explain anything; you must to obey, as must be cos just works 
and works very well, please if you dont know check [How to use this guide](#how-to-use-this-guide) 
section before starts:

## Install alpine linux

> **Warning**: if you already have alpine running just foward to [0 - Environment](#0---setup-environment) part!

Those commands are for any distro, it will create a disk and runs a virtual 
machine to install Alpine Linux 3.20 as base system OS for webmin setup.

You can use alpine 3.20, to 3.22, or edge... any alpine will work since 3.20
to install GLPI system. In this part we use 3.22 but any other version still work!

> **Warning** will need to change the php command name package at older versions

```
mkdir -p /home/general/VM/alpine322 && cd /home/general/VM/alpine322

qemu-img create -f raw computerint1alpine-vitualdisk1-file.raw 6G

wget -c -t8 --no-check-certificate http://dl-cdn.alpinelinux.org/alpine/v3.22/releases/x86_64/alpine-extended-3.22.0-x86_64.iso

/usr/bin/qemu-system-x86_64  -m 2048 -name "computerint1alpine322" \
 -cpu host -machine q35 \
 -device virtio-net,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3222-:22,hostfwd=tcp::9080-:80,hostfwd=tcp::9443-:443 \
 -device virtio-keyboard -device virtio-mouse -device virtio-tablet -device virtio-vga,max_outputs=1 \
 -drive file=computerint1alpine-vitualdisk1-file.raw,format=raw \
 -cdrom alpine-extended-3.22.0-x86_64.iso -boot order=cd,once=d,menu=off
```

* When start it, will ask for root just write "root" and enter to start the command `setup-alpine`

#### the setup-alpine command procedure

* keyboard and variant, example for Latin is es and after then es-winkeys
* hostname: just hit enter, it's the name of the computer, must be only strings.
* Network: select the eth0 one that is the network cable and answer dhcp.
* Network (again): only happends if you have wifi or second card.. must ignore it
* DNS Options: It is recommended to use 8.8.8.8 and none for the domain
* Root: password for the administrative account, take care and dont forgive it
* Timezone: use UTC only for one OS, otherwise America/Panama or something similar
* Proxy Options: Use none if you are connecting directly to the Internet.
* NTP Options: Use chrony the packet already in the medium (extended).
* APK mirror: if you are over slow or no interent, type Skip or none
* User: modern alpine releases allows user creation, skip by typing no
* SSH Options: Use openssh the package that already comes in the medium (extended).
* Root allow: here you must type yes because we do not setup user yet!
* SSH key: just type here none
* Disk Options: Use sda as the entire hard drive present will be used.
* Mode: Select sys to install the system on disk.

Then reboot and take care if you are not using virtual machine of ip address

## 1 - Setup environment

Do not miss or bypass any package or command, or glpi will silend fails, 
this first part will setup main and community repositories, later install 
dependencies for the installation and also core running GLPI software, 
and at last will setup console font, please if you dont know check [How to use this guide](#how-to-use-this-guide) 
section, otherwise runs all of the following commands as root user:

```
cat > /etc/apk/repositories << EOF
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add openssl perl perl-net-ssleay perl-io-socket-ssl perl-io-tty \
 perl-datetime perl-datetime-timezone perl-datetime-locale attr diffutils \
 at dcron man-pages nano binutils coreutils readline shared-mime-info \
 grep gawk sed attr dialog lsof less groff procps wget curl terminus-font \
 file findutils gawk tree pciutils usbutils lshw tzdata tzdata-utils \
 zip unzip p7zip xz tar cabextract cpio binutils lha gzip lz4 \
 ethtool musl-locales musl-locales-lang  arch-install-scripts util-linux \
 docs iproute2-minimal psmisc net-tools lsof curl wget apkbuild-cpan

rc-update add consolefont boot


cat > /etc/hosts << EOF
127.0.0.1	provision.domino.ver provision localhost.localdomain localhost
::1		localhost localhost.localdomain
EOF

echo "provision.domino.ver"
```

> **Warning**: execute all the commands before, if you dont know how to, check [How to use this guide](#how-to-use-this-guide)

## 2 - Install prerequisites

GLPI is a Web application that will need: a webserver, PHP and a database. 
All of these so continue as root user and runs all the following commands:

```
apk add sed apache2 apache2-utils apache2-error apache2-ssl apache2-ctl

mkdir -p /var/www/localhost/htdocs /var/log/apache2
sed -i -r 's#^Listen.*#Listen 80#g' /etc/apache2/httpd.conf
sed -i -r 's#^ServerTokens.*#ServerTokens Minimal#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_alias.so.*#LoadModule alias_module modules/mod_alias.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_rewrite.so.*#LoadModule rewrite_module modules/mod_rewrite.so#g' /etc/apache2/httpd.conf
chown -R apache:www-data /var/www/localhost/
chown -R apache:wheel /var/log/apache2
echo "it works" > /var/www/localhost/htdocs/index.html
rc-update add apache2 default

IP=$(ip a s | grep 'inet ' | grep -Eo '([0-9]*\.){3}[0-9]*' | grep -v '127.0' | head -n1)
HN=$(hostname)
echo "$IP $HN" >> /etc/hosts

rc-service apache2 restart
```

Now setups php

```
apk add php82-opcache php82-openssl php82-json php82-bcmath php82-mbstring \
 php82-bz2 php82-zip php82-tidy php82-calendar php82-intl php82-pspell \
 php82-ctype php82-dev php82-dom php82-enchant php82-fileinfo php82-shmop \
 php82-simplexml php82-dom php82-sysvmsg php82-sysvsem php82-sysvshm \
 php82-tokenizer php82-xml php82-xmlreader php82-iconv \
 php82-xmlwriter php82-xsl php82-xmlwriter php82-sodium \
 php82-exif php82-gd php82-pcntl php82-mysqli php82-pdo php82-pdo_mysql \
 php82-sockets php82-curl php82-pear php82-phar php82-session \
 php82 php82-apache2 

sed -i -r 's|.*cgi.fix_pathinfo=.*|cgi.fix_pathinfo=1|g' /etc/php*/php.ini
sed -i -r 's#.*safe_mode =.*#safe_mode = Off#g' /etc/php*/php.ini
sed -i -r 's#.*expose_php =.*#expose_php = Off#g' /etc/php*/php.ini
sed -i -r 's#memory_limit =.*#memory_limit = 536M#g' /etc/php*/php.ini
sed -i -r 's#upload_max_filesize =.*#upload_max_filesize = 128M#g' /etc/php*/php.ini
sed -i -r 's#post_max_size =.*#post_max_size = 256M#g' /etc/php*/php.ini
sed -i -r 's#^file_uploads =.*#file_uploads = On#g' /etc/php*/php.ini
sed -i -r 's#^max_file_uploads =.*#max_file_uploads = 12#g' /etc/php*/php.ini
sed -i -r 's#^allow_url_fopen = .*#allow_url_fopen = On#g' /etc/php*/php.ini
sed -i -r 's#^.default_charset =.*#default_charset = "UTF-8"#g' /etc/php*/php.ini
sed -i -r 's#^.max_execution_time =.*#max_execution_time = 150#g' /etc/php*/php.ini
sed -i -r 's#^max_input_time =.*#max_input_time = 90#g' /etc/php*/php.ini
sed -i -r 's#^session.cookie_secure =.*#session.cookie_secure = on#g' /etc/php*/php.ini
sed -i -r 's#^session.cookie_samesite =.*#session.cookie_samesite = Lax#g' /etc/php*/php.ini

cat > /var/www/localhost/htdocs/index.php << EOF 
<?php
phpinfo();
?>
EOF
```

Now confiuguration of the database

```
apk add tzdata mysql mysql-client mariadb-doc mariadb-server-utils

mysql_install_db --user=mysql --datadir=/var/lib/mysql

rc-service mariadb start

mysqladmin -u root password toor1

cat > /etc/my.cnf.d/mariadb-timezone.cnf << EOF
[mysqld]
default-time-zone = 'America/Panama'
EOF
mysql_tzinfo_to_sql /usr/share/zoneinfo/ | mysql -u root -ptoor1 mysql

sed -i "s|.*max_allowed_packet\s*=.*|max_allowed_packet = 100M|g" /etc/mysql/my.cnf
sed -i "s|.*max_allowed_packet\s*=.*|max_allowed_packet = 100M|g" /etc/my.cnf.d/mariadb-server.cnf
sed -i "s|.*bind-address\s*=.*|bind-address=0.0.0.0|g" /etc/mysql/my.cnf
sed -i "s|.*bind-address\s*=.*|bind-address=0.0.0.0|g" /etc/my.cnf.d/mariadb-server.cnf
sed -i "s|.*skip-networking.*|#skip-networking|g" /etc/mysql/my.cnf
sed -i "s|.*skip-networking.*|#skip-networking|g" /etc/my.cnf.d/mariadb-server.cnf

rc-service mariadb restart

rc-update add mariadb

mkdir -p /usr/share/webapps/adminer

wget https://github.com/vrana/adminer/releases/download/v5.3.0/adminer-5.3.0.php -O /usr/share/webapps/adminer/adminer-5.3.php

rm -rf /usr/share/webapps/adminer/index.php
ln -s adminer-5.3.php /usr/share/webapps/adminer/index.php

cat > /etc/apache2/conf.d/adminer.conf << EOF
Alias /adminer /usr/share/webapps/adminer/
<Directory /usr/share/webapps/adminer/>
    Require all granted
    DirectoryIndex index.php
</Directory>
EOF

rc-service apache2 restart
```

## 3 - Download and setup GLPI

GLPI does not have a way to automate the installation, but at the end 
of the document we have a full automation way.

```
apk add aria2

cd /tmp

aria2c https://github.com/glpi-project/glpi/releases/download/10.0.18/glpi-10.0.18.tgz

tar zxvf glpi-10.0.18.tgz

mkdir -p /usr/share/webapps && rm -rf /usr/share/webapps/glpi

mv glpi /usr/share/webapps/

cat > /etc/apache2/conf.d/glpi.conf << EOF
Alias /glpi /usr/share/webapps/glpi/
<Directory /usr/share/webapps/glpi/>
    Require all granted
    DirectoryIndex index.php
</Directory>
EOF

cat > /usr/share/webapps/glpi/public/.htaccess << EOF
RewriteBase /
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)\$ index.php [QSA,L]
EOF

sed -i '3,$ s/^#//' /usr/share/webapps/glpi/.htaccess

chown -R apache:www-data /usr/share/webapps/glpi/files
chown -R apache:www-data /usr/share/webapps/glpi/config

rc-service apache2 restart
```

After runs last command, we need to run installer of database so continue 
as root user and runs following commands to setup database

```
mysql -u root -ptoor -e "CREATE DATABASE glpi;"
mysql -u root -proot -e "CREATE USER 'glpiudb'@'%' IDENTIFIED BY 'glpi.1';FLUSH PRIVILEGES;"
mysql -u root -proot -e "CREATE USER 'glpiudb'@'localhost' IDENTIFIED BY 'glpi.1';FLUSH PRIVILEGES;"
mysql -u root -ptoor -e "GRANT ALL PRIVILEGES ON glpi.* TO 'glpiudb';"
mysql -u root -ptoor -e "GRANT ALL PRIVILEGES ON glpi.* TO 'glpiudb'@'localhost' WITH GRANT OPTION;"
mysql -u root -ptoor -e "GRANT SELECT ON mysql.time_zone_name TO 'glpiudb'@'localhost';"

cd /usr/share/webapps/glpi

php82 bin/console db:install -f --default-language=es_VE --no-telemetry -H localhost -d glpi -u glpiudb -p glpi.1

php bin/console task:unlock --all
```

Now you can visit the http://provision.domino.ver/glpi , default user accounts are:

* glpi/glpi admin account,
* tech/tech technical account,
* normal/normal “normal” account,
* post-only/postonly post-only accoun

## 4 - Configuration for remote access

The installation only provides core functionality of GLPI, but for full access 
apart of the web UI you will need ssh setup so runs the following commands to 
setup the SSH software on the GLPI server and then fix any extra parameters by 
example when to install extra modules of the GLPI system, so please if you 
dont know check [How to use this guide](#how-to-use-this-guide) 
section, otherwise runs all of the following commands as root user:

```
apk add doas bash shadow shadow-uidmap musl-locales musl-locales-lang \
 e2fsprogs btrfs-progs exfat-utils f2fs-tools dosfstools xfsprogs jfsutils zfs \
 acpi patch coreutils mdadm e2fsprogs-extra attr smartmontools doas-sudo-shim \
 iproute2 netpbm poppler-utils libjpeg-turbo-utils perl-socket6 libqrencode-tools

cat > /etc/doas.d/apkgeneral.conf << EOF
permit  general as root cmd apk
permit  general as root cmd service
EOF

useradd -m -U -c "" -s /bin/bash -G wheel,input,disk,floppy,cdrom,dialout,audio,video,lp,netdev,games,users,ping,wheel general

for u in $(ls /home); do for g in disk lp floppy audio cdrom dialout video lp netdev games users ping wheel; do addgroup $u $g; done;done

echo "general:general.1" | chpasswd

sed -i -r 's|#PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config

echo AllowUsers general >> /etc/ssh/sshd_config

service sshd restart
```

Now everything will be using the "general" user, so RUNS "su -l general" 
or LOGIN WITH general USER (same password so obvious), this user will have 
the power to install packages and also restart services as super user. 
Also will be from now the only user to get access using ssh to the machine.

## How to use this guide

This guide **structure all the commands in blocks, each block its separated by a line spaced**, 
so you must **type each line as is.. and hit enter**, so you noted that then you 
typed each separated clocks of commands, copy/type only blocks separated by an empty line, 
all new(next) lines are made by just enter. the terminal will detect if must execute or not.

This guide is for install process, many parts will need you understand minimal 
knowledge of linux.

This guide assumed you have a serial port allowed in the targeted computer, also 
its important you shuold understand the way of the configuration in this guide.

> **Warning**  Some Linux or/and Mac terminals have security cut/paste locks, so 
> if you paste, the first line will be preceded by garbage, check always the first char of your paste.

## see also

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
- Matrix
  - 👥 https://matrix.to/#/#alpine-linux-english:matrix.org

# LICENSE

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [[alpine/copyright.md](../../alpine/copyright.md)](https://venenux.github.io/alpine-wiki/#/alpine/copyright)
