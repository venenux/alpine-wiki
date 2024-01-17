fail2ban for alpine linux
==========================

Monitor the **log** of common services to spot patterns in authentication failures 
and takes configured actions based on those failures.

Oficial alpine wiki is a crap so we started a professional focused alpine docuentation.

> **Warning**: you should care to read LICENSE of this document, at the end here!

## Introduction

When fail2ban is configured to monitor the logs of a service, it looks at a filter 
that has been configured specific to that service. The filter is designed to 
identify authentication failures for that specific service through the use of 
(unfortunatelly) complex regular expressions and then translated to a rule 
for banning offending origin.

### Regular expressions

Regular expressions are a common templating language used for pattern matching, 
basically it consists of two types of characters:

* the regular literal characters and
* the metacharacters

These metacharacters are the ones which give the power to the regular expressions.
It defines these regular expression patterns into an internal variable `failregex` 
and basically for the **filter** of the **log**.

### Filters and failregex

By default, Fail2ban includes **filter files** for common services. When a **log** 
from any **service**, like a web server, matches the `failregex` in its filter, a 
predefined **action** is executed for that service.

### Actions

The **action** can be configured to do many different things, with defaults to `ban` 
the offending host/IP address by modifying the local **firewall rules**.

Those actions can be expanded to, for example, send an email to your system administrator.

### Retries

By default, action will be taken when a **number authentication failures** have 
been detected **in a period or time** defined in minutes, with a defined default 
ban time. This is configurable.

### Jails

The gruping of **retries** + **actions** + **filters** using **regular expressions**, 
are **defined as jails**, each one contains those definitions and will be 
**translated into a set of rules for the firewall**.

When using the default **iptables firewall**, fail2ban creates a new set of firewall 
rules, also called a chain, when the service is started. It adds a new rule to 
the INPUT chain that sends all TCP traffic directed at port of the service to 
the new chain. In the new chain, it inserts a single rule that returns to the INPUT chain.
The chain and associated rules are removed if the Fail2ban service is stopped.

## fail2ban alpine

Fail2ban is configured through several files located within a hierarchy under 
the `/etc/fail2ban/` directory as default.

The package was started since 0.9 but at alpine 3.9 was packaged a 0.10 stable 
version of fail2ban so any tutorial of howto will for for any alpine version.

### package

| alpine package  | since  | Brief usage                     | Related package       |
| --------------- | ------ | ------------------------------- | --------------------- |
| fail2ban        | v3.8   | Main fail2ban service and files | monit whois py3-dnspython  |
| fail2ban-test   | v3.14  | fail2ban helper test scripts    |                       |
| fail2ban-openrc | v3.10  | init scripts and startup files  | monit whois py3-dnspython  |
| fail2ban-doc    | v3.10  | manpages and extra files        | man-pages mandoc      |
| fail2ban-pyc    | v3.18  | compiled version of the pythons | fail2ban whois monit  |

### Configurations

The configurations are defined by the fail2ban service scrip, after install 
need a backend database, and also a dedicated user, also has configurations files.

| Artifac         | Name             | Defaults or packaged       | Customizable |
| --------------- | ---------------- | -------------------------- | ------------ |
| Binary program  | fail2ban-server  | `/usr/bin/fail2ban-server` | no           |
| Binary manager  | fail2ban-client  | `/usr/bin/fail2ban-client` | no           |
| Daemon script   | fail2ban         | `/etc/init.d/fail2ban`     | no           |
| Daemon user     | fail2ban         | `/var/lib/fail2ban/`       | opt. run as root |
| Daemon socket   | fail2ban.sock    | `/var/run/fail2ban/fail2ban.sock` | no,depends |
| Loggin files    | logtarget        | `/var/log/fail2ban.log`    | no,depends   |
| Database files  | dbfile           | `/var/lib/fail2ban/fail2ban.sqlite3` | no,depends |
| Config dir      | N/A              | `/etc/fail2ban/`           | no           |
| Config global   | fail2ban.conf    | `/etc/fail2ban/fail2ban.conf` | no           |
| Config file     | jail.conf        | `/etc/fail2ban/jail.conf`  | no           |
| Customizing     | jail.local, *.local | `/etc/fail2ban/jail.d/`    | yes,depends  |

### What you should know

Default configuration files cannot be touch, are only references and managed by 
the package upgrades, server own changes may be incompatible with some future 
versions, you shouldn't edit it in-place. In resume:

