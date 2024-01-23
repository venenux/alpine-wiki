# alpine mysql professional

MySQL is the most popular database manager in free software for two simple 
reasons (which are not the best technical reasons):

* It's simple and very easy to use
* It is very similar to M$ crap SQL and is also used in other systems

In the world of Linux Alpine, there is a software that provides it, it is 
the MariaDB, here we have a brief of info about compatibility and differences, 
but in short there's no great differences, if you have doubts check the 
section MariaDB_vs_MySQL at the end

In the wiki there are two approaches for its use, the professional one 
(for servers and deploys, this document) and the fast and simple usage 
(for developers and/or enthusiasts), this document: [../tutorials/alpine-howto-mysql.md](../tutorials/alpine-howto-mysql.md)

## MariaDB

MariaDB is a community-developed fork of the MySQL relational database 
management system intended to remain free under the GNU GPL. It is notable 
for being led by the original developers of MySQL, who forked it due to 
concerns over its acquisition by Oracle.

This page assumed that you have a general knowledge about MariaDB, so if 
you are new to MySQL first take a look at the  [../tutorials/alpine-howto-mysql.md](../tutorials/alpine-howto-mysql.md) 
for information about how you can start with SQL and MySQL.

## Installation

Alpine Linux has dummy counterparts packages for those that are not close to 
that change from mysql to mariadb naming packages, please check the next section
page for more information.

Take in consideration that the user mysql was created during instalation of packages, 
in the initialization section two users will be created in database 
init: `root` and `mysql`, and in that point only if are in their respective system 
accounts, will be able to connect to the database service.

```
apk add mysql mysql-client
```

That will install the most used ones.. `mariadb-cient` and `mariadb-server`, 
rest of packages are brief described in next section for more information.

## Mysql alpine packages

Here are listed in orden of relevance for production server

| MySQL name package     | Since Alpine: | Brief usage                                                                     | Related package         |
|------------------------|---------------|---------------------------------------------------------------------------------|-------------------------|
| mysql                  | v2            | it's a dummy package to easy install of mariadb                                 | mariadb                 |
| mysql-client           | v2            | it's a dummy package to easy install of commands tools                          | mariadb-client          |
| mariadb                | v2            | server equivalent to mysql-server                                               | mariadb-common          |
| mariadb-client         | v2            | connection command line and tools                                               | mariadb-common          |
| mariadb-doc            | v3.0          | manpages are there!                                                             | man man-pages           |
| mariadb-connector-odbc | edge          | coding or making OS level connections, to any DB without libs install           | .                       |
| mariadb-connector-c    | v3.8          | coding connection on C sources                                                  | mariadb-connector-c-dev |
| mariadb-backup         | v3.8          | to external backup devices, not widely used, in past was inside mariadb package | .                       |
| mariadb-server-utils   | v3.8          | server commands not widely used, in past was inside mariadb package             | .                       |
| mariadb-dev            | v3.1          | Need for compilations depends on source code                                    | .                       |
| mariadb-test           | v3.3          | testing suite from MariaDB tools                                                | .                       |
| mariadb-mytop          | v3.9          | data performance monitoring                                                     | .                       |
| mariadb-plugin-rocksdb | v3.9          | plain key-value event relational for data                                       | .                       |
| mariadb-static         | v3.8          | static libs for static non depends linking in builds                            | .                       |
| mariadb-embedded       | v3.9          | the libmysqld identical interface as the C client                               | mariadb-embedded-dev    |
| mariadb-embedded-dev   | v3.9          | use the normal mysql.h and link with libmysqld instead of libmysqlclient        | mariadb-dev             |
| mariadb-openrc         | v3.8          | separate scripts, in past was embebed on server package                         | .                       |
| tzdata                 | v3            | need to process mysql tzinfo sql settings using `mysql_tzinfo_to_sql`           |                         |

## Initialization

The alpine package of MySQL/MariaDB are like normal tarball of MySQL one, admins 
must be know what they want.. there's no automatic window-like here.

The datadir are located to `/var/lib/mysql` and must be owned by the mysql user 
and group. You can modify this behavior but must edit the service file 
at `/etc/init.d` directory. Also, you need to set `datadir=<YOUR_DATADIR>` under 
section `[mysqld]` at the config file so.

* Initialize the main mysql database, and the data dir as standardized to `/var/lib/mysql` by the rc script
* Then initialize the service, root account and socket connection are enabled without password at this point
* Configure if will be run as service on each boot start of the computer machine
* Setup the root account by asignes a proper password, this are purely paranoid. due next step already do that!

