K3B einrichten
==============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Das Brennprogramm K3B erfreut sich außerordentlicher Beliebtheit, selbst bei
jenen, die eigentlich nicht KDE verwenden. Allerdings erfordert das Brennen von
CDs eine Menge Zugriffsrechte. Dieser Artikel beschreibt wie K3B eingerichtet
werden kann, um es als normaler Benutzer ohne Einschränkungen zu verwenden.

Installation
------------

-  `sysutils/k3b <https://www.google.com/search?q=sysutils/k3b&btnI=lucky>`__
   in den FreeBSD Ports.
-  `sysutils/k3b-kde4 <https://www.google.com/search?q=sysutils/k3b-kde4&btnI=lucky>`__
   in den FreeBSD Ports.
-  `sysutils/k3b <https://www.google.com/search?q=sysutils/k3b&btnI=lucky>`__
   in Pkgsrc.

FreeBSD
-------

Unter FreeBSD benötigt K3B die SCSI-Emulation, zusätzliche
Zugriffsrechte und Einträge in der Datei ``/etc/fstab``.

Zugriffsrechte
~~~~~~~~~~~~~~

K3B benötigt zum Brennen Zugriffsrechte auf die entsprechenden
CD-Devices. Folgender Befehl liefert eine Liste:

::

   $ ls /dev/cd*

Außerdem wird Zugriff auf ``/dev/xpt0`` und ``/dev/pass*`` benötigt.
Leider kann sich die Nummer der betroffenen **pass**-Devices ändern (zum
Beispiel wenn beim Booten ein externes USB-Laufwerk vorhanden ist),
deshalb muss Zugriff auf alle **pass**-Devices gegeben werden. Um das
Risiko zumindest etwas einzudämmen, sollte der Zugriff auf eine
vertrauenswürdige Benutzergruppe beschränkt werden. Üblich ist die
Gruppe **operator**.

Wie Zugriffsrechte vergeben werden ist im `devfs-HowTo <devfs>`__
beschrieben.

Hier ist ein funktionsfähiges Beispiel für die Datei
``/etc/devfs.rules``.

::

   [localrules=10]
   add path 'cd*'          mode 0660 group operator
   add path 'pass*'        mode 0660 group operator
   add path 'xpt0'         mode 0660 group operator

aus K3B mounten
~~~~~~~~~~~~~~~

Da K3B beim Mounten nicht auf HAL aufsetzt, benötigt es einen Eintrag in
der Datei ``/etc/fstab`` und einen Mountpunkt im Homeverzeichnis des
aktuellen Benutzers. Der Eintrag in der ``/etc/fstab`` kann etwa so
aussehen:

::

   /dev/cd0        .mnt/cd0    cd9660  ro,noauto   0   0
   /dev/cd1        .mnt/cd1    cd9660  ro,noauto   0   0

Zu beachten ist hierbei, dass vor dem Mountpunkt kein ``/`` steht.
Dadurch wird der Mountpunkt relativ zum aktuellen Verzeichnis
betrachtet. Üblicherweise werden Programme vom Home-Verzeichnis des
aktuellen Nutzers aus gestartet. Deshalb sollte jeder Nutzer in der
Gruppe mit den entsprechenden Rechten, üblicherweise **operator**, der
K3B einsetzen will, folgenden Befehle ausführen:

::

   $ mkdir -p ~/.mnt/cd0 ~/.mnt/cd1

Verzeichnisse und ``fstab``-Einträge sollten natürlich nur für
vorhandene Laufwerke angelegt werden.

Zu guter Letzt muss noch Mounten für normale Benutzer aktiviert werden.
Das geht mit folgendem Kommando:

::

   # sysctl vfs.usermount=1

Permanent wird diese Änderung durch einen Eintrag in der Datei
``/etc/sysctl.conf``.

::

   vfs.usermount=1

K3B 2.0/k3b-kde4
~~~~~~~~~~~~~~~~

Ab K3B 2.0, die Version die mit KDE4 läuft, muss der HAL Daemon aktiv
sein, damit K3B die vorhanden Laufwerke entdeckt. HAL kann zur Laufzeit
mit folgendem Befehl aktiviert werden:

::

   # service hald onestart

Um HALD beim Booten automatisch zu starten, müssen die folgenden 2
Einträge in die Datei ``/etc/rc.conf`` gemacht werden:

::

   dbus_enable="YES"
   hald_enable="YES"

Verweise
--------

-  `devfs </howto/devfs>`__ - devfs-Howto
-  `K3B </anwendungen/K3B>`__
-  `sysutils/k3b <https://www.google.com/search?q=sysutils/k3b&btnI=lucky>`__
   in den FreeBSD Ports.
-  `sysutils/k3b <https://www.google.com/search?q=sysutils/k3b&btnI=lucky>`__
   in Pkgsrc.
-  http://k3b.org die K3B Homepage.

* :ref:`genindex`

Zuletzt geändert: |date|

