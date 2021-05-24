USB-Stick Installation
======================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-netbsd.png

Die schnellste Möglichkeit zum Installieren und Testen von NetBSD bieten
Emulatoren wie QEMU. Leider lässt sich QEMU auf sehr aktuellen Systemen mit dem
Compiler gcc4 noch nicht kompilieren.  Ausserdem wird das Test-System in einer
komplett simulierten Hardware-Umgebung gebootet; es ist so nicht überprüfbar,
ob das System später auf der realen Hardware wirklich booten wird.

Das immer noch sehr übliche Brennen von Installations-CDs oder -DVDs ist
dagegen vielen Anwendern lästig geworden, da die Zuverlässigkeit von
Rohlingen, Brennern und den ständig wechslenden Firmwares oft sehr
schlecht ist.

Als alternative Installationsmedien bieten sich USB-Sticks an oder
externe USB- oder Firewire-Laufwerke. Der folgende Artikel beschreibt
die Schritte, die unter NetBSD nötig sind, um einen USB-Stick später als
Installationsmedium booten zu können.

Den USB-Stick bootbar machen
----------------------------

Das Einrichten des USB-Sticks erfolgt auf einem NetBSD-System. Neben
einem neuen oder bereits gebrauchten USB-Stick mit mindestens 256MB
Speicherkapazität wird ein CD-Image der gewünschten NetBSD-Version
benötigt. Das ca. 200MB grosse CD-Image ist im Internet auf vielen
FTP-Servern verfügbar.

Generell empfiehlt es sich, als ersten Schritt den Master Boot Record
(MBR) neu zu installieren. Dies ist zwar nicht unbedingt notwendig,
stellt aber sicher, dass der USB-Stick booten wird::

  $ su # fdisk -i /dev/sd1d

Ausserdem ist es empfehlenswert, zunächst unter *fdisk(8)* eine
NetBSD-Partition einzurichten:

::

  # fdisk -u /dev/rsd1d
  Disk: /dev/rsd1d
  NetBSD disklabel disk geometry:
  cylinders: 974, heads: 128, sectors/track: 8 (1024 sectors/cylinder)
  total sectors: 997375
  BIOS disk geometry:
  cylinders: 974, heads: 128, sectors/track: 8 (1024 sectors/cylinder)
  total sectors: 997375
  Do you want to change our idea of what BIOS thinks? [n] n
  Partition table:
  0: Primary DOS with 32 bit FAT (sysid 11)
     start 8, size 997367 (487 MB, Cyls 0-973/127/8)
  1: <UNUSED>
  2: <UNUSED>
  3: <UNUSED>
  Bootselector disabled.
  Which partition do you want to change?: [none] 0
  The data for partition 0 is:
  Primary DOS with 32 bit FAT (sysid 11)
     start 8, size 997367 (487 MB, Cyls 0-973/127/8)
  sysid: [0..255 default: 11] 169
  start: [0..974cyl default: 8, 0cyl, 0MB] ↵
  size: [0..974cyl default: 997367, 974cyl, 487MB]
  bootmenu: [] ↵
  Partition table:
  0: NetBSD (sysid 169)
      start 8, size 997367 (487 MB, Cyls 0-973/127/8)
  1: <UNUSED>
  2 : <UNUSED>
  3: <UNUSED>
  Bootselector disabled.
  Which partition do you want to change?: [none] ↵
  We haven't written the MBR back to disk yet.  This is your last chance.
  Partition table:
  0: NetBSD (sysid 169)
      start 8, size 997367 (487 MB, Cyls 0-973/127/8)
  1: <UNUSED>
  2: <UNUSED>
  3: <UNUSED>
  Bootselector disabled.
  Should we write new partition table? [n] y

Zusätzlich sollte die NetBSD-Partition unter fdisk noch auf "Active"
gesetzt werden:

::

  # fdisk -a sd1
  Disk: /dev/rsd1d
  NetBSD disklabel disk geometry:
  cylinders: 974, heads: 128, sectors/track: 8 (1024 sectors/cylinder)
  total sectors: 997375
  BIOS disk geometry:
  cylinders: 974, heads: 128, sectors/track: 8 (1024 sectors/cylinder)
  total sectors: 997375
  Partition table:
  0: NetBSD (sysid 169)
      start 8, size 997367 (487 MB, Cyls 0-973/127/8)
  1: <UNUSED>
  2: <UNUSED>
  3: <UNUSED>
  Bootselector disabled.
  Do you want to change the active partition? [n] y
  Choosing 4 will make no partition active.
  active partition: [0..4 default: 4] 0
  Are you happy with this choice? [n] y

NetBSD nutzt eine eigene Partitionstabelle, nicht die von *fdisk*. Diese
wird mit *disklabel(8)* angelegt:

