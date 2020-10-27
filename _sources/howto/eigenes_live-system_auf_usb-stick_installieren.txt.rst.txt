Eigenes Live-System auf USB-Stick installieren
==============================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Um ein FreeBSD "immer dabei" zu haben besteht die Möglichkeit es auf
einen USB-Stick zu installieren. Da eine normale Installation einige
Nachteile mit sich bringt und ggf. sogar gar nicht booten will ist hier
ein ein Weg beschrieben um ein Live-System für einen USB-Stick zu
erstellen. Der Stick wird als Live-System gebaut, wie man es von CDs
kennt, und wird daher keine Änderungen speichern. Somit hat man immer
ein sauberes System zur Hand.

Um ein System zu bauen welches auch Änderungen speichert sind lediglich
kleine Änderungen an der Partitionierung und/oder der fstab vorzunehmen.

Image erstellen
---------------

Zunächst wird ein Image auf der Festplatte erzeugt welches am Ende auf
den USB-Stick kopiert wird. Dieses Image kann dann abseits des
USB-Sticks erstellt und gepflegt werden und erlaubt es recht schnell
mehrere USB-Sticks anzufertigen.

Um die genauen Geometriedaten des Sticks zu bekommen wird er an den
Rechner angeschossen und per ``'dmesg``' die Zeile gesucht welche seine
Daten angibt. In diesem Beispiel handelt es sich um einen 1GB-USB-Stick.

Stick aus dmesg auslesen
~~~~~~~~~~~~~~~~~~~~~~~~

Ein dmesg liefert dann:

::

   da0: 979MB (2004992 512 byte sectors: 64H 32S/T 979C)

Image anlegen
~~~~~~~~~~~~~

Um nun ein Image mit exakt dieser Größe auf der Festplatte anzulegen
wird per ``'dd``' eine leere Datei erzeugt: 

::

  % dd if=/dev/zero of=stick.img bs=512 count=2004992

Datei als Device einbinden
~~~~~~~~~~~~~~~~~~~~~~~~~~

Um auf der Datei arbeiten zu können wie auf einem echten Datenträger
muss sie als Device eingebunden werden. Dies wird über den Weg einer
Memorydisk mittels ``'mdconfig``' erledigt: 

::

  # mdconfig -f stick.img md0

Das erzeugte Device heißt also ``'md0``' und sollte nun unter ``/dev`` zu
finden sein.

Partitionieren
~~~~~~~~~~~~~~

Als nächstes müssen auf dem Device die nötigen Partitionen und die
entsprechende Partitionstabelle angelegt werden. Es wird hierbei mit dem
``'GPT``'-Partitionsschema gearbeitet.

GPT-Tabelle anlegen
^^^^^^^^^^^^^^^^^^^

Zuerst wird die Partitionstabelle angelegt 

::

  # gpart create -s GPT /dev/md0

Partitionstabelle anzeigen
^^^^^^^^^^^^^^^^^^^^^^^^^^

Mit dem folgenden Befehl kann immer angezeigt werden wie die
Partitionstabelle aktuell aussieht. Dies kann auch zwischendurch
sinnvoll sein um sich einen Überblick zu verschaffen. 

::

  # gpart show md0

Bootcode in das Device schreiben
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Damit das Device am Ende auch booten kann muss es mit dem entsprechenden
bootcode versehen werden. 

::

  # gpart bootcode -b /boot/pmbr md0

Bootpartition anlegen und mit Bootdaten füllen
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nun wird noch eine Partition mit den Bootdaten benötigt von welcher das
System dann starten kann. Diese wird dann noch mit dem entsprechenden
Bootcode versehen.

::

  # gpart add -b 34 -s 128 -t freebsd-boot md0
  # gpart bootcode -p /boot/gptboot -i 1 md0

BSD-Partition anlegen
^^^^^^^^^^^^^^^^^^^^^

Nun können die gewünschten Partitionen für das System angelegt werden.
In diesem Fall wird nur eine Partition für das komplette System
angelegt.

::

  # gpart add -b 162 -s 2004797 -t freebsd-ufs /dev/md0

Dateisystem anlegen
^^^^^^^^^^^^^^^^^^^

Nachdem die Systempartition angelegt wurde muss sie noch mit einem
Dateisystem versehen werden. Hier wird ein UFS2 Dateisystem mit
Softupdates erschaffen, welches noch ein Label-Namen erhält. Dies ist
wichtig, dass das System später auch an verschiedenen USB-Ports starten
kann!

::

  # newfs -U -L STICKBSD /dev/md0p2

Mounten des Devices
^^^^^^^^^^^^^^^^^^^

Nachdem nun die Systempartition angelegt und formatiert ist kann sie
gemountet werden um sie weiter zu bearbeiten.

::

  # mount /dev/ufs/STICKBSD /mnt

Laden aktueller Systemquellen
-----------------------------

Zunächst sind die aktuellen Systemquellen erforderlich. Diese können per
cvsup heruntergeladen werden.

Kompilieren des Systems
-----------------------

Das System wird aus den Quellen kompiliert und installiert. Zunächst
wird im Verzeichnis ``/usr/src`` sowohl die "Welt" als auch der Kernel
kompiliert. Da in diesem Fall der Buildrechner ein ``amd64``-System ist
muss der Buildtarget noch auf ``i386`` geändert werden um auch auf
älteren Systemen ohne ``amd64``-kompatible-CPU lauffähig zu sein. Sollte
der Buildrechner selbst ein ``i386``-System sein ist die Angabe der
``TARGET``- und ``TARGET_ARCH``-Anweisungen unnötig.

