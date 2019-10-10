Festplattenverschlüsselung mit Geli und USB Boot
================================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Einleitung
----------

Vorraussetzungen
~~~~~~~~~~~~~~~~

-  USB Bootfähiges Mainboard
-  FreeBSD DVD (wegen Install Daten und Fixit Shell)
-  USB Stick
-  HDD Terminologie: ad10
-  USB Stick Terminologi: da0

Man sollte schon einen relativ guten USB Stick nehmen. Ich hatte erst
einen 6€ noname 2GB USB Stick und es kamen nur LBA Fehler Mit einem 20€
USB Stick von Sony gab es keine Probleme mehr.

Zielsetzung
~~~~~~~~~~~

Eine 1TB HDD soll folgendermaßen formatiert werden

::

   ad10s1 (40GB)
   a   /       300MB
   b   swap        2GB
   c   raw disk    
   d   /usr        20GB
   e   /var        5GB
   f   /tmp        5GB
   g   /root       1GB
   h   /home       5GB (rest)

   ad10s2 (Rest)
   d   /data

Wobei ad10s1 Die Systempartition ist, welche direkt beim booten
eingehangen wird. Dies geschiet mittels Password und einem Key, welcher
sich auf dem USB Stick befindet. ad10s2 Ist die Datenpartition und kann
nach belieben eingehangen werden.

Das ganze System soll von einem unverschlüsselten USB Stick booten,
welcher nach dem Start entfernt werden kann.

Hinweis
~~~~~~~

Achtung ich gehe davon aus, dass es sich um eine leere Festplatte
handelt. Diese wird neu formatiert, falls sich Daten darauf befinden,
dann vorher Backups erstellen!

Installation
------------

Fixit Shell starten
~~~~~~~~~~~~~~~~~~~

-  FreeBSD mit der Install DVD Booten.
-  Fixit Shell starten ( sysinstall->fixit->2 CDROM/DVD )

Setup Permissions
~~~~~~~~~~~~~~~~~

