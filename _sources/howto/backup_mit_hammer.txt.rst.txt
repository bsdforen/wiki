Backup mit HAMMER
=================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-dragonflybsd.png

DragonFlys Dateisystem HAMMER bietet einige nützliche Funktionen, die
sich leicht für ein Backup nutzen lassen.

-  Speicherung der Veränderungen ähnlich einem Versionskontrollsystem
-  Anlegen von Snapshots
-  Spiegelung eines Pseudo-Dateisystems (PFS) auf andere Blockdevices

Die Benutzerdaten sollten in einem eigenen PFS liegen, um sie einfach
auf ein anderes Device zu spiegeln.

Alle Befehle, die das PFS verändern, müssen mit root Rechten --am besten
via ``sudo``-- aufgerufen werden.

Wenn noch keine Spiegelung erfolgte, wird zuerst ein Device mit einem
HAMMER Dateisystem benötigt. Am einfachsten lässt sich dieses mit

::

   $ newfs_hammer -L BACKUP /dev/da8s1

erstellen. In diesem Fall ist eine externe Festplatte unter
``da8s1`` verfügbar. Danach sollte das Dateisystem eingebunden und ein
Verzeichnis ``/pfs`` angelegt werden, um die Pseudo-Dateisystem
anzulegen. Diese werden später die Sicherung aufnehmen. Unter ``/pfs``
liegen per Konvention die Pseudo-Dateisysteme.

::

   $ mount_hammer /dev/da8s1 /mnt
   $ mkdir /mnt/pfs

Anschließend kann mit

::

   $ hammer -v mirror-copy /pfs/home /mnt/pfs/backup.home

die Spiegelung gestartet werden. Sollte das Pseudo-Dateisystem
``backup.home`` noch nicht existieren, wird es auf Nachfrage erstellt.

Nun kann mit obigem Befehl die Spiegelung zu jedem beliebigem Zeitpunkt
fortgesetzt werden --- auch wenn parallel gearbeitet wird. Alle auf dem
Quell-PFS vorhandenen Snapshots werden ebenfalls übertragen. Sie lassen
sich auch anzeigen: 

::

  $ hammer snapls /mnt/pfs/backup.home

In regelmäßigen Abständen sollte auch das Backup PFS von der Historie
bereinigt werden. Andernfalls besteht die Gefahr, dass irgendwann auf
dem Device kein Platz mehr vorhanden ist.

::

   $ hammer cleanup /mnt/pfs/backup.home

Um gewisse Stände im Zugriff zu haben, können spezielle Links
(Snapshots) angelegt werden. Dazu sollte ein separates Verzeichnis
``/snaps`` angelegt werden, dass die Snapshots aufnimmt.

::

   $ mkdir /mnt/snaps

Nun kann dort ein Snapshot mit einem Zeitstempel erzeugt werden.

::

   $ cd /mnt/snaps
   $ hammer snapshot ../pfs/backup.home .

Wenn die HAMMER Dateisysteme die Version 3 oder neuer verwenden, können
Snapshots auch in den Metadaten eingetragen werden. Damit könnte auf das
Snapshot Directory und die entsprechenden symbolischen Links verzichtet
werden.

Die Snapshots in den Metadaten können optional einen Kommentar
enthalten. Damit sieht der Befehl zum Erstellen des Snapshots so aus:

::

   $ hammer snapq /mnt/pfs/backup.home "Backup Kommentar"

Mittels

::

   $ hammer snapls /mnt/pfs/backup.home

können die Snapshots aufgelistet werden.

.. warning::

  Wenn parallel gearbeitet werden oder die Übertragung über das Netz erfolgen
  soll, ist es möglicherweise sinnvoll nicht die gesamte zur verfügung stehende
  Bandbreite zur Übertragung zu nutzen. Mit

  ::

    $ hammer -v -b5m mirror-copy /pfs/home /mnt/pfs/backup.home

  lässt sich die verwendete Bandbreite beschränken. In dem Beispiel wird mit
  maximal 5 MegaByte pro Sekunde kopiert.

Soll die Sicherung permanent erfolgen, kann statt ``mirror-copy`` der
Parameter ``mirror-stream`` verwendet werden, dann wird das Kopieren
nicht beendet. Sobald neue Transaktionen auf dem
Quell-Pseudo-Dateisystem vorhanden sind, werden die Daten auf das
Ziel-Pseudo-Dateisystem gespiegelt. So kann ein Live-Mirror eingerichtet
werden.

* :ref:`genindex`

Zuletzt geändert: |date| 

