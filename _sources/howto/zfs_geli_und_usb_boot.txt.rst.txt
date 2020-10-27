ZFS (raidz), Geli und USB Boot
==============================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Einleitung
----------

Info
~~~~

Ich habe hier exakt meine Anleitung aufgeschrieben, wie ich mein System
installiert habe. Es gibt allerdings noch ein paar Unklarheiten, die ich
nicht klären konnte.

.. note::

  * Beim Booten muss immer noch ein Password angegeben werden, da ich geli mit
    -b (siehe unten) initialisieren musste. Ich hatte es versucht ohne diese
    Option, dann hat ZFS allerdings keine ``*.eli`` Devices mehr gefunden, weil
    die Platten nicht während der boot-time entschlüsselt worden sind. Evtl.
    hat noch jemand eine Idee wie man es bewerkstelligen kann, dass man kein
    Passwort während der boot-time braucht.

  * Mir ist noch nicht ganz klar inwiefern der Sicherheitsunterschied und auch
    die Performance zwischen 128bit und 256bit AES Verschlüsselung ist. Bitte
    ggf. erweitern.

Vorraussetzungen
~~~~~~~~~~~~~~~~

-  USB Bootfähiges Mainboard
-  FreeBSD DVD (wegen Install Daten und Fixit Shell) ab 8-RC1
-  USB Stick
-  3x HDD Terminologi: ad10, ad12, ad14
-  USB Stick Terminologi: da0

Zielsetzung
~~~~~~~~~~~

Es soll ein komplett verschlüsseltes FreeBSD System mit ZFS (raidz1)
aufgesetzt werden, was von einem USB Stick startet.

Installation
------------

Fixit Shell starten
~~~~~~~~~~~~~~~~~~~

-  FreeBSD mit der Install DVD Booten.
-  Fixit Shell starten ( sysinstall->fixit->2 CDROM/DVD )

Setup Permissions
~~~~~~~~~~~~~~~~~

Die fixit shell gibt am Anfang den Hinweis /etc/pwd und /etc/group zu
symlinken, da ansonsten die Rechte beim entpacken von tar archiven
verloren gehen könnten. Da diese Dateien allerdings schon existieren,
lösche ich diese erst und linke sie neu.

::

   Fixit> rm /etc/pwd.db
   Fixit> rm /etc/spwd.db
   Fixit> rm /etc/group
   Fixit> ln -s /mnt2/etc/pwd.db /etc/pwd.db
   Fixit> ln -s /mnt2/etc/spwd.db /etc/spwd.db
   Fixit> ln -s /mnt2/etc/group /etc/group

Sonstige Vorbereitungen
~~~~~~~~~~~~~~~~~~~~~~~

::

   ### needed for kernel modules
   Fixit> ln -s /dist/boot/kernel /boot/kernel
   Fixit> ln -s /dist/lib /lib

   ### load kernel modules
   Fixit> kldload geom_eli
   Fixit> kldload zfs

Create gpart containter
~~~~~~~~~~~~~~~~~~~~~~~

::

   Fixit> gpart create -s GPT ad10
   Fixit> gpart create -s GPT ad12
   Fixit> gpart create -s GPT ad14

Das Ganze sieht dann bei mir mit 3 1TB HDDs so aus:

::

   Fixit> gpart show

   =>            34 1953522988     ad10    GPT  (932G)
                 34 1953522988             - free -  (932G)

   =>            34 1953522988     ad12    GPT  (932G)
                 34 1953522988             - free -  (932G)

   =>            34 1953525101     ad14    GPT  (932G)
                 34 1953525101             - free -  (932G)

.. _create-gpart-containter-1:

Create gpart containter
~~~~~~~~~~~~~~~~~~~~~~~

So wie ich es gelesen habe, sollte man die swap Partition nicht mit in
den ZFS Container packen, darum wird dieser extra gebaut und wir zweigen
uns von jeder Platte jeweils 1GB ab.

