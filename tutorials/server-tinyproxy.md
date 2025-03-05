# alpine server Proxy browser network

`proxy` term that is applied to the service as man in the middle!

There are several types of proxy, in this case we will use as gateway for 
internet browsing, this is that through the proxy server others will navigate 
no matter are devices or machines.

## What is the usage for?

* ** Navigate as if it were another machine ** the machine that connects to 
the proxy will be behind, and the proxy machine is the one that will give the 
face, means that for effects to the sisited server or website visited, the 
machine server visited wil be not the real website, it's just the proxy, and 
the machine that connects to the proxy remains partially hidden.
* ** Navigation for confined devices ** if you have many devices or machines 
that are locked or behind a firewall, you can configure the proxy so that all 
of these machine can navigate through it, the server that has the proxy must 
be a machine accessible both by these machines limited as well as being able 
to navigate to provide such functionality.
* ** Navegacion for limited devices ** this example is the most used, if the 
internet signal is wifi but only one machine has wifi equipment, you can do 
that the machine with internet wifi brings to those that do not have wifi by 
using the service of the proxy, of course those that do not have wifi must be 
all on the same network, the one that has wifi is configured as a proxy and 
means that the wifi machine would have at least two network interfaces, one 
to get the internet and the other wired interface to brings the proxy web.

## What use was it in the past?

Formerly it was used to simulate a faster internet, this was because the proxy 
server kept the pages and resources that had already been visited, so every 
time a machine asks you to navigate the proxy it was the page saved so which 
simulated higher speed.

Today this is then called "proxy cache" and is only used for limited resources 
or as a means of performance, but not very optimal becouse nomadays the network 
is tremendously fast even at slow speeds.

## Tinyproxy installation

This tutorial will configure alpine server as a proxy server but without cache.

### preparation

* configure a hostname so you don't reject our packages
* normal repositories added and updated
* update repositories


```
hostname venenux &  echo 'hostname="venenux"'> /etc/conf.d/hostname & echo "venenux" > /etc/hostname

cat > /etc/apk/repositories << EOF; $(echo)
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d '.' -f1.2)/main
http://dl-cdn.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d '.' -f1.2)/community
EOF

apk update
```

### Tinyproxy service installation

* install minimum packages
* add the service at startup

```
apk add tinyproxy tinyproxy-doc tinyproxy-openrc

rc-update add tinyproxy
```

> ** Warning **: the `eudev` package is very necessary to automatically detect the device, the `sed` package is necessary because the extended regular expression


After installation:

* Configuration files
     * `/etc/tinyproxy/tinyproxy.conf` the MAIN configuration file
     * `/etc/tinyproxy/filter` whitelist / black file either by URL or domain
* program files
     * `/usr/bin/tinyproxy` the executable which in turn also has the daemon
* service files
     * `/etc/init.d/tinyproxy` openrc script by alpine
     * `/var/log/tinyproxy/tinyproxy.log` log file assumed by daemon, must be the same in conf


### Configuration - basic operation

The alpine package does not fit anything well configured, it must be adjusted 
so that can run as service daemon.

This is because in dockers and containers `tinyproxy` needs to work 
without being a service that belongs to anyone, that's why he is not tied to 
the group or neither to the user in the configuration file.

The most important part is the combination of the filters, this will be 
addressed in the advanced configuration section. Here it will be configured 
for a simple service on a pc:

* configure to run the service in their own user and group
* configure port, web proxies use 8888, use this for compatibility
* if you have a single network card, use it for input and output
* configure the maximum number of machines that can use this proxy with 10
* allow any ip to connect and use the proxy
* configure log and run pid files as same as daemon boot service it need
* enable service and start (or restart) proxy service

```
sed -Ei 's|^.?User.*|User tinyproxy|g' /etc/tinyproxy/tinyproxy.conf
sed -Ei 's|^.?Group.*|Group tinyproxy|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?Port.*|Port 8888|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?BindSame.*|BindSame yes|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?MaxClients.*|MaxClients 10|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?Allow[[:space:]]|#Allow |g' /etc/tinyproxy/tinyproxy.conf

mkdir -p /var/run/tinyproxy && chown tinyproxy /var/run/tinyproxy
sed -Ei 's|^.?LogFile.*|LogFile "/var/log/tinyproxy/tinyproxy.log"|g' /etc/tinyproxy/tinyproxy.conf
sed -Ei 's|^.?PidFile.*|PidFile "/var/run/tinyproxy.pid"|g' /etc/tinyproxy/tinyproxy.conf
rc-update add tinyproxy && rc-service tinyproxy restart
```