The `fail2ban.conf` file configures some operational settings like the way the 
daemon logs info, and the socket and pid file it will use. The main configuration, 
however, is specified in the files that define the per-application `jails`.

1. Don't change `jail.conf`, instead just create `jail.local` and only put modified variables only.
2. Don't change `filter.conf`, Instead, create `filter.local` and insert only override/append settings.

Any values defined in `jail.local` will override those in `jail.conf` but if none 
is defined so will use the defaults in `jail.conf` same for `filter.conf`.

### instalacion fail2ban alpine

* update apk reposiotries
* installed fail2ban packages with ipv6 support included also
* installed required packages becouse alpine developers are stupid
* add to init the startup script service
* before configure be sure to stop the service

```
apk update

apk add fail2ban iptables ip6tables

apk add monit whois py3-dnspython fail2ban-pyc logrotate python3

rc-update add fail2ban

/sbin/service fail2ban stop
```

### configuration fail2ban with defaul firewall

* enable iptables with ipv6 support with also ipv4
* create the local jail config file
    * by default alpine should use common as debian, as geento does
    * dont use dns, attackers can change the domain with fake dns
    * ban the ofending address with 16 minutes bantime, then will be unbaned
    * window of 10 minutes to pay attention to when looking for repeated attakers
    * maximun failed attemps in the last window of "findtime" period
    * ignore the local loopback ip address
    * ignore the seflt ip address of the server or machine
    * increment the ban time if we found more attemps on each new try attack
    * do not ban more than six days to avoid changed dinamic ip addresses
    * search on all jails, so if multiple services are under attackers
    * use non shitstemd service backend, cos we used openrc
    * auto enconding of the log files text to filter on searchs
    * disable all jails, we only enabled only specific services
    * use normal non aggressive mode for banning actions
    * use the iptables firewall backend
* start the service
* restart log service so be in sync all the services

```
rc-update add ip6tables && rc-update add iptables 

cat > /etc/fail2ban/jail.local << EOF
[INCLUDES]
before = paths-debian.conf
[DEFAULT]
usedns               = no
bantime              = 16m
findtime             = 10m
maxretry             = 2
ignoreip             = 127.0.0.1
ignoreself           = true
bantime.increment    = true
bantime.maxtime      = 6d
bantime.overalljails = true
backend              = polling
logencoding          = auto
enabled              = false
mode                 = normal
banaction            = iptables-multiport
EOF

/sbin/service fail2ban restart

/sbin/service syslog restart || /usr/sbin/service rsyslog restart
```

## Firewall and services

By default, Fail2ban uses iptables. However, configuration of most firewalls 
and services is straightforward. Unfortunatelly firewalling packages in alpine 
is so crap so we have only one options (without complicated customizations) 
so relies on `ufw`. Inclusive shorewall/awall is crap in such configurations.

### configuring fail2ban with shorewall default action

Any shorewall or awall on alpien have issues, and specially becouse alpine 
developers dont have a good support on xquisite configs, so is not recommended.

Upstream fail2ban also dont have good behaviour with shorewall, see 
https://github.com/fail2ban/fail2ban/issues/2031#issuecomment-361612869 
cos **shorewall action does not have any "anchor" ATM, neither chain nor table or 
something similar**.

Alpine awall it has some improvements and now does not depends on shorewall, 
but still is not well integrated to, so, avoid to use it if you install fail2ban.

### configuring fail2ban with firewalld default action

Not recommended, is a feladora python based crap with multiple upstream issues:

* https://github.com/fail2ban/fail2ban/issues/1609#issuecomment-303085942
* https://github.com/fail2ban/fail2ban/issues/2666#issuecomment-601620071
* https://github.com/fail2ban/fail2ban/issues/2503#issuecomment-533105500

Firewalld has two backends; iptables and nftables. In the iptables backend the 
generic "accept existing connections" occurs before rules addded via `--direct` 
are executed and therefore rules added to block traffic won't actually block 
pre-existing connections. When the nftables backend was added this was fixed: 
the rules added with --direct are always executed before the generic "accept 
existing connections".

Conclusion: firewalld always have default tricks not managed by fail2ban!

### configuring fail2ban with ufw default action

This will sustitute the iptables fully by ufw (but still you will need iptables):