```
mysql_install_db --user=mysql --datadir=/var/lib/mysql

rc-service mariadb start

rc-update add mariadb default

mysqladmin -u root password toor
```

After that, all are initializated to proceed with configuration, now can be done 
using the `mysql_secure_installation` script at the next section.

Alternative steps for first steps is the most recents versions of the package, 
the rc-scripts only flavour has the "setup" argument that provide a way to init 
the service configuration with `rc-service mariadb setup`

## Configuration

In order to finish setup into, now **MariaDB** provide this **script called 
`mysql_secure_instalation` that also are present as `mariadb-secure-installation` too**.
This script provides minimal and security setup to the database, and here are 
the questions made explained:

1. **Enter current password for root (enter for none)**: this are if you 
previously setup as we done in previous section a root password, just provide it 
and press enter, **must be provided due we already set previously** and from now, 
this sript will access to the engine and alter many setting on the database. 
Correct responds are: `OK, successfully used password, moving on...`
2. **Switch to unix_socket authentication [Y/n]** Setting the root password or 
using the unix_socket ensures that only admins can log into engine database. 
Since mysql 5.6 and mariadb 10.2 a new auth mechanish are set, by socket authentiaction, 
when system user are same as mysql/mariadb user, in this case, no password are need. 
In production servers this are not the case and must be disabled, so answer NO, 
and response will be: `... skipping.`
3. **Change the root password? [Y/n]** this answer are here only if the first 
one are just enter, or if can provide a better passowrd if no unix socket are 
set. Just press "n" only if you provided a good password, otherwise just change it, 
response will depends on your action. **IF you dont have a bash protected env and 
use it the previously command with mysqladmin.. you must change it here!**
4. **Remove anonymous users? [Y/n]** this permits remove the anonymous user 
created to log using socket authentication, only working on unix-like system. 
In any case, production system must remove it, so answer "Y" and the proper 
respond must be `... Success!.`
5. **Disallow root login remotely? [Y/n]** Normally, root should only be allowed 
to connect from 'localhost'. This ensures that someone cannot guess at the root 
password from the network. For sure answer Y and respond must be `... Success!.`
6. **Remove test database and access to it? [Y/n]** By default, MariaDB comes with 
a database named 'test' for stupid windosers and that anyone can access. This is 
also intended only for testing, and should be removed, so answer "Y" and proper 
respond must be `... Success!.`
7. **Reload privilege tables now? [Y/n]** Reloading the privilege tables will 
ensure that all changes made so far will take effect immediately, so answer "Y" 
and proper respond must be `... Success!.`

After reponse all the questions.. restart the service with `rc-service mariadb restart`

## Configuration files

Due today were influenced by systemd standardization, the famous `my.cnf` are 
not more the main config file for the server engine. Now only few variables are 
defined there and all the settings are provided by independent files into 
the `/etc/my.cnf.d/` directory, user own config files are under `~/.my.cnf` 
config file of each home dir, and are read after global ones; so then we have:

| Config file        | Path and name                    | Versions of Alpine | Contents to configure                     |
|--------------------|----------------------------------|--------------------|-------------------------------------------|
| my.cnf             | /etc/mysql/my.cnf                | v2 to v3.8         | All the directives, Global config file    |
| mariadb-server.cnf | /etc/my.cnf.d/mariadb-server.cnf | since 3.9          | First Global config file, main directives |
| .my.cnf            | $HOME                            | all                | user name only config directives          |

## Localization integration

This apps that depend on MySQL for timezone info started displaying the timezone 
as theiy own in, instead of just UTC, so we need the tzdata package as we described 
in the previous "Alpine packages" section before.

```
apk add tzdata

mysql_tzinfo_to_sql /usr/share/zoneinfo/ | mysql -u root -p mysql
```

Here we used `-p` becouse we already configured previously password accesss.

## Production settings

These setting are only recommended for some server settings, there are the 
recommendation for high production settings:

