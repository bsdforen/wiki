USB-Stick als System-Laufwerk nutzen mit "NetBSD LiveKey"
=========================================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-netbsd.png

Das Besondere von NetBSD LiveKey ist, dass auf dem USB-Stick ein DOS bzw.
Windows-Dateisystem (FAT32) angelegt ist. Das ist praktisch, da sich der Stick
damit auch weiterhin zum Speichern und Austauschen von Daten zwischen den
diversen Betriebssystemen eignet. Das NetBSD-System belegt nämlich nur 64Mb auf
dem USB-Stick. Die Installation sollte auch unter Linux- oder anderen
BSD-Systemen möglich sein (allerdings von mir noch nicht getestet!).

Homepage des Projekts: http://imil.net/nlk

Download des tarfiles: http://imil.net/nlk/livekey.tar.gz

Einen gebrauchten USB-Stick vorbereiten
---------------------------------------

Die Schritte in diesem Abschnitt können bei unbenutzten USB-Sticks
entfallen, da sie normalerweise mit dem FAT32-Dateisystem vor-bespielt
sind. Mithilfe des dialogbasierten ``fdisk -u`` wird der System-Typ der
ersten Partition geändert: von NetBSD (oder was auch immer) auf "Primary
DOS with 32 bit FAT"

::

  # fdisk -u /dev/rsd1d
  Disk: /dev/rsd1d
  NetBSD disklabel disk geometry:
  cylinders: 974, heads: 128, sectors/track: 8 (1024 sectors/cylinder)
  total sectors: 997375
  BIOS disk geometry:
  cylinders: 974, heads: 128, sectors/track: 8 (1024 sectors/cylinder)
  total sectors: 997375
  Do you want to change our idea of what BIOS thinks? [n] ↵
  Partition table:
  0: NetBSD (sysid 169)
  start 8, size 997367 (487 MB, Cyls 0-973/127/8)
  1: <UNUSED>
  2: <UNUSED>
  3: <UNUSED>
  Which partition do you want to change?: [none] 0
  The data for partition 0 is:
  NetBSD (sysid 169)
  start 8, size 997367 (487 MB, Cyls 0-973/127/8)
  sysid: [0..255 default: 169] 11
  start: [0..974cyl default: 8, 0cyl, 0MB] ↵
  size: [0..974cyl default: 997367, 974cyl, 487MB] ↵
  bootmenu: [] ↵
  The bootselect code is not installed, do you want to install it now? [n] n
  Partition table:
  0: Primary DOS with 32 bit FAT (sysid 11)
  start 8, size 997367 (487 MB, Cyls 0-973/127/8)
  1: <UNUSED>
  2: <UNUSED>
  3: <UNUSED>
  Which partition do you want to change?: [none] ↵
  Installed bootfile doesn't support required options.
  Update the bootcode from /usr/mdec/mbr? [n] ↵
  We haven't written the MBR back to disk yet.  This is your last chance.
  Partition table:
  0: Primary DOS with 32 bit FAT (sysid 11)
  start 8, size 997367 (487 MB, Cyls 0-973/127/8)
  1: <UNUSED>
  2: <UNUSED>
  3: <UNUSED>
  Should we write new partition table? [n] y

Da NetBSD die von fdisk erstellte Partitionstabelle nicht nutzt, muss
noch eine zweite, NetBSD-kompatible erstellt werden. Dies geschieht mit
``mbrlabel`` und ``disklabel``. Zunächst wird der Disklabel-Editor
aufgerufen, um alle angzeigten Partitionen zu löschen, da ansonsten das
spätere Aufrufen von ``mbrlabel`` das Disklabel durcheinander bringen
kann:

::

  # disklabel -e -I /dev/sd1d
  (Alle Partitionen löschen!)

Mit ``mbrlabel`` wird die von fdisk in den MBR geschriebene
Partitionstabelle ausgelesen:

::

  # mbrlabel -frw /dev/sd1
  Found MSDOS partition; size 997367 (486 MB), offset 8
  adding MSDOS partition to slot a.
  4 partitions:
  #        size    offset     fstype [fsize bsize cpg/sgs]
  a:    997367         8      MSDOS                     # (Cyl.      0*-    973*)
  Updating in-core and on-disk disk label.

Zuletzt wird noch die Partition d angelegt, die unter NetBSD für das
gesamte Laufwerk steht:

::

  # disklabel -i -I /dev/sd1
  partition> d
  Filesystem type [?] [unused]: ↵
  Start offset ('x' to start after partition 'x') [0c, 0s, 0M]: ↵
  Partition size ('$' for all remaining) [0c, 0s, 0M]: $
    d:    997375         0     unused      0     0        # (Cyl.      0 -    973*)
  partition> W
  Label disk [n] Y
  Label written
  partition> Q


Die Partitionstabellen sind damit fertig angelegt. Es fehlt noch das
eigentliche Aufspielen des neuen Dateisystems mit ``newfs_msdos(8)``:

::

  # newfs_msdos -F 32 /dev/sd1a

Nun kann der eigentliche Installationsvorgang beginnen!

Den USB-Stick bootbar machen: GRUB installieren und konfigurieren
-----------------------------------------------------------------

Der Stick wird gemountet, die von GRUB benötigten Dateien werden installiert::

  # mount_msdos /dev/sd1a /mnt
  # grub-install --root-directory=/mnt /dev/sd1d
  # cd /mnt/grub
  # touch stick

Zur Konfiguration von GRUB wird die Datei menu.lst erstellt und angepasst::

  # vi /mnt/grub/menu.lst
  title NetBSD LiveKey
  root (hd0,0)
  kernel --type=netbsd /livekey/netbsd 
  boot

Die GRUB-shell starten um GRUB in den Master Boot Record (MBR) zu installieren::

  # grub
  grub> find /grub/stick
  (hd2,0)
  grub> root (hd2,0)
  grub> setup (hd2)

Der USB-Stick ist nun bootbar enthält aber noch kein Betriebssystem. Dieses wird im nächsten Schritt installiert.

Die Installation von NetBSD LiveKey
-----------------------------------

Zur Installation des Betriebssystems wird in das Verzeichnis gewechselt,
wohin der USB-Stick gemountet wurde. Anschliessend muss das System nur
noch auf den Stick entpackt werden:

::

  # cd /mnt
  # tar zxvf /home/mark/livekey.tar.gz

Nun kann der USB-Stick gebootet werden (login: root, password ist noch
keins gesetzt)

Mehr zum Thema USB-Sticks und BSD
---------------------------------

-  `USB-Stick-Installation <USB-Stick-Installation>`__
-  `USB-Stick bootfähig machen <USB-Stick bootfähig machen>`__
-  `USB-Stick Unterstützung <USB-Stick Unterstützung>`__

* :ref:`genindex`

Zuletzt geändert: |date|