* installed fail2ban packages with ipv6 support included also ufw
* add to init the **startup script services in specific order**
* before configure be sure to stop the service
* enable iptables with ipv6 support with also ipv4
* create the local jail config file
    * by default alpine should use common as debian, as geento does
    * dont use dns, attackers can change the domain with fake dns
    * ban the ofending address with 16 minutes bantime, then will be unbaned
    * window of 10 minutes to pay attention to when looking for repeated attakers
    * maximun failed attemps in the last window of "findtime" period
    * ignore the local loopback ip address
    * ignore the seflt ip address of the server or machine
    * increment the ban time if we found more attemps on each new try attack
    * do not ban more than six days to avoid changed dinamic ip addresses
    * search on all jails, so if multiple services are under attackers
    * use non shitstemd service backend, cos we used openrc
    * auto enconding of the log files text to filter on searchs
    * disable all jails, we only enabled only specific services
    * use normal non aggressive mode for banning actions
    * use the ufw firewall backend
* start the service
* restart log service so be in sync all the services

```
apk update && apk add fail2ban ufw ip6tables iptables whois py3-dnspython logrotate

rc-update add ip6tables && rc-update add iptables && rc-update add ufw && rc-update add fail2ban 

/sbin/service fail2ban stop

ufw enable

cat > /etc/fail2ban/jail.local << EOF
[INCLUDES]
before = paths-debian.conf
[DEFAULT]
usedns               = no
bantime              = 16m
findtime             = 10m
maxretry             = 2
ignoreip             = 127.0.0.1
ignoreself           = true
bantime.increment    = true
bantime.maxtime      = 6d
bantime.overalljails = true
backend              = polling
logencoding          = auto
enabled              = false
mode                 = normal
banaction            = ufw
EOF

/sbin/service fail2ban restart

/sbin/service syslog restart || /usr/sbin/service rsyslog restart
```


> **Warning** this configuration is not recomended there are multiple issues with it, iptables based backend firewall is the most working solution

The developers dont recommend default firewall action to ufw, 
as https://github.com/fail2ban/fail2ban/discussions/3401#discussioncomment-4075123 
at the end lines, with further explanations.

The ufw is recommended to scanning defeat, because ufw relies on iptables, see 
next section for:

### configuring fail2ban with ufw loging scaning

This will uses the ufw log to ban scaning attemps.

* First configure using default iptables firewall backend as first section here!
* Later add this configurations as by runing those commands:

```
cat > /etc/fail2ban/filter.d/ufw-port-scan.conf << EOF
[Definition]
failregex = ^\s*\S+ kernel:(?: +\[[^\]]+\])? \[UFW (?:LIMIT )?BLOCK\] (?:\b(?:IN=\w+|OUT=|(?:(?!OUT=|IN=)[A-Z]+=[^ \[]*)+) )*SRC=<ADDR> DST=\S+
ignoreregex =
EOF

cat > /etc/fail2ban/jail.d/03-ufw-port-scan.local << EOF
[ufw-port-scan]
logpath  = /var/log/ufw.log
protocol = all
maxretry = 10
EOF

/sbin/service fail2ban restart

/sbin/service syslog restart
```

If you have a very older fail2ban installation this idea is derived from 
the site https://blog.fernvenue.com/archives/ufw-with-fail2ban/

## jails fail2ban

So we will avoid unification of files so modularization will provide 
organization over independient services, by example we can have ssh, and smtp 
but if we uninstall or changed smtp so the script can manage the jails wihout 
affect the ssh service.. so we can remove the smtp jail independiently.

Those jails will depends only on enabled services so you must learn about 
regular expressions and filtering.. we here only provide default most close 
configuration and how you should made each one, using always modularization!

1. firs of all enable relapse jails so attakers will be punished more harder
2. always enable the ssh service jail
3. dont enable any jail that don have respective service installed

### relapse jail

```
cat > /etc/fail2ban/jail.d/00-recidive.conf << EOF
[recidive]
enabled   = true
bantime   = 8d
findtime  = 1d
EOF

/sbin/service fail2ban restart

/sbin/service syslog restart
```

### ssh jail alpine

In this example perfectly prepared for any professional server, 
we used the already packaged `alpine-sshd` files, already provided 
since fail2ban 0.10 version into alpine v3.10 and improved in last versions.

Also in this service we add a custom 19226 port to the internal ssh forked 
service, cos in this example we have two ssh services running. If you 
dessire, or/and if you changed the ssh port, just change "ssh" to the port number 
and that's all, if you havbe multiple ssh services you can defined multiple 
port numbers. But if you renamed the sections `sshd-key` by example you 
must define wicht filter will be used, as we done here:

