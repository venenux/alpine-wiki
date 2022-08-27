# alpine server gitea

Gitea is a community managed lightweight code hosting solution written in Go. 
It is a fork of Gogs. For a more simple guide use the [alpine-howto-gitea-package.md](../community-way/alpine-howto-gitea-package.md)

## Clarifications

1. Gitea was created by a group of users and contributors of the self-hosted Git service Gogs, 
It is a fork of Gogs and is written in Go.
2. There's two ways to deploy, server real one and docker containerized one, best 
performance its server real, and most isolated one are dockerizer way..
3. If even though docker always uses Alpine linux as images, and the software is 
alpine packages, it still has nothing to do with using alpine specific commands.
4. Git is the version control system (VCS) software behind gitea perse, so must 
be installed first. But repositories on server are not same as in clients.. server 
repositories are bare repositories.

## Requirements

* OS required tools:
    * bash
    * grep
    * lsof
    * less
    * curl
    * attr
* CVS command line
    * git
    * git-lfs
* Database backend:
    * sqlite
    * mysql
    * postgresql
* Auth and security:
    * mail
    * gnupg
    * openssl
    * pip
* Packages publish:
    * curl
    * docker

## Preparations

A hostname is a unique name created to identify a machine on a network, 
configured in `/etc/hostname`. (make sure to replace "giteahost" with your desired hostname):

```
hostname giteahost

echo 'hostname="giteahost"' > /etc/conf.d/hostname 

echo "giteahost" > /etc/hostname
```

You should also add the hostname to your hosts file (/etc/hosts), to 
obtain the best results if you have in internat network without DNS.


```
cat > /etc/hosts << EOF
127.0.0.1 giteahost.mydomain.com giteahost localhost.localdomain localhost
::1 localhost localhost.localdomain
EOF

cat > /tmp/tmp.tmp << EOF
127.0.1.1 giteahost.mydomain.com giteahost
EOF

sed -i '/127.0.0.1/  r /tmp/tmp.tmp' /etc/hosts && rm /tmp/tmp.tmp
```

## Installation

Gitea is a golang build application, so practically has no dependencies, but 
for minimal good working instance you should considered minimal installation 
packages. But due is a production server this will lack of doc, manpages and make packages.

1. added and update normal repositories
2. install direct dependences: git, gnupg, bash, coreutils
3. install indirect dependences: grep, lsof, less, curl, binutils, attr
4. setup the user of the gitea

```
cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add bash coreutils grep lsof less curl binutils dialog attr

apk add git git-lfs gnupg gnupg1 sqlite sqlite libs openssl

export PAGER=less
```

This guide does work either if are or not in main or edge the gitea package, 
take note, do not install any more from edge.. so in fact all gitea dependencies 
must be listed and installed before gitea and edge brand are activated, so the following process will guide and show you how to do that; first gain root privileges or access ssh to your alpine server and then:

4. alternate edge repositories (do that only if your alpine version are over 3.9)
5. install gitea from edge repository
6. restore normal repository

```
cat >> /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
http://dl-cdn.alpinelinux.org/alpine/edge/main
http://dl-cdn.alpinelinux.org/alpine/edge/community
EOF

apk update --allow-untrusted

apk add gitea --allow-untrusted

cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update --allow-untrusted
```

**WARNING** if your version of alpine are more ancient like 3.6 or 3.8, do not use 
the edge version, it need upgrading almost to alpine 3.10 to use it, so if you 
are using a older hardware not supported by recent linux kernel, upgrade to alpine 3.10 
and then you can follow this manual. Otherwise just avoit the edge swicht and 
install the normal already provided older gitea package.


## Configurations

Gitea configurations are defined by the gitea service scrip, after install 
need a backend database, and also a dedicated user, also has configurations files.

