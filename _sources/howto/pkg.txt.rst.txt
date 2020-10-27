pkg
===

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

pkg (auch unter dem Entwicklerstichwort pkgng (pkg next generation)) ist
ein Paketverwaltungstool für Binärpakete. Es versteht und sieht nichts
von den Ports und ist somit auch kein Ersatz für portmaster bzw.
portupgrade. Mit pkg lassen sich von externe Repositories Binärpakete
abrufen und installieren. Zuerst wird dabei beim ersten Aufruf ein Index
gezogen wird, der alle in dem angegebenen Repository lagernde Pakete
enthält. Außerdem ist pkg in der Lage sämtliche Abhängigkeiten
aufzulösen, upgrades durchzuführen und Informationen über vorhandene und
installierte Pakete anzuzeigen. Somit stellt es einen Ersatz zu den
bisherigen Binärpaketverwaltungstools pkg_\* dar. Einige Highlights von
pkg sind:

-  Installation von einem, oder mehreren externen Repositories
-  Ein neues und effizienteres Paketformat (.txz), die mehr
   Informationen enthalten und diverse Stages fuer Scripte zur
   Installation
-  Bessere Auflösung von Abhängigkeiten, besonders bei upgrades
-  Die pkg Paketdatenbank befindet sich in */var/db/pkg* und enthält nur
   noch eine einzige Datei

Die Pakete für pkg können automatisch mit
`poudriere <http://fossil.etoilebsd.net/poudriere/doc/trunk/doc/index.wiki>`__
erstellt werden.

Installation und Einrichtung
----------------------------

Vor der Benutzung muss pkg "gebootstrapt" werden, d.h. pkg ist nicht
Bestandteil des Basissystems von FreeBSD. Lediglich das bootstraping
tool ist vorhanden und muss beim ersten Aufruf ausgeführt werden:

::

   # pkg

Das Ergebnis sieht dann so aus. Gibt es schon Pakete im alten Format so
können sie wie empfohlen mit *pkg2ng* in die pkg Datenbank importiert
werden.

::

  pkg The package management tool is not yet installed on your system. Do you
  want to fetch and install it now? [y/N]: y Bootstrapping pkg please wait
  Installing pkg-1.0.11... done If you are upgrading from the old package
  format, first run:

  # pkg2ng


In Benutzung mit den Ports muss in der **/etc/make.conf** ein
*WITH_PKGNG=yes* eingetragen werden, wenn die Pakete über die pkg
Datenbank registriert werden sollen. An der Stelle sollte noch erwähnt
werden, das es generell eine schlechte Idee ist und es sollte unbedingt
vermieden werden die pkg_\* Tools mit pkg zu mischen!

.. note::

  Seit dem `30.10.2013
  <http://lists.freebsd.org/pipermail/freebsd-pkg/2013-October/000107.html>`__
  gibt es offizielle Repositories:

  -  Hauptrepository: pkg.freebsd.org,
  -  Mirrors: pkg.eu.freebsd.org, pkg.us-east.freebsd.org,
     pkg.us-west.freebsd.org

  Inoffizielle Repositories finden sich hier (ungeprüft):

  -  http://mirror.exonetric.net/pub/pkgng/
  -  ftp://ftp.pcbsd.org/pub/mirror/packages

*pkg* sollte mit einem Versionsstand von mindestens Version 1.1.4_8
installiert sein. Das kann mit *pkg -v* überprüft werden. Sollte auf dem
System eine ältere Version installiert sein, bitte erst über die Ports
aktualisieren. Aus der *usr/local/etc/pkg.conf* sollten alle
repository-spezifischen Einträge entfernt werden (wie z. B. PACKAGESITE,
MIRROR_TYPE, PUBKEY). Wenn die Datei damit keine Einträge mehr enthält,
kann sie gelöscht werden.

Erstelle dann ein Repositoryverzeichnis:

::

  % mkdir -p /usr/local/etc/pkg/repos

Erstelle dort eine Datei */usr/local/etc/pkg/repos/FreeBSD.conf* mit
folgendem Inhalt:

::

   FreeBSD: {
     url: "pkg+http://pkg.FreeBSD.org/${ABI}/latest",
     mirror_type: "srv",
     enabled: yes
   }

<note> In der Originalanleitung ist das yes in Anführungszeichen
angeführt, dies führt jedoch zu Fehlermeldungen. </note>

Natürlich sollte das passende Repository eingesetzt werden, z. B.
*pkg.eu.freebsd.org* für Deutschland. Die URL für das Repository hat
weder eine browsable Webseite noch einen DNS Eintrag. Das ist
beabsichtigt, da es ein *SRV* Host ist.

Die man-page *pkg(8)* beschreibt die korrekte Verwendung der Datei. Mit
*pkg search* kann nach Paketen gesucht werden.

Pakete werden wöchentlich aus einem Snapshot der Ports erstellt und zwar
jeden Mittwoch morgen 01:00 UTC. Typischerweise sind sie im Repository
ein paar Tage später verfügbar.

Der Originaltext findet sich im
`Announcment <http://lists.freebsd.org/pipermail/freebsd-pkg/2013-October/000107.html>`__.

Nützliche Quickies
------------------

Eine generelle Übersicht der Befehle liefert **pkg help**. Mit *pkg help
<command>* wird eine manpage zu dem entsprechenden Kommando mit
detaillierten Informationen aufgerufen. Diese können auch direkt über
*man pkg-<command>* erreicht werden. Eine Sammlung der wichtigsten und
häufigsten Befehle mit einer kurzen Erklärung sind hier unten
aufgeführt:

Generelle Befehle zum Installieren und Entfernen von Paketen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

=============================== ====================================================================================================================================================================================================
Befehl                          Kommentar
=============================== ====================================================================================================================================================================================================
pkg install <package name>      Installiert ein Paket aus dem repo
pkg add <path/package_name.txz> Installiert ein Paket von einer lokalen Datei
pkg delete <package name>       Entfernt ein Paket.
pkg autoremove                  Entfernt alle Abhängigkeiten, die als Abhängigkeiten installiert wurden und nicht mehr benötigt werden.
pkg which <absolute path/file>  Gibt den Namen des Pakets, welches die Datei installiert hat.
pkg version                     Vergleicht die Version der aus dem Repo installierten Pakete mit der Version im installierten ports tree, falls vorhanden. Wenn nicht, werden die installierten Pakete mit denen im Repo verglichen.
pkg clean                       Entleert den Paketdaten Cache. Nützlich z.B. nach einem upgrade, um alte Pakete loszuwerden.
=============================== ====================================================================================================================================================================================================

Informationen über installierte Pakete
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

========================== ===========================================================
Befehl                     Kommentar
========================== ===========================================================
pkg info                   Listet alle installierten Pakete
pkg info <package name>    Listet den Name und die Beschreibung des Paketes.
pkg info -f <package name> Detailierte Liste mit sämtlichen Information über das Paket
pkg info -l <package name> Listet alle installierten Dateien von dem Paket
========================== ===========================================================

Fortgeschrittene Kommandos
--------------------------

"pkg query" bietet auch die Möglichkeit ein scriptfreundlicheres
Interface zu nutzen. In dem der Benutzer selbst bestimmt was angezeigt
wird. Dazu wurden diverse Variablen definiert um auf Paketnamen,
Version, Beschreibungen, Nachrichten usw. zuzugreifen. Außerdem können
auf Vergleichsoperatoren zurückgegriffen werden. Ein paar Beispiele:

================================ ============================================================
Befehl                           Kommentar
================================ ============================================================
pkg query '%n %v' <package name> Listet den Namen und Version des Packets
pkg query -a '%n %o'             Gibt eine Liste aller Pakete mit namen und Pfad im portstree
================================ ============================================================

Links
-----

-  `FreeBSD Wikiseite <https://wiki.freebsd.org/pkgng>`__
-  `FreeBSD
   Handbuch <http://www.freebsd.org/doc/en/books/handbook/pkgng-intro.html>`__
-  `pkgng auf Github (offizielle Versionsverwaltung des pkgng
   Codes) <https://github.com/pkgng/pkgng>`__

* :ref:`genindex`

Zuletzt geändert: |date|

