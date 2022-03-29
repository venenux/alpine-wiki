Preparar un alpine, para brindar el servicio PXE TFTP y asi se pueda arrancar otras computadoras con dicho servicio.

## requisitos

* red, opciones:
    * red con dos pc conectadas comunicandose
        * cable punto a punto
    * red con dos pc mediante hub de red
        * cables normales de red
    * red con dhcp ya activa y dos pc
        * cables normales de red
* linux que se desea arancar ejemplo:
    * https://dl-cdn.alpinelinux.org/alpine/v<version>/releases/<arch>/alpine-netboot-<version>.<minor>-<arch>.tar.gz
* internet para los primeros archivos

Cabe destacar qe por ejemplo todo equipo x86 mas antiguo de 2011 solo 
acepta i386 de 32bit por ende la imagen para estos como el DARUMA URMET 
es (si se usa 3.8.1): https://dl-cdn.alpinelinux.org/alpine/v3.8/releases/x86_64/alpine-netboot-3.8.1-x86.tar.gz

Apine comenzo el netboot en 3.8, pero desde 3.11 usa kernel lts por lo que las referencias
para todas las arquitecturas en https://dl-cdn.alpinelinux.org/alpine/v<version>/releases/<arch>/netboot/ cambian

Actualmente para las actuales ARM, AMD64, S390 ls puedes encontrar desde la version 3.9, anteriormente 
solo x86 y x64 eran las provistas para netboot.

## preparacion

Una de las maquinas es la instalar y la otra la que proveera los archivos.

Se necesita que ambas maquinas esten en la misma red, solo en el caso 
de que ya exista DHCP se omite esto del servicio dnsmasq 

#### Maquina 1 servidor PXE y archivos

En esta maquina se necesita alpine instalado y dnsmasq preparado

#### Maquina 2 objetivo instalar PXE

En esta maquina se necesita PXE boot habilitado desde el bios

#### Conexciones

**Caso de un solo cable** Se conectan ambas pc una contra la otra con 
un calbe de punto a punto, la pc 1 tendra una ip fija, y ofrecera con 
el software dnsmasq DHCP a la pc 2 (auque se puede usar tambien en esta estatica).

**Caso de tener hub red** Se conectan ambas pc en el hub normal y 
la pc primaria 1 tendra una ip fija, y ofrecera con el 
software dnsmasq DHCP a la pc 2 (auque se puede usar tambien en esta estatica).

**Caso de tener red y DHCP** Se conectan ambas pc a la red ya provista 
la pc primaria 1 tendra una ip fija, y ofrecera con el 
software dnsmasq DHCP a la pc 2 (auque se puede usar tambien en esta estatica).

## Configuracion maquina 1



```
apk add dnsmasq

mv /etc/dnsmasq.conf  /etc/dnsmasq.conf.backup
```

**Caso de tener red y DHCP** :

Debera colocar una ip fija a la maquina 1 coordinada con el router:
* red 192.168.2.1/255.255.255.0 (un router que ya da dhcp)
* pc1 192.168.2.2/255.255.255.0 gateway 192.168.2.1

```
cat > /etc/dnsmasq.conf << EOF
interface=enp2s0
domain=debian.lan
dhcp-range=192.168.2.2,192.168.2.254,12h
dhcp-option=option:router,192.168.2.1
dhcp-option=option:dns-server,192.168.2.1
dhcp-option=6,8.8.8.8
server=8.8.4.4
dhcp-boot=pxelinux.0,pxeserver,192.168.2.2
pxe-prompt="Press F8 for menu.", 60
pxe-service=x86PC, "Install alpine linux 192.168.2.2", pxelinux
enable-tftp
tftp-root=/srv/tftp
EOF
```

**Caso de tener hub red o solo cable** :

Debera colocar una ip fija a la maquina 1 que actuara como un router:
* red 192.168.2.1/255.255.255.0 (la pc1 con ip estatica en 1)
* pc1 192.168.2.1/255.255.255.0 gateway 192.168.2.1

