# alpine server filebrowser

`filebrowser` its a single application that provides web file browsing pretty ultra faster, 
result of combined technology of golang (as backend) and Vuejs (for frontend).


> Warning: unfortunatelly filebrowser its a compiled app and must be recompile, due muslc of alpine

## Preparations

* setup a hostname, here we use "filebrowser" string as hostname
* added and update normal repositories


```
hostname giteahost

echo 'hostname="filebrowser"' > /etc/conf.d/hostname 

echo "giteahost" > /etc/hostname

cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update
```

## Compilation and Installation

* install dependencies
* install indirect dependences: grep, lsof, less, curl, binutils, attr, lighttpd
* clone repository
* compiling
* installing to path

```
apk add npm nodejs go npm-bash-completion npm-doc go-doc git make

apk add bash coreutils grep lsof less wget binutils attr openssl lighttpd

export PAGER=less

mkdir ~/Devel && cd ~/Devel

git clone https://github.com/filebrowser/filebrowser && cd filebrowser

make build

cp filebrowser /usr/bin/
```

## Configurations

The filebrowser has internal defaults, can just mount the root and the database and that's all, 
the binary itselft is a selft hosted application, just runt and will assume current directory 
as defaults for all the paths

| Artifac         | Name             | Defaults instalation      | Internal defaults        |
| --------------- | ---------------- | ------------------------- | ------------------------ |
| Binary program  | filebrowser      | `/usr/bin/filebrowser`    | any path                 |
| Daemon script   | filebrowser      | `/etc/init.d/filebrowser` | N/A                      |
| Daemon user     | filebrowser      | `/var/lib/filebrowser/`   | N/A                      |
| Group user      | www-data         | `/var/www/`               | N/A                      |
| Rootdir (files) | FB_ROOT          | `/var/lib/filebrowser/`   | `./` (current path)      |
| DB file         | FB_DATABASE      | `/var/lib/filebrowser/filebrowser.db` | `./filebrowser.db` |
| Port socket     | FB_PORT          | 8080                      | 8080                     |
| Address socket  | FB_ADDRESS       | 0.0.0.0                   | 127.0.0.1                |
| BAse URL        | FB_BASEURL       | `/webfilebrowser`         | `/`                      |
| Config global   | FB_CONFIG        | `/etc/filebrowser/filebrowser.json` | `./defaults.json` |
| Admin web user  | FB_USERNAME      | `adminuser`               | `admin`                  |
| Admin password  |                  | Undefined if name changed | `admin`                  |
| Runing user     | filebrowser      | `/var/lib/filebrowser`    | N/A                      |
| Loggin files    | internally       | `/var/log/filebrowser.log` | stdout                  |

WE need to define the most important parts.. DB file, BAse URL and Port were will run:

##### running as unique web service

* Stop any web server and use file browser as the web service
* Run init configuration
* Setup admin user and password
* SEtup that first user as admin user
* Run ad daemon manually

```
service lighttpd stop && service nginx stop && service apache2 stop

filebrowser -d /var/lib/filebrowser/filebrowser.db config init -a 0.0.0.0 -l /var/log/filebrowser.log -p 80 -r /var/lib/filebrowser

filebrowser -d /var/lib/filebrowser/filebrowser.db users add adminuser password
 
filebrowser -d /var/lib/filebrowser/filebrowser.db users update adminuser --perm.admin=true

start-stop-daemon --start --quiet --make-pidfile --pidfile /run/filebrowser.pid --exec /usr/bin/filebrowser --name filebrowser --background -- -d /var/lib/filebrowser/filebrowser.db
```

> Warning: here we previously configure the program, 

### 1- Configure the Database

Choose the configuration option now:

##### Option 1.A Setup backend SQLITE

```
sed -i -r 's#PATH =.*.db#PATH = /var/lib/filebrowser/db/filebrowser.db#g' /var/lib/filebrowser/app.ini

service filebrowser start

rc-update add filebrowser default

service filebrowser status
```

##### Option 1.B Setup backend MySQL

```
apk add mariadb mariadb-client

sed -i "s|.*skip-networking.*|skip-networking|g" /etc/mysql/my.cnf
sed -i "s|.*skip-networking.*|skip-networking|g" /etc/my.cnf.d/mariadb-server.cnf

/etc/init.d/mariadb setup

/etc/init.d/mariadb start

rc-update add mariadb default

service filebrowser start

rc-update add filebrowser default

service filebrowser status
```

##### Option 1.C Setup backend PostgreSQL

```
sed -i "s|||g" /var/lib/postgresql/*/data/pg_hba.conf

apk add postgresql postgresql-client

/etc/init.d/postgresql setup

/etc/init.d/postgresql start

rc-update add postgresql default

service filebrowser start

rc-update add filebrowser default

service filebrowser status
```

### 2- configure the service

After check that is "running" you must setup graphically using a web browser, 
poiting to `http://localhost:3000`, in the case of this document should be 
pointing to `http://filebrowserhost.mydomain.com:3000` and a web landing will show.

The post-installation process happends when you visit the url with the browser.

**The post install page**, will be displayed and only are show when try to use 
the system for the first time, away of the starting page, by example if browse 
the repositories or try to login. You must not forgotten to setup that final 
installation process.

**Database configs** will be depending of the choice made in the steps avobe, 
just give the required credentials, only the case of sqlite does not need complications.

**Administrator account** must be configured before push "install filebrowser", the 
button at the end of the post-configuration page when you first visit the installation. 
Provide an username for admin user, take note "admin" are a reserved word so 
choose another name. after provide passowrd you will continue the installation.

**Redirection to landing** can be a problem cos after proceed the service of filebrowser 
will try to send to "localhost", to fix that, just go to the config file 
at the key value of `ROOT_URL` and check not contains "localhost" in it, change 
to the web url of the server in this case document is `http://filebrowserhost.mydomain.com:3000`

## ANNEXES

#### Setup filebrowser hook for Telegram

1. **Create a empty bot with botfather** visit https://t.me/BotFather and create or reuse a bot from list
    * Copy the **bot ID** (that must end in `_bot` ) and ..
    * Copy the **bot token** (a `number:string` separated format )..
2. **Chat or Add the bot to the desired group or channel** to be able to receive the results of the events of the hooks.
3. **Get the chat/group ID** for that use the https://t.me/myidbot inside of your group or channel, to get the working thing.. if you plan to receive individualy , get the chat id after chat it with the bot.
4. **Create the hook** go to `http://<url>/admin/hooks` (for global hook) or `https://<url>/org/<group>/settings/hooks` and create a new telegram hook
    * Setup the bot token you copied at the firts step
    * Setup the chat id you get it at the thirth step
    * Setup the desired events to be delivered to the chat/telegram
5. You can update and test the hook since version 1.12 of filebrowser.. this process work also for public deploys like forgetto or codeberg.

## see also

* [server-alpine-filebrowser-professional.md](server-alpine-filebrowser-professional.md)
* [server-alpine-filebrowser-professional.md](server-alpine-filebrowser-professional.md)

# LICENSE

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