| Artifac         | Name             | Defaults or packaged    | Customizable |
| --------------- | ---------------- | ----------------------- | ------------ |
| Binary program  | gitea            | `/usr/bin/gitea`        | no           |
| Daemon script   | gitea            | `/etc/init.d/gitea`     | no           |
| Daemon user     | gitea            | `/var/lib/gitea/`       | no           |
| Group user      | www-data         | `/var/www/`             | no           |
| Working dir     | GITEA_WORK_DIR   | `/var/lib/gitea/`       | yes          |
| Customizing     | GITEA_CUSTOM     | `/var/lib/gitea/custom/` | yes,depends  |
| Config global   | gitea.ini        | `/etc/gitea/app.ini`    | yes,depends  |
| Web files       | STATIC_ROOT_PATH | `/usr/share/webapps/gitea/` | no           |
| Data files      | APP_DATA_PATH    | `/var/lib/gitea/data/`  | depends      |
| Git repositories | GITEA_GIT_DIR   | `/var/lib/gitea/git`   | yes          |
| Loggin files    | internally       | `/var/log/gitea`       | no           |
| Database files  | Sqlite/MariaDB/PG | `/var/lib/gitea/db/gitea.db` | yes          |

**Gitea runs as `gitea` user, and `www-data` group**, so are compatible with any web 
deploy in system webservers packages of alpine repositories, but not with any 
other external installation if does not are same as.

Gitea has two configuration files, the system defaults at `/var/lib/gitea/conf/app.ini` 
and modifiable package defaults, at `/etc/gitea/app.ini`. Original files are 
in `/usr/share/webapps/gitea` and are defaults non-modifiable.

**Gitea can be customized**: just take same path from `/usr/share/webapps/gitea/` and 
put in same manner at `/var/lib/gitea/custom/` place.

For alterations see next sections where are defined initialization, customization and configurations.

## Initialization

Gitea just after install does not need many configurations, the daemon service 
will init all the needs, but forced setup gui will be need after initialize

1. start from init script
2. make enable the init script
3. check the runing service
4. visit the gitea service using your web browser

```
service gitea start

rc-update add gitea default

service gitea status
```

After check that is "running" you must setup graphically using a web browser, 
poiting to `http://localhost:3000`, in the case of this document should be 
pointing to `http://giteahost.mydomain.com:3000` and a web landing will show.

## Post install

Using your web browser and pointing to the gitea url path, you will be 
redirected to the post install page, but first you must prepare the 
backend database to be used by the service:

#### Post installation with sqlite

There's no need of preparation, just proceed to "post installation process" section.

#### Post installation with mysql

TODO:

#### Post installation with postgresql

TODO:

#### Post install process

The post-installation process happends when you visit the url with the browser.

**The post install page**, will be displayed and only are show when try to use 
the system for the first time, away of the starting page, by example if browse 
the repositories or try to login. You must not forgotten to setup that final 
installation process.

**Database configs** will be depending of the choice made in the steps avobe, 
just give the required credentials, only the case of sqlite does not need complications.

**Administrator account** must be configured before push "install gitea", the 
button at the end of the post-configuration page when you first visit the installation. 
Provide an username for admin user, take note "admin" are a reserved word so 
choose another name. after provide passowrd you will continue the installation.

**Redirection to landing** can be a problem cos after proceed the service of gitea 
will try to send to "localhost", to fix that, just go to the config file 
at the key value of `ROOT_URL` and check not contains "localhost" in it, change 
to the web url of the server in this case document is `http://giteahost.mydomain.com:3000`

**Theme and templates** are already mentioned in the configuration section, 
just take same path from `/usr/share/webapps/gitea/` and put in same manner 
at `/var/lib/gitea/custom/` place, by example to customize default landing page, 
just take a copy of the `/usr/share/webapps/gitea/templates/home.tmpl` and put 
modified one at `/var/lib/gitea/custom/templates/home.tmpl` as well.


## Tunning instances

Gitea is a single application and can work as instanciating a working path, 
so means you can run as normal application or as system service, inclusive 
you can also run multiple instances in sabe server:

#### System running

Gitea binary itselt cannot be start alone, without parameters will put lot of 
directories and files in the default current path, so to start to use must be 
using the service from the package.

1. Start from init script!
2. Make enabled the init script!
3. Stop from init script

