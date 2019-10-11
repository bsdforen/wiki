DVB unter FreeBSD
=================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel zeigt kurz und beispielhaft, wie man USB-Geräte für den Empfang
von digitalem Rundfunk unter FreeBSD nutzt.  Eingegangen wird auf die in
Deutschland verbreiteten Verfahren DVB-S2 (Satellit), DVB-C (Kabel) und DVB-T
(terrestrischer Rundfunk).

.. note::

  Derzeit deckt dieser Artikel nicht alle möglichen Programme zum
  Rundfunkempfang ab, es wird daher um Erweiterung gebeten.

Voraussetzungen
---------------

Voraussetzung ist erst einmal ein von ``webcamd`` unterstützter
USB-Stick zum Empfang des jeweiligen digitalen Rundfunksystems. Des
weiteren wird ein Computer benötigt, der die gewünschte Bildqualität
dekodieren kann. Bei DVB-S2 und DVB-C ist eine Ausstrahlung von bis zu
1080p Full-HD möglich, dies benötigt einen schnellen Prozessor oder aber
eine Grafikkarte mit VDPAU-Unterstützung. DVB-T ist derzeit auf 720p
begrenzt, hier sollte eine Dualcore-CPU der Mittelklasse ausreichen.

Auf der Softwareseite wird FreeBSD 8.1 oder höher verlangt, getestet
sind derzeit nur FreeBSD/i386 und FreeBSD/amd64. Des Weiteren muss der
Portstree in ``/usr/ports`` aktuell sein. Dieser Artikel geht davon aus,
dass ein funktionsfähiges X.org installiert ist.

Installation
------------

cuse4bsd
~~~~~~~~

`multimedia/cuse4bsd <https://www.google.com/search?q=multimedia/cuse4bsd&btnI=lucky>`__
ist ein Kernelmodul, welches es im Userland laufenden Programmen erlaubt
Devicenodes in ``/dev`` anzulegen. Diese Funktion wird später von
``webcamd`` genutzt. Die Installation erfolgt einfach über die Ports:

::

    # cd /usr/ports/multimedia/cuse4bsd-kmod/ 
    # make install clean

Anschließend wird das Modul geladen:

::

    # kldload cuse4bsd

Es fehlt noch ein Eintrag in ``/boot/loader.conf`` zum automatischen
Laden beim Start des Betriebssystems:

::

    # echo 'cuse4bsd_load="YES"' >> /boot/loader.conf

webcamd
~~~~~~~

`multimedia/webcamd <https://www.google.com/search?q=multimedia/webcamd&btnI=lucky>`__
ist Daemon - also ein im Hintergrund laufender Dienst - welcher es
ermöglicht für "Video4Linux" geschriebene Treiber im Userland
auszuführen. Dieser Dienst ist der Kern eines DVB-Setups unter FreeBSD.
Die Installation geschieht auch hier über die Ports:

::

    # cd /usr/ports/multimedia/webcamd
    # make install clean

Anschließend wird ein Eintrag in der ``/etc/rc.conf`` zum automatischen
Start von ``webcamd`` vorgenommen:

::

    # echo 'webcamd_enable="YES" >> /etc/rc.conf

Abschließend wird der Dienst ``devd`` zum Verarbeiten von
Systemereignissen neu gestartet:

::

    # service devd restart

devfs
~~~~~

Normalerweise werden die Geräte in ``/dev`` mit den Rechten ``600``
angelegt. Dies bedeutet, dass nur ``root`` auf sie zugreifen kann. Für
DVB ist dies unpraktisch bis problematisch, da das zugehörige Programm
in der Regel mit Rechten als unprivilegierter Nutzer läuft. Eine
Möglichkeit sind Einträge in den aktuellen Regelsatz unter
``/etc/devfs.rules`` zum automatischen Setzen erweiterter
Berechtigungen:

::

   add path dvb/adapter*/demux* mode 0660
   add path dvb/adapter*/frontend* mode 0660
   add path dvb/adapter*/dvr* mode 0660

Anpassung des Benutzers
~~~~~~~~~~~~~~~~~~~~~~~

Der Benutzer, der das DVB-Signal nutzen möchte, muss Mitglied der Gruppe
``'webcamd``' sein. Daher wird er in diese eingetragen. Anschließend
sollte er sich aus- und wieder einloggen um sicherzustellen, dass die
Änderung übernommen wurde:

::

   pw groupmod -m username

Funktionstest
~~~~~~~~~~~~~

Wird der USB-Stick nun eingesteckt, sollten in ``/dev/dvb`` ein
Unterverzeichnis ``adapter0`` mit drei Device-Nodes erscheinen. Ist dies
nicht der Fall, kann es drei Gründe haben.

#. Der Stick ist derzeit nicht unterstützt. Hier sei auf das
   BSDForen.de-Forum und die freebsd-multimedia@ Liste verwiesen.
#. Der Stick benötigt eine Firmware. Aus Lizenzgründen liefert
   ``webcamd`` die Firmware nicht mit, nennt aber in seiner Ausgabe (sie
   ist in ``/var/log/messages`` zu finden) ihren Namen. Diese Datei kann
   mit einer Suchmaschine gefunden, heruntergeladen und nach
   ``/boot/modules`` kopiert werden. Alternativ kann man das Perl-Script
   `get_dvb_firmware <http://www.kernel.org/doc/Documentation/dvb/get_dvb_firmware>`__
   runterladen und verwenden. Anschließend den Stick erneut einstecken.
#. Der Stick identifiziert sich nicht als DVB-Gerät. Gerade billigere
   Modelle haben oftmals falsch gefüllte Register zur Erkennung der
   Geräteklasse. Hier hilft nur webcamd als root manuell zu starten:
   <code> # webcamd </code>