::

   ### create swap 2048000 = 1GB | 2048000+34 = 2048034
   Fixit> gpart add -b 34 -s 2048034 -t freebsd-swap ad10
   Fixit> gpart add -b 34 -s 2048034 -t freebsd-swap ad12
   Fixit> gpart add -b 34 -s 2048034 -t freebsd-swap ad14

Mit dem Rest bauen wir die zfs Partitionen

::

   ### create rest for zfs
   Fixit> gpart add -b 2048068 -s 1953522988 -t freebsd-zfs ad10
   Fixit> gpart add -b 2048068 -s 1953522988 -t freebsd-zfs ad12
   Fixit> gpart add -b 2048068 -s 1953525101 -t freebsd-zfs ad14

Bei mir sieht es dann wie folgt aus:

::

   Fixit> gpart show

   =>            34 1953522988     ad10    GPT  (932G)
                 34    2048034        1    freebsd-swap (1G)
            2048068 1951474954        2    freebsd-zfs (931G)

   =>            34 1953522988     ad12    GPT  (932G)
                 34    2048034        1    freebsd-swap (1G)
            2048068 1951474954        2    freebsd-zfs (931G)

   =>            34 1953522988     ad14    GPT  (932G)
                 34    2048034        1    freebsd-swap (1G)
            2048068 1951477067        2    freebsd-zfs (931G)

Geli
~~~~

create Keys
^^^^^^^^^^^

::

   Fixit> mkdir -p /root/keys
   Fixit> dd if=/dev/random of=/root/keys/ad10.key bs=128k count=1
   Fixit> dd if=/dev/random of=/root/keys/ad12.key bs=128k count=1
   Fixit> dd if=/dev/random of=/root/keys/ad14.key bs=128k count=1

Encrypt
^^^^^^^

TODO1:
''''''

Hier muss noch abgewogen werden, ob eine 128bit oder 256bit
Verschlüsselung zum Einsatz kommen soll. Ist 128bit ausreichend?

TODO2:
''''''

Ich habe es irgendwie nur so hinbekommen, dass ich die "-b" Option
(Passwort während dem Bootvorgang eingeben) angeben MUSS, da er sonst
beim booten die Platten nicht entschlüsselt. Es muss noch geklärt
werden, ob es ohne die "-b" Option, also ohne Passwort, sondern nur mit
keys funktioniert.

::

   Fixit> geli init -b -K /root/keys/ad10.key -s 4096 -l 256 /dev/ad10p2
   Fixit> geli init -b -K /root/keys/ad12.key -s 4096 -l 256 /dev/ad12p2
   Fixit> geli init -b -K /root/keys/ad14.key -s 4096 -l 256 /dev/ad14p2

