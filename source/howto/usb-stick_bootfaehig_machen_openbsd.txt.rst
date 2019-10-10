USB-Stick bootfähig machen (OpenBSD)
====================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

Die folgenden Instruktionen löschen alle Daten und einen eventuell
vorhandenen Master Boot Record auf dem verwendeten USB-Stick
unwiederbringlich. Die Werte in den Kommandoparametern sind nur
Beispiele. Alle Operationen erfolgen selbstvertständlich auf eigene
Gefahr.

Securelevel herabsetzen
-----------------------

Melde dich als root an.

Ändere in ``/etc/rc.securelevel`` den Wert von securelevel auf ``-1``

::

   securelevel=-1

Starte das System neu

::

   # reboot

Nach erneutem Anmelden kannst du überprüfen, ob der gewünschte
Securelevel eingestellt ist.

::

   # sysctl kern.securelevel 
   kern.securelevel=-1

Ermitteln der Geometrie des Sticks
----------------------------------

Stecke den Stick in einen Slot.

Du solltest dann neben vielen anderen Dingen in etwa folgendes sehen:

::

   sd0 at scsibus1 targ 1 lun 0: <USB, Flash Disk, 3000> SCSI0 0/direct removable 
   sd0: 247MB, 247 cyl, 64 head, 32 sec, 512 bytes/sec, 506880 sec total 

Diese Werte teilen dir die Geometrie des von dir verwendeten Sticks mit.

Schreiben des Master Boot Record (MBR)
--------------------------------------

::

   # fdisk -i -c 247 -h 64 -s 32 sd0 

Partitionieren des Mediums
--------------------------

::

   # disklabel -f /tmp/fstab -E sd0

Lasse dir dir vorhandenen Partitionen anzeigen:

::

   > p

Lösche die vorhandenen Partitionen:

::

   > d a

Erzeuge eine neue Partition:

::

   > a a
   offset: <enter>
   size: <enter>
   FS type: <enter>
   mount point: /<enter>

Sichern und Beenden:

::

   > q

Formatieren des Mediums
-----------------------

::

   # newfs sd0a

Mounten des Devices
-------------------

::

   # mount /dev/sd0a /mnt

Kopieren des Kernels
--------------------

::

   # cp /bsd /mnt/

Schreiben des Partition Boot Record (PBR)
-----------------------------------------

::

   # cp /usr/mdec/boot /mnt/
   # /usr/mdec/installboot /mnt/boot /usr/mdec/biosboot sd0

Unmounten des Devices
---------------------

::

   # umount /mnt

Securelevel heraufsetzen
------------------------

Ändere in ``/etc/rc.securelevel`` den Wert von ``securelevel=-1`` auf
einen wert deiner Wahl. Standardmässig ist ``1`` eingestellt.

::

   securelevel=1

Starte das System neu

::

   # reboot

Nach erneutem Anmelden kannst du überprüfen, ob der gewünschte
Securelevel eingestellt ist.

::

   # sysctl kern.securelevel
   kern.securelevel=1

Das war es.

Jetzt ist der Stick bootfähig. Es bleibt dir überlassen, auf ihm ein
lauffähiges System zu platzieren.

Quellen
-------

-  `Wie bootet
   OpenBSD/i386? <http://www.openbsd.org/faq/de/faq14.html#Boot386>`__
-  `OpenBSD-Bootloader <http://www.openbsd.org/faq/de/faq8.html#Bootloader>`__
-  `Installieren von
   Bootblocks <http://www.openbsd.org/faq/de/faq14.html#InstBoot>`__
-  `man
   securelevel <http://www.openbsd.org/cgi-bin/man.cgi?query=securelevel&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
-  `man
   sysctl <http://www.openbsd.org/cgi-bin/man.cgi?query=sysctl&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
-  `man
   fdisk <http://www.openbsd.org/cgi-bin/man.cgi?query=fdisk&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
-  `man
   disklabel <http://www.openbsd.org/cgi-bin/man.cgi?query=disklabel&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
-  `man
   newfs <http://www.openbsd.org/cgi-bin/man.cgi?query=newfs&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
-  `man
   mount <http://www.openbsd.org/cgi-bin/man.cgi?query=mount&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
-  `man
   umount <http://www.openbsd.org/cgi-bin/man.cgi?query=umount&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
-  `man
   installboot <http://www.openbsd.org/cgi-bin/man.cgi?query=installboot&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
-  `man
   biosboot <http://www.openbsd.org/cgi-bin/man.cgi?query=biosboot&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__

* :ref:`genindex`

Zuletzt geändert: |date|