```
cat > /etc/fail2ban/jail.d/00-sshd.local << EOF
[sshd]
enabled  = true
mode     = aggressive
filter   = alpine-sshd
port     = ssh,19226
logpath  = /var/log/messages
maxretry = 2
maxlines = 3
[sshd-ddos]
enabled  = true
mode     = aggressive
filter   = alpine-sshd-ddos
port     = ssh,19226
logpath  = /var/log/messages
maxretry = 3
maxlines = 4
[sshd-key]
enabled  = true
filter   = alpine-sshd-key
port     = ssh,19226
logpath  = /var/log/messages
maxretry = 2
bantime  = 2d
EOF

/sbin/service fail2ban restart

/sbin/service syslog restart
```

> **Note** : the bug of non password logins are solved with 0.11+ packages

These above configuration if you use recent 0.11+ pacakge version
addresses the following use cases:

* attempts to login with wrong passwords
* attempts to login by ddos attackers
* attempts to login without private key
* attempts to login with wrong private key
* attempts to login with wrong passphrase aren't logged

### PAM jail for locals

> **Warning**: latesd versions must use `shadow-login` or `util-linux-login` packages

```
apk add shadow linux-pam

cat > /etc/fail2ban/jail.d/01-pam.local << EOF
[pam-all]
enabled  = true
mode     = normal
filter   = pam-generic
port     = all
banaction = iptables-allports
logpath  = /var/log/messages
maxretry = 2
bantime  = 20m
```

The `/var/log/messages` if changed in PAM definitions then must be changed here also

### bind9 jail

This is an especial setup, will need to modify bind9 and configure to check logs 
for special setup, the default bind9 fail2ban will denied any request that is not 
autorized, so only will help for an authoritative setup, this setup here will use 
the default bind9 setup but for rate-limit setup but we recomend a better one 
reading the [server-alpine-bind9-professional.md](server-alpine-bind9-professional.md)

The fail2ban filter line must have: `<HOST>#\S+( \([\S.]+\))?\: rate limit drop`

```
cp /etc/fail2ban/filter.d/named-refused.conf /etc/fail2ban/filter.d/named-refused-ratelimit.conf

sed -i -r 's#failregex=.*#failregex=<HOST>\#\\S+( \\([\\S.]+\\))?\\: rate limit drop#g' /etc/fail2ban/filter.d/named-refused-ratelimit.conf

cat > /etc/fail2ban/jail.d/04-bind.local << EOF
[named-refused-udp]
enabled  = true
port     = domain,953
protocol = udp
filter   = named-refused-ratelimit
logpath  = /var/log/messages
findtime = 1
maxretry = 5
action   = iptables-multiport[name=Named, port=53, protocol=udp]
[named-refused-tcp]
enabled  = true
port     = domain,953
protocol = tcp
filter   = named-refused-ratelimit
logpath  = /var/log/messages
findtime = 1
maxretry = 5
action   = iptables-multiport[name=Named, port=53, protocol=tcp]
EOF

/sbin/service fail2ban restart

/sbin/service syslog restart
```

> **Warning**: here we used `/var/log/syslog` becouse the bind9 by default 
dump all the logs to the system log, that is same in alpine and debian.

### unban ip on ssh service ony

`fail2ban-client set sshd unbanip 10.10.1.2`

### unban all ips

```
fail2ban-client unban --all

/sbin/service fail2ban restart

/sbin/service syslog restart || /usr/sbin/service rsyslog restart
```

### check rules managed in the firewall

```
fail2ban-client -vd

iptables -L
```

## Important issues and notes

The public network environment can be complex and dangerous, with countless 
malicious devices constantly scanning IP addresses on the internet. This means 
that using Fail2ban on a public network requires constant monitoring of logs and 
frequent updates to firewall rules.

### static address vs dynamic address

Use this tool carefully and to add any IP addresses that your service needs to 
connect to as "ignored" IPs of course, must be an static address of course.

### Accidental blocking and service disruption

If you are using Fail2ban in a complex network environment or on a low-end device, 
you may need to be extra vigilant about monitoring the service status.

If you have network issues, with unstable connection, any TCP service that will 
be network connection oriented will suffers from banning, cos constants attemps 
of reconnetions on faulies could be interpreted as DOS attacks or false probes.

## See also

* [README.md](README.md)

## LICENSE

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

For more information check the [alpine/copyright.md](../../alpine/copyright.md)
