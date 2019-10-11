Portmaster
==========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Neben dem bereits etablierten `portupgrade </anwendungen/portupgrade>`__ gibt
es auch Alternativen zum (halb-)automatischen Update der über den Portstree
installierten Anwendungen und Bibliotheken.

Bei
`Portmaster <https://www.google.com/search?q=ports-mgmt/portmaster&btnI=lucky>`__
handelt es sich um ein Leichtgewicht in jeder Hinsicht:

-  es benötigt keine zusätzliche Programmiersprache, da es ein
   Shell-Script ist.
-  es benötigt keine zusätzliche Datenbank, da es die vorhandenen
   Ports-Tree Strukturen nutzt.

Portmaster bietet gegenüber portupgrade noch gravierende Vorteile:

-  Zeitersparnis, da die Datenbank-Maintenance entfällt. Dies macht sich
   vorallem auf Systemen mit einer Vielzahl von installierten Ports
   bemerkbar.
-  Vorgänge, die Benutzerinteraktionen mit sich bringen (z.B. make
   config), werden im Vorfeld abgearbeitet, sodaß der eigentliche
   Update/Upgrade-Prozess ohne weiteren Benutzereingriff erfolgt. Auch
   dies macht sich vorallem bei vielen Ports bemerkbar. Der Benutzer
   braucht nicht während des Prozesses vor dem Bildschirm sitzen, um auf
   weitere Interaktionen zu warten.
-  Informationstexte (pkg-messages) werden nach Abschluß aller Upgrades
   angezeigt.
-  Ebenfalls zum Schluß wird ab Version 2.0 eine Liste der erfolgreich
   bearbeiteten Ports aufgelistet.

Nachteile sind:

-  Portmaster ist nicht fehlertolerant. Der Upgrade/Update-Vorgang wird
   bei einem Fehler während des "Port-Baus" unterbrochen und muß nach
   der Fehlerbeseitigung erneut gestartet werden.

.. note::

  portmaster und portupgrade sollten nicht parallel installiert und benutzt
  werden. Es kann zu Inkonsistenzen in der portupgrade-Datenbank führen.

Schnellkurs
-----------

Alle installierten Ports auf Upgrades prüfen und ggf. Upgrade durchführen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

  # portmaster -a

Mit der Option **-u** (unattended mode) kann dieser Vorgang noch weiter
automatisiert werden. Es findet keine Interaktion mehr mit dem Benutzer
statt, da Default-Werte bei ggf. auftretenden Rückfragen vom System
angenommen werden. Diese Option kann unter Umständen zu
unvorhergesehenen Ergebnissen führen. Sollte dies der Fall sein, sollte
Portmaster erneut ohne diese Option ausgeführt werden. Ein Mittelweg
wäre die Option **-d** (always clean distfiles) um wenigstens nicht nach
der Deinstallation von alten Paketen gefragt zu werden und Portmaster
diese einfach zu deinstallieren.

Einen bestimmten Port Upgraden, Installieren bzw. erneut Installieren
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

  # portmaster <Kategorie/Name des Ports> 

Beispiel: # portmaster sysutils/py-iocage

Bei einem bestimmten Port soll nicht automatisch ein Upgrade durchgeführt werden
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hierzu wird im Pfad **/var/db/pkg/<Name des Ports>/** wird die Datei
+IGNOREME angelegt. Dies kann eine "leere" Datei sein.

::

  # echo > /var/db/pkg/de-openoffice.org-2.2.0/+IGNOREME

Nützliche Optionen
------------------

Es sollen keine Backup-Packages angelegt werden
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Portmaster erstellt in der Grundeinstellung automatisch Packages vom
bereits installierten Port. Dies hat den Vorteil, daß die aktuelle
Version des Ports wieder durch die Vorgängerversion ersetzt werden kann,
sofern es mit der neuen/aktuellen Version Probleme gibt. Mit der Option
**-B** kann die Erstellung solch eines Backup-Packages unterbunden
werden.

Es sollen grundsätzlich Packages angelegt werden
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Portmaster kann mit der Option **-g** angewiesen werden, beim
planmäßigen Update der Ports auch zusätzlich ein Package anzulegen. Dies
ist besonders in einem lokalen Netz sinnvoll, bei dem auf mehreren (und
i.d.R. langsameren) Maschinen FreeBSD-Ports installiert werden. Leider
fehlt Portmaster zur Zeit noch die Option, um bspw. auf diesen Maschinen
vorrangig Packages zu installieren (Stand 04/2008).

Problembeseitigung
------------------

Es kann vorkommen, das die im /var/db/pkg-Tree eingetragenen
Abhängigkeiten nicht mehr stimmen. Hierzu kann man <xterm> # portmaster
--check-depends </xterm> starten. Fehlende oder fehlerhafte
Abhängigkeiten werden angezeigt und können jeweils durch Bestätigung
entfernt werden. Diese Option sollte auch bei einem Wechsel von
portupgrade zu portmaster einmal durchgeführt werden.

Übersicht über die Optionen (unvollständig!)
--------------------------------------------

== =====================================================================================================
-a Es werden alle installierten Ports überprüft und wenn notwendig, auf den neusten Stand gebracht
-B Es werden keine Backup-Packages erstellt
-d Always clean distfiles - Nicht mehr benötigte (meist veraltete) Distfiles werden automatisch gelöscht
-g Es werden zusätzlich aktuelle Packages gebaut
-i Interaktiver Programmablauf - Vor Aktionen kann der Benutzer entscheiden
-u Unattended Mode - Bei Benutzerinteraktionen werden automatisch Default-Werte angenommen
-v weitere Ausgaben beim Programmablauf
== =====================================================================================================

Siehe auch
----------

-  `Portmaster Homepage <http://dougbarton.us/portmaster.html>`__
-  `Paketsysteme </howto/paketsysteme>`__
-  `Ports - Entwicklung und Wartung </ports/entwicklung_und_wartung>`__
-  `Pakete deinstallieren </howto/pakete deinstallieren>`__
-  `Portstree aktualisieren </anwendungen/portsnap>`__

* :ref:`genindex`

Zuletzt geändert: |date|