Die fixit shell gibt am Anfang den Hinweis /etc/*pwd* und /etc/group zu
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

Partitionierung
~~~~~~~~~~~~~~~

Dies geschiet mit fdisk

::

   Fixit> fdisk -i /dev/ad10

Ich habe folgende Werte verwendet

::


   Do you want to change our idea of what BIOS thins?  [n]

   Do you want to change it?   [y]
   sysid               165
   start               63
   size                83885697    # ca 40GB                       
   set beg/end         [n]
   are you happy with this     [y]

   Data for Partition 2
   Do you want to change it?   [y]
   sys             165
   start               83885760
   size                1869636384
   set beg/end         [n]
   are you happy with this     [y]

   set active partition        [y]
   active              [1]
   are you happy with this     [y]
   write partition table       [y]

Bei Bedarf kann die Größe der ersten Partition von ca 40G auch geändert
werden.

Geli
~~~~

Load Geli
^^^^^^^^^

Um Geli laden zu können muss man vorher zwei symlinks zu kernel und lib
erstellen.

::

   Fixit> ln -s /dist/boot/kernel /boot/kernel
   Fixit> ln -s /dist/lib /lib
   Fixit> kldload geom_eli

Create Keys
^^^^^^^^^^^

Jetzt werden die Verschlüsselungskeys für die beiden Partitionen
erzeugt. Den zweiten key kann man natürlich auch später erzeugen, da
dieser für die Installation nicht notwendig ist.

::

   Fixit> mkdir -p /root/keys
   Fixit> dd if=/dev/random of=/root/keys/ad10s1.key bs=128k count=1
   Fixit> dd if=/dev/random of=/root/keys/ad10s2.key bs=128k count=1

Encrypt
^^^^^^^

Die -b Option bedeutet, dass bei dieser Partition schon eine
Passwortabfrage beim booten erfolgt.

::

   Fixit> geli init -b -K /root/keys/ad10s1.key -s 4096 -l 256 /dev/ad10s1     # -b PWD Abfrage beim Boot
       Passwort:
       Passwort wiederholung:

   Fixit> geli init -K /root/keys/ad10s2.key -s 4096 -l 256 /dev/ad10s2        # Wird im running system gemountet
       Passwort:
       Passwort wiederholung:

Attach
^^^^^^

Hier werden die Partitionen entschlüsselt.

::

   Fixit> geli attach -k /root/keys/ad10s1.key /dev/ad10s1
   Fixit> geli attach -k /root/keys/ad10s2.key /dev/ad10s2

Override (Optional)
^^^^^^^^^^^^^^^^^^^

Falls es sich um eine schon benutzte Festplatte handelt, kann diese hier
mit /dev/random komplett überschrieben werden.

::

   Fixit> dd if=/dev/random of=/dev/ad0s1.eli bs=1m 
   Fixit> dd if=/dev/random of=/dev/ad0s2.eli bs=1m 

Slices
~~~~~~

Sollen Bei meinem Setup folgendermaßen aussehen:

::

   ad10s1 (40GB)
   a   /       300MB
   b   swap        2GB
   c   raw disk    
   d   /usr        20GB
   e   /var        5GB
   f   /tmp        5GB
   g   /root       1GB
   h   /home       5GB (rest)

::

   Fixit> bsdlabel -w /dev/ad10s1.eli
   Fixit> bsdlabel -e /dev/ad10s1.eli

Manueller Eintrag mit bsdlabel. Die \* sind Platzhalter und bedeuten,
dass bsdlabel die Werte automatisch nach-berechnet. Folgende Werte habe
ich bei mir eingetragen:

::

   a:  300M    2   4.2BSD
   b:  2G  *   swap
   c:  -- dont edit --
   d:  20G *   4.2BSD
   e:  5G  *   4.2BSD
   f:  5G  *   4.2BSD
   g:  1G  *   4.2BSD
   h:  *   *   4.2BSD

Slices formatieren

::

   Fixit> newfs -L ROOT /dev/ad10s1.elia
   Fixit> newfs -L SWAP /dev/ad10s1.elib
   Fixit> newfs -L USR /dev/ad10s1.elid
   Fixit> newfs -L VAR /dev/ad10s1.elie
   Fixit> newfs -L TMP /dev/ad10s1.elif
   Fixit> newfs -L ROOTHOME /dev/ad10s1.elig
   Fixit> newfs -L HOME /dev/ad10s1.elih

Installieren
~~~~~~~~~~~~

Installations-Directory anlegen, in diesem fall /fixed und die Slices
entsprechend mounten

::

   Fixit> mkdir /fixed
   Fixit> cd /fixed
   Fixit> mount /dev/ad10s1.elia /fixed
   Fixit> mkdir home root tmp usr var
   Fixit> mount /dev/ad10s1.elid /fixed/usr
   Fixit> mount /dev/ad10s1.elie /fixed/var
   Fixit> mount /dev/ad10s1.elif /fixed/tmp
   Fixit> mount /dev/ad10s1.elig /fixed/root
   Fixit> mount /dev/ad10s1.elih /fixed/home

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

USB Stick formatieren und partitionieren

::

   Fixit> fdisk -BI /dev/da0
   Fixit> bsdlabel -B -w /dev/da0s1
   Fixit> newfs /dev/da0s1a

USB Directory erzeugen und einhängen

::

   Fixit> mkdir /usb
   Fixit> mount /dev/da0s1a /usb

Den Boot Ordner vom Installierten System auf den USB Stick kopieren

::

   Fixit> mkdir /usb/boot
   Fixit> cp -Rpv /fixed/boot/ /usb/

Damit man booten kann muss die kernel unter /boot/kernel/kernel zu
finden sein. Also GENERIC nach kernel verschieben.

::

   Fixit> rmdir /usb/boot/kernel
   Fixit> mv /usb/boot/GENERIC /usb/boot/kernel
   Fixit> cd /usb/boot/kernel

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

Den Geli Encryption Key auf den USB Stick kopieren.

::

   Fixit> cp /root/keys/ad10s1.key /usb/ad10s1.key

Damit man die Festplatte entschlüsseln kann muss das Geli Modul geladen
werden und der Pfad für das Keyfile angegeben werden. Hierzu wird
folgender Eintrag in der loader.conf auf dem USB Stick angelegt:

::

   Fixit> vi /usb/boot/loader.conf
       geom_eli_load="YES"
       geli_ad10s1_keyfile0_load="YES"
       geli_ad10s1_keyfile0_type="ad10s1:geli_keyfile0"
       geli_ad10s1_keyfile0_name="/ad10s1.key"

Jetzt muss noch eine fstab auf dem USB Stick angelegt werden, welcher
die root-partition mountet.

::

   Fixit> mkdir /usb/etc
   Fixit> cd /usb/etc
   Fixit> vi fstab
       /dev/ad10s1.elia    /   ufs rw  1   1

Nach dem Mount der root Partition ist der USB Stick aus dem namespace
raus, darum muss noch eine fstab in der root Partition der Festplatte
angelegt werden, wo dann der Rest geladen werden muss.

::

   Fixit> vi /fixed/etc/fstab
       /dev/ad10s1.elib    none    swap    sw  0   0
       /dev/ad10s1.elid    /usr    ufs rw  2   2
       /dev/ad10s1.elie    /var    ufs rw  2   2
       /dev/ad10s1.elif    /tmp    ufs rw  2   2
       /dev/ad10s1.elig    /root   ufs rw  2   2
       /dev/ad10s1.elih    /home   ufs rw  2   2

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

/etc/resolve.conf anpassen

::

   domain      domain.de
   nameserver      217.237.151.51
   nameserver      217.237.149.205

Zusätzliche Packages installieren
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   # mount_cd9660 /dev/acd0 /mnt
   # cd /mnt/7.2-RELEASE/manpages && ./install.sh
   # cd /mnt/7.2-RELEASE/src && ./install.sh all
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

Fehler
------

Sollte der Systemstart nach der Installation nicht klappen, da man evtl.
das keyfile falsch in die loader.conf eingetragen hat oder sicht sonst
irgendwo vertippt hat, kann man immer noch das System mit der DVD
starten, die Fixit shell aufrufen und alles ändern. Hier zu muss aber
wieder Geli geladen werden, die symlinks wie oben entsprechend angepasst
werden, die Partitionen und den USB stick einhängen.

Quellen
-------

`HOWTO: GELI+ZFS for whole system inc. root with boot from USB
stick <http://forums.freebsd.org/showthread.php?t=2775>`__

`FreeBSD/System verschlüsseln mit
geli(8) <https://wiki.uugrn.org/FreeBSD/System_verschl%C3%BCsseln_mit_geli%288%29>`__

`System verschlüsseln mit
geli(8) <https://www.bsdwiki.de/System_verschl%C3%BCsseln_mit_geli%288%29>`__

`FreeBSD Handbook - Disks
Encryption <http://www.freebsd.org/doc/en/books/handbook/disks-encrypting.html>`__

* :ref:`genindex`

Zuletzt geändert: |date|

