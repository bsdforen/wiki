Multihead mit nv
================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Seit einiger Zeit unterstützt der freie Treiber nv(4x) für nVidia-Karten auch
die parallele Nutzung beider Monitorausgänge mit zwei Monitoren. Aufgrund
vieler Nachfragen wird hier einmal kurz aufgezeigt, wie man dies einrichtet und
worauf man achten muss.

Vorraussetzung
--------------

Benötigt wird eine nVidia-Grafikkarte mit zwei Monitorausgängen. Alle
bisher erschienene Modelle werden unterstützt. Weiter benötigt man den
nv-Treiber in Version 2.1.3 oder höher. Dieser ist Teil von X.org 7.3,
kann aber auch mit älteren Versionen kompiliert werden.

Einschränkungen
---------------

nv nutzt zum Ansprechen mehrerer Monitore keinen MergedFB, wie viele
andere Treiber, stattdessen geht er über VBE. Dies beinhaltet als
Nachteil, dass einmal beide Monitore die gleiche Auflösung haben müssen.
Auch müssen diese bereits zur Bootzeit an der Grafikkarte angeschlossen
sein. Die Darstellung läuft unbeschleunigt ab und die xv-Erweiterung für
Videodarstellung funktioniert nicht oder bringt sogar die Darstellung
zum Einfrieren. In diesem Fall muss das System über ssh neu gestartet
werden. Ein Ändern der Orientierung der Monitore ist auch nicht möglich,
sie stehen immer nebeneinander. An welchem Anschluss der Grafikkarte der
linke und an welchem der rechte Monitor hängt, hängt von der verwendeten
Grafikkarte ab.

Einrichtung
-----------

Die Einrichtung ist simpel. Einmal wird in die Device-Section der
xorg.conf eine neue Zeile "Option "Dualhead" "true" eingefügt. Dies
ermöglicht die Ausgabe auf beiden Monitoren. Anschließend wird in der
Screen-Section EIN Monitor konfiguriert. nv dupliziert diese
Einstellungen automatisch auf den zweiten. Folgende Konfiguration
betreibt zwei TFT-Monitore bei 1280x1024 nebeneinander:

::

   Section "Device"
         Identifier  "Card0"
         Driver      "nv"
         Option      "Dualhead" "true"
         BusID       "PCI:2:0:0"
   EndSection

::

   Section "Screen"
         Identifier "Screen0"
         Device     "Card0"
         Monitor    "Monitor0"
         DefaultDepth     24
         SubSection "Display"
                 Viewport   0 0
                 Depth     24
                 Modes "1280x1024"
         EndSubSection
   EndSection

Xinerama
--------

nv unterstützt derzeit kein Xinerama. Es ist jedoch mit einem Hack
möglich, die entsprechende Funktionalität zu erhalten. Hierzu wird im
Paket libXinerama die Datei src/Xinerama.c mit folgendem Code gepatcht
und dann neu gebaut. Siehe:
http://deponie.yamagi.org/freebsd/patches/xinerama.patch

Jetzt kann im Homeverzeichnis eine Datei .fakexinerama angelegt werden,
welche die Geometrie enthält. In der ersten Zeile stehen die Anzahl der
Monitore - normalerweise also 2 - in den folgenden Zeilen dann für jeden
Monitor erst die horizontale Verschiebung, dann die vertikale, die
Breite des Bildes und die Höhe. Die Angaben sind in Pixeln zu machen. Im
obrigen Beispiel mit zwei Monitoren mit je 1280x1024 sieht die
.fakexinerama also so aus:

::

   2
   0 0 1280 1024
   1280 0 1280 1024

Nach einem Neustart von X.org ist Xinerama vorhanden und kann genutzt
werden. Natürlich lässt sich dieser hack auch mit jedem anderen Treiber
kombinieren.

* :ref:`genindex`

Zuletzt geändert: |date|