::

  # cd /usr/src 
  # make TARGET=i386 TARGET_ARCH=i386 buildworld 
  # make TARGET=i386 TARGET_ARCH=i386 buildkernel

Das kompilierte Ergebnis liegt in diesem Fall (des Crosscompiles) unter
``/usr/obj/i386``.

Installieren des Systems
------------------------

Nach Kompilieren des Systems und des Kernels können diese nun in die
neue Partition installiert werden.

::

  # make TARGET=i386 TARGET_ARCH=i386 installworld DESTDIR=/mnt
  # make TARGET=i386 TARGET_ARCH=i386 installkernel DESTDIR=/mnt
  # make TARGET=i386 TARGET_ARCH=i386 distribution DESTDIR=/mnt

``make installworld`` installiert die "Welt", ``make installkernel``
installiert den Kernel und ``make distribution`` legt die Konfigurationsdateien
in /etc an. Wichtig ist hier das ``'DESTDIR``', also das Zielverzeichnis
richtig anzugeben.

Nun ist der erste Schritt getan.

Ports verfügbar machen
----------------------

Um das installierte System ein wenig umfangreicher an Software
auszustatten werden die Ports des Buildrechners per Nullfs-Mount zur
Verfügung gestellt. Somit wird kein Platz im Device verschwendet.

::

  # cd /mnt/usr 
  # mkdir ports 
  # mount_nullfs -o ro /usr/ports /mnt/usr/ports

Nun muss noch die Datei ``/mnt/etc/make.conf`` mit folgendem Inhalt
ergänzt oder erzeugt werden damit die Ports nicht im Original-Portbaum
gespeichert oder gebaut werden.:

::

   WRKDIRPREFIX=/tmp
   DISTDIR=/tmp/distfiles
   PACKAGES=/tmp/packages

Damit nun das ``/tmp``-Verzeichnis des Images nicht überläuft wird für
die Dauer des Portbaus Festplattenplatz für das Verzeichnis verwendet.
Also wird nochmals ein Nullfs-Mount (diesmal jedoch mit Schreibrechten)
durchgeführt. Angenommen ein zur Verfügung gestelltes Verzeichnis trägt
den Namen ``/usr/home/USER/temp-fuer-stick``. Dann lautet der Befehl:

::

  # nullfs_mount /usr/home/USER/temp-fuer-stick /mnt/tmp

Am Ende aller Installationen kann das Verzeichnis wieder gelöscht werden.

Anpassen des Systems
--------------------

Nun werden Änderungen direkt am Image-System vorgenommen.

Ins Image wechseln
~~~~~~~~~~~~~~~~~~

Um nun das System noch weiter anzupassen wird mittels ``chroot`` ins
neue Image gewechselt: 

::

  # chroot /mnt /bin/csh

/etc/fstab anlegen
~~~~~~~~~~~~~~~~~~

Um später ein schönes Live-System zu haben wird das ``/``-Verzeichnis
ReadOnly eingebunden. Anschließend werden zwei RAM-Disks erstellt welche
Änderungen am System bis zum Neustart speichern können. Das Verzeichnis
``/var`` enthält statische Daten welche jedoch im Betrieb Änderungen
erfahren. Daher wird das ReadOnly-Verzeichnis ``/var`` nun per
``'unionfs``' mit einer RAM-Disk zusammengeführt.

::

    /dev/ufs/STICKBSD    /            ufs     ro                1    1
    md                   /mvar        mfs     rw,-s10M,noatime  0    0
    md                   /tmp         mfs     rw,-s5M,noatime   0    0
    /mvar                /var         unionfs rw                0    0
    proc                 /proc        procfs  rw                0    0

Anschließend muss ``/mdvar`` noch angelegt werden.

::

  # mkdir /mvar

Weitere Anpassungen
~~~~~~~~~~~~~~~~~~~

Weitere Anpassungen können nun am System vorgenommen werden. So können
nun noch Ports installiert, Benutzer angelegt oder Konfigurationen
verändert werden. Vor Allem sollte die ``/etc/rc.conf`` erzeugt und der
Hostnamen des Systems definiert werden. Zudem kann die
``/boot/loader.conf`` angepasst werden um bestimmte Module zu laden,
z.B. für Sound, Netzwerk, etc.

root-Kennwort ändern
~~~~~~~~~~~~~~~~~~~~

Das Kennwort des Benutzers root sollte nun noch geändert werden.

::

  # passwd

Aufräumen
---------

Um nun alle arbeiten zu beenden wird die ``chroot``-Umgebung verlassen
(falls nicht schon geschehen). Die Image-Partition wird ausgehängt und
das Memorydevice freigegeben.

::

  # umount /mnt # mdconfig -d -u md0

Nun kann noch das temporäre Verzeichnis, welches für eventuelles
Port-bauen verwendet wurde gelöscht werden.

::

  # rm -r /usr/home/USER/temp-fuer-stick

Image installieren
------------------

Das Image, welches nun fertig sein sollte, kann mit folgendem Befehl auf
den USB-Stick bekannt werden. Der Stick darf hierfür nicht gemounted
sein! Vorausgesetzt der Stick heisst /dev/da0.

::

  # dd if=stick.img of=/dev/da0 bs=4096

Änderungen
----------

Um Änderungen am System durchzuführen wird das Image einfach wieder per
``'mdconfig -f``' gemounted und kann verändert werden. Anschließend wird
das Image neu auf den Stick geschoben.

* :ref:`genindex`

Zuletzt geändert: |date|