Sendersuchlauf
~~~~~~~~~~~~~~

Ist der Stick erkannt worden, ist der nächste Schritt die Durchführung
eines Sendersuchlaufes. Hierbei ist zu beachten, dass bei DVB-T Sticks
die beigelegten Antennen zumeist von eher niedriger Qualität sind. Für
beste Ergebnisse sollte sie während des Suchlaufs direkt vor der Scheibe
eines Fensters platziert, oder aber eine aktiv verstärkte Antenne
genutzt werden.

Es gibt mehrere Scanprogramme. An dieser Stelle wird das Tool ``w_scan``
genutzt. In den Ports kann es unter
`multimedia/w_scan <https://www.google.com/search?q=multimedia/w_scan&btnI=lucky>`__
gefunden werden und besitzt keinerlei weitere Abhängigkeiten. Es kann
Senderlisten in verschiedenen Formaten generieren, zudem unterstützt es
alle derzeit im Einsatz befindlichen DVB-Formate.

Die erzeugte Ausgabe ist die ``channels.conf``, eine Liste aller Sender
mit ihren Parametern. Diese kann in verschiedener Form erzeugt werden,
wobei die Form mittels einer Option für ``w_scan`` festgelegt wird. Die
folgende Tabelle schlüsselt auf, welcher Parameter für welches Prorgramm
genutzt werden muss. Hierbei ist beachten, dass Kaffein eine integrierte
Scanfunktion besitzt.

============ =============================
Option       Programme
============ =============================
Keine Option VDR, MythTV
-k           kaffein
-G           Gstreamer dvbsrc plugin
-M           mplayer
-X           mencoder, xine / libxine, zap
============ =============================

Des Weiteren müssen, je nach verwendetem DVB-System, eventuell noch
weitere Parameter angegeben werden. Dies ist zum Beispiel der zu
nutzende Satellit bei DVB-S2. Der Aufruf mittels

::

    # w_scan -h

bietet eine Liste aller Optionen. Grundsätzlich sollten Optionen nur
dann angegeben werden, wenn sie auch benötigt werden! Unnütze Optionen
können zu Problemen führen.

Ein Aufruf per

::

    # w_scan [optionen] >> channels.conf

führt den Suchlauf durch. Je nach DVB-System, Anzahl der Sender und dem
Wohnort kann dieser zwischen 10 und etwa 60 Minuten dauern. Anschließend
wurde eine ``channels.conf`` generiert. Es empfiehlt sich nun mit einem
Editor die Sendernamen manuell anzupassen, da die automatisch
generierten Namen recht umständlich sind.

Abspielprogramme
~~~~~~~~~~~~~~~~

mplayer / mencoder
^^^^^^^^^^^^^^^^^^

`multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
ist auch als das "Schweizer Taschenmesser in Sachen Video" bekannt und
kann als solches natürlich auch DVB abspielen. Dabei sind zuerst zwei
Voraussetzungen zu erfüllen:

#. mplayer und mencoder müssen mit Unterstützung für "v4l" gebaut sein.
#. Es muss eine kompatible ``channels.conf`` in ``~/.mplayer`` liegen.

``mplayer`` kann dann einfach per

::

    # mplayer dvb://sendername

aufgerufen werden. Da viele aber nicht alle DVB-Sendungen interlaced
sind, empfiehlt sich die Nutzung eines Deinterlace-Filters. Nutzer von
vdpau drücken hierfür während der Wiedergabe "D", alle anderen können
z.B. per

::

    -vd yadif

einen Softwarefilter einfügen und diesen per "D" wahlweise aktivieren
oder deaktivieren. Da DVB-Streams eine gewisse Neigung zu Rucklern haben
ist es zudem ratsam einen Cache zu nutzen:

::

    -cache 2048

Während der laufenden Sendung können die Kanäle durch die Tasten "h" und
"k" durchgeschaltet werden. Allerdings dauert der Kanalwechsel recht
lange. Eventuelle, alternative Tonspuren werden per "#" ausgewählt.

Es können auch Sendungen aufgenommen werden. Hierbei gibt es zwei
Möglichkeiten. Die erste ist ein "dummer" Dump des Streams durch
mplayer. Der große Vorteil dieser Methode ist, dass praktisch keine
CPU-Last generiert wird. Auch sehr langsame Maschinen, die zur
Wiedergabe zu langsam sind, können damit aufnehmen. Nachteilig ist der
vergleichsweise große Platzbedarf von etwa 10 Megabyte pro Minute bei
einer Sendung in 720p. Folgendes Kommando nimmt zum Beispiel die ARD für
zwei Stunden auf und schreibt die Ausgabe nach ``./film.ts``.

::

    # mplayer -dumpfile film.ts -dumpstream -endpos 2:00:00 dvb://ard

Alternativ ist eine direkte Umwandlung in ein anderes Dateiformat mit
`multimedia/mencoder <https://www.google.com/search?q=multimedia/mencoder&btnI=lucky>`__
möglich. Diese Methode erzeugt kleine Dateien, ist aber recht aufwendig.
Ein 720p Stream per x264 codiert lastet einen Phenom II 940 mit 4x 3GHz
z.B. zu ~60% aus. Folgendes Kommando nimmt die ARD für 2 Stunden auf und
speichert die Aufnahme als H.264-Stream nach ``./film.avi``.

::

    #  mencoder -cache 2048 -oac mp3lame -ovc x264 -lameopts cbr:br=192 \
       x264encopts bitrate=700:preset=slow:threads=4 -vf yadif -endpos \
       02:00:00 -really-quiet -o film.avi dvb://ard

* :ref:`genindex`

Zuletzt geändert: |date|

