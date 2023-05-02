# alpine server gitea

Gitea is a community managed lightweight code hosting solution written in Go. 
It is a fork of Gogs.

**This is a fast guide how to using our alpine package, for more elaborated and professional, use the 
[server-alpine-gitea-professional.md](server-alpine-gitea-professional.md) document.**

## Preparations

* setup a hostname
* added and update normal repositories


```
hostname giteahost

echo 'hostname="giteahost"' > /etc/conf.d/hostname 

echo "giteahost" > /etc/hostname

cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update
```

## Installation

* install the gitea
* install indirect dependences: grep, lsof, less, curl, binutils, attr
* install direct dependences: git, gnupg, bash, coreutils
* alternate the edge repository [explanations check here](server-alpine-gitea-professional.md#installation)
* install gitea last version from edge repository no matter what alpine version you have
* restore normal repository

```
apk add bash coreutils grep lsof less curl binutils dialog attr

apk add git git-lfs gnupg gnupg1 sqlite sqlite libs openssl

export PAGER=less

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

## Configurations

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

Configuration must depends of the **backend database files**:

| Database   | Needs setup | Default Location |
| ---------- | ----------- | ---------------- |
| Sqlite     | No          | `/var/lib/gitea/db/gitea.db` |
| MySQL      | Yes         | `/run/mysqld/mysqld.sock` |
| PostgreSQL | Yes         | `localhost:5232` |

### 1- Configure the Database

Choose the configuration option now:

##### Option 1.A Setup backend SQLITE

```
sed -i -r 's#PATH =.*.db#PATH = /var/lib/gitea/db/gitea.db#g' /var/lib/gitea/app.ini

service gitea start

rc-update add gitea default

service gitea status
```

##### Option 1.B Setup backend MySQL

```
apk add mariadb mariadb-client

sed -i "s|.*skip-networking.*|skip-networking|g" /etc/mysql/my.cnf
sed -i "s|.*skip-networking.*|skip-networking|g" /etc/my.cnf.d/mariadb-server.cnf

/etc/init.d/mariadb setup

/etc/init.d/mariadb start

rc-update add mariadb default

service gitea start

rc-update add gitea default

service gitea status
```

##### Option 1.C Setup backend PostgreSQL

```
sed -i "s|||g" /var/lib/postgresql/*/data/pg_hba.conf

apk add postgresql postgresql-client

/etc/init.d/postgresql setup

/etc/init.d/postgresql start

rc-update add postgresql default

service gitea start

rc-update add gitea default

service gitea status
```

### 2- configure the service

After check that is "running" you must setup graphically using a web browser, 
poiting to `http://localhost:3000`, in the case of this document should be 
pointing to `http://giteahost.mydomain.com:3000` and a web landing will show.

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

## ANNEXES

#### Setup Gitea hook for Telegram

1. **Create a empty bot with botfather** visit https://t.me/BotFather and create or reuse a bot from list
    * Copy the **bot ID** (that must end in `_bot` ) and ..
    * Copy the **bot token** (a `number:string` separated format )..
2. **Chat or Add the bot to the desired group or channel** to be able to receive the results of the events of the hooks.
3. **Get the chat/group ID** for that use the https://t.me/myidbot inside of your group or channel, to get the working thing.. if you plan to receive individualy , get the chat id after chat it with the bot.
4. **Create the hook** go to `http://<url>/admin/hooks` (for global hook) or `https://<url>/org/<group>/settings/hooks` and create a new telegram hook
    * Setup the bot token you copied at the firts step
    * Setup the chat id you get it at the thirth step
    * Setup the desired events to be delivered to the chat/telegram
5. You can update and test the hook since version 1.12 of gitea.. this process work also for public deploys like forgetto or codeberg.

## see also

* [server-alpine-gitea-professional.md](server-alpine-gitea-professional.md)
* [server-alpine-gitea-professional.md](server-alpine-gitea-professional.md)

# LICENSE

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
