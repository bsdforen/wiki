Festplatte einrichten
=====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Wenn eine weitere (neue) Festplatte in das System integriert werden
soll, gibt es zwei Möglichkeiten dies zu tun. Zunächst mit
``sysinstall``, einem Programm welches zum Basissystem gehört oder
manuell mit dem "Boardwerkzeug" des Basissystems.

Im folgenden Artikel werden klassische MBR-Partitionen mit BSD-Slices
angelegt, wie sie ein PC-BIOS benötigt. In dem Schwesterartikel
`festplatte_einrichten_gpt <festplatte_einrichten_gpt>`__ ist
beschrieben, wie man GPT-Partitionen für EFI bzw. große Festplatten
anlegt.

.. note::

  In diesem Artikel werden Arbeitsschritte beschrieben, **die eine oder mehrere
  Festplatten des System löschen können**. Daher kann **keine Gewähr für die
  Korrektheit** dieser Arbeitsschritte übernommen werden. Die Nutzung dieses
  Artikels erfolgt somit **auf eigene Gefahr**! Es sind **unbedingt
  Sicherheitskopien anzulegen**, wenn sich wichtige Daten auf der/den
  Festplatten befinden!

.. warning::

  Ein oft vorkommendes Problem ist, dass FreeBSD einem nicht erlaubt, die
  Festplatte zu partitionieren. Dies liegt daran, dass dieser wichtige Bereich
  vom System geschützt ist. Um den Schutz aufzuheben, damit die Festplatte
  eingerichtet werden kann, muss eine sysctl-Einstellung gemacht werden:

  sysctl kern.geom.debugflags=16

sysinstall
----------

FIXME

Der Slice-Editor von sysinstall (also das FreeBSD fdisk tool) kann von
den Ports nachinstalliert werden. Man findet diesen unter dem Namen
``sysutils/sfdisk``.

Manuelle Einrichtung
--------------------

Die in diesem Beispiel genutzte Festplatte wird beim Hochfahren des
System als ``/dev/ad3`` erkannt. Auch später lässt sich mit ``dmesg``
das Log vom Start des System erneut anzeigen und die angeschlossenen
Platten ermitteln. Da die Festplatte als zusätzliches Speichermedium im
System eingesetzt werden soll, wird hier nur eine Partition mit der
gesamten Festplatten-Kapazität angelegt.

Löschen der Platte
~~~~~~~~~~~~~~~~~~

Bei einer gebrauchten Platte ist es unter Umständen sinnvoll, die
vorhandenen Daten zunächst zu Löschen. Hierbei werden ggf. auch
Hardwareschäden der Platte erkennbar. Der folgenden Befehl löscht die
Daten der **kompletten** Festplatte **unwiderruflich**! Dieser Schritt
kann auch übersprungen werden.

::

  # dd if=/dev/zero of=/dev/ad3 bs=1k count=1

Slice in der Partitionstabelle erstellen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Der Slice nimmt den gesamten Platz auf der Festplatte ein.

============================= ==============================
nicht bootfähig               bootfähig
============================= ==============================
# fdisk -I ad3                # fdisk -IB ad3
============================= ==============================

BSD-Partitionstabelle erstellen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

================================== =====================================
nicht bootfähig                    bootfähig
================================== =====================================
# bsdlabel -w ad3s1                # bsdlabel -B -w ad3s1
================================== =====================================

Die Partition ``a`` nimmt jetzt den gesamten Platz auf dem Slice ein.

BSD-Partitionstabelle anpassen (optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ggf. kann die Festplatte in mehrere BSD-Partitionen unterteilt werden.
Eine Standardfall hierfür ist z.B. das Einrichten einer
Dateisystempartition und einer Swap-Partition auf einer Platte:

::

  # bsdlabel -e ad3s1

Danach kann man in einem Editor (Standard: vi) die Paritionstabelle in
Text-Tabellenform anpassen:

::

   # /dev/da3s1:
   8 partitions:
   #        size   offset    fstype   [fsize bsize bps/cpg]
     a:        *        *    4.2BSD     2048 16384 28528
     b:       2G        *      swap
     c: 78242913        0    unused        0     0         # "raw" part, don't edit

In der Spalte **size** sind auch Angaben in Gigabyte (Einheit ``G``) und
Megabyte (Einheit ``M``) möglich.

Sehr hilfreich ist hierbei die Verwendung des Joker-Zeichens ``*``,
welches in den Spalten folgende Bedeutung hat:

#. **offset:** bsdlabel soll den offset berechnen, was lästige
   Berechnungen erspart.
#. **size:** für Parition ``a`` soll der Rest der Festplatte verwendet
   werden

Beim Abspeichern und Verlassen des Editors wird bei korrekter Syntax das
Label auf von der o.g. menschengerechten Form in Sectoren umgewandelt
und auf Platte geschrieben.

Da die Rückmeldung etwas "knapp" sind, sollte die Paritionierung wie
folgt noch mal geprüft werden: 

::

  # bsdlabel ad3s1

Dateisystem erstellen
~~~~~~~~~~~~~~~~~~~~~

::

  # newfs /dev/ad3s1a

Softupdates ggf. einschalten
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mit den Softupdates kann die Performance beim Schreiben auf die
Festplatte erhöht werden. Mehr Informationen findet man im Handbuch.

::

  # tunefs -n enable /dev/ad3s1a

temporär nach /mnt mounten
~~~~~~~~~~~~~~~~~~~~~~~~~~

Nachdem nun die Platte partitioniert und eingerichtet wurde, kann sie in
das Dateisystem gemountet werden. In diesem Beispiel nach **/mnt**:

::

  # mount /dev/ad3s1a /mnt

/etc/fstab
~~~~~~~~~~

Sollen die Partitionen der neuen Festplatte bei jedem Neustart
automatisch gemountet werden, wird ein neuer Eintrag in der
**/etc/fstab** notwendig. Die hier dargestellten Schritte erzeugen eine
Partition in einem Slice über die gesamte Platte, die nicht bootbar ist.
Wenn eine bootbare Partition benötigt wird, so findet sich im
FreeBSD-Handbuch ein Beispiel dazu.

Siehe auch
----------

-  `FreeBSD-Handbuch <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/install-steps.html>`__
-  `Softupdates <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/configtuning-disk.html#SOFT-UPDATES>`__
-  `/etc/fstab <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/mount-unmount.html>`__

* :ref:`genindex`

Zuletzt geändert: |date|

