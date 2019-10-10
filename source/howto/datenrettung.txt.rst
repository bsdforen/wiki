Datenrettung
============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Grundsätzlich sollte niemand diesen Artikel lesen müssen, denn mit einer
möglichst aktuellen Datensicherung ist eine defekte oder auch nur vermurkste
Festplatte schnell rekonstruiert.

Hier findet man Anleitungen für Backups:

-  `Backup über das Netzwerk </howto/Backup über das Netzwerk>`__

Da aber offensichtlich das Kind in den Brunnen gefallen ist, folgen hier
einige Möglichkeiten, wie man unter Umständen Daten wieder herstellen
kann.

.. warning::

  -  Rettungsversuche, wenn möglich, im Single-User-Mode ausführen. Noch
     besser ist ein Zweitrechner in dem keine weiteren Festplatten (bis
     auf eine System/Backupplatte ) vorhanden sind.
  -  Bei sehr wichtigen Daten sollte man unter Umständen eine
     professionelle (sprich kostenintensive) Herstellung der Daten in
     Anspruch nehmen.
  -  Die Erfolgschancen richten sich stark nach der Ursache des Defektes.
     Wenn die Festplatte beispielsweise einen Hardwareschaden hat, kann es
     nahezu unmöglich werden, die Daten zu retten.
  -  Die hier aufgeführten Möglichkeiten können erfolgreich sein, sie
     können aber auch genau das Gegenteil bewirken und ggf. Daten komplett
     unbrauchbar machen!
  -  Alle hier gemachten Angaben erfolgen natürlich ohne Gewähr!

Variante 1 (FreeBSD)
--------------------

**Schadensbild:**

-  Festplatte hat Schwierigkeiten beim Hochfahren, läuft mehrmals an und
   schafft aber die Initialisation und wird mit den BSD-Partitionen im
   /dev-Ordner eingehängt.
-  Untypische Geräusche, die auf ein Kalibration der Schreib/Leseköpfe
   hinweisen.
-  **fsck** liefert eine vielzahl von Lesefehlern, bricht zum Teil ab.

**Lösung in diesem Fall:**

Mit **dd** die betroffene Partition auf eine andere Festplatte mit
ausreichend Platz als Datei kopieren. Achtung - Auf der "freien"
Partition muß mindestens so viel Platz sein, um die komplette Partition
hinein zu kopieren, unabhängig wie groß die Belegung auf der defekten
Partition ist.

::

  # dd if=/dev/ad4s1f of=/freierplatz/defektepartition.image

Sollten Lesefehler beim Kopieren auftreten, könnnen diese mit der Angabe
der dd-Option ``'conv=noerror,sync``' unterdrückt werden und die
fehlenden Bytes in der Image-Datei werden mit Nullen aufgefüllt.

Es erscheint keine weitere Ausgabe. Erst wenn die Partition komplett
kopiert ist, kommt eine Meldung, wieviele Daten übertragen wurden. Je
nach Partitionsgröße kann das Kopieren eine Weile dauern. Mit
Ctrl-T/Strg-T kann man sich während des Kopiervorgangs anzeigen lassen,
wieviel Daten bisher kopiert wurden.

Jetzt kann die defekte Partition als Device **/dev/md0** eingehängt
werden:

::

  # mdconfig -f /freierplatz/defektepartition.image md0

Bevor man die Partition mounten kann, muss mit fsck ein "sauberer
Zustand" hergestellt werden. Es muß hier zwingend der Filesystem-Typ
angegeben werden, da diese Information für fsck nicht vorliegt. Im
Beispielfall ist es **ufs**.

::

  # fsck -y -t ufs /dev/md0

Wenn fsck erfolgreich abgeschlossen hat, kann man die Partition mounten
und sehen, ob noch unzerstörte Daten in dem Imagefile vorhanden sind.

::

  # mount /dev/md0 /mnt # cd /mnt # ls -alg [...]

Sollten Dateien fehlen, kann es sein, daß ``fsck`` in den Ordner
lost+found kopiert hat. In den lost+found-Ordner werden alle Dateien
kopiert, bei denen z.B. der Dateiname nicht mehr zu rekonstruieren oder
die zugehörige Directory beschädigt war.

Um die Image-Datei wieder freizugeben, folgt noch ein:

::

  # mdconfig -d -u md0

Tools in den Ports (FreeBSD)
----------------------------

-  `sysutils/dd_rescue <https://www.google.com/search?q=sysutils/dd_rescue&btnI=lucky>`__
   - Ein Tool, ähnlich wie `dd <dd>`__, mit dem man fehlerbehaftete
   Medien lesen kann
-  `sysutils/ddrescue <https://www.google.com/search?q=sysutils/ddrescue&btnI=lucky>`__
   - dito ?!
-  `sysutils/gpart <https://www.google.com/search?q=sysutils/gpart&btnI=lucky>`__
   - Wiederherstellen von verschiedenen Partitionstabellen und
   Dateisysteme
-  `archivers/gzrecover <https://www.google.com/search?q=archivers/gzrecover&btnI=lucky>`__
   - Daten aus defekten gzip-Archiven wiederherstellen
-  `sysutils/fatback <https://www.google.com/search?q=sysutils/fatback&btnI=lucky>`__
   - Wiederherstellen von FAT12/16/32 Dateisysteme
-  `sysutils/foremost <https://www.google.com/search?q=sysutils/foremost&btnI=lucky>`__
   - Wiederherstellen von gelöschten Dateien mittels Headers & Footers
-  `sysutils/ffs2recov <https://www.google.com/search?q=sysutils/ffs2recov&btnI=lucky>`__
   - Wiederherstellen von UFS2 Dateisystem
-  `sysutils/magicrescue <https://www.google.com/search?q=sysutils/magicrescue&btnI=lucky>`__
   - Wiederherstellen von gelöschten Dateien auf Block-Geräten
-  `sysutils/recoverdm <https://www.google.com/search?q=sysutils/recoverdm&btnI=lucky>`__
   - Dieses Programm hilft beim Wiederherstellen von Platten mit
   defekten Sektoren (Dateien und komplette Geräte)
-  `sysutils/scan_ffs <https://www.google.com/search?q=sysutils/scan_ffs&btnI=lucky>`__
   - Wiederherstellen von UFS1 & UFS2 disklabel
-  `sysutils/sleuthkit <https://www.google.com/search?q=sysutils/sleuthkit&btnI=lucky>`__
   - Beweismittelanalyse & Disk Forensik

* :ref:`genindex`

Zuletzt geändert: |date| 

