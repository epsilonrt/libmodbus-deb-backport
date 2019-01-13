# Backport _testing_ libmodbus 3.1.4

Source: [SimpleBackportCreation](https://wiki.debian.org/fr/SimpleBackportCreation)

On va faire un backport des paquets  libmodbus depuis la distribution Debian 
Testing.  

Ici nous le faisons sur une architecture `arm64`, mais cela peut se faire sur 
toutes les architectures disposant du paquet `libmodbus5` en version 3.14-2.

Si vous souhaitez juste installer les paquets libmodbus en version 3.1.4-2 créés à 
partir de ce tutoriel, il vous suffit d'installer le dépôt 
[piduino.org](http://apt.piduino.org), en effectuant les opérations suivants :

    $ wget -O- http://www.piduino.org/piduino-key.asc | sudo apt-key add -
    $ sudo add-apt-repository 'deb http://apt.piduino.org stretch piduino'
    $ sudo apt update

Puis, vous pouvez installer :

    $ sudo apt install libmodbus-dev

Ou mettre à jour :

    $ sudo apt upgrade

Ce dépôt fournit les paquets Debian Stretch et Ubuntu Xenial pour les 
architectures `arm64`, `armhf` et `arm64`. Dans le cas de Xenial, il faudra 
remplacer `stretch` par `xenial` dans la ligne `add-apt-repository`.

## Procédure pour le backport

### Installez les outils d'empaquetage pour Debian

    $ sudo apt install --no-install-recommends packaging-dev debian-keyring devscripts equivs libdistro-info-perl

### Cloner le dépôt et y descendre

    $ git clone https://github.com/epsilonrt/libmodbus-deb-backport.git
    $ cd libmodbus-deb-backport

### Trouvez quelle est la version disponible dans l'archive Debian

    $ rmadison libmodbus5 --architecture arm64
    libmodbus5 | 3.0.6-1       | oldstable  | arm64
    libmodbus5 | 3.0.6-2       | stable     | arm64
    libmodbus5 | 3.1.4-2       | testing    | arm64
    libmodbus5 | 3.1.4-2       | unstable   | arm64

### Téléchargez le fichier source .dsc de la version Testing

Avec votre navigateur, allez à la page http://packages.debian.org/testing/libmodbus5 
et cherchez le fichier dsc et copiez l'adresse du lien : 


    $ dget -x http://deb.debian.org/debian/pool/main/libm/libmodbus/libmodbus_3.1.4-2.dsc
    dget: retrieving http://deb.debian.org/debian/pool/main/libm/libmodbus/libmodbus_3.1.4-2.dsc
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100   334  100   334    0     0    813      0 --:--:-- --:--:-- --:--:--   814
    100  1946  100  1946    0     0   2987      0 --:--:-- --:--:-- --:--:--  2987
    dget: retrieving http://deb.debian.org/debian/pool/main/libm/libmodbus/libmodbus_3.1.4.orig.tar.gz
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100   340  100   340    0     0   1308      0 --:--:-- --:--:-- --:--:--  1312
    100 92596  100 92596    0     0   168k      0 --:--:-- --:--:-- --:--:--  168k
    dget: retrieving http://deb.debian.org/debian/pool/main/libm/libmodbus/libmodbus_3.1.4-2.debian.tar.xz
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100   344  100   344    0     0   2398      0 --:--:-- --:--:-- --:--:--  2405
    100  7540  100  7540    0     0  27982      0 --:--:-- --:--:-- --:--:-- 27982
    libmodbus_3.1.4-2.dsc:
          Good signature found
       validating libmodbus_3.1.4.orig.tar.gz
       validating libmodbus_3.1.4-2.debian.tar.xz
    All files validated successfully.
    dpkg-source: info: extraction de libmodbus dans libmodbus-3.1.4
    dpkg-source: info: extraction de libmodbus_3.1.4.orig.tar.gz
    dpkg-source: info: extraction de libmodbus_3.1.4-2.debian.tar.xz
    dpkg-source: info: mise en place de Fix-typo.patch
    dpkg-source: info: mise en place de Fix-float-endianness-issue-on-big-endian-arch.patch

Modifiez en fonction de votre distribution, par exemple pour xenial `amd64` :

    $ dget -x -u https://launchpad.net/ubuntu/+archive/primary/+sourcefiles/libmodbus/3.1.4-2/libmodbus_3.1.4-2.dsc

### Installez les dépendances de `libmodbus`

    $ sudo apt install --no-install-recommends debhelper asciidoc xmlto psmisc

Dans le cas de xenial, il faut que la version de debhelper soit en 10 au 
minimum.

    $ rmadison debhelper --architecture arm64
     debhelper | 9.20131227ubuntu1           | trusty           | all
     debhelper | 9.20160115ubuntu3           | xenial           | all
     debhelper | 10.2.2ubuntu1~ubuntu16.04.1 | xenial-backports | all
     debhelper | 11.1.6ubuntu1               | bionic           | all
     debhelper | 11.1.6ubuntu2               | bionic-updates   | all
     debhelper | 11.3.2ubuntu1               | cosmic           | all
     debhelper | 12ubuntu1                   | disco            | all

On voit qu'il faut l'installer depuis le dépôt xenial-backports :

    $ sudo apt install -t xenial-backports debhelper
    $ sudo apt install --no-install-recommends asciidoc xmlto psmisc
    

### Mettez un numéro de révision de backport dans le journal des modifications

    $ cd libmodbus-3.1.4
    $ dch --local ~epsi+ --distribution stretch  "build for stretch-backports."

Modifiez en fonction de votre distribution, par exemple pour xenial:

    $ dch --local ~epsi+ --distribution xenial  "build for xenial-backports."


### Correction du fichier `libmodbus-dev.docs` 

Cette opération doit être effectuée que si la version de `debhelper` est 
inférieure à 11 (cas de `stretch` et `xenial`, mais pas pour `bionic`)

    $ cp ../libmodbus-dev.docs debian

### Compilez correctement le paquet, sans signature GPG

    $ fakeroot debian/rules binary
    dh binary --with autoreconf --exclude=.la
       dh_testdir -O--exclude=.la
       dh_update_autotools_config -O--exclude=.la
       dh_autoreconf -O--exclude=.la
    [.............]
    dpkg-deb: building package 'libmodbus5-dbgsym' in '../libmodbus5-dbgsym_3.1.4-2~epsi+1_arm64.deb'.
    dpkg-deb: building package 'libmodbus5' in '../libmodbus5_3.1.4-2~epsi+1_arm64.deb'.
    dpkg-deb: building package 'libmodbus-dev' in '../libmodbus-dev_3.1.4-2~epsi+1_arm64.deb'.

Si l'on souhaite recompiler après une modification, il faudra au préalable faire un `clean`

    $ fakeroot debian/rules clean


### Vérifiez les paquets

    $ cd ..
    $ dpkg -I libmodbus5_3.1.4-2~epsi+1_arm64.deb 
     nouveau paquet Debian, version 2.0.
     taille 29022 octets : archive de contrôle=1329 octets.
         516 octets,    17 lignes      control              
         435 octets,     6 lignes      md5sums              
          23 octets,     1 lignes      shlibs               
        2387 octets,    72 lignes      symbols              
          60 octets,     2 lignes      triggers             
     Package: libmodbus5
     Source: libmodbus
     Version: 3.1.4-2~epsi+1
     Architecture: arm64
     Maintainer: SZ Lin (林上智) <szlin@debian.org>
     Installed-Size: 71
     Depends: libc6 (>= 2.17)
     Section: libs
     Priority: optional
     Multi-Arch: same
     Homepage: http://libmodbus.org/
     Description: library for the Modbus protocol
      A Modbus library written in C, to send/receive data with a device which
      respects the Modbus protocol. This library can use a serial port or an
      Ethernet connection.
      .
      This package contains the shared library.