```
rc service gitea start

rc-update add gitea default
```

To stop just run `rc-service gitea stop`

#### Standard running

A manual start without init script can be done but its recommended to 
indicate to use the files installed on the system (by example) as is:

1. stop any running instance
2. make a command to run with proper arguments

```
rc-service gitea stop

GITEA_WORK_DIR='/var/lib/gitea' /usr/bin/gitea web --config /etc/gitea/app.ini
```

This commands will start the gitea manually as standar alone, but 
will use the config files and installed files from the package.

#### Multiple instances

As same manner you can setup multiple instances of gitea, by many ways:

* By just run "Standar" command with specific alternate config files, but using new user
* By just run "Service" fork new unit file with alternate config files, but using new user

By example to run another instance using the system files as starting 
point **as Standard alone** application:

1. stop the main daemon
2. added a shell restricted and setup a new restricted user for instance of gitea "2"
3. create the new user that will run the another instance of gitea
4. copy the config file to use as template config file for instance of gitea "2"
5. create directories for future new files of the instance tha will run
6. change the variables and configuration to point to the new user of the instance
7. fix and set right permissions for the new user of the instance "2"
8. run the new instance **as "Standard" alone** instance named gitea "2"

```
rc-service gitea stop

apk add tcsh

add-shell '/bin/csh'

adduser -S -D -h /var/lib/gitea2 -s /bin/bash -g '' gitea2

cp /etc/gitea/app.ini /var/lib/gitea2/gitea2.ini

mkdir -p /var/lib/gitea2/db
mkdir -p /var/lib/gitea2/log
sed -i -r 's#ROOT = /.*#ROOT = /var/lib/gitea2/git#g' /var/lib/gitea2/gitea2.ini
sed -i -r 's#RUN_USER.*#RUN_USER = gitea2#g' /var/lib/gitea2/gitea2.ini
sed -i -r 's#APP_DATA_PATH.*#APP_DATA_PATH = /var/lib/gitea2/data#g' /var/lib/gitea2/gitea2.ini
sed -i -r 's#PATH = /.*#PATH = /var/lib/gitea/db/gitea2.db#g' /var/lib/gitea2/gitea2.ini
sed -i -r 's#^ROOT_PATH = /.*#ROOT_PATH = /var/lib/gitea2/log#g' /var/lib/gitea2/gitea2.ini

chown -R gitea2:www-data /var/lib/gitea2
chmod 0755 /var/lib/gitea2/db
chmod 0755 /var/lib/gitea2/log
chmod 0755 /var/lib/gitea2

GITEA_WORK_DIR='/var/lib/gitea2' /usr/bin/gitea web --config /var/lib/gitea2/gitea2.ini
```

By example to run another instance using the system files as starting 
point **as System running** application:

1. stop the main daemon and setup a new restricted user for instance of gitea "2"
2. create the new user that will run the another instance of gitea
3. copy the config file to use as template config file for instance of gitea "2"
4. create directories for future new files of the instance tha will run
5. change the variables and configuration to point to the new user of the instance
6. copy the unit service to use a new service template
7. change the variables and configuration to point to the new instance user
8. fix and set right permissions for the new user of the instance "2"
9. run the new instance **as "Standard" alone** instance named gitea "2"

