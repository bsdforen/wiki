Xfce4 mit gdm
=============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Beitrag soll Schritt für Schritt durch die Installation des `GNOME
Display Managers (GDM) </anwendungen/gdm>`__ führen und die Integration von
`xfce4 </anwendungen/xfce4>`__ in Selbigen aufzeigen.

Installation
------------

Folgende Ports müssen installiert sein:

-  `x11/gdm <https://www.google.com/search?q=x11/gdm&btnI=lucky>`__
-  `x11-wm/xfce4 <https://www.google.com/search?q=x11-wm/xfce4&btnI=lucky>`__

Konfiguration
-------------

Die Datei ``/usr/local/share/xsessions/xfce4.desktop`` muss editiert
werden:

::

   [Desktop Entry]
   Encoding=UTF-8
   Name=Xfce4
   Exec=/usr/local/bin/startxfce4
   Icon=
   Type=Application

wobei darauf zu achten ist, dass der Pfad zum Binary "startxfce4" in der
Zeile "Exec" auch korrekt gesetzt ist.

Testen
------

Ein Start von::

  # gdm

sollte nun den GDM präsentieren und man kann links oben unter Sessions "Xfce4"
anwählen.

GDM beim Booten starten
-----------------------

Um den GDM beim Booten automatisch zu starten in **/etc/rc.conf**
folgende Zeile einfügen:

::

   gdm_enable="YES"

Siehe auch
----------

-  `fluxbox_mit_gdm <fluxbox_mit_gdm>`__

* :ref:`genindex`

Zuletzt geändert: |date|