After configuring and restarting the service, you can test it by 
this command line in shell like this:

```
apk add curl curl-doc

export http_proxy="http://localhost:8888/" && curl -I http://google.com
```

### Advanced configuration

#### Configure tinyproxy for hidden internet navigation

```
sed -Ei 's|^.?User.*|User tinyproxy|g' /etc/tinyproxy/tinyproxy.conf
sed -Ei 's|^.?Group.*|Group tinyproxy|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?Port.*|Port 8888|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?BindSame.*|BindSame yes|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?Allow[[:space:]]|#Allow |g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?DisableViaHeader.*|DisableViaHeader Yes|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?XTinyproxy.*|XTinyproxy No|g' /etc/tinyproxy/tinyproxy.conf

viaproxyname=$(head -n1 < <(fold -w10 < <(tr -cd 'a-z0-9' < /dev/urandom)))
sed -Ei "s|^.?ViaProxyName.*|ViaProxyName \"${viaproxyname}\"|g" /etc/tinyproxy/tinyproxy.conf

mkdir -p /var/run/tinyproxy && chown tinyproxy /var/run/tinyproxy
sed -Ei 's|^.?LogFile.*|LogFile "/var/log/tinyproxy/tinyproxy.log"|g' /etc/tinyproxy/tinyproxy.conf
sed -Ei 's|^.?PidFile.*|PidFile "/var/run/tinyproxy/tinyproxy.pid"|g' /etc/tinyproxy/tinyproxy.conf
rc-update add tinyproxy && rc-service tinyproxy restart
```

#### Configure tinyproxy to filter what is browsing

```
touch /etc/tinyproxy/filter && echo "mocosoft" >> /etc/tinyproxy/filter
touch /etc/tinyproxy/filter && echo "*.facebook.com" >> /etc/tinyproxy/filter

sed -Ei 's|^.?Filter[[:space:]].*|Filter \"/etc/tinyproxy/filter\"|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?FilterURLs[[:space:]].*|FilterURLs On|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?Allow[[:space:]]|#Allow |g' /etc/tinyproxy/tinyproxy.conf

mkdir -p /var/run/tinyproxy && chown tinyproxy /var/run/tinyproxy
sed -Ei 's|^.?LogFile.*|LogFile "/var/log/tinyproxy/tinyproxy.log"|g' /etc/tinyproxy/tinyproxy.conf
sed -Ei 's|^.?PidFile.*|PidFile "/var/run/tinyproxy.pid"|g' /etc/tinyproxy/tinyproxy.conf
rc-update add tinyproxy && rc-service tinyproxy restart
```

#### Configure tinyproxy to filter who can navigate

```
default_if=$(ip route list | awk '/^default/ {print $5}')
netmaskip=$(ip -o -f inet addr show ${default_if} | awk '{print $4}')

sed -Ei "s|^.?Allow[[:space:]]127.0.0.1|#Allow ${netmaskip}|g" /etc/tinyproxy/tinyproxy.conf
sed -Ei 's|^.?Allow[[:space:]]127.0.0.1|#Allow 127.0.0.1|g' /etc/tinyproxy/tinyproxy.conf

sed -Ei 's|^.?MaxClients.*|MaxClients 10|g' /etc/tinyproxy/tinyproxy.conf

mkdir -p /var/run/tinyproxy && chown tinyproxy /var/run/tinyproxy
sed -Ei 's|^.?LogFile.*|LogFile "/var/log/tinyproxy/tinyproxy.log"|g' /etc/tinyproxy/tinyproxy.conf
sed -Ei 's|^.?PidFile.*|PidFile "/var/run/tinyproxy.pid"|g' /etc/tinyproxy/tinyproxy.conf
rc-update add tinyproxy && rc-service tinyproxy restart
```

# LICENSE

** CC BY-NC-SA **: The project allows reusers to distribute, remix, adapt and build on the material.
in any medium or format only for non-commercial purposes, and only provided that the attribution is provided
to the creators involved. If you remix, adapt or build on the material, you must obtain the license of the modified material.
material under identical terms includes the following elements:

* ** POR **: credit must be granted to the creator of each content respectively, starting with the first contributor.
* ** NC ** – Non-commercial uses of the work are only allowed, with exceptions if you complete a problem here!
* ** SA ** – Adaptations must be shared under the same terms, you must obey these terms and not change them.
