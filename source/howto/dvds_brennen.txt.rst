DVDs brennen
============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Dieser Artikel über das Brennen von DVDs gibt nicht nur praktische
Beispiele, sondern beleuchtet auch Hintergründe über die verschiedenen
Formate.

.. note::

  Dieser Artikel stammt von **[moR-pH-euS]** und wurde zuerst `im Forum
  <http://www.bsdforen.de/showthread.php?t=1894>`__ veröffentlicht.

Erklärungen zu DVD-Standards
----------------------------

Die meisten werden wahrscheinlich wissen, dass es zwei verschiedene
Standards beim DVD-Brennen gibt: DVD+ und DVD-. Um etwas Klarheit in
diesen Dschungel von verschieden Formaten zu bringen, versuche ich in
den beiden folgenden Abschnitten einmal die Medien und die wesentlichen
Unterschiede zwischen diesen beiden "Standards" zu erklären, da man
schon wissen sollte wofür die einzelnen Begriffe eigentlich stehen, wenn
man sich schon damit beschäftigt.

Erklärung der Medien
~~~~~~~~~~~~~~~~~~~~

Man muss zu allererst die 5 verschiedenen Standards der beschreibbaren
DVD-Medien unterscheiden:

DVD-R
^^^^^

Der Urahn der beschreibbaren DVD. Das DVD-Forum, das die ursprünglichen
Standards für die DVDs erarbeitet hat (die sind z.B. auch fuer die
Region Codes und CSS verantwortlich), steht auf dieser Seite und hat
diesen Standard auch offiziell anerkannt. Die DVD-R ist stark an der
DVD-ROM angelehnt und wird deswegen auch von den meisten DVD-Playern
abgespielt - von ca. 90% (sie hat eine ähnliche Datenstruktur und
Reflexionseigenschaft wie die DVD-ROM)

DVD-RW
^^^^^^

Die DVD-RW ähnelt eigentlich auch der DVD-ROM in der Datenstruktur, kann
aber zu Problemen mit DVD-Playern und Laufwerken führen, da die
Reflexionseigenschaften der einer DVD-9 ähneln (also einer DVD-ROM mit
9GB) und der Laser anhand der Reflexion das Medium erkennt, er dann
leicht "durcheinander" kommt und nach falschen Layern sucht (er sucht
dann z.B. nach der halbtransparenten schicht der DVD-9 anstatt der
vorhandenen RW-Schicht) - wird auf ungefähr 75% aller DVD-Player
abgespielt (dieselbe Zahl gilt auch fuer DVD+RW). Die DVD-RW kann bis zu
1000 x beschrieben werden.

DVD+R
^^^^^

Einige Mitglieder des DVD-Forums, haben den DVD+ Standard geschaffen und
sich zur DVD+RW Alliance zusammengeschlossen. Dieser Standard weicht
etwas von der Norm ab. Dieses Format wurde allerdings vom Konsortium
nicht abgesegnet, da einer der Hauptgruende fuer dieses neue Format
darin besteht keine Lizenzkosten an das DVD-Forum zu zahlen. DVD+R wird
auf ca. 85% aller DVD-Player abgespielt.

DVD+RW
^^^^^^

Der wesentliche Unterschied zu DVD-RW liegt darin, dass +RW aufgrund
einer anderen Struktur des Spurrandes beim Brennen Sektoren genauer
adressieren kann. Dank des hochfrequenten Wobble können Brenner ohne den
Umweg von Land Pre-Pits? und Linking Sectors direkt an vorhandene
Datenbestände anknüpfen - auch Lossless Linking genannt. Zudem können
+RW-Writer die Medien auch in Teilen neu beschreiben - interessant für
standalone Recorder. Für PC-Laufwerke ist diese Fähigkeit dagegen
weniger relevant. Da die DVD+RW gleiche Reflexionseigenschaften wie die
DVD-RW hat, kann es zu oben genannten Problemen führen.

DVD-RAM
^^^^^^^

DVD-RAM führt ein Schattendasein. Es wurde eigentlich für PCs
entwickelt, d.h. es wurde von vorneherein als reines Datenspeichermedium
entwickelt. Die Medien lassen sich bis zu 100.000 x beschreiben. Durch
die besonderen Reflexionseigenschaften der Medien können nur wenige
DVD-ROM-Laufwerke RAM-Medien lesen, das gilt auch fuer DVD Player. Die
Medien sind dabei in Caddys, ausser DVD-RAM-Medien vom Typ2 die sich aus
dem Caddy herausnehmen lassen. Das DVD-RAM Format hat sich bis jetzt nur
in Japan durchgesetzt, dabei wird es wohl auch bleiben, auch wenn es
noch DVD-Brenner gibt die auch RAM beherrschen. Hier steht auch das
DVD-Forum dahinter, die diesen Standard unterstützen.