```
cat > /etc/dnsmasq.conf << EOF
interface=enp2s0
domain=debian.lan
dhcp-range=192.168.2.2,192.168.2.254,255.255.255.0,12h
dhcp-boot=pxelinux.0,pxeserver,192.168.2.1
dhcp-option=option:dns-server,192.168.2.1
dhcp-option=6,8.8.8.8
server=8.8.4.4
pxe-prompt="Press F8 for menu.", 60
pxe-service=x86PC, "Install alpine linux 192.168.2.1", pxelinux
enable-tftp
tftp-root=/srv/tftp
EOF
```

**Archivos para installer**:

El instalador esperara lso archivos en el ftp desde donde esta el PXE.

```
mkdir /srv/tftp/
cd /srv/tftp/

wget https://dl-cdn.alpinelinux.org/alpine/v3.11/releases/x86/alpine-netboot-3.11.13-x86.tar.gz

tar xfz netboot.tar.gz

wget http://archive.debian.org/debian-archive/debian/dists/wheezy/main/installer-i386/current/images/SHA256SUMS

wget http://archive.debian.org/debian-archive/debian/dists/wheezy/Release

wget http://archive.debian.org/debian-archive/debian/dists/wheezy/Release.gpg

chmod -R 755 /srv/tftp/

service dnsmasq restart

netstat -tulpn | grep dnsmasq
```

Si todo esta correcto los puertos 53, 67 y 69 estaran abierto.. sino use 
el `uwf allow` con `<puerto>/<protocolo>` con puerto con los de arriba, 
y protocolo ambos el UDP y el TCP.

**Archivos para repos offline**:

El instalador una vez iniciado buscara un web repositorio, y si no 
tienes internet no podra seguir, asi que podemos emularlo con 
un repositorio desde el ISO debian (cualquiera netinstall, dvbd/cd, etc)
localmente con lighttpd, apache2 etc..

Si estas instalando un debian oldstable, intentara usar "stable" como 
distro ademas por lo que aun con internet fallaria con versiones viejas.

```
apt-get install lighttpd

lighty-enable-mod accesslog  dir-listing status

lighty-disable-mod unconfigured

service lighttpd restart

cd /srv/tftp/

apk add wget

wget https://dl-cdn.alpinelinux.org/alpine/v3.11/releases/x86/alpine-standard-3.11.13-x86.iso

apk add coreutils

mkdir /srv/tftp/alpine-iso
mount alpine-standard-3.11.13-x86.iso /srv/tftp/alpine-iso -o loop
cp -rf /srv/tftp/debian-iso/* /var/www/ -o loop
```

Adicional para que funcione debe alterarse estos archivos

* /srv/tftp/debian-installer/i386/boot-screens/txt.cfg
* /srv/tftp/debian-installer/i386/boot-screens/adtxt.cfg
* /srv/tftp/debian-installer/i386/boot-screens/spk.cfg

La linea `append` hacia el final agregar siempre `debian-installer/allow_unauthenticated=true`
sino no podra usar los repos ni archivos installer desde la red local.

## Configuracion maquina 2

Vaya al menú de inicio de la maquina y seleccione inicio de red como el 
dispositivo de inicio principal (en los mas actuales sistemas puede 
seleccionar el dispositivo de inicio sin ingresar a la configuración 
del BIOS con solo presionando una tecla durante BIOS POST comunmente F11).

En los URMET DARUMA MT 1000 si tiene clave arrancara con peticion DHCP, 
cuando pulse "n" unicamente.

* despues de arrancar debera pulsar F8 y escoger la opcion alpine
* durante la instalacion debera escoger un mirror manualmente
    * el mirror url seria 192.168.2.2 que es la maquina 1
    * el directorio raiz puede ser "/main/" o simplemente "/"

Ambas maquinas deben estar activas durante la instalacion.

Despues de terminar la instalacion se puede configurr internet y demas 
en la maquina pc 2 por lo que quizas un archivo iso mas completo es lo 
mejor.

