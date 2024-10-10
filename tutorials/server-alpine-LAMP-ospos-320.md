# alpine3.20 + Apache2 + Mysql + Php8 + Ospos4

**Warning** those are for **php 8.3 (php83) due composer dependencies on alpine 3.20** 
if you runs olders use [server-alpine-LAMP-ospos-314.md](server-alpine-LAMP-ospos-314.md)

* [Install alpine linux](#install-alpine-linux)
* [0 - Environment](#0---setup-environment)
* [1 - Apache2](#1---apache2)
* [2 - Php](#2---php)
* [3 - DBMS Mysql](#3---databases-mysql)
* [4 - ospos](#4---ospos)
* [How to use this guide](#how-to-use-this-guide)
* [LICENSE](#LICENSE)

This document will not explain anything you must to obey, as must be cos just works and works very well, please if you dont know check [How to use this guide](#how-to-use-this-guide) section before starts:

## Install alpine linux

```
mkdir -p /home/general/VM/alpine320 && cd /home/general/VM/alpine320

qemu-img create -f raw computerint1alpine-vitualdisk1-file.raw 6G

wget -c -t8 --no-check-certificate http://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-extended-3.20.0-x86_64.iso

/usr/bin/qemu-system-x86_64  -m 2048 -name "computerint1alpine320" \
 -cpu host -machine q35 \
 -device rtl8139,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3222-:22,hostfwd=tcp::9080-:80,hostfwd=tcp::9443-:443 \
 -device virtio-keyboard -device virtio-mouse -device virtio-tablet -device virtio-vga,max_outputs=1 \
 -drive file=computerint1alpine-vitualdisk1-file.raw,format=raw \
 -cdrom alpine-extended-3.20.0-x86_64.iso -boot d
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

Then reboot and if you are using a virtual machine change the line `-boot d` to ` -boot c`

## 0 - Setup environment

```
cat > /etc/apk/repositories << EOF
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add man-pages nano binutils coreutils readline \
 sed attr dialog lsof less groff wget curl terminus-font \
 file lz4 gawk tree pciutils usbutils lshw tzdata tzdata-utils \
 zip p7zip xz tar cabextract cpio binutils lha acpi musl-locales musl-locales-lang \
 e2fsprogs btrfs-progs exfat-utils f2fs-tools dosfstools xfsprogs jfsutils \
 arch-install-scripts util-linux docs

rc-update add consolefont boot
```

## 1 - apache2

```
apk add apache2 apache2-utils apache2-error apache2-proxy-html apache2-proxy

mkdir -p /etc/skel/Devel
mkdir -p /var/www/localhost/cgi-bin /var/www/localhost/htdocs /var/log/apache2
sed -i -r 's#^Listen.*#Listen 80#g' /etc/apache2/httpd.conf
sed -i -r 's#^ServerTokens.*#ServerTokens Minimal#g' /etc/apache2/httpd.conf
chown -R apache:www-data /var/www/localhost/
chown -R apache:wheel /var/log/apache2
sed -i -r 's#.*LoadModule.*modules/mod_cgid.so.*#LoadModule cgid_module modules/mod_cgid.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_cgi.so.*#LoadModule cgi_module modules/mod_cgi.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_alias.so.*#LoadModule alias_module modules/mod_alias.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*ScriptAlias /cgi-bin/.*#    ScriptAlias /cgi-bin/ "/var/www/localhost/cgi-bin"#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_usertrack.so.*#LoadModule usertrack_module modules/mod_usertrack.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_userdir.so.*#LoadModule userdir_module modules/mod_userdir.so#g' /etc/apache2/httpd.conf
sed -i -r 's#public_html#Devel#g' /etc/apache2/conf.d/userdir.conf
sed -i -r 's#AllowOverride.*#AllowOverride All#g' /etc/apache2/conf.d/userdir.conf
sed -i -r 's#/usr/lib/libxml2.so.*#/usr/lib/libxml2.so.2#g' /etc/apache2/conf.d/proxy-html.conf

rc-update add apache2 default

rc-service apache2 restart

echo "it works" > /var/www/localhost/htdocs/index.html
for i in /home/*; do mkdir $i/Devel ; done
```

For testing open a browser and go to `http://<webserveripaddres>` but for secure way or SSL support: 
https://venenux.github.io/alpine-wiki/#/tutorials/server-alpine-LAMP-professional-fast-forward

## 2 - PHP

```
apk add php83-opcache php83-openssl php83-json php83-bcmath php83-mbstring php83-bz2 \
 php83-ctype php83-dev php83-dom php83-enchant php83-fileinfo php83-shmop php83-simplexml php83-tidy \
 php83-tokenizer php83-sysvmsg php83-sysvsem php83-sysvshm php83-xml php83-xmlreader \
 php83-xmlwriter php83-xsl php83-zip php83-intl php83-gettext php83-pspell php83-calendar \
 php83-exif php83-gd php83-pcntl php83-gmp php83-imap php83-session php83-curl php83-pear \
 php83-phar php83-doc php83-embed php83-posix php83-fpm php83-cgi php83-dba php83-mysqli \
 php83-mysqlnd php83-odbc php83-pgsql php83-sodium php83-sqlite3 php83-apache2 \
 php83-pdo php83-pdo_dblib php83-pdo_mysql php83-pdo_odbc php83-pdo_pgsql php83-pdo_sqlite

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
mkdir -p /var/run/php-fpm83/
sed -i -r 's|^.*listen.owner = .*|listen.owner = apache|g' /etc/php*/php-fpm.d/www.conf
sed -i -r 's|^.*listen.group = .*|listen.group = www-data|g' /etc/php*/php-fpm.d/www.conf
sed -i -r 's|^.*listen.mode = .*|listen.mode = 0660|g' /etc/php*/php-fpm.d/www.conf
chown apache:www-data /var/run/php-fpm83

sed -i -r 's|^.*listen =.*|listen = /run/php-fpm83/php-fpm.sock|g' /etc/php83/php-fpm.d/www.conf
sed -i -r 's|^pid =.*|pid = /run/php-fpm83/php-fpm.pid|g' /etc/php83/php-fpm.conf
rc-update add php-fpm83
rc-service php-fpm83 restart

sed -i -r 's|.*LoadModule.*modules/mod_mpm_event.so.*|LoadModule mpm_event_module modules/mod_mpm_event.so|g' /etc/apache2/httpd.conf
sed -i -r 's|.*LoadModule.*modules/mod_mpm_prefork.so.*|#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so|g' /etc/apache2/httpd.conf
rm /etc/apache2/conf.d/php*.conf
cat >> /etc/apache2/conf.d/php83-fpm.conf << EOF
<FilesMatch \\.php\$>
   <If "-f %{REQUEST_FILENAME}">
    SetHandler "proxy:unix:/run/php-fpm83/php-fpm.sock|fcgi://localhost"
   </If>
</FilesMatch>
EOF
rc-update add apache2
rc-service apache2 restart

echo -e "<?php\nphpinfo( );\n?>" > /var/www/localhost/htdocs/index.php
```

## 3 - Databases mysql

```
apk add mysql mysql-client mariadb-doc mariadb-server-utils mariadb-mytop

mysql_install_db --user=mysql --datadir=/var/lib/mysql

sed -i "s|.*max_allowed_packet\s*=.*|max_allowed_packet = 100M|g" /etc/mysql/my.cnf
sed -i "s|.*max_allowed_packet\s*=.*|max_allowed_packet = 100M|g" /etc/my.cnf.d/mariadb-server.cnf
sed -i "s|.*bind-address\s*=.*|bind-address=0.0.0.0|g" /etc/mysql/my.cnf
sed -i "s|.*bind-address\s*=.*|bind-address=0.0.0.0|g" /etc/my.cnf.d/mariadb-server.cnf
sed -i "s|.*skip-networking.*|#skip-networking|g" /etc/mysql/my.cnf
sed -i "s|.*skip-networking.*|#skip-networking|g" /etc/my.cnf.d/mariadb-server.cnf
rc-update add mariadb
rc-service mariadb restart

mysqladmin -u root password root

mkdir -p /usr/share/webapps/adminer && wget https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1.php -O /usr/share/webapps/adminer/adminer-4.8.1.php

ln -s adminer-4.8.1.php /usr/share/webapps/adminer/index.php
cat >> /etc/apache2/conf.d/adminer.conf << EOF
Alias /adminer /usr/share/webapps/adminer/
<Directory /usr/share/webapps/adminer/>
    Require all granted
    DirectoryIndex index.php
</Directory>
EOF
rc-service apache2 restart
```

## 4 - ospos

We build ospos from master branch, by default our user MUST be "general":

```
apk add doas bash shadow shadow-uidmap doas musl-locales musl-locales-lang

cat > /etc/doas.d/apkgeneral.conf << EOF
permit nopass general as root cmd apk
EOF
useradd -m -U -c "" -s /bin/bash -G wheel,input,disk,floppy,cdrom,dialout,audio,video,lp,netdev,games,users,ping,wheel general
for u in $(ls /home); do for g in disk lp floppy audio cdrom dialout video lp netdev games users ping wheel; do addgroup $u $g; done;done
echo "general:general" | chpasswd
```

Now everithing will be using "general" user, so RUNS "su -l general" OR LOGIN WITH general USER!

```
doas apk add git git-doc nodejs nodejs-doc npm npm-doc

mkdir -p /home/general/Devel && cd Devel
git clone https://github.com/opensourcepos/opensourcepos && cd opensourcepos

composer install

npm install

npm run build

mysql -u root -proot -e "CREATE SCHEMA ospos;"
mysql -u root -proot -e "CREATE USER 'admin'@'%' IDENTIFIED BY 'pointofsale';GRANT ALL PRIVILEGES ON ospos . * TO 'admin'@'%' IDENTIFIED BY 'pointofsale' WITH GRANT OPTION;FLUSH PRIVILEGES;"
mysql -u root -proot -e "CREATE USER 'admin'@'localhost' IDENTIFIED BY 'pointofsale';GRANT ALL PRIVILEGES ON ospos . * TO 'admin'@'localhost' IDENTIFIED BY 'pointofsale' WITH GRANT OPTION;FLUSH PRIVILEGES;"
mysql -u admin -ppointofsale -D ospos < database/database.sql

sed -i -r 's#logger.threshold.*#logger.threshold = 9#g' .env
sed -i -r 's#app.db_log_enabled.*#app.db_log_enabled = true#g' .env

chown -R general:www-data /home/general/Devel/opensourcepos
```


#### Tune apache2 rewrite and permissions

TODO


#### Results

* OSPOS: `http://<ip>/~general/opensourcepos/`
* MYSQL: `http://<ip>/adminer/`
* FILES: `/home/general/Devel/opensourcepos/`

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
if you paste, the first line will be preceded by garbage, check always the first char of your paste.

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
