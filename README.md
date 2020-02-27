# Backport _testing_ libmodbus 3.1.4 vers libmodbusepsi

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

    $ sudo apt install libmodbusepsi-dev

Ou mettre à jour :

    $ sudo apt upgrade

Ce dépôt fournit les paquets Debian Stretch et Ubuntu Xenial pour les 
architectures `arm64`, `armhf` et `arm64`. Dans le cas de Xenial, il faudra 
remplacer `stretch` par `xenial` dans la ligne `add-apt-repository`.

## Procédure pour le backport

### Installez les outils d'empaquetage pour Debian

    $ sudo apt install --no-install-recommends packaging-dev debian-keyring devscripts equivs libdistro-info-perl

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


### Cloner le dépôt et créer les paquets

    $ git clone https://github.com/epsilonrt/libmodbus-deb-backport.git
    $ cd libmodbus-deb-backport
    $ git checkout epsilonrt
    $ ./build

Les fichiers deb sont créés dans le répertoire parent.

La commande pour compilez correctement le paquet, sans signature GPG, en cas d'erreur :

    $ fakeroot debian/rules binary
    dh binary --with autoreconf --exclude=.la
       dh_testdir -O--exclude=.la
       dh_update_autotools_config -O--exclude=.la
       dh_autoreconf -O--exclude=.la
    libtoolize: putting auxiliary files in AC_CONFIG_AUX_DIR, 'build-aux'.
    [.............]
    dpkg-deb: building package 'libmodbusepsi5-dbgsym' in '../libmodbusepsi5-dbgsym_3.1.4-5_amd64.deb'.
    dpkg-deb: building package 'libmodbusepsi5' in '../libmodbusepsi5_3.1.4-5_amd64.deb'.
    dpkg-deb: building package 'libmodbusepsi-dev' in '../libmodbusepsi-dev_3.1.4-5_amd64.deb'.

Si l'on souhaite recompiler après une modification, il faudra au préalable faire un `clean`

    $ fakeroot debian/rules clean


### Vérifiez les paquets

    $ cd ..
    $ dpkg -I ../libmodbusepsi5_3.1.4-5_amd64.deb 
     nouveau paquet Debian, version 2.0.
     taille 31676 octets : archive de contrôle=1340 octets.
         588 octets,    19 lignes      control              
         458 octets,     6 lignes      md5sums              
          31 octets,     1 lignes      shlibs               
        2473 octets,    74 lignes      symbols              
          60 octets,     2 lignes      triggers             
     Package: libmodbusepsi5
     Source: libmodbusepsi
     Version: 3.1.4-5
     Architecture: amd64
     Maintainer: Pascal Jean (epsilonrt) <epsilonrt@gmail.com>
     Installed-Size: 79
     Depends: libc6 (>= 2.15)
     Conflicts: libmodbus5
     Replaces: libmodbus5
     Section: libs
     Priority: optional
     Multi-Arch: same
     Homepage: http://libmodbus.org/
     Description: library for the Modbus protocol (epsilonrt version)
      A Modbus library written in C, to send/receive data with a device which
      respects the Modbus protocol. This library can use a serial port or an
      Ethernet connection.
      .
      This package contains the shared library.


