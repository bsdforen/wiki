TV-Karte benutzen
=================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieses Howto beschreibt die Benutzung einer Brooktree kompatiblen TV-Karte
unter FreeBSD.

When you have a Philips SAA7134 chipset instead, you might want to take
a look at `download.purpe.com <http://download.purpe.com/>`__, a guy
from asia has written a driver for FreeBSD 5.

Anleitung
---------

Entweder man fügt folgende Zeilen der Kernelconfig hinzu (siehe
bktr(4)):

::

   device bktr
   device iicbus
   device iicbb
   device smbus

Oder man lädt das bktr-Modul mittels ``kldload``. Zum automatischen
Laden beim Startup siehe ``loader.conf``.

TV-Programme
------------

Viewer (zu finden unter
`multimedia <https://www.google.com/search?q=multimedia&btnI=lucky>`__):

-  `multimedia/xawtv <https://www.google.com/search?q=multimedia/xawtv&btnI=lucky>`__
   http://linux.bytesex.org/xawtv
-  `multimedia/fxtv <https://www.google.com/search?q=multimedia/fxtv&btnI=lucky>`__
   http://people.freebsd.org/~rhh/fxtv
-  `multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
   http://www.mplayerhq.hu

EPG (ersetzt die Fernsehzeitung ):

-  `multimedia/nxtvepg <https://www.google.com/search?q=multimedia/nxtvepg&btnI=lucky>`__
   http://nxtvepg.sourceforge.net

Teletext:

-  `misc/vbidecode <https://www.google.com/search?q=misc/vbidecode&btnI=lucky>`__

Zu mplayer
----------

Mit mplayer TV sehen ist zwar nicht ganz einfach, aber der Aufwand lohnt
sich wenn man die Qualität sieht.

::

   # mplayer -tv driver=bsdbt848:device=/dev/bktr0:input=1:channels= tv://

Sieht kompliziert aus, aber so kompliziert ist es nun auch wieder nicht:

Hinter chanlist folgt eine Liste (mit Kommata getrennt) von den
TV-Sendern und deren Namen bsp.: E10-ARD,SE11-VOX oder man nimmt die
vorgefertigte Liste chanlist=europe-west

Sender(Deutschland):

::

   E10-ARD
   E8-ZDF
   E4-RTL2
   SE20-pro7
   SE11-VOX
   E9-SAT1
   E11-RTL
   SE6-Kabel1
   SE13-MTV
   S22-VIVA
   S25-SUPERRTL
   SE4-KIKA
   SE14-N24
   SE18-EuroSport
   S23-DSF
   SE5-ntv
   E7-BBC
   SE12-CNN

Standardmäßig kann man mit den Tasten "k" und "h" durch die Kanäle
navigieren.

Man kann auch andere Tasten belegen, indem man eine Datei anlegt, hier
``~/.mplayer/tvkeys`` mit dem folgenden Inhalt.

::

   LEFT tv_step_channel -1
   RIGHT tv_step_channel +1
   UP tv_step_channel +1
   DOWN tv_step_channel -1

Diese Datei gibt man an mplayer mit ``-input conf=tvkeys`` weiter. So
hat man hier zum Beispiel die Cursortasten zum Sendernavigieren belegt.

Wenn Du kein Bild siehst, spiele mit ``input`` herum (meistens ist es 0
oder 1).

Wenn du keinen Sound hast:

#. Überprüfe mittels ``mixer(8)`` die Lautstärke.
#. Füge ``audioid`` hinzu (auch damit musst du herum spielen, bei mir
   ist es 2)

Mit `mplayer </anwendungen/mplayer>`__ und
`mencoder </anwendungen/mplayer#mencoder>`__ aufnehmen:

::

   # mencoder -tv driver=bsdbt848:device=/dev/bktr0:input=1:chanlist=europe-west:width=768:height=576 -ovc lavc -lavcopts vcodec=mpeg4:vbitrate=900 -oac mp3lame -lameopts cbr:br=64 -vop pp=lb,crop=720:544:24:16 -o output.avi tv://

Dies nimmt den aktuellen Sender im MPEG4-Format auf. ``vbitrate`` ändert
die Qualität des Videos (in Bits pro Sekunde) und ``br`` die Qualität
des Sounds (mp3 in kBit pro Sekunde).

Um nun einen bestimmten Sender aufzunehmen, einfach anstelle
``chanlist=europe-west``

::

   channel=E6-ARD

Wenn du nun noch ``-endpos 02:00:00`` hinzufügst, nimmt er exakt zwei
Stunden lang auf. Um einen vollständigen Videorekorder zu ersetzen,
starte den Befehl zur Aufnahme im crontab oder besser mit ``at(1)`` zu
einer bestimmten Zeit.

Wenn du keinen Sound hast: Überprüfe ob das recording device (siehe
mixer) und die Lautstärke stimmt. Wenn du, um in mplayer etwas zu hören,
audioid benötigst, füge dieses einfach in mencoder mit an.

Ich hoffe ein paar Leuten damit geholfen zu haben (und danke an lamer!).

Hauppauge Karte mit Fernbedienung
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  "urmon", siehe
   http://www.monkey.org/freebsd/archive/freebsd-multimedia/200305/msg00039.html

Troubleshooting
---------------

Sollten bei der Konfiguration Probleme auftreten (z.b. kein Sound),
solltest Du mit dmesg überprüfen, ob die TV-Karte überhaupt richtig
erkannt wird. Ist das nicht der Fall, kannst Du in
``/usr/src/sys/dev/bktr/bktr_card.h`` nachschauen, ob die Karte dort
aufgelistet ist. Wenn ja, wird sie mit
``sysctl hw.bt848.card=<kartennummer>`` festgelegt. Sollte dies zum
Erfolg führen, trägst du den Wert in ``/etc/sysctl.conf ein``, damit die
Karte auch nach dem nächsten Booten noch funktioniert. Ist die Karte
allerdings nicht in ``bktr_card.h`` zu finden, bleiben Dir noch die
sysctls

-  hw.bt848.card
-  hw.bt848.tuner
-  hw.bt848.reverse_mute
-  hw.bt848.format
-  hw.bt848.slow_msp_audio

Bei Karten mit dem Brooktree-878-Chip, heißen diese vermutlich
entsprechend. Details zu den einzelnen sysctls befinden sich in
``bktr(4)``.

* :ref:`genindex`

Zuletzt geändert: |date|