::

  # disklabel -i -I
  partition> a
  Filesystem type [?] [MSDOS]: 4.2BSD
  Start offset ('x' to start after partition 'x') [0.0078125c, 8s, 0.00390625M]: ↵
  Partition size ('$' for all remaining) [973.991c, 997367s, 486.996M]: $
  partition> d
  Filesystem type [?] [unused]: ↵
  Start offset ('x' to start after partition 'x') [0c, 0s, 0M]: ↵
  Partition size ('$' for all remaining) [973.999c, 997375s, 487M]: ↵
  partition> W
  Label disk [n]? y
  Label written
  We haven't written the MBR back to disk yet.  This is your last chance.
  Should we write new partition table? [n] y


Die Partitionen stehen nun fest. Es fehlt noch das Dateisystem auf der
Partition sd1a, das mit *newfs(8)* angelegt wird:

::

  # newfs /dev/sd1a

Allerdings kann die Partition noch nicht gebootet werden. Die folgenden
Schritte sind abschließend notwendig:

::

  # mount /dev/sd1a /mnt 
  # cp /usr/mdec/boot /mnt 
  # installboot -v -o timeout=5 /dev/rsd1a /usr/mdec/bootxx_ffsv1

Den USB-Stick als Image sichern
-------------------------------

Der USB-Stick ist nun generell bootfähig (auch wenn ihm das zu bootende
System noch fehlt!). Diese Prozedur erscheint vergleichsweise aufwendig.
Deshalb ist es sinnvoll, von dem Stick ein Image zu erzeugen, das bei
Bedarf einfach auf weitere USB-Sticks kopiert werden kann. Dafür können
die Programme *dd(1)* oder das schnellere ``sdd`` genutzt werden:

::

  # dd if=/dev/rsd1d of=/home/mark/usb_boot.img bs=1m

Das Image auf einen zweiten USB-Stick kopieren:

::

  # dd if=/home/mark/usb_boot.img of=/dev/rsd2d bs=1m

Die Installations-Kernel und -Sets auf den USB-Stick kopieren
-------------------------------------------------------------

Der USB-Stick ist nun bootfähig, aber ihm fehlt noch das, was eigentlich
gebootet werden soll: das Betriebssystem oder wie in userem Fall das
NetBSD-Installations-System mit dem Installations-Kernel. Das spätere
Betriebssystem steckt in den Installations-Sets, die während der
Installation entpackt und auf die Festplatte kopiert werden. Auch diese
Installations-Sets werden auf dem USB-Stick benötigt.

Am einfachsten ist es, die komplette Installations-CD herunterzuladen,
die als CD-Image auf vielen FTP-Servern liegt:

::

  $ cd /home/mark/
  $ ftp -a ftp:ftp.netbsd.org/pub/NetBSD/iso/3.0.1/i386cd-3.0.1.iso

Um die benötigten Dateien aus dem CD-Image auf den USB-Stick kopiern zu können,
muss die Image-Datei ins Dateisystem eingebunden werden:

::

  $ su
  # vnconfig -c vnd0 /home/mark/imagefile.iso
  # mount -t cd9660 /dev/vnd0d /image/

Der Installations-Kernel und die Sets werden dann auf den gemounteten USB-Stick
kopiert:

::

  # mount /dev/sd1a /stick/
  # cp /imag/binary/kernel/netbsd-INSTALL.gz /stick/netbsd.gz
  # cp -R /image/binary/sets/ /stick/sets/

Danach ist der USB-Stick fertig eingerichtet. Nach dem Runterfahren des
Computers, muss der USB-Stick im BIOS als das zu bootende Laufwerk eingestellt
werden. NetBSD sollte im Anschluss daran einwandfrei booten und mit einem
blauen Installations-Menü grüssen.

Anmerkungen zur Installation
----------------------------

Eigentlich läuft die Installation nun problemlos und nach dem vertrauten
Muster ab. Der folgende Schritt könnte jedoch Probleme bereiten:

::

  Ihre Festplatte ist nun bereit für die Installation der Kernel- und 
  Distributionspakete. [...]

Hier muss das richtige Medium ausgewählt werden, und zwar:

::

  >f: Ungemountetes Dateisystem

Mit RETURN bestätigen und es erscheint der folgende Screen:

::

  Geben Sie das noch nicht gemountete lokale Gerät und dessen entsprechendes
  Verzeichnis an, im dem die Distribution zu finden ist. (Das Verzeichnis muss
  .tgz Dateien enthalten)

Hier sind die Optionen wie folgt zu setzen (aber dies ist nur ein Beispiel,
bitte gegebenenfalls anpassen!!):

::

  a: Gerät           sd0a
  b: Dateisystem     ffs
  c: Basispfad       
  d: Verzeichnis     /sets

Sind die Optionen richtig gewählt, sollte die Installation wie gewohnt und erfolgreich verlaufen.

Mehr zum Thema USB-Sticks und BSD
---------------------------------

-  `USB-Stick als System-Laufwerk nutzen mit "NetBSD LiveKey" <USB-Stick als System-Laufwerk nutzen mit "NetBSD LiveKey">`__
-  `USB-Stick Unterstützung <USB-Stick Unterstützung>`__

* :ref:`genindex`

Zuletzt geändert: |date|

