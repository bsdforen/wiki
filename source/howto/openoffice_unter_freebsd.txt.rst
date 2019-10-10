OpenOffice unter FreeBSD
========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Diese Anleitung soll bei der Installation des deutschsprachigen
``OpenOffice.org`` helfen. Eine Einführung in dieses Programm finden Sie unter
`OpenOffice </anwendungen/OpenOffice>`__.

Installation über Package
-------------------------

Besitzer einer aktuellen FreeBSD-Version (> 6.x) können die englische
OpenOffice-Version bequem mit::

  # pkg_add -r openoffice.org

installieren.

Installation über externes Package
----------------------------------

-  `ftp://ooopackages.good-day.net/pub/OpenOffice.org/FreeBSD/ Beta-Versionen und aktuelle OpenOffice-Packages <ftp://ooopackages.good-day.net/pub/OpenOffice.org/FreeBSD/ Beta-Versionen und aktuelle OpenOffice-Packages>`__
-  `http://live.prooo-box.org/openoffice.org/freebsd_dvd/ aktuelle OpenOffice-Packages <http://live.prooo-box.org/openoffice.org/freebsd_dvd/ aktuelle OpenOffice-Packages>`__

Nach dem Herunterladen des gewünschten Package können Sie mit::

  # pkg_add <Package-Name>

das Package installieren.

Die Installation externer Packages setzt einige Abhängigkeiten voraus.
Die Installation ist nicht erfolgreich, wenn eine oder mehrere dieser
Abhängigkeiten ausgegeben werden, diese Packages müssen zuerst
installiert werden. Danach sollte die Installation von OpenOffice keine
Probleme mehr bereiten.

Mehr Auswahl ist auf der `Liste inoffizieller OpenOffice
Pakete </anwendungen/OpenOffice aus inoffiziellen Paketen>`__ zu finden.

Installation über Ports
-----------------------

Da die Zeit zum Kompilieren von OpenOffice.org einige Stunden in
Anspruch nehmen kann, ist die Nutzung bereits kompilierter Pakete
sinnvoll. Es macht aber auch Sinn, OpenOffice.org über die Ports selbst
zu Kompilieren, da viele OOO viele Optionen zur Verfügung stellt.

In der Datei
**/usr/ports/editors/openoffice.org-2/files/Makefile.knobs** sind einige
der Einstellmöglichkeiten aufgeführt:

-  WITH_GNUGCJ
-  WITH_KDE
-  WITH_CUPS
-  WITHOUT_MOZILLA
-  WITHOUT_GNOMEVFS
-  WITH_EVOLUTION2
-  WITH_SYSTEM_FREETYPE
-  ALL_LOCALIZED_LANGS
-  LOCALIZED_LANG

Zur Zeit wird ein Arbeitsspeicher von 2 GB und ein freier
Plattenspeicher von bis zu 35 GB empfohlen, je nach eingestellten
Optionen.

Lesen Sie bitte vor der Installation aus den Ports die
`Installationsanleitung <http://porting.openoffice.org/freebsd/>`__
aufmerksam durch!

OpenOffice.org einrichten
-------------------------

Nach der Programminstallation müssen Sie OpenOffice für jeden Benutzer
einzeln einrichten. Dazu müssen Sie das OpenOffice-Setup aufrufen. Der
entsprechende Befehl für das OpenOffice-Setup wird Ihnen während der
Programminstallation (pkg_add) mitgeteilt.

Für OpenOffice.org 1.1.4::

  % openoffice.org-1.1.4-setup

Für OpenOffice.org 2.2::

  % openoffice.org-2.2.1

Deutsche Rechtschreibeprüfung installieren
------------------------------------------

Die deutsche Rechtschreibeprüfung (ooodict) muss nach der
Programminstallation nachinstalliert werden:

-  OpenOffice starten, ``% openoffice.org-1.1.4-swriter``
   oder ``% openoffice.org-2.2.1``
-  OpenOffice-Menü -> Datei -> Autopilot/Assistenten -> Weitere
   Wörterbücher installieren
-  Assistenten durchlaufen
-  OpenOffice-Menü -> Extras -> Optionen -> Spracheinstellung ->
   Linguistik -> Bearbeiten-Knopf
-  Im erscheinenden Dialog-Fenster die gewünschte Sprache einstellen

Fehlerbehebung
--------------

Schriftprobleme
~~~~~~~~~~~~~~~

Bei Problemen mit fehlenden oder schlecht lesbaren Schriften erhält man
unter `Bessere Schriften - Abschnitt
OpenOffice <Bessere_Schriften#OpenOffice>`__ weitere Informationen.

Weiterführende Informationen
----------------------------

-  http://porting.openoffice.org/freebsd

* :ref:`genindex`

Zuletzt geändert: |date|

