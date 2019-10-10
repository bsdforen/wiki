Kerio Mailserver
================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieses Howto beschreibt die Installation sowie Einrichtung des Kerio
Mailservers unter FreeBSD. Beim Erstellen dieser Anleitung wurde FreeBSD in der
Version 6.1 sowie die Demo des Kerio Mailserver Version 6.1.4 verwendet. Das
Ganze sollte aber auch mit einigen Anpassungen mit neueren oder auch aelteren
Versionen der genannten Software funktionieren.

Das Howto setzt voraus, das ein bereits installiertes FreeBSD-System
vorhanden ist und dieses nun mit dem Kerio Mailserver ergaenzt werden
soll.

.. note::

  Kerio gibt offiziell keine Unterstützung fuer dessen Mailserver unter FreeBSD
  bekannt. Auch wird offiziell kein Support fuer solche Installationen
  gewaehrt. Da es sich um eine reine Linux-Software handelt, verwenden wir hier
  den Linux-Binary-Support von FreeBSD. Die Software läuft jedoch von der
  Performance her fast besser als unter Linux :) Größere Belastungstests mit
  mehreren Userzugriffen gleichzeitig habe ich bis jetzt jedoch nicht gemacht.
  Probleme oder Bugs konnte ich während der Tests keine feststellen.

Webseite des Kerio-Mailservers: http://www.kerio.com/kms_home.html

Warum dieses Howto
------------------

Mit FreeBSD und Samba lässt sich ja schon mal ein super
Domaenencontroller fuer Windows Clients einrichten. Fehlt halt nur noch
der Ersatz fuer dieses komische Mailserverding namens M$ Exchange
Server. Und genau hier kommt eben der Kerio Mailserver zum Einsatz.
Neben dem komfortablen Webmail bietet diese Lösung auch die Möglichkeit
die bestehenden Outlook-Clients zu verbinden. Der normale Benutzer kann
also keinen Unterschied zu einer Exchange-Lösung erkennen. Zusätzlich
ist die Performance beim Synchronisieren der Daten gegenüber der
M$-Lösung einiges besser. Außerdem ist der Kerio Mailserver wirklich
günstig und läuft eben auch unter FreeBSD einwandfrei, wenn auch nicht
offiziell.

Preise Kerio Mailserver: http://www.kerio.com/kms_price.html

Lange Rede, kurzer Sinn, jetzt gehts los :)

Installation
------------

freeBSD Linux-Binary support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Da, wie bereits erwaehnt, der Kerio-Mailserver eine Linuxsoftware ist,
muessen wir unter FreeBSD den Linux-Binary-Support installieren.

Aktiviert also die Unterstuetzung in /etc/rc.conf mit folgender Zeile:

::

   linux_enable="YES"

Danach benötigen wir noch ein Linux-Base-System. Ich habe mich hier fuer
Fedora Core 4 entschieden, da dieses System auch von Kerio unterstützt
wird. Außerdem ist es das Standard-Linux System für FreeBSD.

::

   # cd /usr/ports/emulators/linux_base-fc4/
   # make install clean

Nach der Installation empfiehlt sich ein Neustart, damit die Anweisungen
in der rc.conf neu geladen werden.

Herunterladen der Installationsdateien, Installation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nun die Demo-Version des Servers sowie der Administrationskonsole unter
http://www.kerio.com/kms_download.html herunterladen. Ich habe die
beiden RPM-Dateien unter /usr/src/kerio/ abgelegt und extrahiere die
Dateien nun mit folgenden Befehlen:

::

   # cd /compat/linux
   # rpm2cpio /usr/src/kerio/kms6-linux-i386 | cpio -i --make-directories 
   # rpm2cpio /usr/src/kerio/kms6-admin-linux-i386 | cpio -i --make-directories 

Ergänzen der fehlenden Linux-libs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Damit der Server läuft müssen wir noch ein paar Linux-libs nach
/compat/linux/lib/ kopieren. Ich hab die benötigten Dateien von einem
Suse-System "geklaut" und stelle sie zum Download zur Verfügung.

http://www.netclan.ch/fbsd-kerio/

Bei Problemen findet ihr im FreeBSD-Handbuch ausführliche Informationen
ueber den Linux-Support und Libraries.

http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/linuxemu-lbc-install.html

Folgende libs werden von der mailserver Bin-Datei verwendet:

::

   # ldd /compat/linux/opt/kerio/mailserver/mailserver
   libpthread.so.0 => /lib/obsolete/linuxthreads/libpthread.so.0 (0x28d3b000) 
   libz.so.1 => /usr/lib/libz.so.1 (0x28d8f000) 
   libuuid.so.1 => /lib/libuuid.so.1 (0x28da2000) 
   libcrypt.so.1 => /lib/libcrypt.so.1 (0x28da5000) 
   libresolv.so.2 => /lib/libresolv.so.2 (0x28dd2000) 
   libdl.so.2 => /lib/libdl.so.2 (0x28de4000) 
   libstdc++.so.5 => /usr/lib/libstdc++.so.5 (0x28de8000) 
   libm.so.6 => /lib/obsolete/linuxthreads/libm.so.6 (0x28ea2000) 
   libgcc_s.so.1 => /lib/libgcc_s.so.1 (0x28ec8000) 
   libc.so.6 => /lib/obsolete/linuxthreads/libc.so.6 (0x28ed2000) 
   /lib/ld-linux.so.2 (0x28d1d000)

Konfiguration, erster Start
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nun könnt ihr mit

::

   # /compat/linux/opt/kerio/mailserver/cfgwizard

die Grundeinstellungen des Mailservers vornehmen. Dieses Wizard ist so
ziemlich selbsterklärend. Achtet darauf, dass ihr beim letzten Punkt
"Message Store Directory" ein Verzeichnis auf einem Laufwerk mit
genügend Speicherplatz wählt.

Nun könnt ihr den Server mit

::

   # /compat/linux/opt/kerio/mailserver/mailserver 

starten.

Wenn der Server erfolgreich gestartet wurde, könnt ihr mit einem
Webbrowser auf das Login und die Web-Adminkonsole zugreifen.

::

   Login:
   https://ip-oder-hostname/

::

   Web-Admin:
   https://ip-oder-hostname/admin/

Administrationstool
~~~~~~~~~~~~~~~~~~~

Sobald der Mailserver läuft, koennt ihr via der Administrationskonsole
die es offiziell fuer Windows, Mac OS X und Linux gibt auf den Server
verbinden und diesen nach euren Vorstellungen einrichten.

Falls ihr einen X-Server auf dieser Maschine installiert habt, könnt ihr
mit das Admintool nach einem

::

   # cp /compat/linux/opt/kerio/admin/libqt-mt.so.3* /compat/linux/lib

mit dem Befehl

::

   # /compat/linux/opt/kerio/admin/kadmin 

starten.

Am besten erstellt ihr noch einen Startscript und legt diesen unter
/usr/local/etc/rc.d/ ab, damit der Mailserver beim Starten des Systems
hochfährt.

Schlussteil
-----------

Nun wünsche ich euch viel Spass und Erfolg beim Administrieren und
Nutzen des Kerio Mailservers.

Kritik, Erfahrungsberichte, Fragen hier:
http://www.bsdforen.de/showthread.php?t=15550

* :ref:`genindex`

Zuletzt geändert: |date|

