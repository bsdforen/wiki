Fluxbox mit gdm
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Beitrag soll Schritt für Schritt durch die Installation des `GNOME
Display Managers (GDM) </anwendungen/gdm>`__ führen und die Integration von
`Fluxbox </anwendungen/Fluxbox>`__ in Selbigen aufzeigen.

Installation
------------

Folgende Ports müssen installiert sein:

-  `x11/gdm <https://www.google.com/search?q=x11/gdm&btnI=lucky>`__
-  `x11-wm/fluxbox <https://www.google.com/search?q=x11-wm/fluxbox&btnI=lucky>`__

Konfiguration
-------------

Die Datei ``/usr/local/share/gnome/xsessions/Fluxbox.desktop`` muss
editiert werden:

::

   [Desktop Entry]
   Encoding=UTF-8
   Name=Fluxbox
   Exec=/usr/X11R6/bin/fluxbox
   Icon=
   Type=Application

wobei darauf zu achten ist, dass der Pfad zu dem Binary "fluxbox" in der
Zeile "Exec" auch richtig gesetzt ist.

Testen
------

Ein Start von

::

   # gdm 

sollte nun den GDM präsentieren und man kann links oben unter Sessions
"Fluxbox" anwählen.

GDM beim Booten starten
-----------------------

Um den GDM beim Booten automatisch zu starten in /etc/rc.conf folgende
Zeile einfügen:

::

   gdm_enable="YES"

Siehe auch
----------

-  `xfce4 mit gdm <xfce4 mit gdm>`__

* :ref:`genindex`

Zuletzt geändert: |date|

