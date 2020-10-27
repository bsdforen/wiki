Xine
====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

`Xine <http://xinehq.de/>`__ ist ein freier Multimediaplayer. Er spielt
CDs, DVDs und VCDs ab. Außerdem unterstützt er Plugins zum Abspielen von
AVI, MOV, WMV und MP3. Diese Formate werden allerdings nur von der
lokalen Festplatte gespielt. Video-Streams kann xine auch empfangen und
wiedergeben. Es werden die meisten Formate und viele andere unterstützt.
xine selber ist eine Library, die von mehreren Frontends benutzt wird.
Zu den Frontends gehören einmal die
xine-ui\ `Screenshot </Glossar/Screenshot>`__,
`Totem <http://www.gnome.org/projects/totem>`__,
`Kaffeine <http://www.gnome.org/projects/totem>`__ oder das
Mozilla-Plugin.

Features
--------

|Xine.jpg|

-  Verschiedene Frontends
-  Installation der Skins direkt aus dem Internet
-  Navigationbuttons wie start, pause, stop, suchen,...
-  Untertitel
-  Menüs
-  Audio und Untertitel wählen
-  Playlisten
-  Konfigurationsdialog
-  Vollbildwiedergabe

Konfiguration
-------------

Zur Konfiguration ist nicht viel nötig. xine biete ein
Konfigurationsmenü. Beim ersten Start wird dieses Menü gestartet und man
kann gewünschte Optionen wie die Eingabe- und Ausgabeplugins ändern.
Ansonsten ist um normale DVDs abspielen zu können normalerweise keine
weitere Konfiguration nötig. Um Originaldvds abspielen zu können ist
etwas Arbeit nötig. Zuerst muss man einen Symlink dvd in /dev auf sein
DVD-Laufwerk setzen:

::

    # cd /dev
    # ln -sf ${DVD-Device} dvd
    # ln -sf ${DVD-Device} rdvd
    # echo 'link ${DVD-Device} dvd' >> /etc/devs.conf
    # echo 'link ${DVD-Device} rdvd' >> /etc/devs.conf

Dann muss man Schreibrechte setzen und die library libdvdcss
installieren:

::

    # cd /dev
    # chmod +w dvd rdvd
    # cd /usr/ports/multimedia/libdvdcss && make install clean
    # pkg_add -r libdvdcss

Nun sollte man auch Originaldvds abspielen können. Wichtig: Das Laufwerk
darf vor dem Abspielen nicht gemountet sein.

Siehe auch
----------

-  `multimedia/xine <https://www.google.com/search?q=multimedia/xine&btnI=lucky>`__

.. |Xine.jpg| image:: images/xine.jpg
   :class: align-right
   :width: 192px

* :ref:`genindex`

Zuletzt geändert: |date|

