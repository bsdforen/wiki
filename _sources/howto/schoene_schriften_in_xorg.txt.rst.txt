Schöne Schriften in Xorg
========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

.. note::

  Dieser Artikel ist veraltet und bedarf dringender Überarbeitung!

Die Konfiguration der Schriften ist inzwischen kein Hexenwerk mehr, leider sind
die Anweisungen im FreeBSD Handbuch veraltet. Die Anweisungen hier sind für
alle Systeme mit Xorg gültig. Lediglich Pfadangaben sind noch FreeBSD
spezifisch.

Schriftarten installieren
-------------------------

Unter FreeBSD sollten die folgenden Ports installiert werden:

-  `x11-fonts/urwfonts <https://www.google.com/search?q=x11-fonts/urwfonts&btnI=lucky>`__
-  `x11-fonts/bitstream-vera <https://www.google.com/search?q=x11-fonts/bitstream-vera&btnI=lucky>`__
-  `x11-fonts/webfonts <https://www.google.com/search?q=x11-fonts/webfonts&btnI=lucky>`__

Letzteres ist ein Satz Schriften von Microsoft, die von vielen Websites
vorausgesetzt werden. Es geht natürlich auch ohne, aber vor allem beim
Austausch von Dokumenten mit Windows-Nutzern ist es nützlich die
Schriften auf dem Rechner zu haben.

Schriftarten aktivieren
-----------------------

Zuerst müssen die Schriften Xorg bekannt gemacht werden. Dazu müssen die
Verzeichnisse mit den Schriften in der ``xorg.conf`` eingetragen werden.
Unter FreeBSD liegt die Datei je nach Wahl des Administrators unter
``/etc/X11/xorg.conf`` oder ``/usr/local/etc/X11/xorg.conf``.

::

   Section "Files"
       RgbPath     "/usr/local/share/X11/rgb"
       ModulePath  "/usr/local/lib/xorg/modules"
       FontPath    "/usr/local/lib/X11/fonts/misc/"
       FontPath    "/usr/local/lib/X11/fonts/TTF/"
       FontPath    "/usr/local/lib/X11/fonts/OTF"
       FontPath    "/usr/local/lib/X11/fonts/Type1/"
       FontPath    "/usr/local/lib/X11/fonts/100dpi/"
       FontPath    "/usr/local/lib/X11/fonts/75dpi/"

       FontPath    "/usr/local/lib/X11/fonts/URW/"
       FontPath    "/usr/local/lib/X11/fonts/bitstream-vera/"
       FontPath    "/usr/local/lib/X11/fonts/webfonts/"
   EndSection

Mit den folgenden Kommandos kann ein Pfad auch zur Laufzeit aktiviert
werden:

::

   $ xset fp+ PFAD
   $ xset fp rehash

Darstellung konfigurieren
-------------------------

Einstellungen zur Darstellungen werden im Ordner ``fonts`` im
Verzeichnis mit den Konfigurationsdateien vorgenommen. Unter FreeBSD
wäre der vollständige Pfad ``/usr/local/etc/fonts``. Dort gibt es zwei
Verzeichnisse, ``conf.avail`` und ``conf.d``. In ``conf.avail`` liegen
fertige Konfigurationsschnipsel. In ``conf.d`` werden die
Konfigurationen die verwendet werden sollen verlinkt. Das geht
folgendermaßen:

::

   # cd /usr/local/etc/fonts/conf.d
   # ln -s ../conf.avail/70-no-bitmaps.conf

70-no-bitmaps.conf verbietet die Verwendung von Bitmap-Fonts.
Bitmap-Fonts sind sehr hässlich beim Skalieren und es gibt keine
Kantenglättung für sie. Es gibt keinen Grund mehr sie auf einem
Desktop-Rechner einzusetzen.

Die meisten sinnvollen Konfigurationen sind bereits verlinkt. Lediglich
die Kantenglättung muss vom Administrator selbst aktiviert werden. Der
Name einer Konfigurationsdatei für Kantenglättung beginnt immer mit 10.
Für die meisten Anwender ist ``10-sub-pixel-rgb.conf`` die richtige
Wahl.

Die Änderungen werden nach einem Neustart von Xorg gültig.

Verweise
--------

-  `x11-fonts <https://www.google.com/search?q=x11-fonts&btnI=lucky>`__
   - Schriftsammlungen in den FreeBSD Ports.

* :ref:`genindex`

Zuletzt geändert: |date|

