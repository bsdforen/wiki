USB-Stick Unterstützung
=======================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Diese Anleitung ist als Ergänzung zur `Automount
</howto/Automount>`__-Anleitung gedacht. Es wird erläutert, wie man mit devd
unter FreeBSD 5.x automatisch USB-Massenspeicher wie USB-Stick, Digitalkamera
und USB-Wechselfestplatte einhängen kann. Und als Zugabe, wie man FreeBSD auf
einen bootbaren USB-Stick bringt!

<note>Wechselmedien wie zum Beispiel DVD, CD-ROM, Iomega Zipdrive können
nur mit dem Automounter ``amd`` automatisch eingebunden werden. Sehen
Sie dazu bitte in der Automount-Anleitung unter:
`Automount </howto/Automount>`__ nach.</note>

USB-Massenspeicher automatisch mounten
--------------------------------------

devd
----

Der ab FreeBSD 5.x integrierte Dienst devd reagiert auf
Hardware-Ereignisse wie zum Beispiel USB-Gerät angehängt, USB-Gerät
entfernt, Laptop-Akku fast leer. Devd wird über die Konfigurationsdatei
/etc/devd.conf eingestellt, diese lädt auch automatisch zusätzliche
``.conf``-Dateien aus dem Verzeichnis ``/etc/devd``. Ich möchte hier
eine Lösung für das automatische Einhängen des USB-Sticks und die
automatische Anlegung von Links auf dem KDE-Desktop präsentieren.

Diese Lösung ist sicher verbesserungsfähig und muss an die individuellen
Anforderungen angepasst werden!

**/etc/devd/my.conf**

::

   # USB-Massenspeicher (Sony-Digitalkamera, USB-Stick)
   attach 0 {
    device-name "umass[[0-9]]+";
    action "(sleep 2; /usr/local/sbin/mountumass)&";
   };

Kein detach => Umount nach dem physikalischen Entfernen des Gerätes ist
ungesund!

**/etc/fstab**

::

   /dev/da0 /mnt/zip msdos rw,noauto,nodev,nosuid,-Lde_CH.UTF-8 0 0
   /dev/da0s4 /mnt/zip msdos rw,noauto,nodev,nosuid,-Lde_CH.UTF-8 0 0
   /dev/da1 /mnt/umass msdos rw,noauto,nodev,nosuid,-Lde_CH.UTF-8 0 0
   /dev/da1s1 /mnt/umass msdos rw,noauto,nodev,nosuid,-Lde_CH.UTF-8 0 0

**/usr/local/sbin/mountumass**

::

   #!/bin/csh
   /sbin/mount /dev/da1s1 && /usr/local/sbin/createumasslink && exit
   /sbin/mount /dev/da1 && /usr/local/sbin/createumasslink && exit

**/usr/local/sbin/createumasslink** => Erstellt Links auf jedem
KDE-Desktop

::

   #!/bin/csh
   ln -s /mnt/umass/ /root/Desktop/USB-Massenspeicher
   foreach i (`find /usr/home -mindepth 1 -maxdepth 1 -type d`)
     ln -s /mnt/umass/ ${i}/Desktop/USB-Massenspeicher
   end

**/usr/local/sbin/umountumass** => Wird über KDE-Menüeintrag aufgerufen

::

   #!/bin/csh
   sudo /sbin/umount /mnt/umass && /usr/local/sbin/deleteumasslink && exit

**/usr/local/sbin/deleteumasslink**

::

   #!/bin/csh
   rm -f /root/Desktop/USB-Massenspeicher
   foreach i (`find /usr/home -mindepth 1 -maxdepth 1 -type d`)
     rm -f ${i}/Desktop/USB-Massenspeicher
   end

**/usr/local/sbin/mountzip** => Wird über KDE-Menüeintrag aufgerufen.
Automatische Einhängung mit amd möglich => siehe dazu:
`Automount <Automount>`__

::

   #!/bin/sh
   sudo mount /dev/da0 && ln -s /mnt/zip ~/Desktop/Iomega\ Zip && exit
   sudo mount /dev/da0s4 && ln -s /mnt/zip ~/Desktop/Iomega\ Zip && exit

**/usr/local/sbin/umountzip** => Wird über KDE-Menüeintrag aufgerufen
Nicht vergessen: sudo mit der Datei /usr/local/etc/sudoers
konfigurieren!

::

   #!/bin/sh
   sudo umount /mnt/zip && rm ~/Desktop/Iomega\ Zip

FreeBSD vom USB-Stick booten
----------------------------

Schon ein 16MB-USB-Stick reicht! Hier einige Links zu verschiedenen
Lösungen und Anleitungen:

-  http://devcorner.schlenker-webdesign.de/cms.24.html
-  http://www.m0n0.ch/
-  http://neon1.net/misc/minibsd.html - MiniBSD
-  http://frenzy.org.ua/eng/ (frenzy booten -> install2flash starten ->
   fertig)

Weitere Informationen
---------------------

-  `http://www.bsdforen.de/showthread.php?t=8364 Weitere Lösungen für das Einhängen des USB-Stick <http://www.bsdforen.de/showthread.php?t=8364 Weitere Lösungen für das Einhängen des USB-Stick>`__
-  `http://www.de.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/usb-disks.html USB-Speichermedien unter FreeBSD <http://www.de.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/usb-disks.html USB-Speichermedien unter FreeBSD>`__
-  `http://www.freebsd.org/cgi/man.cgi?query=ehci&apropos=0&sektion=0&manpath=FreeBSD+5.4-RELEASE+and+Ports&format=html  -  man ehci <http://www.freebsd.org/cgi/man.cgi?query=ehci&apropos=0&sektion=0&manpath=FreeBSD+5.4-RELEASE+and+Ports&format=html  -  man ehci>`__
-  `EHCI-Spezifikationen <http://www.intel.com/technology/usb/ehcispec.htm>`__

* :ref:`genindex`

Zuletzt geändert: |date|