Note:
'''''

Wenn der Fehler "geli: Cannot open /var/backups/ad10.eli: No such file
or directory." kommt, dann einfach /var/backups anlegen

Attach
^^^^^^

::

   Fixit> geli attach -k /root/keys/ad10.key /dev/ad10p1
   Fixit> geli attach -k /root/keys/ad12.key /dev/ad12p1
   Fixit> geli attach -k /root/keys/ad14.key /dev/ad14p1

Override (Optional)
^^^^^^^^^^^^^^^^^^^

Falls es sich um eine schon benutzte Festplatte handelt, kann diese hier
mit /dev/random komplett überschrieben werden.

::

   Fixit> dd if=/dev/random of=/dev/ad10p1.eli bs=1m 
   Fixit> dd if=/dev/random of=/dev/ad12p1.eli bs=1m 
   Fixit> dd if=/dev/random of=/dev/ad14p1.eli bs=1m 

ZFS
~~~

Raidz Pool anlegen
^^^^^^^^^^^^^^^^^^

::

   Fixit> zpool create -m /tank tank raidz1 ad10p2.eli ad12p2.eli ad14p2.eli

Mountpoints erzeugen
^^^^^^^^^^^^^^^^^^^^

::

   Fixit> zfs create tank/usr
   Fixit> zfs create tank/var
   Fixit> zfs create tank/tmp
   Fixit> zfs create tank/root
   Fixit> zfs create tank/home

Access Time ausschalten
^^^^^^^^^^^^^^^^^^^^^^^

::

   Fixit>zfs set atime=off tank

Zpool Exportieren
^^^^^^^^^^^^^^^^^

Dies ist erforderlich, da beim export und danach Import die zpool.cache
Datei erzeugt wird. Diese muss auf dem USB Stick sein, da ZFS sonst
nicht weitermacht.

::

   Fixit> mkdir /boot/zfs
   Fixit> zpool export tank
   Fixit> zpool import tank

Installieren
~~~~~~~~~~~~

Installations Pfad angeben und Base und Kernel installieren

::

   Fixit> export DESTDIR=/fixed
   Fixit> cd /dist/7.2-RELEASE/base && ./install.sh
   Fixit> cd /dist/7.2-RELEASE/kernels && ./install.sh generic

### Anmerkung: Bei der Base installation kamen folgende Fehlermeldungen:

-  /root/.profile: Cant create 'root/.profile' : Cross-Devide link
-  ./root/.cshrc: Cant create 'root/.cshrc' : Cross-Devide link
-  tar: Error exit delayed from previous errors

Diese konnte ich aber misachten, da alles trotzdem funktioniert Sollte
jemand wissen was hier los ist, bitte ändern

USB Stick Setup
~~~~~~~~~~~~~~~

formatieren und partitionieren
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

   Fixit> gpart create -s GPT da0
   Fixit> gpart add -b 34 -s 16 -t freebsd-boot da0
   Fixit> gpart add -b 50 -s 31301548 -t freebsd-ufs da0
   Fixit> gpart bootcode -b /dist/boot/pmbr -p /dist/boot/gptboot -i 1 da0
   Fixit> newfs -O2 /dev/da0p2

USB Directory erzeugen und einhängen
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

   Fixit> mkdir /usb
   Fixit> mount /dev/da0p2 /usb

Boot Ordner
^^^^^^^^^^^

Den Boot Ordner vom Installierten System auf den USB Stick copieren

::

   Fixit> mkdir /usb/boot
   Fixit> cp -Rpv /tank/boot/ /usb/

Kernel
^^^^^^

Damit man booten kann muss die kernel unter /boot/kernel/kernel zu
finden sein. Also GENERIC nach kernel verschieben.

::

   Fixit> rmdir /usb/boot/kernel
   Fixit> mv /usb/boot/GENERIC /usb/boot/kernel
   Fixit> cd /usb/boot/kernel

Faster Boot
^^^^^^^^^^^

Damit der Systemstart schneller geht, kann man noch die kernel und die
Kernelmodule gzippen

::

   Fixit> gzip *   # compress to have faster loading

Um Platz zu sparen kann man auch in /usb/boot/kernel die nichtbenötigten
Module löschen. Definitiv benötigte Module sind:

-  zlib.ko
-  crypto.ko
-  acpi.ko
-  geom_eli.ko
-  kernel
-  zfs.ko
-  opensolaris.ko

Copy Encryption Keys
^^^^^^^^^^^^^^^^^^^^

Den Geli Encryption Keys auf den USB Stick kopieren.

::

   Fixit> cp -r /root/keys /usb/

Copy Zpool Cache
^^^^^^^^^^^^^^^^

copy zpool.cache file to usb stick

::

   Fixit> mkdir /usb/boot/zfs
   Fixit> cp /boot/zfs/zpool.cache /usb/boo/zfs/

Geli Aktivieren
^^^^^^^^^^^^^^^

Damit man die Festplatte entschlüsseln kann muss das Geli Modul geladen
werden und der Pfad für das Keyfile angegeben werden. Hierzu wird
folgender Eintrag in der loader.conf auf dem USB Stick angelegt:

::

   Fixit> vi /usb/boot/loader.conf
       geom_eli_load="YES"

       geli_ad10p2_keyfile0_load="YES"
       geli_ad10p2_keyfile0_type="ad10p2:geli_keyfile0"
       geli_ad10p2_keyfile0_name="/keys/ad10.key"

       geli_ad12p2_keyfile1_load="YES"
       geli_ad12p2_keyfile1_type="ad12p2:geli_keyfile1"
       geli_ad12p2_keyfile1_name="/keys/ad12.key"

       geli_ad14p2_keyfile2_load="YES"
       geli_ad14p2_keyfile2_type="ad14p2:geli_keyfile2"
       geli_ad14p2_keyfile2_name="/keys/ad14.key"

ZFS Aktivieren
^^^^^^^^^^^^^^

ZFS Support to the loader

::

   Fixit> vi /usb/boot/loader.conf
       zfs_load="YES"
       vfs.root.mountfrom="zfs:tank

Boot Time verkürzen
^^^^^^^^^^^^^^^^^^^

::

   Fixit> vi /usb/boot/loader.conf
       autoboot_delay="1"

local fstab
^^^^^^^^^^^

Nach dem Mount der root Partition ist der USB Stick aus dem namespace
raus, darum muss noch eine fstab in der root Partition der Festplatte
angelegt werden, wo dann der Rest geladen werden muss.

::

   Fixit> vi /fixed/etc/fstab
       # Device    Mountpoint  FStype  Options Dump    Pass#
       /dev/ad10p1.eli none        swap    sw  0   0
       /dev/ad12p1.eli none        swap    sw  0   0
       /dev/ad14p1.eli none        swap    sw  0   0

       tank        /       zfs rw,noatime  0   0
       tank/usr    /usr        zfs rw,noatime  0   0
       tank/var    /var        zfs rw,noatime  0   0
       tank/tmp    /tmp        zfs rw,noatime  0   0
       tank/root   /root       zfs rw,noatime  0   0
       tank/home   /home       zfs rw,noatime  0   0

Jetzt ist man fertig und kann das System rebooten. Beim reboot muss man
dann das Passwort für die ad10s1 Partition eingeben.

Post-Installation
-----------------

Post Steps
~~~~~~~~~~

Super User Passwort setzen

::

   # passwd

Add user with wheel group

::

   # adduser

Set Timezone

::

   # tzsetup

set up network card

::

   # ifconfig re0 add 192.168.0.133 netmask 255.255.255.0

/etc/resolv.conf anpassen

::

   domain      domain.de
   nameserver      217.237.151.51
   nameserver      217.237.149.205

Zusätzliche Packages installieren
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   # mount_cd9660 /dev/acd0 /mnt
   # cd /mnt/8.0-RC1/manpages && ./install.sh
   # cd /mnt/8.0-RC1/src && ./install.sh all
   ...

Ports Collection installieren
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hierzu bnötigt man eine funktionierende Internetkonfiguration.

::

   # mkdir /usr/ports
   # cd /usr/ports
   # portsnap fetch extract

rc.conf
~~~~~~~

::

   ifconfig_re0="inet 192.168.0.133  netmask 255.255.255.0"    # hier natürlich das entsprechende Interface und IP wählen.
   defaultrouter="192.168.0.1"                 # Auch hier wirder dementsprechend anpassen
   hostname="hostname.domain.de"

   sshd_enable="YES"                       # ssh aktiveren
   syslogd_enable="YES"
   syslogd_flags="-ss"     # disable logging from remote host

Directory Permissions setzen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Man kann jetzt noch die Directory Permissions von root und den
systemusern verbessern.

::

   # chmod 700 /root
   # chmod 700 /home/<USER>

* :ref:`genindex`

Zuletzt geändert: |date|

