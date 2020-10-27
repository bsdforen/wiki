Unreal Tournament 2004
======================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Im Handel gibt es verschiedene Versionen von Unreal Tournament, mit und ohne
Linux Installer. Im folgenden wird die manuelle Einrichtung des Spiels, ohne
Linux Installer, beschrieben.

Benötigte Pakete
----------------

Folgende Pakete werden zur Installation benötigt:

-  `archivers/unshield <https://www.google.com/search?q=archivers/unshield&btnI=lucky>`__
   um Dateien aus Microsoft Cabinet (.cab) Dateien zu extrahieren.
-  `devel/linux-sdl12 <https://www.google.com/search?q=devel/linux-sdl12&btnI=lucky>`__
   eine Multimedia Bibliothek, welche unter anderem für die
   Grafikausgabe zuständig ist.
-  `audio/linux-openal <https://www.google.com/search?q=audio/linux-openal&btnI=lucky>`__
   kümmert sich um die Soundeffekte.

Installation
------------

Von der UT2K4 DVD werden folgende Dateien benötigt: data1.cab,
data1.hdr, data2.cab, data3.cab, data4.cab, data5.cab und data6.cab. Die
cabs werden alle in ein beliebiges Verzeichnis kopiert und entpackt:

# unshield x /pfad/zur/data1.cab -d /mein_spiele_verzeichnis/ut2k4

Das Spiel Patchen
-----------------

Damit das Spiel funktioniert wird der aktuellste
`Patch <http://download.beyondunreal.com/fileworks.php/official/ut2004/ut2004-lnxpatch3369-2.tar.bz2>`__
von UT2K4 benötigt, dieser wird zugleich entpackt. Der entpackte Patch
mit seinen sechs Verzeichnissen wird nun in den
**mein_spiele_verzeichnis/ut2k4** Ordner verschoben. Wenn alles richtig
gemacht wurde, befinden sich nun vier ausführbare Dateien im
**/mein_spiele_verzeichnis/ut2k4/System** Verzeichnis.

Benötigte Libs kopieren
-----------------------

Zusätzlich werden zwei Libs ins **ut2k4/System** Verzeichnis kopiert,
eine aus dem **linux-sdl-**, sowie eine aus dem **linux-openal**-Paket.

::

   # cp /usr/compat/linux/usr/lib/libSDL-1.2.so.0.7.3 /mein_spiele_verzeichnis/ut2k4/System/libSDL-1.2.so.0
   # cp /usr/compat/linux/usr/lib/libopenal.so.0.0.0 /mein_spiele_verzeichnis/ut2k4/System/openal.so

CD-Key hinzufügen
-----------------

Zum Online Spielen wird ein CD Key benötigt. Dafür wird die Textdatei
**ut2k4/System/cdkey** erstellt, diese enthält die eigene Seriennummer
in GROßBUCHSTABEN.

Spiel starten
-------------

Damit ist die UT2K4 Installation abgeschlossen. Das Spiel startet indem
in das **ut2k4/Sytem** Verzeichnis gewechselt und folgendes eingegeben
wird:

::

   # ./ut2004-bin

verwandte Artikel
-----------------

Es empfiehlt sich zu diesem Thema auch folgenden Artikel gelesen zu
haben.

-  `FreeBSD - Spiele <hauptseite>`__

Links
-----

-  `Offizielle Webpräsenz von Unreal
   Tournament <http://www.unrealtournament.com/>`__
-  `inUnreal.de <http://unreal.ingame.de/index.php>`__ Die große
   deutsche Unreal Seite. Infos, Tips, Tricks ...
-  `planetunreal.com <http://www.planetunreal.com/>`__ Große
   englischsprachige Seite zu Unreal.

* :ref:`genindex`

Zuletzt geändert: |date|

