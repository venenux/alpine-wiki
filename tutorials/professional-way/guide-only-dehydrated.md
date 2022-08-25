## dehydrated

**Ultra simple letsencrypt/acme client** implemented as a shell-script - *just add water* 😆

#### features

**PROS**

* multi domain 
* using webservers
* full setup

**CONS**

* no package usage, direct provider upstream
* just commands no explanations
* only http-01 methods

#### requirements

* the domain (here we use venenux.com) must has valid DNS
* alpine must be 3.8+ recomended 3.10 or 3.12

#### instalation

```
apk del acme.sh

apk add openssl curl wget bash

wget https://raw.githubusercontent.com/dehydrated-io/dehydrated/master/dehydrated -O /usr/bin/dehydrated

chmod 755 /usr/bin/dehydrated
```

#### main configuration

```
mkdir -p /etc/dehydrated/
cat > /etc/dehydrated/config << EOF
CONFIG_D=/etc/dehydrated/conf.d
BASEDIR=/var/lib/dehydrated
WELLKNOWN="\${BASEDIR}/acme-challenges"
DOMAINS_TXT="/etc/dehydrated/domains.txt"
EOF

mkdir -p /etc/dehydrated/conf.d

cat > /etc/dehydrated/domains.txt << EOF
venenux.com www.venenux.com altern.venenux.com
EOF

cat > /etc/dehydrated/conf.d/00_defaultaccount.sh << EOF
CONTACT_EMAIL="mckaygerhard@venenux.com"
EOF

mkdir -p /var/lib/dehydrated/certs

mkdir -p /var/lib/dehydrated/acme-challenges/

mkdir -p /var/lib/dehydrated/hooks.d

cat > /var/lib/dehydrated/hooks.sh << EOF
#!/bin/bash
for file in /var/lib/dehydrated/hooks.d/*
do
    if [ -f "\${file}" ]; then
        \${file} "\$@"
    fi
done
EOF

chmod +x /var/lib/dehydrated/hooks.sh

mkdir /etc/dehydrated/conf.d/
cat > /etc/dehydrated/conf.d/01_defaulthooks.sh << EOF
HOOK="/var/lib/dehydrated/hooks.sh"
EOF

/usr/bin/dehydrated --register --accept-terms --challenge http-01
```

#### initial cert file

```
mkdir -p /etc/ssl/certs/

openssl req -x509 -days 1460 -nodes -newkey rsa:4096 \
   -subj "/C=VE/ST=Bolivar/L=Upata/O=VenenuX/OU=Systemas:hozYmartillo/CN=localhost" \
   -keyout /etc/ssl/certs/localhost.pem -out /etc/ssl/certs/localhost.pem

chmod 640 /etc/ssl/certs/localhost.pem

chown root:www-data /etc/ssl/certs/localhost.pem

cp /etc/ssl/certs/localhost.pem /etc/ssl/certs/venenux.com.pem
```

#### setup for lighttpd

```
apk add lighttpd

sed -i -r 's#alias.url =#alias.url +=#g' /etc/lighttpd/mod_cgi.conf
cat > /etc/lighttpd/mod_dehydrated.conf << EOF
alias.url += (
 "/.well-known/acme-challenge/" => "/var/lib/dehydrated/acme-challenges/",
)
EOF
itawxrc="";itawxrc=$(grep 'include "mod_dehydrated.conf' /etc/lighttpd/lighttpd.conf);[[ "$itawxrc" != "" ]] && echo listo || sed -i -r 's#.*include "mime-types.conf".*#include "mime-types.conf"\ninclude "mod_dehydrated.conf"#g' /etc/lighttpd/lighttpd.conf

rc-service lighttpd restart

cat > /etc/lighttpd/mod_ssl.conf << EOF
server.modules += ("mod_openssl")
\$HTTP["scheme"] == "http" {
    \$HTTP["host"] =~ ".*" {
        url.redirect += (".*" => "https://%0\$0")
    }
}
\$SERVER["socket"] == "0.0.0.0:443" {
 include "mod_ssl_conf.conf" 
}
\$SERVER["socket"] == "[::]:443" {
 server.use-ipv6 = "enable"
 include "mod_ssl_conf.conf" 
}
EOF

cat > mod_ssl_conf.conf << EOF
ssl.engine  = "enable"
ssl.pemfile = "/etc/ssl/certs/localhost.pem"
   \$HTTP["host"] =~ "(^other|www\.venenux.com)" {
        ssl.pemfile = "/etc/ssl/certs/venenux.com.pem"
    }
ssl.cipher-list = "ECDHE-RSA-AES256-SHA384:AES256-SHA256:RC4:HIGH:!MD5:!aNULL:!EDH:!AESGCM"
ssl.honor-cipher-order = "enable"
EOF

rc-service lighttpd restart
```

#### setup for apache2


#### periodic updates

```
rm /etc/periodic/*/dehydrated*

cat > /etc/periodic/monthly/dehydrated << EOF
#!/bin/bash
/usr/bin/dehydrated --cleanup 
/usr/bin/dehydrated -x --cron --challenge http-01  --force 

cp -f /var/lib/dehydrated/certs/venenux.com/combined.pem /etc/ssl/certs/venenux.com.pem
chmod 640 /etc/ssl/certs/venenux.com.pem
chown root:www-data /etc/ssl/certs/venenux.com.pem

/sbin/service lighttpd restart
/sbin/service nginx restart
/sbin/service apache2 restart
EOF

chmod 755 /etc/periodic/monthly/dehydrated
```

#### executing and testing

```
/etc/periodic/monthly/dehydrated

```
### Anexes : combined pem hook


```
#!/usr/bin/env bash
deploy_cert() {
    local DOMAIN="${1}" KEYFILE="${2}" CERTFILE="${3}" FULLCHAINFILE="${4}" CHAINFILE="${5}" TIMESTAMP="${6}"
    echo "Executing deploy_cert hook $0"
    echo " + Creating combined.pem (a combined privkey.pem + cert.pem)"
    
    cd "$(dirname "${CERTFILE}")" && {
        cat "${KEYFILE}" "${CERTFILE}" > "combined-${TIMESTAMP}.pem" && \
        ln -sf "combined-${TIMESTAMP}.pem" "combined.pem" && {
            # Loop over all files of this type
            for filename in "combined-"*".pem"; do
              # Check if current file is in use, remove if unused
              if [[ ! "${filename}" = "combined-${TIMESTAMP}.pem" ]]; then
                echo " + Removing unused combined certificate file: ${filename}"
                rm "${filename}"
              fi
            done
        }
    }
}
HANDLER="$1"; shift
if [[ "${HANDLER}" = "deploy_cert" ]]; then
  "$HANDLER" "$@"
fi
```