```
rc-service gitea stop

apk add tcsh

add-shell '/bin/csh'

adduser -S -D -h /var/lib/gitea2 -s /bin/bash -g '' gitea2

cp /etc/gitea/app.ini /var/lib/gitea2/gitea2.ini

mkdir -p /var/lib/gitea2/db
mkdir -p /var/lib/gitea2/log
sed -i -r 's#ROOT = /.*#ROOT = /var/lib/gitea2/git#g' /var/lib/gitea2/gitea2.ini
sed -i -r 's#RUN_USER.*#RUN_USER = gitea2#g' /var/lib/gitea2/gitea2.ini
sed -i -r 's#APP_DATA_PATH.*#APP_DATA_PATH = /var/lib/gitea2/data#g' /var/lib/gitea2/gitea2.ini
sed -i -r 's#PATH = /.*#PATH = /var/lib/gitea/db/gitea2.db#g' /var/lib/gitea2/gitea2.ini
sed -i -r 's#^ROOT_PATH = /.*#ROOT_PATH = /var/lib/gitea2/log#g' /var/lib/gitea2/gitea2.ini

cp /etc/init.d/gitea /etc/init.d/gitea2

sed -i -r 's#name=.*#name=gitea2#g' /etc/init.d/gitea2
sed -i -r 's#command_user=.*#command_user=gitea2#g' /etc/init.d/gitea2
sed -i -r 's#/etc/gitea/app.ini#/var/lib/gitea2/gitea2.ini#g' /etc/init.d/gitea2
sed -i -r 's#/var/lib/gitea#/var/lib/gitea2#g' /etc/init.d/gitea2
sed -i -r 's#/var/log/gitea#/var/lib/gitea2/log#g' /etc/init.d/gitea2
sed -i -r 's#/run/gitea.pid#/run/gitea2.pid#g' /etc/init.d/gitea2

chown -R gitea2:www-data /var/lib/gitea2
chmod 0755 /var/lib/gitea2/db
chmod 0755 /var/lib/gitea2/log
chmod 0755 /var/lib/gitea2

rc service gitea2 start

rc-update add gitea2 default
```

**CAUTION** if you runs multiple instances, each one must have different port, 
this means you must check in the config `app.ini` file (like the `gitea2.ini` )
that after the póst setup procedure, the port are correct and different, if not just, 
before or after change it with `HTTP_PORT` and `ROOT_URL` keys.

## Serving web gui

The gitea by itselft its also a web service, but you have three options to made this:

1. **Root hijacking web server**: by configuring as the main web service, this is 
using the port 80 instead of the default 3000 number. Of course the disadvantage 
is that will be the only web service over the standar http port
2. **Proxy [sub]domain web service**: by configurin as service behind another web server, 
but using a hole domain for the service, this still is a variant of the first case, 
cos will run behind a web server, that will reverse proxy the resquest to the gitea service, 
this setup is only usefully if you run multiple domains in same web service, so gitea 
dont hijack the hole web service, and only hijack one domain of the web service.
3. **Proxy subpath of the web service**: by configuring as part of the same web service, 
just inside a path, event a hole domain or subdomnain, this is pretty usefully cos the 
service of gitea will coexist with others in sabe path domain. This is a variant of 
the second case, using a web server as reverse proxy.

The options 2 and 3 with usage of domain filtering is useful for multiple instances, 
and the real reason of this mixed configuration of webserver+gitea setup.

#### Root hijacking web server

Just remove or stop any service over the web http port and configure the gitea service 
over the web http standart port, by default 80 for http. The domain or any ip request 
over the service will show the gitea service as web page.

#### Proxy [sub]domain web service

This will need a web server and gitea service, the web server will do those process:

1. filter the domain
2. trap the request and reverse/proxy to the backend service (gitea) port

**Usin apache2**