| Config setting               | Default         | Recommended        | Explanation                                                                                   |
|------------------------------|-----------------|--------------------|-----------------------------------------------------------------------------------------------|
| main ram                     | 2G              | 8G - 16G           | MariaDB/MySQL can run with 512M or 1G of ram, high production must use minimun of 4G or more. |
| data dir disk type           | any             | SSD                | MariaDB/MySQL must run with faster SSD if high request will be produced.                      |
| collation_server             | utf8_unicode_ci | utf8_unicode_ci    | With 8mb4 some characters are able to use more than a single byte.                            |
| character_set_client         | iso8859-1       | utf8 or utf8mb4    | Important due are standard, for left to right use 8mb4 flavor                                 |
| max_connections              | 151             | 100                | total_connections = total_processes * (total_threads + script_servers + 1)                    |
| max_heap_table_size          | 16M             | 32M                | allocate more memory to memory tables, for memory storage engines                             |
| tmp_table_size               | 16M             | 32M                | It allows the sub queries to remain more in memory, making them faster                        |
| join_buffer_size             | 32M             | 64M                | It allows the join queries to remain more in memory rather in temp files                      |
| innodb_file_format           | unset           | Barracuda          | will allow longer indexes for important most used tables                                      |
| innodb_large_prefix          | unset           | 1                  | Must set if Barracuda file format are choosen                                                 |
| innodb_buffer_pool_size      | 128M            | 456M               | hold as much tables and indexes in system memory as is possible                               |
| innodb_read_io_threads       | 16              | 32                 | On high I/O systems, a value greater than 1 may allow the disk I/O to be more sequential      |
| innodb_write_io_threads      | 16              | 32                 | Only if you have SSD storage for the data MySQL/MariaDB database and temp files               |
| innodb_buffer_pool_instances | 1               | 2 or 4             | Only for older MySQL/MariaDB engines,                                                         |
| innodb_io_capacity           | 200             | 1200 - 2600        | Only if you have SSD storage for the data MySQL/MariaDB database and temp files               |
| innodb_io_capacity_max       | 200             | 2400 - 5200        | Only if you have SSD storage for the data MySQL/MariaDB database and temp files               |

If you have SSD disks, use the recommended suggestion, otherwise, use minimum 
suggested. If you have physical hard drives, use `2000 * <number of active drives>` 
in the array. If using NVMe or PCIe Flash, much larger numbers as high as 200000 
can be used, but those lasted storage devices will have a short life of course.

Newer system Alpine packages can set in independent files in any case those 
commands always works and where are not apply just will ignore the output:

```
cat > /etc/my.cnf.d/mariadb-server-default-highload.cnf << EOF
[mysqld]
collation_server = utf8_unicode_ci
character_set_server = utf8
max_connections     = 200
max_heap_table_size = 32M
tmp_table_size      = 32M
join_buffer_size    = 64M
innodb_file_format  = Barracuda
innodb_large_prefix = 1
innodb_buffer_pool_size = 320M
innodb_flush_log_at_timeout = 3
innodb_read_io_threads  = 32
innodb_write_io_threads = 32
innodb_io_capacity     = 2000
innodb_io_capacity_max = 8000
innodb_buffer_pool_instances = 1
EOF

rc-service mariadb restart

rc-update add mariadb default
```


On older Alpine system must set config files for MAX ALLOWED PACKETS to minimun 
proper amount:

```
sed -i "s|.*max_allowed_packet\s*=.*|max_allowed_packet = 200M|g" /etc/mysql/my.cnf
sed -i "s|.*max_allowed_packet\s*=.*|max_allowed_packet = 200M|g" /etc/my.cnf.d/mariadb-server.cnf
```

