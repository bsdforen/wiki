Audio-CD rippen
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Der Artikel behandelt das Kopieren und Konvertieren von Musik auf Audio-CDs.
Soweit nicht anders angegeben, gelten die Tipps für alle drei BSDs.

Das Rippen von Audio-CDs ist ein Kinderspiel, so man denn die richtigen
Programme installiert hat.

Die nötigen Ports für FreeBSD
-----------------------------

#. `audio/dagrab`_
#. `audio/bladeenc`_
#. evtl. `audio/vorbis-tools`_ und/oder `audio/abcde`_

Beide z.B. mittels **pkg install** installieren.

Die nötigen Pakete aus Pkgsrc
-----------------------------

In ports/audio:

#. `audio/cdparanoia`_
#. `audio/bladeenc`_
#. evtl. `audio/vorbis-tools`_ und/oder `audio/abcde`_

Auslesen der kompletten CD
--------------------------

Unter FreeBSD
~~~~~~~~~~~~~

::

   # dagrab -d /dev/acd0c -v -C -N -a

Alternativ unter Open- bzw. NetBSD
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   # cdparanoia -w -q -B

Einen bestimmten Titel der CD auslesen
--------------------------------------

FreeBSD speziell
~~~~~~~~~~~~~~~~

::

   # dagrab -d /dev/acd0c -v -C -N -i 3

Dadurch wird Titel 3 als Wav-Datei auf die Platte kopiert.

Alternativ (alle drei BSDs)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   # cdparanoia -w -B -q 3

Auch hier wird Lied 3 auf die Platte kopiert.

Titelinformationen zur CD
-------------------------

FreeBSD
~~~~~~~

Will man mehr Informationen zur CD (über cddb):

::

   # dagrab -d /dev/acd0c -v -C -N -a -f @FDS

Das schöne an **dagrab** ist, dass bei einer bestehenden Internetanbindung in
der **cddb** nach der CD und dem Titel gesucht wird und dies dann auch erkannt
wird.

OpenBSD
~~~~~~~

Für den Zugriff auf CDDB kann man unter OpenBSD z.B. **mp3cddb** einsetzen. Die
Bedienung ist recht einfach:

::

   # cd album
   # mp3cddb *

Nun wird der passende Eintrag in die Datei **cddbinfo.txt** geschrieben:

::

   # mp3cddbtag cddbinfo.txt

NetBSD
~~~~~~

Für NetBSD gibt es anscheinend nur Perl- bzw. Pythonmodule oder Tools
mit graphischer Oberfläche spez. für Zugriff auf die CDDB. abcde (siehe
unten) wäre hier wohl die einzige Lösung für die Konsole.

MP3 erstellen
-------------

MP3 erstellen mit:

::

   # bladeenc -br 256 -crc *.wav

--> Alle im Verzeichnis vorhandenen Wav-Dateien werden mit 256 kbit in
MP3 umgewandelt.

Ogg erstellen
-------------

Dazu muss vorbis-tools aus den Ports (in allen drei BSD Ports enthalten)
installiert sein.

::

   # oggenc -b 256 *

--> Alle im Verzeichnis vorhandenen Dateien werden mit 256 kbit in Ogg
umgewandelt

MP3 zusammenfügen
-----------------

Gerade bei Audiobooks gibt es verschiedene Titel die alle
zusammengehören. Das wären dann einige einzelne mp3-Dateien, diese kann
man einfach zu einer verschmelzen:

::

   # cat 1.mp3 2.mp3 3.mp3 ... > ALLES.mp3

Alternative: CD vollautomatisch rippen mit abcde
------------------------------------------------

Das Programm **abcde** erledigt das Auslesen, Umwandeln in OGG/MP3 und das
Umbenennen und Setzen der ID3-Tags vollautomatisch in einem Schritt und
sogar parallel.

Nach der Installation, muss in der `abcde.conf` das richtige Laufwerk angegeben
werden, z.B. so:

::

  CDROM=/dev/cd0

Die anderen Parameter sind evtl. auch interessant, aber mit den
Standardeinstellungen erzeugt abcde OGG-Dateien in guter Qualität.

.. _audio/dagrab: https://www.google.com/search?q=audio/dagrab&btnI=lucky
.. _audio/bladeenc: https://www.google.com/search?q=audio/bladeenc&btnI=lucky
.. _audio/vorbis-tools: https://www.google.com/search?q=audio/vorbis-tools&btnI=lucky
.. _audio/abcde: https://www.google.com/search?q=audio/abcde&btnI=lucky
.. _audio/cdparanoia: https://www.google.com/search?q=audio/cdparanoia&btnI=lucky

* :ref:`genindex`

Zuletzt geändert: |date|

Original Eintrag: 2007/07/03 17:01 Elwood
