Xfce4
=====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

`Xfce <http://www.xfce.org>`__ ist ein kompakter und modularer Window-Manager.
Die aktuelle Version ist 4.8 (Stand: Mai 2011) In dieser Version ist die
HAL-Funktionaltät komplette entfernt und durch *devd* ersetzt worden, da im
Linux-Kernel *hal* in Zukunft nicht weiter unterstützt wird. Da *devd* unter
FreeBSD zur Zeit nicht unterstützt wird, funktionieren einige Teile/Module
nicht!

Basis-Installation
------------------

Der
`x11-wm/xfce4 <https://www.google.com/search?q=x11-wm/xfce4&btnI=lucky>`__-Port
ist ein `metaport </kompendium/metaport>`__ für die xfce4-Basismodule.
Folgende `ports </x11/ports>`__ werden mit diesem Metaport installiert:

====================================================================================================== =============================================
`x11-wm/xfce4-wm <https://www.google.com/search?q=x11-wm/xfce4-wm&btnI=lucky>`__                       XFCE
`x11-wm/xfce4-session <https://www.google.com/search?q=x11-wm/xfce4-session&btnI=lucky>`__             Sessionmanagement
`x11-wm/xfce4-panel <https://www.google.com/search?q=x11-wm/xfce4-panel&btnI=lucky>`__                 Panels auf dem Desktop
`x11-wm/xfce4-desktop <https://www.google.com/search?q=x11-wm/xfce4-desktop&btnI=lucky>`__             Desktophandling
`sysutils/xfce4-utils <https://www.google.com/search?q=sysutils/xfce4-utils&btnI=lucky>`__             Hilfsmittel/Programme
`sysutils/xfce4-settings <https://www.google.com/search?q=sysutils/xfce4-settings&btnI=lucky>`__      
`x11-themes/gtk-xfce-engine <https://www.google.com/search?q=x11-themes/gtk-xfce-engine&btnI=lucky>`__
`x11-fm/thunar <https://www.google.com/search?q=x11-fm/thunar&btnI=lucky>`__                           Dateimanager (siehe auch `thunar <thunar>`__)
`misc/xfce4-appfinder <https://www.google.com/search?q=misc/xfce4-appfinder&btnI=lucky>`__             Anwendungssuchprogramm (optional)
`deskutils/orage <https://www.google.com/search?q=deskutils/orage&btnI=lucky>`__                       Kalender (optional)
`editors/mousepad <https://www.google.com/search?q=editors/mousepad&btnI=lucky>`__                     Einfacher Texteditor (optional)
`print/xfce4-print <https://www.google.com/search?q=print/xfce4-print&btnI=lucky>`__                   Druckmanagement (optional)
`x11/Terminal <https://www.google.com/search?q=x11/Terminal&btnI=lucky>`__                             Terminalemulation (optional)
`audio/xfce4-mixer <https://www.google.com/search?q=audio/xfce4-mixer&btnI=lucky>`__                   Audiomixer (optional)
`x11/gdm <https://www.google.com/search?q=x11/gdm&btnI=lucky>`__                                       Gnome Login Manager (optional)
====================================================================================================== =============================================

Plugins, Module und Anwendungen
-------------------------------

Folgende xfce4-Komponenten werden nicht automatisch mit dem Metaport
installiert und sind daher optional:

================================================================================================================================ =====================================================================
`audio/xfce4-mpc-plugin <https://www.google.com/search?q=audio/xfce4-mpc-plugin&btnI=lucky>`__                                   Plugin für MPD (Music Player Daemon)
`deskutils/xfce4-notes-plugin <https://www.google.com/search?q=deskutils/xfce4-notes-plugin&btnI=lucky>`__                       Notizzettel-Plugin für die Panels
`deskutils/xfce4-notification-daemon <https://www.google.com/search?q=deskutils/xfce4-notification-daemon&btnI=lucky>`__         Benachrichtigungs-Daemon
`deskutils/xfce4-xkb-plugin <https://www.google.com/search?q=deskutils/xfce4-xkb-plugin&btnI=lucky>`__                           Tastaturlayout-Umschalter-Plugin
`devel/xfce4-dev-tools <https://www.google.com/search?q=devel/xfce4-dev-tools&btnI=lucky>`__                                     Eine Zusammenstellung von Werkzeugen für xfce-Entwickler
`mail/xfce4-mailwatch-plugin <https://www.google.com/search?q=mail/xfce4-mailwatch-plugin&btnI=lucky>`__                         Plugin, welches neue Mails anzeigt (biff)
`misc/xfce4-artwork <https://www.google.com/search?q=misc/xfce4-artwork&btnI=lucky>`__                                           Zusätzliches Artwork (Wallpaper etc)
`misc/xfce4-weather-plugin <https://www.google.com/search?q=misc/xfce4-weather-plugin&btnI=lucky>`__                             Panel-Plugin welches aktuelles Wetter anzeigt
`multimedia/xfce4-media <https://www.google.com/search?q=multimedia/xfce4-media&btnI=lucky>`__                                   Kompakter Mediaplayer auf xine-Basis
`multimedia/xfce4-xmms-controller-plugin <https://www.google.com/search?q=multimedia/xfce4-xmms-controller-plugin&btnI=lucky>`__ Panel-Plugin zur Steuerung von xmms
`multimedia/xfce4-xmms-plugin <https://www.google.com/search?q=multimedia/xfce4-xmms-plugin&btnI=lucky>`__                       Panel-Plugin zur Steuerung von xmms
`net-im/xfce4-messenger-plugin <https://www.google.com/search?q=net-im/xfce4-messenger-plugin&btnI=lucky>`__                     Instant-Messenger-Plugin
`sysutils/xfce4-battery-plugin <https://www.google.com/search?q=sysutils/xfce4-battery-plugin&btnI=lucky>`__                     Panel-Plugin für Akkukontrolle
`sysutils/xfce4-cpugraph-plugin <https://www.google.com/search?q=sysutils/xfce4-cpugraph-plugin&btnI=lucky>`__                   Plugin, welches CPU-Load anzeigt
`sysutils/xfce4-fsguard-plugin <https://www.google.com/search?q=sysutils/xfce4-fsguard-plugin&btnI=lucky>`__                     Plugin, welches die Festplattenbelegung anzeigt
`sysutils/xfce4-genmon-plugin <https://www.google.com/search?q=sysutils/xfce4-genmon-plugin&btnI=lucky>`__                       Systemmonitor Plugin. Zeigt Temperatur, Lüftergeschwindigkeit etc. an
`sysutils/xfce4-minicmd-plugin <https://www.google.com/search?q=sysutils/xfce4-minicmd-plugin&btnI=lucky>`__                     Plugin, welches eine Shell-Eingabe ermöglicht
`sysutils/xfce4-netload-plugin <https://www.google.com/search?q=sysutils/xfce4-netload-plugin&btnI=lucky>`__                     Plugin, welches den Load eines Netzwerkgeräts anzeigt
`sysutils/xfce4-systemload-plugin <https://www.google.com/search?q=sysutils/xfce4-systemload-plugin&btnI=lucky>`__               Plugin, welches den Systemload anzeigt
`sysutils/xfce4-wavelan-plugin <https://www.google.com/search?q=sysutils/xfce4-wavelan-plugin&btnI=lucky>`__                     Plugin, welches diverse Infos zu WaveLAN-Geräten anzeigt
`textproc/xfce4-dict-plugin <https://www.google.com/search?q=textproc/xfce4-dict-plugin&btnI=lucky>`__                           Plugin, welches ein Wort übersetzt
`www/xfce4-smartbookmark-plugin <https://www.google.com/search?q=www/xfce4-smartbookmark-plugin&btnI=lucky>`__                   Ein Lesezeichen-Plugin
`x11/xfce4-clipman-plugin <https://www.google.com/search?q=x11/xfce4-clipman-plugin&btnI=lucky>`__                               Clipboard-Modul
`x11/xfce4-quicklauncher-plugin <https://www.google.com/search?q=x11/xfce4-quicklauncher-plugin&btnI=lucky>`__                   Quicklauncher-Plugin
`x11/xfce4-screenshooter-plugin <https://www.google.com/search?q=x11/xfce4-screenshooter-plugin&btnI=lucky>`__                   Erstellen von Screenshots
`x11/xfce4-taskmanager <https://www.google.com/search?q=x11/xfce4-taskmanager&btnI=lucky>`__                                     Taskmanager für xfce
`x11/xfce4-xfapplet-plugin <https://www.google.com/search?q=x11/xfce4-xfapplet-plugin&btnI=lucky>`__                             Plugin für Xfapplets
`x11-clocks/xfce4-datetime-plugin <https://www.google.com/search?q=x11-clocks/xfce4-datetime-plugin&btnI=lucky>`__               Tageszeitplugin
`x11-clocks/xfce4-timer-plugin <https://www.google.com/search?q=x11-clocks/xfce4-timer-plugin&btnI=lucky>`__                     Timerplugin
================================================================================================================================ =====================================================================

Weitere Informationen
---------------------

Für weitere Anwendungen,Plugins und Module, für die es u.U. noch keine
Ports gibt, lohnt sich der Blick auf http://goodies.xfce.org.

-  Plugins: http://goodies.xfce.org/projects/panel-plugins/start
-  Thunar-Plugins: http://goodies.xfce.org/projects/thunar-plugins/start
-  Anwendungen: http://goodies.xfce.org/projects/applications/start
-  Artwork: http://goodies.xfce.org/projects/artwork/start

XFCE4 Deutsch
-------------

In die .xinitrc bärig neijagen:

::

   export LANG=de_DE.ISO8859-15
   export LC_CTYPE=de_DE.ISO8859-15
   export LC_MESSAGES=de_DE.ISO8859-15

* :ref:`genindex`

Zuletzt geändert: |date|