Only allow local connections on cases where there's only one server or no 
expected to connect from others, DONT DO THAT IS YOU HAVE A PRODUCTION SERVER, 
instead use [fail2ban mysql jail](server-alpine-fail2ban-professional.md#mysql-jail):

```
sed -i "s|.*bind-address\s*=.*|bind-address=127.0.0.1|g" /etc/mysql/my.cnf
sed -i "s|.*bind-address\s*=.*|bind-address=127.0.0.1|g" /etc/my.cnf.d/mariadb-server.cnf
```

If are not in domain controller, dont search for hostnames to improve performance 
responses (ideal for local only servers):

```
sed -i "s|.*skip-networking.*|skip-networking|g" /etc/mysql/my.cnf
sed -i "s|.*skip-networking.*|skip-networking|g" /etc/my.cnf.d/mariadb-server.cnf
```

Set default charset to UTF8 only, cos some software cant interpreted the UFT8MB4 new one, 
in newer versions (since Alpine v3.9), just added a new file to added thus customization, 
but older versions (below Alpine v3.8) of the package does not have a charset section, 
so you must added manually to the main configuration in each respective section:

```
cat > /etc/my.cnf.d/mariadb-server-default-charset.cnf << EOF
[client]
default-character-set = utf8

[mysqld]
collation_server = utf8_unicode_ci
character_set_server = utf8

[mysql]
default-character-set = utf8
EOF
```

## Updating or comming from upgrading

Mayor Upgrades beetween Alpine linux version are so easy as change the repository 
version, but the MySQL/MariaDB engine need some extra steps when this are performed:

Upgrade databases on major releases Upon a major version release of mariadb (for 
example mariadb-10.1.10-1 to mariadb-10.1.18-1), it is wise to upgrade databases:

1. keep the old database (mysql sheme) structure of the engine daemon, currently 
this are not more the case, today this not make sense anymore
2. upgrade the MariaDB/MySQL packages, of course with must be done if the upgrade 
process to mayor alpine version does not!
3. run the `mysql_upgrade -u root -p` script, providing the password or root, 
(from the new package version) **against the old still-running database** (mysql sheme). 
This will produce some error messages; however, the upgrade will succeed.
4. Restart the service

If are unable to run `mysql_upgrade` because MySQL cannot start try run MySQL in 
safemode with `mysqld_safe --datadir=/var/lib/mysql/` command and then run 
the `mysql_upgrade -u root -p` script after.

## Relevant important notes

#### File system notes about the databases managed

**BTRFS Notes**


If the database (in /var/lib/mysql) resides on a btrfs file system, you should 
consider disabling Copy-on-Write for the directory before creating any database 
(schemes), after initialization you can enabled again. But .. on every database 
creation (scheme creation), you must disabled again, to avoid corrupted data.

**ZFS Bock sizes**

ZFS, unlike most other file systems, has a variable record size, or what is 
commonly referred to as a block size. By default, the recordsize on ZFS is 128KiB, w
hich means it will dynamically allocate blocks of any size from 512B to 128KiB 
depending on the size of file being written. Most RDBMSes work in 8KiB-sized 
blocks by default. Although the block size is tunable for MySQL/MariaDB use 
an 8KiB block size by default.

It is usually desirable to tune ZFS instead to accommodate the databases, using 
a command such as zfs set recordsize=8K /var/lib/mysql (or change /var/lib/mysql 
to the mount point where /var/lib/mysql resides) and in the interest of saving 
memory, it is best to simply disable ZFS's caching of the database's file data 
and let the database do its own job with zfs set primarycache=metadata /var/lib/mysql 
(or change /var/lib/mysql to the mount point where /var/lib/mysql resides).

But beware, these kinds of tuning parameters are only if RDBMSes are setup in 
dedicated partitions, if your root and of course database are all in one partition, 
dont do that.

#### Restore root password

```
rc-service mysql stop

kill  `cat /run/mysqld/mysqld.pid`

/usr/bin/mysqld --datadir=/var/lib/mysql --pid-file=/run/mysqld/mysqld.pid --skip-grant-tables --skip-networking &

mysql -e "ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';FLUSH PRIVILEGES;ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';FLUSH PRIVILEGES;set password = password('MyNewPass');"

kill  `cat /run/mysqld/mysqld.pid`

rc-service mariadb restart
```

## MariaDB vs MySQL

It is more a matter of compatibility than of performance and characteristics 
(with the arrival of MySQL v8) .. and it depends on whether there is a purely 
business and support approach "zero concern".

MySQL, being from Oracle, establishes limits if a license is not purchased, 
MariaDB has a large connection pool, more than 200,000 connections, while MySQL 
has a smaller connection pool if it is not licensed.

However, MariaDB does not support data masking and dynamic column while MySQL 
supports it, also MariaDB although it has 12 new storage engines while MySQL has 
less these are very new and MySQL's are widely known and tested.

In terms of performance, MariaDB is only a little faster than MySQL, this is 
because MySQL implements more business features, but this is only noticeable 
using these many features.

Which is more optimal this is not clear .. in general MySQL should be less, 
and MariaDB faster, there is a third option which is Percona which is the same 
MySQL service but with special aggressive optimization patches for servers. 
Percona mysql code must be compiled in Alpine linux.

#### Comparison table

| Characteristic             | MariaDB                                | MySQL                               |
|----------------------------|----------------------------------------|-------------------------------------|
| Storage Engines            | up to 12 but many in development stage | less but well tested                |
| Performance                | just a little faster                   | less, there is almost no difference |
| Initial version            | 2009 (5.3)                             | 1995 (3.0)                          |
| Data masking               | no                                     | yes                                 |
| dynamic columns            | no                                     | yes                                 |
| Monitoring                 | SQLyog                                 | MySQLworkbench                      |
| Routing                    | MariaDB MaxScale                       | Mysql Router                        |
| Analytics                  | MariaDB ColumnStore                    | not have                            |
| Git starred times (github) | around 3.6k                            | around 6k                           |

## About copyright material

**CC BY-NC-SA**: If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

Please check our [../alpine/copyright.md](../alpine/copyright.md).