Erklärung der Wesentlichen Unterschiede zwischen DVD+ und DVD-
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

DVD-
^^^^

Firmen die dahinter stehen bzw. das Konsortium für diesen Standard
bilden: z.B. Pioneer, Apple, Hitachi, NEC, Samsung, Sharp, Sony

Kopierschutz:

Auf den Medien sind die ersten Sektoren durch die Produktion bereits
vollgeschrieben, damit die Keys zum entschluesseln von DVDs (CSS) nicht
1:1 kopiert werden können.

DVD+
^^^^

Firmen die dahinter stehen bzw. das Konsortium für diesen Standard
bilden: z.B. Phillips, HP, Sony, Verbatim, Ricoh, Yamaha, Thomson, Dell

Kopierschutz:

Der CSS-Bereich ist bei dieser Art Medien nicht vollgeschrieben, da der
Brenner mit der Firmware, also hardwareseitig, den Kopierschutz
darstellt.

DVD+ ist multisessionfähig

Fazit:
~~~~~~

Meine Empfehlung wäre es sich einen Multi-DVD-Brenner zu holen, der alle
oben aufgeführten Standards brennen und verstehen kann (DVD+, DVD- und
DVD-RAM, das habe ich zumindest gemacht mit dem lg-4040b). Da sich bis
heute noch nicht abgezeichnet hat, welcher standard sich durchsetzen
wird, ist das die zur Zeit beste Lösung.

DVDs-Brennen unter FreeBSD
--------------------------

Hardware
~~~~~~~~

Zuerst solltet ihr einmal mit "dmesg" sehen, ob euer Brenner überhaupt
richtig erkannt wird. Bei mir sieht die Ausgabe so aus:

::

   # dmesg
   acd0: DVD-R <HL-DT-ST DVDRAM GSA-4040B> at ata1-slave PIO4

Multimedia-Software unter Unix (rippen, decoden, brennen)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Das hier ist mal eine Liste mit Tools die ich kenne/gefunden habe, ist
sicherlich eine unvollständige Liste, deswegen ergänzt bitte als
Anschluss am Thread eure Erfahrungen mit den Tools bzw. postet die Tools
die ich vergessen habe, damit ich sie später hier noch einbauen kann.
Danke!

Rippen, Encoden:
~~~~~~~~~~~~~~~~

transcode
^^^^^^^^^

Ein sehr mächtiges Tool um Filme zu rippen, encoden, transcoden,
multiplexen usw.

http://www.transcoding.org/cgi-bin/transcode bzw.

::

   /usr/ports/multimedia/transcode

dvd::rip
^^^^^^^^

Wie der Name schon vermuten lässt, ist es ein Perl-Programm um DVDs auf
die Platte zu rippen (ist ein Frontend für transcode)
http://www.exit1.org/dvdrip/ bzw.

::

   /usr/ports/multimedia/dvdrip

mencoder
^^^^^^^^

Mit `mencoder </anwendungen/mplayer>`__ kann man ebenfalls DVDs in einem
Rutsch rippen und konvertieren. Dazu gibt es auch `ein HowTo mit Wrapper
Skript <DVD-Rip mit mencoder>`__.

cp
^^

Wer einfach nur ein `Image <ISO Images von optischen Medien erzeugen>`__
für eine spätere identische Kopie haben will. Kann einfach mit dem
Kommando

::

   # cp /dev/acd0 image.iso

eine ISO image anlegen. Von einem solchen Image kann auch ein `Device
erzeugt werden <ISO Images mounten>`__ um es zu mounten oder mit einer
DVD Software abzuspielen.

LDVD9to5
^^^^^^^^

Mit diesem Tool soll man eine DVD-9 auf ein DVD-5 format bekommen, d.h.
das die 9gb-DVD auf einen normalen DVD-Rohling komprimiert wird, habe
ich allerdings noch nicht getestet http://ldvd9to5.gff-clan.net/ bzw.

::

   /usr/ports/multimedia/ldvd9to5

KLVEMKDVD
^^^^^^^^^

