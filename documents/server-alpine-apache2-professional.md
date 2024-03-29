## Apache2

The most famous cos was the first liux server that was widelly deployed. 
The apache2 server is the main choice when you want to get support in development, 
cos some setup are in a way of granular customization, with extended documentation.

The recommendation its to use apache2 behind a reverse proxy setup, such like 
lighttpd or hiawatta servers.
Currently the most lazy and slow server .. just for windosers that wants to learn.. 

## Table of Contents

- [Apache2](#apache2)
    - [Apache2 Installation](#apache2-installation)
    - [Controlling Apache2](#controlling-apache2)
- [Apache2 Configuration](#apache2-configuration)
    - [Status special page](#status-special-page)
    - [CGI bin directory support](#cgi-bin-directory-support)
    - [Descriptive error or special pages](#descriptive-error-or-special-pages)
    - [Userdir public_html support](#userdir-public_html-support)
    - [Apache2 alpine proxy modules setup](#apache2-alpine-proxy-modules-setup)
    - [Apache2 SSL support](#apache2-ssl-support)
- [Lest Encrypt](#lest-encrypt)
- [see also](#see-also)
- [LICENSE](#license)

### Apache2 Installation

1. run apk for need pacakges
2. make the htdos public web root directories
3. configure the default ports and server information
4. setup the permissions
5. added the service to the boot process
6. start the service
7. put a default page to test the service

```
apk add apache2 apache2-utils

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

> **Warning** : alpine packagers are a mess, the apache2 default configuration is not ordened so all the conf files under `/etc/apache2/conf.d/` will be loaded with no specific order.

### Controlling Apache2

**Start apache2**: After the installation lighttpd is not running. As we made in first section was started already but if you want to start lightttpd manually use: `rc-service apache2 start`

You will get a feedback about the status.

 `* Starting apache2...                                         [ ok ]`

**Stop apache2**: If you want to stop the web server use stop in the same way of previous command: `rc-service apache2 stop`

**Restart lighttpd**: After changing the configuration file lighttpd needs to be restarted. `rc-service lighttpd restart`

### Apache2 Configuration

**If you just want to serve simple HTML pages apache2 can be used out-of-box. No further configuration needed.**

Due to the minimalism of alpine linux, unfortunately the apache2 packaging is the worst ever seen, its configuration file makes it impossible to configure with only single line commands so the commands for quick configuration with cares of overwriting are very dedicated.

#### Status special page

Taking care of the status web server: those special pages are just minimal info of the running web server, are need to view from outside in a case of emergency, do not take the wrong approach of hide behind a filtered ip or filtered network, you must have access in all time in all the web to see problems. The creation of the directory in the htdocs main root web files are just to remember you so then can avoid hiring a staff that becomes indispensable, thus allowing to save costs in knowledge theft by technical staff.

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

#### CGI bin directory support

By default packages assign a directory under localhost main domain, other linux uses a global cgi directory and aliasing.. the most profesional way, but think about it, this per domain configuration allows isolation:

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

#### Descriptive error or special pages

This pages will be show to visitors when a page or path are not in the server, or when a internal error happened, this are to do not show a horrible message of development to visitors.. and just a nice message or "away from here" message:

1. install the errors package
2. restart the service

```
apk add apache2-error

rc-service apache2 restart
```

All about error documents are define at `/etc/apache2/conf.d/multilang-errordoc.conf`, you can customized byt redefine the error alias and the error codes. The right way is to make a symlink from `/var/www/error-pages` over each document and if there's any customized remove the symlink and create the alternate error page there.

#### Userdir public_html support

As vendors of web sites do, with this each user created in the unix system can serve owned web pages witout being root or gain access to sense files:

1. create the directory for put the html files due alpine crap does not follow any standard
2. enable the module in the webserver
3. set the user directory in the config file
4. restart the service to see the changes at the browser per user

```
mkdir -p /etc/skel/public_html

for i in /home/*; do mkdir $i/public_html ; done

sed -i -r 's#.*LoadModule.*modules/mod_usertrack.so.*#LoadModule usertrack_module modules/mod_usertrack.so#g' /etc/apache2/httpd.conf
sed -i -r 's#.*LoadModule.*modules/mod_userdir.so.*#LoadModule userdir_module modules/mod_userdir.so#g' /etc/apache2/httpd.conf

sed -i -r 's#^UserDir .*#UserDir public_html#g' /etc/apache2/conf.d/userdir.conf

rc-service lighttpd restart
```

> **Warning**  as we said.. alpine policy is to be most upstream equal possible, almost like packagers are lazy? NO! just dont put any thing about root user access, but well, you must know what are you doing, by the addition of `UserDir disabled root postmaster` you will denied specific users due security.

If you change the user dir , then you must change the directory definition at the last block.

#### Apache2 alpine proxy modules setup

The error of the XML file in the proxy modules are due the incomplete right made package:

```
httpd: Syntax error on line 481 of /etc/apache2/httpd.conf: Syntax error on line 13 of /etc/apache2/conf.d/proxy-html.conf: Cannot load /usr/lib/libxml2.so into server: Error loading shared library /usr/lib/libxml2.so: No such file or directory
```

The right setup of any proxy module or redirection is:

1. install the proxy apache2 packages
2. fix the configuration of the modules
3. setup a conf file for your redirection
4. restart the service

```
apk add apache2-proxy-html apache2-proxy

sed -i -r 's#/usr/lib/libxml2.so#/usr/lib/libxml2.so.2#g' /etc/apache2/conf.d/proxy-html.conf

cat >> /etc/apache2/conf.d/myproxy.conf << EOF
    ProxyPreserveHost On
    ProxyRequests off
    AllowEncodedSlashes NoDecode
    ProxyPass / http://127.0.0.1:3002/ nocanon
EOF

service apache2 restart
```

> **Warning**  of course, the `myproxy.conf` is hypothetical, for didactic purposes, here it is only exemplified that the error is corrected in the step of the sed command to work.

#### Apache2 SSL support

The package as we said is made in a limited way, and only has a unique config file at `/etc/apache2/conf.d/ssl.conf`.

Best way to do that are by independient include files, Debian counterpart has a good mechanism that enables configuration files, but that is not the case here, so we must deal with the random loading of the modules.

We need to created a sefl-signed certificate, so openssl are need in any case either if used a remote made certificate:

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

sed -i -r 's#^SSLCertificateKeyFile.*/etc/#\#SSLCertificateKeyFile /etc/#g' /etc/apache2/conf.d/ssl.conf
sed -i -r 's#^SSLCertificateFile.*/etc/#SSLCertificateFile /etc/ssl/certs/localhost.pem#g' /etc/apache2/conf.d/ssl.conf
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

Best is to made a redirection inside of the document root of port 80 deifintion
also put a `<IfModule mod_ssl.c>` conditional at the beginning of the ssl config file.

## Lest Encrypt

To obtain a real certificate, use our best guide for: 

Check the document [guide-only-dehydrated.md](guide-only-dehydrated.md) there's also a specific section to setup apache2.

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
