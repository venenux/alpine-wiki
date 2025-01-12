# Linux + Apache2 + Mysql + Php5

The recommendation its to use apache2 behind a reverse proxy setup, such like 
lighttpd or hiawatta servers. This version will only use PHP 5.6, for users with 
recents alpine like 3.12 use document [server-alpine-LAMP-php7-fast-forward.md](server-alpine-LAMP-php7-fast-forward.md)
for php8 alpine up to 3.15 use document [server-alpine-LAMP-php8-fast-forward.md](server-alpine-LAMP-php8-fast-forward.md)
for most moderd alpines up to 3.20 [server-alpine-LAMP-professional-fast-forward.md](server-alpine-LAMP-professional-fast-forward.md)


* [1 - Apache2](#1-apache2)
    * [Apache2 Status special page](#apache2-status-special-page)
    * [Apache2 CGI bin directory support](#apache2-cgi-bin-directory-support)
    * [Apache2 SSL support](#apache2-ssl-support)
* [2 - Php](#2-php)
    * [php install](#php-install)
    * [configuration of php](#configuration-of-php)
    * [configuration of apache2 and php-fpm](#configuration-of-apache2-and-php-fpm)
* [3 - Databases: Sqlite, Postgres, Mysql, ODBC](#3-databases-mysql-postgresql-sqlite-odbc)
    * [Mysql Instalation and configuration](#mysql-instalation-and-configuration)
    * [PostgreSQL instalation and configuration](#postgresql-instalation-and-configuration)
    * [ODBC instalation and configuration](#odbc-instalation-and-configuration)
* [4 - Extra needs](#extra-needs)

## 1 - apache2

Due to the minimalism of alpine linux, unfortunately the apache2 packaging is the worst ever seen, its configuration file makes it impossible to configure with only single line commands so the commands for quick configuration with cares of overwriting are very dedicated.
Currently the most lazy and slow server .. just for windosers that wants to learn.. 

#### Apache2 Installation

1. run apk for need pacakges
2. make the htdos public web root directories
3. configure the default ports and server information
4. setup the permissions
5. added the service to the boot process
6. start the service
7. put a default page to test the service

```
apk add sed apache2 apache2-utils

mkdir -p /var/www/localhost/htdocs /var/log/apache2

sed -i -r 's#^Listen.*#Listen 80#g' /etc/apache2/httpd.conf

sed -i -r 's#^ServerTokens.*#ServerTokens Minimal#g' /etc/apache2/httpd.conf

chown -R apache:www-data /var/www/localhost/

chown -R apache:wheel /var/log/apache2

rc-update add apache2 default

rc-service apache2 restart

echo "it works" > /var/www/localhost/htdocs/index.html
```

For testing open a browser and go to `http://<webserveripaddres>` and you will see "it works". The "webserveripaddres" are the ip address of your setup/server machine.

#### Apache2 Status special page

1. Enable the mod_status at the config files
2. change path in the config file, we are using security by obfuscation later by auth module
3. change the restriction of the status pages, currently we just remove it
4. restart the service to see changes at the browser

```
mkdir -p /var/www/localhost/htdocs/stats

sed -i -r 's#.*LoadModule.*modules/mod_info.so.*#LoadModule info_module modules/mod_info.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_status.so.*#LoadModule status_module modules/mod_status.so#g' /etc/apache2/httpd.conf

sed -i -r 's#tion /server-status#tion /stats/server-status#g' /etc/apache2/conf.d/info.conf
sed -i -r 's#tion /server-info#tion /stats/server-info#g' /etc/apache2/conf.d/info.conf

sed -i -r 's#.*Require host.*#\#    Require host#g' /etc/apache2/conf.d/info.conf
sed -i -r 's#.*Require ip.*#\#    Require ip#g' /etc/apache2/conf.d/info.conf

rc-service apache2 restart
```

#### Apache2 CGI bin directory support

1. create the directory due packager dont make any reference to that neither in the useradd
2. enable the mod_userdir in the config file
3. get sure alias module is also enabled
4. setup and enable the config cgi file path
5. restart the service to see changes at the browser

```
mkdir -p /var/www/localhost/cgi-bin

sed -i -r 's#.*LoadModule.*modules/mod_cgid.so.*#LoadModule cgid_module modules/mod_cgid.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_cgi.so.*#LoadModule cgi_module modules/mod_cgi.so#g' /etc/apache2/httpd.conf

sed -i -r 's#.*LoadModule.*modules/mod_alias.so.*#LoadModule alias_module modules/mod_alias.so#g' /etc/apache2/httpd.conf

sed -i -r 's#.*ScriptAlias /cgi-bin/.*#    ScriptAlias /cgi-bin/ "/var/www/localhost/cgi-bin"#g' /etc/apache2/httpd.conf

rc-service apache2 restart
```

After that, all the files under the `/var/www/localhost/cgi-bin` directory will be procesed under `http://localhost/cgi-bin/` path to executed due the directives defined in the line 482 of the config file.

#### Apache2 Descriptive error or special pages

1. install the errors package
2. restart the service

```
apk add apache2-error

rc-service apache2 restart
```

All about error documents are define at `/etc/apache2/conf.d/multilang-errordoc.conf`, you can customized byt redefine the error alias and the error codes. The right way is to make a symlink from `/var/www/error-pages` over each document and if there's any customized remove the symlink and create the alternate error page there.

#### Apache2 Userdir public_html support

1. create the directory for put the html files due alpine crap does not follow any standard
2. enable the module in the webserver
3. set the user directory in the config file
4. restart the service to see the changes at the browser per user

```
mkdir -p /etc/skel/Devel

for i in /home/*; do mkdir $i/Devel ; done

sed -i -r 's#.*LoadModule.*modules/mod_usertrack.so.*#LoadModule usertrack_module modules/mod_usertrack.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_userdir.so.*#LoadModule userdir_module modules/mod_userdir.so#g' /etc/apache2/httpd.conf

sed -i -r 's#^UserDir .*#UserDir Devel#g' /etc/apache2/conf.d/userdir.conf

rc-service apache2 restart
```

> **Warning**  as we said.. alpine policy is to be most upstream equal possible, almost like packagers are lazy? NO! just dont put any thing about root user access, but well, you must know what are you doing, by the addition of `UserDir disabled root postmaster` you will denied specific users due security.

#### Apache2 alpine proxy modules setup

The error of the XML file in the proxy modules are due the incomplete right made package:

```
httpd: Syntax error on line 481 of /etc/apache2/httpd.conf: Syntax error on line 13 of /etc/apache2/conf.d/proxy-html.conf: Cannot load /usr/lib/libxml2.so into server: Error loading shared library /usr/lib/libxml2.so: No such file or directory
```

1. install the proxy apache2 packages
2. fix the configuration of the modules
3. setup a conf file for your redirection
4. restart the service

```
apk add sed apache2-proxy-html apache2-proxy

sed -i -r 's#/usr/lib/libxml2.so.+#/usr/lib/libxml2.so.2#g' /etc/apache2/conf.d/proxy-html.conf

cat >> /etc/apache2/conf.d/myproxy.conf << EOF
    ProxyPreserveHost On
    ProxyRequests off
    AllowEncodedSlashes NoDecode
    ProxyPass / http://127.0.0.1:3002/ nocanon
EOF

service apache2 restart
```

> **Warning**  of course, the **`myproxy.conf` is hypothetical, for didactic purposes**, it will fails if port 3002 does not offers something, it is only exemplified that the error is corrected in the step of the sed command to work.

#### Apache2 SSL support

1. install openssl and apache-ssl
2. create the self signed certificate
3. set proper permissions
4. setup the cert file for combined pem
5. setup the port for the openssl protocol module
6. setup the allowed negociations, by example allow TLS 1.0 (default deny sslv3 and tls1)
7. setup the allowed protocols, by example allow also olders ones like TLS 1.0
8. activate the mod_redirect in case of global http to https redirections
9. restart the service to see changes

```
apk add openssl apache2-ssl

mkdir -p /etc/ssl/certs/

openssl req -x509 -days 1460 -nodes -newkey rsa:4096 \
   -subj "/C=VE/ST=Bolivar/L=Upata/O=VenenuX/OU=Systemas:hozYmartillo/CN=$(hostname -d)" \
   -keyout /etc/ssl/certs/localhost.pem -out /etc/ssl/certs/localhost.pem

chmod 640 /etc/ssl/certs/localhost.pem
chown apache:www-data /etc/ssl/certs/localhost.pem

sed -i -r 's#^SSLCertificateKeyFile.*/etc/.*#\#SSLCertificateKeyFile /etc/#g' /etc/apache2/conf.d/ssl.conf
sed -i -r 's#^SSLCertificateFile.*/etc/.*#SSLCertificateFile /etc/ssl/certs/localhost.pem#g' /etc/apache2/conf.d/ssl.conf
sed -i -r 's#^SSLCertificateChainFile.*#SSLCertificateChainFile /etc/ssl/certs/localhost.pem#g' /etc/apache2/conf.d/ssl.conf
sed -i -r 's#\#.*SSLCertificateChainFile.*#SSLCertificateChainFile /etc/ssl/certs/localhost.pem#g' /etc/apache2/conf.d/ssl.conf

sed -i -r 's#^Listen.*#Listen 443#g' /etc/apache2/conf.d/ssl.conf
sed -i -r 's#^<VirtualHost.*#<VirtualHost _default_:443>#g' /etc/apache2/conf.d/ssl.conf

sed -i -r 's#^SSLProtocol.*#SSLProtocol all#g' /etc/apache2/conf.d/ssl.conf

sed -i -r 's#^SSLCipherSuite.*#SSLCipherSuite HIGH:MEDIUM:ALL:!MD5:!RC4:!3DES#g' /etc/apache2/conf.d/ssl.conf
sed -i -r 's#^SSLProxyCipherSuite.*#SSLProxyCipherSuite HIGH:MEDIUM:ALL:!MD5:!RC4:!3DES#g' /etc/apache2/conf.d/ssl.conf

rc-service apache2 restart
```

> **Warning** this configuration:
> 1. This is a permissive configuration full compatible wtith older and newer browsers.
> 2. to only allow most secure protocols and a bit of compatibilty, set to `SSLProtocol all -TLSv1 -SSLv3`
> 3. to only allow most secure negociations and a bit of compat, set to `SSLCipherSuite HIGH:MEDIUM:ECDHE:!MD5:!RC4:!3DES:!ADH`
> 4. to only allow most secure negociations and a bit of compat, set proxy to `SSLProxyCipherSuite HIGH:MEDIUM:ECDHE:!MD5:!RC4:!3DES:!ADH`

## 2 - PHP

The php packages in alpine are only one version at time.

> **Warning** Those instructions are for **php5.6 and alpine 3.8+** if you runs 
> olders alpine from 3.8 most of those packages not exits yet!
> for most moderd alpines up to 3.20 [server-alpine-LAMP-professional-fast-forward.md](server-alpine-LAMP-professional-fast-forward.md)

#### php install

1. Install php base packages for php runtime features
2. Install php internationalization and languaje support
3. Install php graphics support packages
4. install php autentication and extra protocol support
5. Install the cli package and then the specific engine:
    * install the fpm package for the fpm backend
    * install the cgi package for the cgi backend
6. install the database handlers
    * install berkerly, mysql, ODBC, postgres, sqlite
    * install the DBO abstration and their backends, mysql, ODBC, postgres, sqlite
7. install the php apache2 module package

```
apk add php5-opcache php5-openssl php5-json php5-bcmath php5-bz2 \
 php5-ctype php5-dev php5-dom php5-enchant php5-shmop \
 php5-sysvmsg php5-sysvsem php5-sysvshm php5-xml php5-xmlreader \
 php5-xsl php5-zip php5-intl php5-gettext php5-pspell php5-calendar \
 php5-exif php5-gd php5-pcntl php5-gmp php5-imap php5-curl php5-pear \
 php5-phar php5-doc php5-embed php5-posix php5-fpm php5-cgi php5-dba php5-mysqli \

apk add php5-fpm

apk add php5-cgi

apk add php5-mysql php5-odbc php5-pgsql php5-sqlite3

apk add php5-pdo php5-pdo_dblib php5-pdo_mysql php5-pdo_odbc php5-pdo_pgsql php5-pdo_sqlite

apk add php5-apache2
```

> **Warning** Those instructions are for **php5 and alpine 3.8+** if you runs 
> olders alpine from 3.8 most of those packages not exits yet!
> for most moderd alpines up to 3.20 [server-alpine-LAMP-professional-fast-forward.md](server-alpine-LAMP-professional-fast-forward.md)

#### configuration of php

1. Use fix.pathinfo
2. Set safe mode off
3. Dont expose php code if something fails
4. Set amount of memory limit for execution to 536Mb (most servers are minimum of 1 GB of RAM)
5. Set upload size to 128Mb as maximun.
6. Set POST max size to 256Mb based on the upload max size limit.
7. Turn on the URL open method
8. Set default charset to UTF-8 for increased compatibility
9. Increase the execution time and the input time for.

```
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
```

#### configuration of apache2 and php-fpm

1. Create directory for php socket and pid files, set owner, **WARNING** MUST BE EQUAL to openrc defined!
2. Set into configuration file paths for socket and pid, **WARNING** MUST BE EQUAL to openrc defined!
3. Set into configuration file owner for apache2 and mode, **WARNING** MUST BE EQUAL to openrc defined!
4. eable the line of event module uncommenting and disable the line of prefork module by commenting
5. add the php fpm service and activate the apache2 service **WARNING** since alpine v3.16 its php8 not php7
6. start the php fpm service
7. now here you will choose the way to handle php:
    * add the handler into apache2 configuration for php using module and restart apache2
    * add the handler into apache2 configuration for php using fpm and restart apache2

```
mkdir -p /var/run/php5-fpm/
chown apache:www-data /var/run/php5-fpm

sed -i -r 's|^.*listen =.*|listen = /run/php5-fpm/php-fpm.sock|g' /etc/php*/php-fpm.d/www.conf
sed -i -r 's|^.*listen =.*|listen = /run/php5-fpm/php-fpm.sock|g' /etc/php*/php-fpm.conf
sed -i -r 's|^pid =.*|pid = /run/php5-fpm/php-fpm.pid|g' /etc/php*/php-fpm.conf
sed -i -r 's|^pid =.*|pid = /run/php5-fpm/php-fpm.pid|g' /etc/php*/php-fpm.conf
sed -i -r 's|^pidfile=.*|pidfile=/run/php5-fpm/php-fpm.pid|g' /etc/init.d/php-fpm

sed -i -r 's|^user = .*|user = apache|g' /etc/php*/php-fpm.conf
sed -i -r 's|^user = .*|user = apache|g' /etc/php*/php-fpm.d/www.conf
sed -i -r 's|^group = .*|group = www-data|g' /etc/php*/php-fpm.conf
sed -i -r 's|^group = .*|group = www-data|g' /etc/php*/php-fpm.d/www.conf
sed -i -r 's|^.*listen.owner = .*|listen.owner = apache|g' /etc/php*/php-fpm.conf
sed -i -r 's|^.*listen.owner = .*|listen.owner = apache|g' /etc/php*/php-fpm.d/www.conf
sed -i -r 's|^.*listen.group = .*|listen.group = www-data|g' /etc/php*/php-fpm.conf
sed -i -r 's|^.*listen.group = .*|listen.group = www-data|g' /etc/php*/php-fpm.d/www.conf
sed -i -r 's|^.*listen.mode = .*|listen.mode = 0660|g' /etc/php*/php-fpm.conf
sed -i -r 's|^.*listen.mode = .*|listen.mode = 0660|g' /etc/php*/php-fpm.d/www.conf

sed -i -r 's|.*LoadModule.*modules/mod_mpm_event.so.*|LoadModule mpm_event_module modules/mod_mpm_event.so|g' /etc/apache2/httpd.conf
sed -i -r 's|.*LoadModule.*modules/mod_mpm_prefork.so.*|#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so|g' /etc/apache2/httpd.conf

rc-update add php-fpm
rc-update add apache2

rc-service php-fpm start

mv /etc/apache2/conf.d/php5-module.conf /etc/apache2/conf.d/php5-module.conf.disabled
cat > /etc/apache2/conf.d/php5-fpm.conf << EOF
<FilesMatch \\.php\$>
   <If "-f %{REQUEST_FILENAME}">
    SetHandler "proxy:unix:/run/php5-fpm/php-fpm.sock|fcgi://localhost"
   </If>
</FilesMatch>
EOF

echo -e "<?php\nphpinfo( );\n?>" > /var/www/localhost/htdocs/index.php

rc-service apache2 start
```

> **WARNING** the two last steps are mutualy exclusive, only one way of php handle can be made.

> **Note** here we use **sockets, so its more faster rather fcgi** steps made by oficial wiki.

## 3 - Databases - MySQL postgreSQL SQLite ODBC

In alpine the only option is using Mariadb or Postgresql, ODBC only provides TDS and Postgresql, 
rest of options are only in edge or using other linuxes

#### adminer to MANAGE ANY DATABASE SERVERS

The most secure for web Mysql managemend its adminer, cos phpmyadmin its too polite
Adminer permits to manage any kind of database, including odbc, postgresql and mysql/mariadb.

```
mkdir -p /usr/share/webapps/adminer

wget https://github.com/pematon/adminer/releases/download/v4.12/adminer-4.12.php -O /usr/share/webapps/adminer/adminer-4.12.php

rm -rf /usr/share/webapps/adminer/index.php
ln -s adminer-4.12.php /usr/share/webapps/adminer/index.php

cat > /etc/apache2/conf.d/adminer.conf << EOF
Alias /adminer /usr/share/webapps/adminer/
<Directory /usr/share/webapps/adminer/>
    Require all granted
    DirectoryIndex index.php
</Directory>
EOF

rc-service apache2 restart
```

The administrator must use the exact URL
`http://<ipaddress>/adminer/index.php` There are two reasons: there\'s
no directory listing and there\'s no direct PHP index reference on the
web server, all because of paranoid settings.

#### Mysql Instalation and configuration

1. install the packages necesary to LAMP usage, not only to deploy a database, including the mytop monitor
2. Initialize the main mysql database, and the data dir as standardized to /var/lib/mysql by the rc script
3. Then initialize the service, root account and socket connection are enabled without password at this point
4. Set up the root account by asigning a proper password. This is pure paranoia. the next step does just that!
5. On older Alpine system you must set config files for MAX ALLOWED PACKETS to minimun proper amount:
6. Allow any source connections in cases where we must test so much and connect to develop
7. Add the start service process, but don't set it as a boot process because networking needs to already be running.
8. Restart the service to apply changes.

```
apk add mysql mysql-client mariadb-doc mariadb-server-utils

mysql_install_db --user=mysql --datadir=/var/lib/mysql

rc-service mariadb start

mysqladmin -u root password toor

sed -i "s|.*max_allowed_packet\s*=.*|max_allowed_packet = 100M|g" /etc/mysql/my.cnf
sed -i "s|.*max_allowed_packet\s*=.*|max_allowed_packet = 100M|g" /etc/my.cnf.d/mariadb-server.cnf

sed -i "s|.*bind-address\s*=.*|bind-address=0.0.0.0|g" /etc/mysql/my.cnf
sed -i "s|.*bind-address\s*=.*|bind-address=0.0.0.0|g" /etc/my.cnf.d/mariadb-server.cnf
sed -i "s|.*skip-networking.*|#skip-networking|g" /etc/mysql/my.cnf
sed -i "s|.*skip-networking.*|#skip-networking|g" /etc/my.cnf.d/mariadb-server.cnf

rc-service mariadb restart

rc-update add mariadb
```

For information and more deep setup check [../documents/server-alpine-mysql-professional.md](../documents/server-alpine-mysql-professional.md)

> **Warning:** the configuration here is for quick access and fist development, security is minimal only! for secure configuration set to specific bind-address or if you dont want remote access uncomment skip-networking

#### PostgreSQL instalation and configuration

1. install postgres packages **WARNING** since alpine v3.15 the pacakge has name scheme version number
2. Initialize the main cluster database, and the data dir as standartized
3. Allow any source connections in cases where we must test so much and connect to develop
4. Then initialize the service, root account and socket connection are enabled without password at this point
5. Add the start service process, but don't set it as a boot process because networking needs to already be running.
6. Set up the root account by asigning a proper password. This is pure paranoia. the next step does just that!


```
apk add postgresql postgresql-client postgresql-contrib postgresql-libs postgresql-orafce

rc-service postgresql setup

sed -r -i "s|.*listen_addresses.*=.*|listen_addresses=\*|g" /var/lib/postgresql/*/data/postgresql.conf*

rc-service postgresql start

rc-update add postgresql

psql -U postgres
  create user postfix with password '******';
  create database postfix owner postfix;
  \c postfix
  create language plpgsql;
  \q
```

#### ODBC instalation and configuration

1. install the base packages for odbc and the available alpine odbc backends to access
2. configure the postgresql ODBC module in the system, for ansi and unicode flavours
2. configure the freetds ODBC module in the system for MSSQL and Sybase databases
2. configure the MDBtools ODBC module in the system for MDB M$ databases

```
apk add unixodbc psqlodbc freetds

odbcinst -u -d -n PostgreSQL 
cat > /tmp/tmppg.tmp << EOF
[PostgreSQL]
Description = PostgreSQL ODBC (ANSI version)
Driver = /usr/lib/psqlodbca.so
Debug = 0
CommLog = 1
[PostgreSQL-Unicode]
Description = PostgreSQL ODBC (Unicode version)
Driver = /usr/lib/psqlodbcw.so
Debug = 0
CommLog = 1
EOF
odbcinst -i -d -f /tmp/tmppg.tmp

odbcinst -u -d -n FreeTDS
cat > /tmp/tmptds.tmp << EOF
[FreeTDS]
Description	= TDS module (Sybase/MS SQL)
Driver		= /usr/lib/libtdsodbc.so.0
CPTimeout	= 
CPReuse		= 
odbcinst -i -d -f /tmp/tmptds.tmp
```

> **Warning** the packages for mysql and sqlite are `sqliteodbc` and `mariadb-connector-odbc` but only available for edge.

> **Warning** if your alpine version is too older, you cannot use the mysql edge package for odbc unfortunatelly


## Extra needs

#### Lest Encrypt to provide real certifications

To obtain a real certificate, you must have a external direct public ip, so for local LAMP common deploys are nonsense. 

Check the document [guide-only-dehydrated.md](guide-only-dehydrated.md) there's also a specific section to setup apache2.

## see also

- 🗯 IRC
  - 💬 `##alpine_telegram_english`
  - 💬 `#alpine_linux_english`
- 📱 Telegram https://t.me/alpine_linux
  - 🇬🇧 https://t.me/alpine_linux_english
  - 🇷🇺 https://t.me/alpine_linux_pycckuu (dual english russian, low activity)
  - 🇨🇴 https://t.me/alpine_linux_espanol
  - 🇧🇬 https://t.me/alpine_linux_news (dual english bulgarian, low activity)
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

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
