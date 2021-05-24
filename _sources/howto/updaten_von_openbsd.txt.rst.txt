Updaten von OpenBSD
===================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

Empfohlene Methode - syspatch und pkg_add
-----------------------------------------

Seit OpenBSD 6.1 ist der gesamte Updateprozess sehr einfach gestaltet.

.. note::

  Eine genaue, aktuelle Anleitung findet sich auch in der OpenBSD FAQ https://www.openbsd.org/faq/index.html - da diese ständig aktualisiert wird, macht es sicher sinn dort reinzuschauen.

Einfach mit

`syspatch <https://man.openbsd.org/syspatch>`_

das Basissystem aktualisieren und dann mit

'pkg_add -u <https://man.openbsd.org/pkg_add>`_

noch alle Binärpakete aktualisieren - Fertig.

Der folgende Abschnitt wird nicht empfohlen und es gibt nur in extrem seltenen Ausnahmesituationen einen Grund ihn zu verwenden.


Update nach der alten Methode - sollte heutzutage fast nie mehr verwendet werden
--------------------------------------------------------------------------------

.. note::

  Die folgenden Inhalte sind sehr alt, vermutlich von ca. 2007 und wurden auf expliziten Wunsch im Wiki belassen.


Dieser Artikel behandelt das Updaten eines `OpenBSD </OpenBSD>`__-Systems
*innerhalb* einer Version. Das Upgraden von einer Version auf eine nachfolgende
wird hier beschrieben: http://openbsd.org/faq/de/upgrade41.html

Bevor man ein OpenBSD-System aktualisiert, sollte man sich mit der
grundlegenden Struktur des selbigen auskennen.

OpenBSD ist ein Betriebssystem. Das bedeutet, dass es nach der
Installation mit allen grundlegenden Programmen ausgestattet ist. Diese
Applikationen genügen auch den hochgesteckten Sicherheitsanforderungen,
denen sich die OpenBSD-Entwickler unterwerfen.

Sollte man zusätzliche Programme benötigen, so kann man diese
nachinstallieren. Das kann auf zwei verschiedenen Wegen passieren:

Die erste, `empfohlene
Methode <http://www.openbsd.org/faq/de/faq8.html#PortsvsPkgs>`__ ist die
Verwendung von vorkompilierten Packages, die man zum Beispiel von einem
der `OpenBSD-Mirrors <http://www.openbsd.org/ftp.html>`__ via FTP
herunterladen kann. Die andere Methode besteht in der Verwendung von
Ports, mit deren Hilfe man den Quellcode der Applikationen
herunterladen, auf der eigenen Maschine übersetzen, daraus Packages
bauen und diese installieren kann.

Da sich die Verwendung von OpenBSD für Anwendungsbereiche mit erhöhten
Sicherheitsanforderungen anbietet, will die Installation zusätzlicher
Software, wie oben beschrieben, gut ueberlegt sein, da die
OpenBSD-Entwickler aufgrund des hohen Aufwandes nicht in der Lage sind,
alle zusätzlich zum Standardsystem installierten Programme auf ihre
Sicherheit hin zu prüfen.

Überwachen des Standardsystems
------------------------------

Es empfiehlt sich, die Mailingliste
`security-announce <http://lists.openbsd.org/cgi-bin/mj_wwwusr?user=&passw=&func=lists-long-full&extra=security-announce>`__
zu beziehen, sowie regelmässig einen Blick auf die `Patches der
verwendeten OpenBSD-Version <http://www.openbsd.org/de/errata.html>`__
zu werfen.

Man kann auch von `Undeadly <http://www.undeadly.org>`__ einen `RSS
Feed <http://undeadly.org/cgi?action=errata>`__ abonnieren.

Überwachen der Packages/Ports
-----------------------------

In der Mailingliste
`ports-security <http://lists.openbsd.org/cgi-bin/mj_wwwusr?user=&passw=&func=lists-long-full&extra=ports-security>`__
werden aktuelle Mitteilungen zur Sicherheit der Ports gepostet. Ob
Packages aus Sicherheitsgründen aktualisiert wurden, kann man anhand der
Liste der `korrigierten
Packages <http://www.openbsd.org/pkg-stable.html>`__ ueberprüfen.

Diese beiden Methoden eigenen sich dazu, den Überblick über
*sicherheitsrelevante* Updates innerhalb der Packages/Ports zu behalten.

Wenn man auf seinem System Ports verwendet, kann man auch ein Update des
Ports-Tree durchführen und danach überprüfen, ob Ports aktualisiert
wurden.

Updaten des Standardsystems
---------------------------

