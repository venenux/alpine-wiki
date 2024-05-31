In production web, **LAMP** means **L**inux + **A**pache + **M**ysql +
**P**hp installed and integrated, but today, more and more, *Apache* is
being replaced by *Nginx* or *Lighttpd* and *MySQL* by *Mariadb*. The
*LAMP* documents are:

## 1. The web server: Lighttpd

[lighttpd](https://www.lighttpd.net/) is a simple, standards-compliant,
secure, and flexible web server, Nginx is the most used due to beeing
manageable by ISP panel\'s software, but **lighttpd performs better.
Nginx cannot process fast-cgi programs**. For more lighttpd information,
consult the [Production Web server:Lighttpd](../documents/server-alpine-lighttpd-professional.md) wiki page.

### Lighttpd Installation

Production environment will handle only needed packages. So no doc or
managers allowed:

1. run apk for needed packages

```
apk add lighttpd gamin
```

### Lighttpd first configuration

1. make the htdos public web root directories
2. change default port to production one, http is used with 80
3. use **FAM style (gamin)** file alteration monitor, increases performance **ONLY ON 3.4 to 3.8 releases of alpine linux!!!**
4. use linux event handler, increases performance due Alpine are linux only
5. added the service to the default runlevel, not to boot, because networking needs to be active first
6. started the web server service
7. Enable the mod_status at the config files
8. change path in the config file, we are using security by obfuscation
9. restart the service to see changes at the browser

```
mkdir -p /var/www/localhost/htdocs/stats /var/log/lighttpd /var/lib/lighttpd

sed -i -r 's#\#.*server.port.*=.*#server.port          = 80#g' /etc/lighttpd/lighttpd.conf

sed -i -r 's#\#.*server.event-handler = "linux-sysepoll".*#server.event-handler = "linux-sysepoll"#g' /etc/lighttpd/lighttpd.conf

chown -R lighttpd:lighttpd /var/www/localhost/

chown -R lighttpd:lighttpd /var/lib/lighttpd

chown -R lighttpd:lighttpd /var/log/lighttpd

rc-update add lighttpd default

rc-service lighttpd restart

echo "it works" > /var/www/localhost/htdocs/index.html

sed -i -r 's#\#.*mod_status.*,.*#    "mod_status",#g' /etc/lighttpd/lighttpd.conf

sed -i -r 's#.*status.status-url.*=.*#status.status-url  = "/stats/server-status"#g' /etc/lighttpd/lighttpd.conf

sed -i -r 's#.*status.config-url.*=.*#status.config-url  = "/stats/server-config"#g' /etc/lighttpd/lighttpd.conf

rc-service lighttpd restart
```

**For testing, open a browser and go to `http://<webserveripaddres>`.
You will see "it works"**. The "webserveripaddres" is the ip address
of your setup/server machine.

**OPTIONAL:** alpine packagers are a mess, removed FAM on recent, so
older releases of alpine can use compiled FAM packages with
`sed -i -r 's#.*server.stat-cache-engine.*=.*# server.stat-cache-engine = "fam"#g' /etc/lighttpd/lighttpd.conf`
**FAM/gamin was deprecated since lighttpd 1.4.36**, and in older releases 
produces problems so if you are running older version of lighttpd 
just avoid usage of FAM/gamin on it!

## 2. php scripting: PHP fpm

In Alpine there are two main languages for programming dynamic web
pages: PHP and LUA. Alpine is minimalist so not all PHP packages are
need in most cases. Both repositories must be enabled (main and
community). Here we explain the most common use in production.

#### PHP Installation

Since alpine 3.16 only php 8 are available, php 7 was only from alpine 3.8 to 3.15, there are external repos on venenux git alpine builds!

```
apk add php81-common php81-doc php81-phar php81-posix php81-pspell php81-session \
 php81-cgi php81-fpm php81-curl php81-simplexml php81-sockets php81-bcmath php81-calendar \
 php81-ctype php81-dom php81-embed php81-enchant php81-exif php81-fileinfo php81-ftp \
 php81-pear php81-gd php81-iconv php81-gettext php81-gmp php81-imap php81-intl \
 php81-mbstring php81-openssl php81-opcache php81-pcntl php81-bz2 php81-zip \
 php81-tidy php81-tokenizer php81-xml php81-xmlreader php81-xmlwriter php81-xsl
```

Those are the basic most need packages, the following packages allow to connect to databases.
A special case is `php81-odbc`. Unless the others, that are able php to
connect to only specific database ODBC allows to connect to multiple databases!

```
apk add php81-dba php81-mysqli php81-mysqlnd php81-odbc php81-pgsql php81-sqlite3
```

There are also special database connection packages called PDO, install if you need:

```
apk add php81-pdo php81-pdo_dblib php81-pdo_mysql php81-pdo_odbc php81-pdo_pgsql php81-pdo_sqlite
```

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

#### PHP-FPM Configuration

1. Create directory for php socket and pid files, MUST BE EQUAL to openrc defined!
2. Set into configuration file the socket path, MUST BE EQUAL to openrc defined!
3. Set into configuration file the pid file path, MUST BE EQUAL to openrc defined!
4. Define the permissions for access of the service by socket, not by tpc/ip for security
5. setup adn restart the service!

```
mkdir -p /var/run/php-fpm7/

chown lighttpd:root /var/run/php-fpm7

sed -i -r 's|^.*listen =.*|listen = /var/run/php-fpm81/php81-fpm.sock|g' /etc/php*/php-fpm.d/www.conf

sed -i -r 's|^pid =.*|pid = /run/php-fpm81/php81-fpm.pid|g' /etc/php*/php-fpm.conf

sed -i -r 's|^.*listen.mode =.*|listen.mode = 0640|g' /etc/php*/php-fpm.d/www.conf

rc-update add php-fpm81 default

service php-fpm81 restart
```

The PHP-FPM defines a master process with a process pool for each
service request. By default, there\'s only one process pool, www.

Default values are good for starting, but will need tuning later. The
best is a static one, but testing is needed to get the right
configuration.

### Lighttpd and PHP-FPM

The web server comes with a minimal config file, so we must handle all
the required settings:

1. create the cgi directory aliasing, we will trick real cgi path by alising technique
2. enable the mod_alias at the config file, real path will be hidden due security
3. be sure and disable the fastcgi-php module cos we must use php by fpm module
4. and then enable the fastcgi-php-fpm specific module then
5. redefine the index filename handlers for security
6. write a much much better approach of the php handler in the local server using the socket
7. configure the php engine to use also the socket for direct connection locally only
8. restart the service to see changes at the browser
9. write a index file name with php info echoes!

```
mkdir -p /var/www/localhost/cgi-bin

sed -i -r 's#\#.*mod_alias.*,.*#    "mod_alias",#g' /etc/lighttpd/lighttpd.conf

sed -i -r 's#.*include "mod_cgi.conf".*#   include "mod_cgi.conf"#g' /etc/lighttpd/lighttpd.conf

sed -i -r 's#.*include "mod_fastcgi.conf".*#\#   include "mod_fastcgi.conf"#g' /etc/lighttpd/lighttpd.conf

sed -i -r 's#.*include "mod_fastcgi_fpm.conf".*#   include "mod_fastcgi_fpm.conf"#g' /etc/lighttpd/lighttpd.conf

sed -e '/index-file.names/ s/^#*/#/' -i /etc/lighttpd/lighttpd.conf

cat > /etc/lighttpd/mod_fastcgi_fpm.conf << EOF
server.modules += ( "mod_fastcgi" )
index-file.names += ( "index.php" )
fastcgi.server = (
    ".php" => (
      "localhost" => (
        "socket"                => "/var/run/php-fpm81/php81-fpm.sock",
        "broken-scriptfilename" => "enable"
      ))
    )
EOF

sed -i -r 's|^.*listen =.*|listen = /var/run/php-fpm81/php81-fpm.sock|g' /etc/php*/php-fpm.d/www.conf

sed -i -r 's|^.*listen.owner = .*|listen.owner = lighttpd|g' /etc/php*/php-fpm.d/www.conf

sed -i -r 's|^.*listen.group = .*|listen.group = lighttpd|g' /etc/php*/php-fpm.d/www.conf

sed -i -r 's|^.*listen.mode = .*|listen.mode = 0660|g' /etc/php*/php-fpm.d/www.conf

rc-service php-fpm81 restart

rc-service lighttpd restart

echo "<?php echo phpinfo(); ?>" > /var/www/localhost/htdocs/info.php
```

For testing, open a browser and go to `http://<webserveripaddres>/info.php`. 
You will see the info as used in production. There\'s no sense givig too much 
information to crackers. The "webserveripaddres" is the ip address of your setup/server
machine.

After that, all the files with php will be procesed faster than used a
host based. Under the `/var/www/localhost/cgi-bin` directory will be
shown as http://localhost/cgi-bin/ path.

## 3. The DBMS part: mysql/mariadb

Alpine Linux has dummy counterpart packages for those not changed from
*mysql* to *mariadb*.

#### mysql Installation

Take into consideration the user `mysql` was created during package
instalation. In the initialization section two users will be created in
database init: `root` and `mysql`, and at that point only if they are in
their respective system accounts, will they be able to connect to the
database service.

## 3 - Databases - MySQL postgreSQL SQLite ODBC

In alpine the only option is using Mariadb or Postgresql, ODBC only provides TDS and Postgresql, 
rest of options are only in edge or using other linuxes

#### adminer to MANAGE ANY DATABASE SERVERS

This part will use the previosu configurations to setup the DBMS web management, 
The most secure for web Mysql managemend its adminer, cos phpmyadmin its too polite
Adminer permits to manage any kind of database, including odbc, postgresql and mysql/mariadb 
in only one interface!

```
mkdir -p /usr/share/webapps/adminer

wget https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1.php -O /usr/share/webapps/adminer/adminer-4.8.1.php

ln -s adminer-4.8.1.php /usr/share/webapps/adminer/index.php

sed -i -r 's#\#.*mod_alias.*,.*#    "mod_alias",#g' /etc/lighttpd/lighttpd.conf

cat > /etc/lighttpd/mod_adminer.conf << EOF
alias.url += (  "/manag/adminer/" => "/etc/adminer/" )
$HTTP["url"] =~ "^/manag/adminer/" {
    dir-listing.activate = "disable"
    index-file.names := ( "index.php", "index.html", "adminer-4.8.1.php" )
}
EOF

itawxrc="";itawxrc=$(grep 'include "mod_adminer.conf' /etc/lighttpd/lighttpd.conf);[[ "$itawxrc" != "" ]] && echo listo || sed -i -r 's#.*include "mime-types.conf".*#include "mime-types.conf"\ninclude "mod_adminer.conf"#g' /etc/lighttpd/lighttpd.conf

sed -i -r 's#\#.*mod_alias.*,.*#    "mod_alias",#g' /etc/lighttpd/lighttpd.conf

sed -i -r 's#.*include "mod_cgi.conf".*#   include "mod_cgi.conf"#g' /etc/lighttpd/lighttpd.conf

checkssl="";checkssl=$(grep 'include "mod_adminer.conf' /etc/lighttpd/lighttpd.conf);[[ "$checkssl" != "" ]] && echo listo || sed -i -r 's#.*include "mod_cgi.conf".*#include "mod_cgi.conf"\ninclude "mod_adminer.conf"#g' /etc/lighttpd/lighttpd.conf

rc-update add lighttpd default

rc-service lighttpd restart
```

This configurations assumes you already runs all this guide!

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
apk add mysql mysql-client mariadb-doc mariadb-server-utils mariadb-mytop

mysql_install_db --user=mysql --datadir=/var/lib/mysql

rc-service mariadb start

mysqladmin -u root password toor

sed -i "s|.*max_allowed_packet\s*=.*|max_allowed_packet = 100M|g" /etc/mysql/my.cnf
sed -i "s|.*max_allowed_packet\s*=.*|max_allowed_packet = 100M|g" /etc/my.cnf.d/mariadb-server.cnf

sed -i "s|.*bind-address\s*=.*|bind-address=0.0.0.0|g" /etc/mysql/my.cnf
sed -i "s|.*bind-address\s*=.*|bind-address=0.0.0.0|g" /etc/my.cnf.d/mariadb-server.cnf

rc-service mariadb restart

rc-update add mariadb
```


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
apk add unixodbc psqlodbc freetds mdbtools-odbc 

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
odbcinst -u -d -f /tmp/tmppg.tmp

cat > /tmp/tmptds.tmp << EOF
[FreeTDS]
Description	= TDS module (Sybase/MS SQL)
Driver		= /usr/lib/libtdsodbc.so.0
CPTimeout	= 
CPReuse		= 
odbcinst -u -d -f /tmp/tmptds.tmp

cat > /tmp/tmpmdb.tmp << EOF
[MDBTools]
Description	= MDB modules for ODBC
Driver		= /usr/lib/odbc/libmdbodbc.so
CPTimeout	= 
CPReuse		= 
odbcinst -u -d -f /tmp/tmpmdb.tmp
```

> **Warning** the packages for mysql and sqlite are `sqliteodbc` and `mariadb-connector-odbc` but only available for edge.

> **Warning** if your alpine version is too older, you cannot use the mysql edge package for odbc unfortunatelly


## see also

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

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
