Qemu
====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Qemu ist eine virtuelle Maschine. Es können folgende Gastsysteme benutzt
werden: Linux, Windows, \*BSD und Mac OS X. Es wird die komplette
Hardware für das Gastsystem emuliert (siehe auch
`Wikipedia <http://de.wikipedia.org/wiki/Qemu>`__).

Vorwort
-------

Für die allgemeine Bedienung von QEMU sehe man sich bitte die
`Manpage <http://www.freebsd.org/cgi/man.cgi?query=qemu&apropos=0&sektion=0&manpath=FreeBSD+6.1-RELEASE+and+Ports&format=html>`__
an. Für einen Screenshot von QEMU siehe bitte
`hier <http://www.flickr.com/photo_zoom.gne?id=129916359&size=o&context=set-72157594517723353>`__.

Der von QEMU emulierte Rechner kann über eine Bridge an das eigene
Netzwerk "angeschlossen" werden, und verhält sich dann so als wäre er am
selben Switch wie der Hostrechner. Alternativ kann man QEMU auch mit
SLIRP (http://slirp.sourceforge.net) betreiben, hier ist die
Konfiguration einfacher und es funktioniert auch unter FreeBSD 5.x ohne
root-Rechte. Der Nachteil dabei: es können keine ICMP Pakete nach
draussen geschickt werden (wie z.B. für ping(8) benötigt). Normale TCP-
oder UDP-Verbindungen funktionieren aber.

Installation von QEMU
---------------------

QEMU bringt ein optionales Kernelmodul mit, was die Emulation um einiges
schneller laufen lässt. Deshalb sollte man QEMU über den Port mit der
Option WITH_KQEMU bauen:

::

   # cd /usr/ports/emulators/qemu
   # sudo make -DWITH_KQEMU install clean

Hierduch wird automatisch der Port kqemu-kmod mitinstalliert, der nach
jedem Kernelupdate ebenfalls neu kompiliert werden muss.

Als Beispiel soll FreeBSD als Clientsystem installiert werden. Zuerst
wird eine neue Imagedatei angelegt, die die Installation beherbergen
soll:

::

   # qemu-img create FreeBSD.img 4G

Damit wird eine 4 GB große Datei angelegt, die als erste Festplatte
dient.

QEMU mit Bridge
---------------

Um die Bridge beim Start von QEMU zu erstellen, wird das folgende Skript
unter /usr/local/bin/qemu_start gespeichert:

::

   #!/bin/sh
   kldstat -n kqemu || kldload kqemu
   kldstat -n if_tap || kldload if_tap
   kldstat -n bridge || kldload bridge
   sysctl net.link.ether.bridge.enable=1
   sysctl net.link.ether.bridge.config=rl0,$1

Hier muss **rl0** mit der Bezeichnung der eigenen Netzwerkkarte ersetzt
werden.

Außerdem *qemu_start* ausführbar machen mit:

::

   # sudo chmod 755 /usr/local/bin/qemu_start

Jetzt kann QEMU mit den Parametern *-net nic -net tap,script=qemu_start*
gestartet werden und man hat im emulierten System ein Netzwerkinterface,
das sich so verhält, als sei es mit dem Hostsystem in einem
physikalischen Netzwerk.

QEMU mit SLIRP
--------------

Die Alternative zur Bridge ist der Netzwerkmodus mit SLIRP. Hier bekommt
das Netzwerkinterface des Clientsystems die IP-Informationen von der
virtuellen Umgebung per DHCP. Um SLIRP benutzen zu können, muss QEMU nur
mit den Parametern *-net nic -net user* gestartet werden.

Installation des Clientsystems
------------------------------

Nachdem die Imagedatei angelegt wurde, brauch man für die Installation
nur noch ein CD-Image und kann beginnen. Wird vom CD-Laufwerk aus
installiert, muss zuvor das Modul "aio" geladen werden, da sonst Qemu
beim Zugriff auf ATAPI-Geräte abbricht:

::

   # kldstat -n aio || kldload aio

Nach der Installation von ist das Modul nicht mehr erforderlich, außer
man will auf das CD-Laufwerk zugreifen.

Bridge
------

::

   # sudo qemu -net nic -net tap,script=qemu_start -hda FreeBSD.img -cdrom 6.0-RELEASE-i386-disc1.iso -boot d -k de

SLIRP
-----

::

   # qemu -net nic -net user -hda FreeBSD.img -cdrom 6.0-RELEASE-i386-disc1.iso -boot d -k de

Booten des Clientsystems
------------------------

Danach kann QEMU auch wie folgt gestartet werden:

.. _bridge-1:

Bridge
------

::

   # sudo qemu -net nic -net tap,script=qemu_start -hda FreeBSD.img -k de

.. _slirp-1:

SLIRP
-----

::

   # qemu -net nic -net user -hda FreeBSD.img -k de

QEMU ohne Grafik
----------------

Im emulierten FreeBSD folgenden Eintrag in die Datei */boot.config*
machen:

::

   -h

Außerdem in */etc/ttys* die Zeile

::

   ttyd0   "/usr/libexec/getty std.9600"   dialup  off secure

ersetzen durch

::

   ttyd0   "/usr/libexec/getty std.9600"   vt100  on secure

Jetzt das emulierte FreeBSD runterfahren und QEMU beenden.

QEMU nun folgendermaßen starten:

.. _bridge-2:

Bridge
------

::

   # sudo qemu -net nic -net tap,script=qemu_start -hda FreeBSD.img -nographic -parallel null -k de

.. _slirp-2:

SLIRP
-----

::

   # qemu -net nic -net user -hda FreeBSD.img -nographic -parallel null -k de

Getestet mit
------------

-  QEMU Version: 0.8 Host: FreeBSD 5.4-RELEASE/i386 Client: FreeBSD
   6.0-RELEASE/i386
-  QEMU Version: 0.8 Host: FreeBSD 7.0-CURRENT/i386 Client: Linux

Fehler / Bugs
-------------

-  Sollte HZ vom Client größer sein als vom Host gibt es evtl. Timing
   Probleme.
-  Die verwendete Bridge Implementierung existiert ab FreeBSD 7.0 nicht
   mehr, hier **if_bridge** verwenden.

Verweise
--------

-  `Eine Beispielinstallation mit Windows
   XP <http://www.bsdbox.de/?page_id=18>`__
-  `Ein Forenthread hierzu (Modul
   aio) <http://www.bsdforen.de/showthread.php?t=16925>`__
-  `emulators/qemu <https://www.google.com/search?q=emulators/qemu&btnI=lucky>`__
-  `emulators/qemu <https://www.google.com/search?q=emulators/qemu&btnI=lucky>`__

* :ref:`genindex`

Zuletzt geändert: |date|