#. Variante: Gehe auf die Seite mit den `Patches der verwendeten
   OpenBSD-Version <http://www.openbsd.org/de/errata.html>`__, lade die
   benötigten Patches herunter und kompiliere die betroffenen Binaries
   neu. Wie du das machst, ist innerhalb der Patch-Files beschrieben.
   Vorher solltest du die Datei ``src.tgz`` vom FTP oder von CD in
   ``/usr`` entpacken.
#. Variante: Du aktualisierst den Source-Tree und übersetzt sowohl den
   Kernel als auch das System neu.

Aktualisieren des Source-Tree
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Die von mir bevorzugte Methode, die neuesten Systemquellen
herunterzuladen, ist die Verwendung von cvsup. Diese Applikation ist auf
OpenBSD in der Standardversion nicht vorhanden. Es muss installiert
werden.

Installieren von cvsup
^^^^^^^^^^^^^^^^^^^^^^

Zuerst ermittele ich die aktuelle Version des Paketes, indem ich mich
anonym auf ``ftp.openbsd.org`` anmelde und dort im Verzeichnis
``/pub/OpenBSD/<RELEASE NUMMER>/packages/i386/`` nachsehe.

Wenn ich den genauen Namen des Paketes kenne, installiere ich es auf
meiner Maschine mit:

::

   # pkg_add -v ftp://ftp.openbsd.org/pub/OpenBSD/<RELEASE NUMMER>/packages/i386/cvsup-xxxx-no_x11.tgz 

Konfiguration von cvsup
^^^^^^^^^^^^^^^^^^^^^^^

Nach der Installation von cvsup kann ich dessen Verwendung vorbereiten:
Ich erzeuge mir eine cvsup-Steuerdatei.

Beispiel: cvsup-src-file

::

   ##################################################
   *default release=cvs 
   *default delete use-rel-suffix 
   *default umask=002 
   *default host=anoncvs.de.openbsd.org 
   *default base=/usr 
   *default prefix=/usr 
   *default tag=OPENBSD_3_8

   OpenBSD-src 
   ##################################################

Das Tag **OPENBSD_3_8** legt fest, dass ich die Quellen **inklusive**
der neuesten Patches, den sogenannten *patch branch*, laden will.

Starten von cvsup
^^^^^^^^^^^^^^^^^

Danach starte ich das Update:

::

   # cvsup -g -L 2 cvsup-src-file

Neubau des Kernels
~~~~~~~~~~~~~~~~~~

Sichere den aktuellen Kernel:

::

   # cp /bsd /bsd.old 

Konfiguriere den Kernel:

::

   # cd /usr/src/sys/arch/i386/conf 
   # config GENERIC 

Baue den Kernel:

::

   # cd ../compile/GENERIC 
   # make clean && make depend && make 

Kopiere den Kernel an seinen Platz:

::

   # cp /usr/src/sys/arch/i386/compile/GENERIC/bsd / 

Starte das System neu:

::

   # reboot

Sollten der neue Kernel Probleme bereiten, so starte die alte,
gesicherte Kernelversion am Bootprompt:

::

   boot> bsd.old

Neubau des Systems
~~~~~~~~~~~~~~~~~~

::

   # cd /usr/src 
   # find . -type l -name obj | xargs rm 
   # make cleandir 
   # rm -rf /usr/obj/* 
   # make obj 
   # cd /usr/src/etc 
   # make DESTDIR=/ distrib-dirs 
   # cd /usr/src 
   # make build

Updaten der Packages/Ports
--------------------------

Packages
~~~~~~~~

Ein

::

   # export PKG_PATH="ftp://ftp.de.openbsd.org/pub/OpenBSD/$(uname -r)/packages/$(uname -m)"
   # pkg_add -ui -F update -F updatedepends

sollte eigentlich alle Pakete auf den neuesten Stand bringen.

Aktualisieren des Ports-Tree
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

per cvs
^^^^^^^

Nachdem man die CVSROOT auf den neuen Release umgestellt hat:

::

   # cd /usr/ports
   # cvs -d$CVSROOT -q up -P

rm & tar
^^^^^^^^

::

   # rm -rf /usr/ports
   # wget ftp://ftp.leo.org/pub/OpenBSD/<RELEASE NUMMER>/ports.tar.gz
   # mv ports.tar.gz /usr
   # cd /usr
   # tar xzf ports.tar.gz
   # rm ports.tar.gz

Prüfen auf aktualisierte Ports
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Ein

::

   # /usr/ports/infrastructure/build/out-of-date

sollte alle "veralteten" Ports anzeigen.

Sollte ein Port veraltet sein:

::

   # cd /usr/ports/<kategorie>/<port>
   # make update

Links, sources and cosmetics: coming soon...

* :ref:`genindex`

Zuletzt geändert: |date|