Das ist ein Tool für KDE, dass auch auf die Bibliotheken von KDE und QT
aufbaut, ist allerdings noch ein Preview-Release;
http://lvempeg.sourceforge.net/klvemkdvd.html

dvdbackup
^^^^^^^^^

dvdbackup ist ein Kommandozeilen-Tool um Video-DVDs auf Festplatte zu
sichern (inklusive Menu, Kapitel, ...).
http://dvd-create.sourceforge.net/index.shtml bzw.

::

   /usr/ports/sysutils/dvdbackup

Authoring
~~~~~~~~~

Tools zum erstellen einer DVD http://dvdauthor.sourceforge.net/ bzw.

::

   /usr/ports/multimedia/dvdauthor

Daten brennen
~~~~~~~~~~~~~

Wir brauchen jetzt noch Software, um mit dem Brenner auch mal zu
brennen... Ich benutze hier nur die dvd+rw-tools und growisofs; In den
Ports findet sich unter

::

   /usr/ports/sysutils/dvd+rw-tools

mit diesem Port kann man DVD-R, DVD-RW, DVD+R und DVD+RW brennen; also:

::

   # cd /usr/ports/sysutils/dvd+rw-tools
   # make install clean

Es gibt auch noch die GNU dvdrecord Tools, die ein verändertes cdrecord
für DVD-Brenner darstellen, darauf gehe ich aber nicht ein. Wir
installieren uns zusätzlich noch cdrecord bzw. die cdrtools, da diese
auch mkisofs enthalten; also:

::

   # cd /usr/ports/sysutils/cdrtools
   # make install clean

Zuerst backen wir uns einen neuen Kernel mit den folgenden Optionen
(ohne diese Optionen funktioniert es nicht, es solllten eigentlich alle
Optionen bereits im Kernel enthalten sein, ausser atapicam)

::

   device ata
   device atapicd
   device atapicam
   device scbus
   device cd
   device pass

Das gilt für FreeBSD 4.x und auch FreeBSD 5.x... Diese Optionen sind
wichtig, da ansonsten der Brenner nicht über die IDE-SCSI-Emulation
angesprochen werden kann und genau über diesen Weg funktionieren die
Brennprogramme. Wie man einen Kernel kompiliert, könnt ihr im
`FreeBSD-Handbuch <http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/kernelconfig.html>`__
nachlesen.

Alternativ kann ab FreeBSD 6 atapicam auch als Modul geladen werden. Die
restlichen Devices sind teil des Standardkernels (so kann auch auf den
Neustart verzichtet werden).

::

   # kldload atapicam

Nach einem Neustart (hoffe mal das eure Kiste auch mit dem neuen Kernel
hochfährt ;-) schauen wir mal, ob die Laufwerke jetzt auch ueber
IDE-SCSI-Emulation noch gefunden werden; Das machen wir am besten mit
camcontrol bzw. cdrecord und die Ausgabe sieht bei mir so aus:

::

   root@freebsd-wk.home# camcontrol devlist
   <HL-DT-ST DVDRAM GSA-4040B A300> at scbus1 target 1 lun 0 (cd0,pass0)
   <PIONEER DVD-ROM DVD-115 1.08> at scbus3 target 0 lun 0 (cd1,pass1)

   root@freebsd-wk.home# cdrecord -scanbus
   Cdrecord 2.00.3 (i386-unknown-freebsd5.1) Copyright (C) 1995-2002 J\xf6rg Sc
   hilling
   Using libscg version 'schily-0.7'
   scsibus1:
   1,0,0 100) *
   1,1,0 101) 'HL-DT-ST' 'DVDRAM GSA-4040B' 'A300' Removable CD-ROM
   1,2,0 102) *
   1,3,0 103) *
   1,4,0 104) *
   1,5,0 105) *
   1,6,0 106) *
   1,7,0 107) *
   scsibus3:
   3,0,0 300) 'PIONEER ' 'DVD-ROM DVD-115 ' '1.08' Removable CD-ROM
   3,1,0 301) *
   3,2,0 302) *
   3,3,0 303) *
   3,4,0 304) *
   3,5,0 305) *
   3,6,0 306) *
   3,7,0 307) *

Das funktioniert schon mal... nun haben wir durch die DVD+RW-Tools drei
neue Kommandos:

::

   # dvd+rw-booktype

   Usage: dvd+rw-booktype [-dvd-rom-spec|-dvd+rw-spec|-dvd+r-spec|-inq] \
              [-media|-unit|-unit+rw|-unit+r] /dev/dvd 

