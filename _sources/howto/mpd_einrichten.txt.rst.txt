MPD einrichten
==============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

**MPD**, der **Music Player Daemon**, ist ein Musik Player, der über eine
TCP/IP-Verbindung ferngesteuert werden kann. Dazu werden seperate
Client-Programme verwendet. Dieser Artikel erklärt wie der Music Player Daemon
sinnvoll eingerichtet werden kann.

Installation
------------

-  `audio/musicpd <https://www.google.com/search?q=audio/musicpd&btnI=lucky>`__
   in den FreeBSD Ports.
-  `audio/musicpd <https://www.google.com/search?q=audio/musicpd&btnI=lucky>`__
   in Pkgsrc.
-  `audio/mpd <https://www.google.com/search?q=audio/mpd&btnI=lucky>`__
   in den OpenBSD Ports.

FreeBSD
-------

Der FreeBSD Port bringt ein **rc(8)** Skript mit um den Daemon beim
Booten zu starten. Vorher muss dieser natürlich konfiguriert werden.

Benutzer anlegen
~~~~~~~~~~~~~~~~

Das Skript startet den MPD als Benutzer **root**. Das ist natürlich kein
besonders begrüßenswertes Verhalten. Zum Glück kann der MPD sich jedoch
zu einem normalen Benutzer degradieren. Dazu wird erst ein mal ein
Benutzer angelegt als der der Daemon-Prozess später laufen soll:

::

   # adduser
   Username: musicpd
   Full name: Music Player Daemon
   Uid (Leave empty for default): 
   Login group [musicpd]: 
   Login group is musicpd. Invite mpd into other groups? []: 
   Login class [default]: daemon
   Shell (sh csh tcsh bash rbash nologin) [sh]: nologin
   Home directory [/home/musicpd]: 
   Use password-based authentication? [yes]: no
   Lock out the account after creation? [no]: yes
   Username   : musicpd
   Password   : <disabled>
   Full Name  : Music Player Daemon
   Uid        : 1003
   Class      : daemon
   Groups     : musicpd 
   Home       : /home/musicpd
   Shell      : /usr/sbin/nologin
   Locked     : yes
   OK? (yes/no): yes
   Add another user? (yes/no): no
   Goodbye!

Nach dem Anlegen des Benutzers wird noch ein Playlist-Verzeichnis
benötigt in dem später die abgespeicherten Playlists abgelegt werden;

::

   # mkdir -p /home/musicpd/playlists
   # chown musicpd:musicpd /home/musicpd/playlists

Daemon konfigurieren
~~~~~~~~~~~~~~~~~~~~

Als nächstes muss die Datei ``/usr/local/etc/musicpd.conf`` angelegt
werden.

::

   music_directory     "/mnt/vault/music"
   playlist_directory  "~/playlists"
   db_file         "~/database"
   log_file        "~/log"
   error_file      "~/error"
   pid_file        "~/pid"
   state_file      "~/state"
   user            "musicpd"

   bind_to_address     "localhost"

   audio_output {
       type        "oss"
       name        "OSS"
   }

Bis auf **music_directory** und **bind_to_address** können diese
Einstellungen unverändert übernommen werden.

aktivieren
~~~~~~~~~~

Zu guter Letzt muss der MPD noch gestartet werden. Dazu wird erst einmal
in die ``/etc/rc.conf`` folgendes eingetragen:

::

   musicpd_enable="YES"

Ein Reboot kann mit folgendem Kommando vermieden werden:

::

   # /usr/local/etc/rc.d/musicpd start

Verweise
--------

-  `Music Player Daemon </anwendungen/mpd>`__
-  Die **musicpd.conf(5)** Manual Page.
-  Die **musicpd(1)** Manual Page.
-  `audio/musicpd <https://www.google.com/search?q=audio/musicpd&btnI=lucky>`__
   in den FreeBSD Ports.
-  `audio/musicpd <https://www.google.com/search?q=audio/musicpd&btnI=lucky>`__
   in Pkgsrc.
-  `audio/mpd <https://www.google.com/search?q=audio/mpd&btnI=lucky>`__
   in den OpenBSD Ports.
-  http://musicpd.org die offizielle Homepage.
-  http://mpd.wikia.com/wiki/Clients eine Liste von Clients.

* :ref:`genindex`

Zuletzt geändert: |date|

