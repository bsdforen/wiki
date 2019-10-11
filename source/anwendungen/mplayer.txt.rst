mplayer
=======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Mplayer ist ein vielseitig einsetzbarer Kommandozeilen
Mediaplayer mit verschiedenen graphischen Oberflächen, der auch zum
Kodieren eingesetzt werden kann. |mplayer| 

Installation
------------

Sowohl in den `FreeBSD </FreeBSD>`__ Ports als auch unter Pkgsrc sind
mplayer und die Encoder-Komponente mencoder separate Pakete. In den
`OpenBSD </OpenBSD>`__ Ports hingegen werden beide von einem Port
installiert.

-  `multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
   (FreeBSD)
-  `multimedia/mencoder <https://www.google.com/search?q=multimedia/mencoder&btnI=lucky>`__
   (FreeBSD)
-  `multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
   (pkgsrc)
-  `multimedia/mencoder <https://www.google.com/search?q=multimedia/mencoder&btnI=lucky>`__
   (pkgsrc)
-  `x11/mplayer <https://www.google.com/search?q=x11/mplayer&btnI=lucky>`__
   (OpenBSD)

Graphische Oberflächen
----------------------

Die GTK Oberfläche wird normalerweise mitgebaut und kann mit
**gmplayer** aufgerufen werden. Alternativ gibt es auch
`kmplayer <kmplayer>`__ für `KDE <KDE>`__.

mencoder
--------

Mit der mencoder Komponente von mplayer kann Film und Ton in
verschiedene Formate umgewandelt werden. Auch `DVD
Rips </howto/DVD-Rip mit mencoder>`__ sind damit möglich.

Widescreen Monitor
------------------

Mplayer geht davon aus dass der Monitor ein Seitenverhaeltnis von 4:3
hat. Alle Widescreen Benutzer muessen folgendes in **~/.mplayer/config**
eintragen:

::

   monitoraspect=16:9

oder als Option **-monitoraspect 16:9** beim starten des Mplayer
mitgeben. Bei einem 16 zu 10 Monitor muss natürlich **16:10** verwendet
werden.

Streams abspeichern
-------------------

Mit folgendem Aufruf kann man Streams auf der Festplatte speichern:

::

   # mplayer -dumpstream -dumpfile /tmp/stream.rm rtsp://server/directory/stream.rm

Weiter Wiki Artikel mit Mplayer/Mencoder
----------------------------------------

-  `Film-DVD Erstellen </howto/Film-DVD Erstellen>`__
-  `DVD-Rip mit mencoder </howto/DVD-Rip mit mencoder>`__

Verweise
--------

-  Die mplayer `Homepage <http://www.mplayerhq.hu>`__.
-  Die
   `Dokumentation <http://www.mplayerhq.hu/DOCS/HTML/de/index.html>`__
   und die
   `Manpage <http://www.mplayerhq.hu/DOCS/man/de/mplayer.1.html>`__ auf
   Deutsch (nicht immer vollständig).
-  Die
   `Dokumentation <http://www.mplayerhq.hu/DOCS/HTML/en/index.html>`__
   und die
   `Manpage <http://www.mplayerhq.hu/DOCS/man/en/mplayer.1.html>`__ auf
   Englisch.
-  `multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
   und
   `multimedia/mencoder <https://www.google.com/search?q=multimedia/mencoder&btnI=lucky>`__
   in den FreeBSD Ports.
-  `multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
   und
   `multimedia/mencoder <https://www.google.com/search?q=multimedia/mencoder&btnI=lucky>`__
   in Pkgsrc.
-  `x11/mplayer <https://www.google.com/search?q=x11/mplayer&btnI=lucky>`__
   in den OpenBSD Ports.

.. |mplayer| image:: images/mplayer.png
   :class: align-right
   :width: 180px

* :ref:`genindex`

Zuletzt geändert: |date|