Hiermit kann man den Booktype vor dem Brennen anscheinend ändern; mit
dem Booktype erkennt der DVD-Player, um welchen Typ von DVD es sich
handelt (DVD+R, DVD-ROM usw.) hier findet ihr noch ein paar
Informationen http://www.dvdplusrw.org/resources/bitsettings.html

::

   # dvd+rw-format

   * DVD±RW/-RAM format utility by <appro@fy.chalmers.se>, version 4.10.
   - usage: dvd+rw-format [-force[=full]] [-lead-out|-blank[=full]]
            [-ssa[=none|default|max]] /dev/cdrom

Mit diesem Kommando könnt ihr DVD+RW und DVD-RWs formatieren; (bei
Intensio DVD-RWs hatte der Erstersteller des vorliegenden HOWTOs
Probleme).

::

   # dvd+rw-mediainfo
   usage: dvd+rw-mediainfo /dev/dvd

Wie der Kommandoname schon vermuten lässt, bekommt ihr damit
Informationen über den eingelegten Rohling; mit growisofs könnt ihr
Daten-DVDs brennen. Als kleines Beispiel soll folgende Befehlszeile
dienen:

::

   # growisofs -Z /dev/cd0 -R -J *

Dieses Kommando brennt alle Dateien im aktuellen Verzeichnis auf die DVD
(als Dateisystem ISO9660 mit Joliet und Rock-Ridge? als Erweiterung).
Bei Dual-Layer-DVDs sollte die "-speed=N"-Option zusätzlich verwendet
werden, da growisofs sonst mit einem I/O-Fehler den Dienst verweigert.
Weitere Beispiele zum Brennen von Daten mit growisofs finden sich in der
Manpage.

Um die volle Geschwindigkeit ausnutzen zu können, muss DMA aktiviert
sein. DMA ist für CD/DVD-Laufwerke bei FreeBSD bis einschließlich 5.2.1
standardmäßig deaktiviert; bei FreeBSD 5.3 und höher sollten solche
Laufwerke bereits im DMA-Modus laufen. Um es zu aktiveren, muss man
folgenden Eintrag in die /boot/loader.conf einfügen und das System
neustarten:

hw.ata.atapi_dma=1

Ohne DMA würde ein dmesg z.b. folgendes ausgeben:

::

   # dmesg

   acd0: CDRW <HL-DT-STCD-RW/DVD DRIVE GCC-4240N> at ata1-master PIO4

Mit DMA:

::

   # dmesg

   acd0: CDRW <HL-DT-STCD-RW/DVD DRIVE GCC-4240N> at ata1-master UDMA33

Weiter weist die SCSI-Emulation damit eine viel höhere
Übertragungsgeschwindigkeit aus:

::

   cd0 at ata1 bus 0 target 0 lun 0
   cd0: <HL-DT-ST RW/DVD GCC-4240N 1202> Removable CD-ROM SCSI-0 device
   cd0: 33.000MB/s transfers

Anmerkung zum DMA-Modus
~~~~~~~~~~~~~~~~~~~~~~~

"Ich (tUNIX) bin unter FreeBSD 5.3 mit einem LG 4160B (Firmware: A302)
in die DMA-Falle getappt. Da bei mir DMA-Modus für Atapi-Geräte
standardmässig aktiviert war, habe ich diesem Parameter auch keine
Aufmerksamkeit geschenkt. Die DVDs liessen sich auch einwandfrei
beschreiben. Allerdings bekam ich beim Lesen der Daten nach dem Mounten
massig Fehler, welche ständig in einem Abbruch des Lesevorganges
mündeten.

Erst nach dem Ausschalten des DMA-Modus lief alles wie gewünscht."

Daten löschen
-------------

Eine DVD-RW könnt ihr mit diesem Befehl löschen:

::

   # dvd+rw-format -blank /dev/cd1

Quellen
-------

-  http://www.dvdforum.com/forum.shtml
-  http://www.dvdrw.com/
-  http://www.dvdrhelp.com/
-  http://www.heise.de/ct/02/25/112/default.shtml
-  http://fy.chalmers.se/~appro/linux/DVD+RW/
-  http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/creating-dvds.html
-  http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/creating-cds.html

Verweise
--------

-  `k3b einrichten </howto/k3b einrichten>`__ - KDE-Brennersoftware

* :ref:`genindex`

Zuletzt geändert: |date|

