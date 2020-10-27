Mounten als Benutzer
====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Immer wieder stoßen Neulinge auf das Problem, dass sich Floppy und CD-ROM nur
als User root mounten lassen.

Ehemalige Windows-User können schon meist mit dem Begriff "mounten"
nichts anfangen und ehemalige Linux-Benutzer sind es gewohnt, dass der
Automounter sofort nach der Installation funktioniert und so ein Mounten
von CD-ROM und Floppy möglich ist.

Das Mounten von CD und Floppy für normale Nutzer kann allerdings eine
Sicherheitslücke sein. Deshalb ist es auf den meisten
`BSD </kompendium/BSD>`__\ s nicht vorkonfiguriert.

Was auf Servern und PCs mit mehreren Usern sicher sinnvoll erscheint,
ist bei dem heimischen PC doch eher lästig. Wer möchte schon immer
mittels "su" root werden und dann das jeweilige Medium mounten, oder wer
will "sudo" dafür einsetzen, welches auch wieder ein Passwort verlangt.

Sysctl
------

Damit das Kommando mount überhaupt von anderen Nutzern als root
verwendet werden kann müssen `Sysctl </kompendium/Sysctl>`__ Variablen
geändert werden. Je nach System, Securelevel und Variable kann man
Änderungen zur Laufzeit vornehmen, machmal ist jedoch ein Neustart
notwendig.

FreeBSD
~~~~~~~

Als Benutzer root sollte man

::

   # sysctl vfs.usermount=1

eingeben. Diese Einstellung kann man in der Datei ``/etc/sysctl.conf``
festhalten, damit sie auch nach einem Neustart erhalten bleibt. Dazu
trägt man einfach

::

   vfs.usermount=1

in die Datei ein.

NetBSD
~~~~~~

Um die Einstellung zur Laufzeit vorzunehmen benutzt man das Kommando:

::

   # sysctl -w vfs.generic.usermount=1

In die ``/etc/sysctl.conf`` trägt man die Zeile

::

   vfs.generic.usermount=1

ein.

OpenBSD
~~~~~~~

Als root in der Konsole:

::

   # sysctl kern.usermount=1

In ``/etc/sysctl.conf``:

::

   kern.usermount=1

Zugriffsrechte setzen
---------------------

Je nach System gibt es hier ein anderes Vorgehen.

FreeBSD 4.x
~~~~~~~~~~~

Um nun Usern das Mounten von Floppy und CDROM zu erlauben:

::

   # chmod 666 /dev/fd0
   # chmod 666 /dev/acd0c 

Wenn Du andere Floppies und CDROMS hast, musst Du fd0 und acd0c
entsprechend abändern.

Du kannst, wie Du schon vorgenommen hast, auch Usern der Gruppe
"operator" das Mounten von CDROM und floppy erlauben:

::

   # chgrp operator /dev/acd0c
   # chmod 640 /dev/acd0c
   # chgrp operator /dev/fd0
   # chmod 640 /dev/fd0  

FreeBSD ab 5.x
~~~~~~~~~~~~~~

Die vorher hier beschriebene Variante mit Einträgen in der
``/etc/devfs.conf`` setzt Nutzerrechte nur nach dem booten. Wenn man
Usern erlauben will auch zur Laufzeit angebundene Geräte wie einen PDA,
eine externe Festplatte oder eine Memory Disc (zum Beispiel für ISO
images) zu verwenden, braucht man die ``/etc/devfs.rules``. Der Vorteil
dieser Methode ist, dass die Rechte direkt beim Erzeugen des Geräts
gesetzt werden. Also auch bei zur Laufzeit angeschlossenen Geräten (z.B.
USB-Geräte) greifen.

Um neue Regeln festzulegen muss man in der ``/etc/devfs.rules`` ein
neues Ruleset anlegen. Die Manpage schlägt dafür

::

   [localrules=10]

vor. Damit das System die neuen Regeln auch verwendet muss man noch in
die ``/etc/rc.conf`` folgenden Eintrag machen:

::

   devfs_system_ruleset="localrules"

Folgendes Beispiel zeigt eine typische ``/etc/devfs.rules``:

::

   [localrules=10]
   add path 'acd*'     mode 0660 group operator
   add path 'cd*'      mode 0660 group operator
   add path 'md*'      mode 0660 group operator
   add path 'ttyU*'    mode 0660 group operator
   add path 'cuaU*'    mode 0660 group operator

Mit ``add path`` legt man ein Regelwerk an. Mit dem nächsten Parameter
in Anführungszeichen legt man fest wofür diese Regeln gelten. Danach
kann man Zugriffsregeln festlegen. Der Parameter ``mode`` entsprichte
einem ``chmod`` Kommando und ``group`` einem ``chown`` Kommando.

Die Devices im Beispiel haben folgende Bedeutung:

-  'acd*'

   -  ATAPI CD-Rom Laufwerke.

-  'cd*'

   -  SCSI und Cam (z.B. für Brennprogramme) CD-Rom Laufwerke.

-  'md*'

   -  Memory Discs, dazu gehören auch aus Dateien erzeugte Devices, z.B.
      um ein ISO oder Festplatten Image zu mounten.

-  'ttyU*' und 'cuaU*'

   -  Das sind terminal devices die man benötigt um auf PDAs
      zuzugreifen. Z.B. mit dem Programm jpilot.

Zum Schluß kann man noch mit dem Kommando

::

   # /etc/rc.d/devfs restart

Das Regelwerk aktivieren. Alternativ kann man auch den Rechner neu
starten.

NetBSD und OpenBSD
~~~~~~~~~~~~~~~~~~

Der Benutzer, der mounten soll, braucht Lese- und Schreibrechte auf die
Geräte, die er mounten darf. Entweder fügt man ihn der Gruppe
``operator`` hinzu:

::

   # usermod -G operator Benutzer
   # chmod 660 GERÄT

Oder man erstellt eine neue Gruppe, der root und alle Benutzer
zugeordnet werden, die zugreifen dürfen:

::

   # groupadd mounters
   # usermod -G mounters root
   # usermod -G mounters Benutzer
   # chgrp mounters GERÄT
   # chmod 660 GERÄT

Als ``GERÄT`` kann u.a. das CD-Laufwerk mit ``/dev/cd0a`` oder ein
USB-Stick mit ``/dev/sd0e`` angegeben werden.

Mountpoint
----------

Der `Mountpoint </kompendium/Mountpoint>`__ in den ein Benutzer mountet
muss diesem Nutzer gehören. Die einfachste Lösung ist hier sicher ein
Mountpoint im Home Verzeichnis des Benutzers. Für Mounts die immer
wieder vorkommen und mehreren Nutzern zugänglich sein sollten empfiehlt
sich jedoch die Verwendung einse `Automount <Automount>`__\ ers.

Ab Version 4.0 müssen unter **NetBSD** die Optionen ``nodev`` und
``nosuid`` für den Mount-Befehl verwendet werden, z.B.:

::

   /sbin/mount -t cd9660 -o nodev,nosuid /dev/cd0a $HOME/cdrom

* :ref:`genindex`

Zuletzt geändert: |date|

