Thunar
======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

`Thunar <http://thunar.xfce.org>`__ ist ein grafischer Dateimanager für
X-Window unter UNIX. Als Teil der Desktopumgebung
`Xfce <http://www.xfce.org>`__ / `xfce4 <xfce4>`__ kann Thunar mit
Plug-Ins erweitert werden.

Installation
------------

|Screenshot| Thunar wird seit xfce4 V 4.4.x automatisch mit dem
meta-port
`x11-wm/xfce4 <https://www.google.com/search?q=x11-wm/xfce4&btnI=lucky>`__
installiert. Er kann dennoch auch allein aus den Ports installiert
werden:

::

  # cd /usr/ports # portinstall x11-fm/thunar

Als Optionen zum Kompilieren sollten mindestens **DBUS**, **HAL**,
**JPEG** und **STARTUP** gewählt werden.

Post-Installation
-----------------

Theme
~~~~~

Wird nicht die Desktopumgebung Xfce verwendet, nutzt Thunar keine Themes
und kommt darum in einem tristen Erscheinungsbild daher. Folgender
Eintrag in **~/.gtkrc-2.0** behebt dies

::

   gtk-icon-theme-name="Tango"

Die `Theme Tango <http://tango-project.org>`__ lässt sich einfach aus
den Ports installieren::

  # cd /usr/ports/x11-themes/icons-tango/ 
  # make install clean

Selbstverständlich können auch andere `GTK
Themes <http://art.gnome.org/themes>`__ verwendet werden.

D-BUS & HAL
~~~~~~~~~~~

Wurde Thunar mit der Option
`D-BUS <http://www.freedesktop.org/wiki/Software/dbus>`__ und
`HAL <http://www.freedesktop.org/wiki/Software/hal>`__ installiert, so
müssen in der Datei **/etc/rc.conf** die Zeilen

::

   dbus_enable="YES"
   hald_enable="YES"
   polkitd_enable="YES"

eingefügt werden, um beim nächsten Neustart des System aktiviert zu
werden. D-Bus und HAL können ohne Neustart wie folgt gestartet werden:

::

  # /usr/local/etc/rc.d/dbus start # /usr/local/etc/rc.d/hald start

Plugins
~~~~~~~

Folgende Plugins können optional installiert werden:

Archiv-Plugin
^^^^^^^^^^^^^

`archivers/thunar-archive-plugin <https://www.google.com/search?q=archivers/thunar-archive-plugin&btnI=lucky>`__
- Packen und Entpacken von Dateien via Kontext-Menü

Mediatag-Plugin
^^^^^^^^^^^^^^^

`audio/thunar-media-tags-plugin <https://www.google.com/search?q=audio/thunar-media-tags-plugin&btnI=lucky>`__
- Mediatag-Plugin

SVN-Plugin
^^^^^^^^^^

`devel/thunar-svn-plugin <https://www.google.com/search?q=devel/thunar-svn-plugin&btnI=lucky>`__
- SVN-Befehle via Kontext-Menü

Gerätemanager-Plugin
^^^^^^^^^^^^^^^^^^^^

Erst mit dem `Thunar Volume
Manager <https://www.google.com/search?q=sysutils/thunar-volman-plugin&btnI=lucky>`__
ist Thunar in der Lage entfernbare Datenträger (z.B. USB-Stick) und
Geräte (Kameras, Drucker) bequem zu verwalten.

Nach der Installtion des Plugins muss Thunar neu gestartet werden.

Anschliessend kann unter **Edit** -> **Preferences...** im Reiter
**Advanced** das Häckchen bei **Enable Volume Management** gesetzt
werden. Unter dem Link **Configure** im gleichen Reiter können
anschliessend die Optionen für den Datenträger- und Media-Verwaltung
gesetzt werden.

Voraussetzung ist, dass DBUS und HAL installiert und aktiviert sind.

Siehe auch
~~~~~~~~~~

-  `xfce4 <xfce4>`__
-  http://goodies.xfce.org/projects/thunar-plugins/start

.. |Screenshot| image:: http://thunar.xfce.org/images/filewindow-1.png
   :class: align-right
   :width: 180px

* :ref:`genindex`

Zuletzt geändert: |date|

