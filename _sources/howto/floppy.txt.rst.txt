Floppy
======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Diskettenlaufwerke (Floppy) sterben langsam aber sicher aus. Dennoch braucht
man in der Regel immer noch eine Diskette und ein Diskettenlaufwerk, um
beispielsweise das BIOS vom Motherboard auf den neusten Stand zu bringen. Die
hier genannten Beispiele gehen von einem 1,4 MB Laufwerk aus, welches als
Laufwerk /dev/fd0 bzw. A: installiert ist.

Diskette im MSDOS-FAT-Format formatieren
----------------------------------------

Zum Upgraden des Motherboard-BIOS wird in der Regel das MS-DOS Format
FAT benötigt. Folgende Sequenz formatiert eine Diskette in diesem
Format:

::

   # fdformat -f 1440 fd0
   # newfs_msdos -f 1440 fd0

Diskette im Unix-Filesystem-Format (UFS) formatieren
----------------------------------------------------

Selbstverständlich kann man die Diskette auch in `UFS (Unix File
System) <http://de.wikipedia.org/wiki/Unix_File_System>`__ formatieren:

::

   # fdformat -q -f 1440 fd0

Dann entweder mit

::

   # disklabel -B -r -w fd0 fd1440

oder

::

   # bsdlabel -B -r -w fd0 fd1440

labeln. High-Level-Formatieren:

::

   # newfs -c 1 fd0

Wer Softupdates haben möchte:

::

   # newfs -O 2 -U -m 0% -c 1 fd0

Die Diskette braucht nur noch gemountet werden und kann wie eine Platte
im Verzeichnisbaum genutzt werden:

::

   # mount /dev/fd0 /mnt

.. note::

  important> Die Diskette muss ungemountet werden, bevor sie dem Laufwerk
  entnommen werden kann. Das Betriebssystem hat noch nicht zwingenderweise alle
  Daten auf die Disketten zurückgeschrieben und hält die Daten noch im Cache
  vor.

Ein Entnehmen der Diskette kann Datenverlust nach sich ziehen, darum Folgendes
eingeben:

::

   # umount /mnt

Sollte hierbei eine Busy-Fehlermeldung kommen, wird von einem Programm
oder der Shell auf das Laufwerk zugegriffen. Die entsprechenden
Programme müssen u.U. beendet und der Pfad in der Shell geändert werden
z.B. durch die einfache Eingabe von:

::

   # cd

Nun sollte sich die Disketten unmounten lassen.

Partitionieren
--------------

Eine Diskette kann wie jedes andere Storage formatiert werden. Über den
Sinn bei einem so kleinen und langsamen Medium kann man sich natürlich
streiten, technisch möglich ist es jedoch.

Low-Level-Formatieren und labeln:

::

   # fdformat -q -f 1440 fd0
   # bsdlabel -B -r -w fd0 fd1440

Das Label bearbeiten:

::

   # bsdlabel -e /dev/fd0

Bei zwei gleichgroßen Slices kann das beispielsweise so aussehen:

::

   #        size   offset    fstype   [fsize bsize bps/cpg]
     a:     1440        0    unused        0     0
     b:     1440     1440    unused        0     0
     c:     2880        0    unused      512  4096         # "raw" part, don't edit

(Die Parameter fstype, fsize, bsize und bps/cpg können einfach ignoriert
werden, denn sie werden beim Formatieren automatisch gesetzt.)

High-Level-Formatieren:

::

   # newfs /dev/fd0a; newfs /dev/fd0b

Mounten:

::

   # mount /dev/fd0a /mnt/floppy

Zugriff auf MS-DOS FAT-Disketten
--------------------------------

Der Zugriff auf Disketten im MS-DOS Format wird vom Betriebssystem nicht
unterstützt. Es gibt eine Toolsammlung in den Ports , mit der dieses
Problem beseitigt wird: ``emulators/mtools``.

Hier eine Übersicht, welche MS-DOS Befehle unter mtools emuliert werden:

::

   Mtool           MSDOS
   name            equivalent      Description
   -----           ----            -----------
   mattrib         ATTRIB          change MSDOS file attribute flags
   mcd             CD              change MSDOS directory
   mcopy           COPY            copy MSDOS files to/from Unix
   mdel            DEL/ERASE       delete an MSDOS file
   mdir            DIR             display an MSDOS directory
   mformat         FORMAT          add MSDOS filesystem to a low-level format
   mlabel          LABEL           make an MSDOS volume label.
   mmd             MD/MKDIR        make an MSDOS subdirectory
   mrd             RD/RMDIR        remove an MSDOS subdirectory
   mread           COPY            low level read (copy) an MSDOS file to Unix
   mren            REN/RENAME      rename an existing MSDOS file
   mtype           TYPE            display contents of an MSDOS file
   mwrite          COPY            alias for mcopy, will be removed soon

* :ref:`genindex`

Zuletzt geändert: |date|