Install the apache2 server as the tutorial [server-alpine-apache2-professional.md](server-alpine-apache2-professional.md)
no matter if have support for ssl, use the part of the [Apache2 alpine proxy modules setup](server-alpine-apache2-professional.md#apache2-alpine-proxy-modules-setup)
then setup a proxy by addiding a new file for gitea redirections; 
the use of domain filtering is useful for multiple instances, and the 
real reason of this mixed configuration of webserver+gitea setup.

```
apk add apache2 apache2-utils apache2-error apache2-proxy-html apache2-proxy

mkdir -p /var/www/localhost/htdocs /var/log/apache2

sed -i -r 's#^Listen.*#Listen 80#g' /etc/apache2/httpd.conf
sed -i -r 's#^ServerTokens.*#ServerTokens Minimal#g' /etc/apache2/httpd.conf

chown -R apache:www-data /var/www/localhost/
chown -R apache:wheel /var/log/apache2

mkdir -p /var/www/localhost/cgi-bin

sed -i -r 's#.*LoadModule.*modules/mod_cgid.so.*#LoadModule cgid_module modules/mod_cgid.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_cgi.so.*#LoadModule cgi_module modules/mod_cgi.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_alias.so.*#LoadModule alias_module modules/mod_alias.so#g' /etc/apache2/httpd.conf

sed -i -r 's#/usr/lib/libxml2.so#/usr/lib/libxml2.so.2#g' /etc/apache2/conf.d/proxy-html.conf
sed -i -r 's#.*ScriptAlias /cgi-bin/.*#    ScriptAlias /cgi-bin/ "/var/www/localhost/cgi-bin"#g' /etc/apache2/httpd.conf

cat >> /etc/apache2/conf.d/gitea.conf << EOF
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyRequests off
    AllowEncodedSlashes NoDecode
    ProxyPass / http://127.0.0.1:3000/ nocanon
</VirtualHost>
EOF

rc-update add apache2 default

rc-service apache2 restart
```

**Usin lighttpd**

For this, the redirection to the gitea service port is configured, 
it can be through the root route or it can be by filtering the domain, 
the use of domain filtering is useful for multiple instances, and the 
real reason of this mixed configuration of webserver+gitea setup.

```
apk add lighttpd gamin

mkdir -p /var/www/localhost/htdocs /var/lib/lighttpd

chown -R lighttpd:lighttpd /var/www/localhost/
chown -R lighttpd:lighttpd /var/lib/lighttpd
chown -R lighttpd:lighttpd /var/log/lighttpd

sed -i -r 's#\#.*server.port.*=.*#server.port          = 80#g' /etc/lighttpd/lighttpd.conf
sed -i -r 's#\#.*server.event-handler = "linux-sysepoll".*#server.event-handler = "linux-sysepoll"#g' /etc/lighttpd/lighttpd.conf

mkdir -p /var/www/localhost/cgi-bin
sed -i -r 's#\#.*mod_alias.*,.*#    "mod_alias",#g' /etc/lighttpd/lighttpd.conf
sed -i -r 's#.*include "mod_cgi.conf".*#   include "mod_cgi.conf"#g' /etc/lighttpd/lighttpd.conf

sed -i -r 's#\#.*mod_accesslog.*,.*#    "mod_accesslog",#g' /etc/lighttpd/lighttpd.conf
sed -i -r 's#\#.*mod_setenv.*,.*#    "mod_setenv",#g' /etc/lighttpd/lighttpd.conf

sed -i -r 's#\#.*mod_redirect.*,.*#    "mod_redirect",#g' /etc/lighttpd/lighttpd.conf
sed -i -r 's#\#.*mod_proxy.*,.*#    "mod_proxy",#g' /etc/lighttpd/lighttpd.conf

cat > /etc/lighttpd/mod_gitea.conf << EOF
\$HTTP["host"] =~ ".*" {
  \$HTTP["url"] =~ "(^/(\$))" {   
    proxy.server  = ( "" => ("" => ( "host" => "0.0.0.0", "port" => 3000 ))) 
  }
}
EOF

itawxrc="";itawxrc=$(grep 'include "mod_gitea.conf' /etc/lighttpd/lighttpd.conf);[[ "$itawxrc" != "" ]] && echo listo || sed -i -r 's#.*include "mime-types.conf".*#include "mime-types.conf"\ninclude "mod_gitea.conf"#g' /etc/lighttpd/lighttpd.conf

rc-update add lighttpd default

rc-service lighttpd restart
```

#### Proxy [sub]path web service

**Using apache2**

Just the same as the previous method but using a path, in both gitea and apache2.

**Using lighttpd**

Unfortunately you can't configure lighttpd for a gitea sub path, becouse the issue https://github.com/gogs/gogs/issues/4741 
gitea becomes the drama object of a couple of developers that dont care about flexibilty.

So the only way its by the combination of the previous method, using the apache2 
for the real proxy reverse redirection and lighttpd as real frontend web service.

1. setup the proxy reverse with the apache2 method but using sub path
2. setup the proxy reverse with the lighttpd but using sub path to the apache web service

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

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
