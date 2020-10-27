Soundkarte einrichten
=====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Für die Soundkartenunterstützung ist es, entgegen der weitläufigen Meinung,
nicht notwendig den Kernel neu zu kompilieren!  Hier soll aufgezeigt werden,
wie die Soundkarte am einfachsten zum Laufen gebracht werden kann.

Technische Details
------------------

Die Soundkarte wird über das entsprechende Kernelmodul unterstützt. Bei
FreeBSD befinden sich alle Kernelmodule unter ``/boot/kernel/snd_*.ko``.
Sie können mit kldload geladen werden, mit kldstat sieht man alle
geladenen Kernelmodule und kldunload entlädt die Module wieder.

/boot/loader.conf
~~~~~~~~~~~~~~~~~

Mit den richtigen Einträgen in ``/boot/loader.conf`` wird das
Soundkartenmodul beim Booten automatisch geladen. Welche Einträge für
die korrekte Soundkartenunterstützung in ``/boot/loader.conf`` notwendig
sind, verrät Dir ``/boot/defaults/loader.conf``! Als Beispiel die
Einträge in der ``/boot/loader.conf`` für einen Intel ICH-Soundchip:

::

   sound_load="YES"
   snd_ich_load="YES"

Dieses Einfügen von Kernelmodulen beim Boot des Systems funktioniert im
Übrigen auch mit jedem anderen Modul.

Bekannte Soundkarte
~~~~~~~~~~~~~~~~~~~

Suchen Sie Ihre Soundkarte aus der FreeBSD-Hardwarekompatibilitätsliste
heraus. Hier der Link zur Hardwarekompatibilitätsliste von FreeBSD 6.2:

http://www.freebsd.org/releases/6.2R/hardware-i386.html#SOUND

Oberhalb des Eintrags Ihrer Soundkarte finden Sie den Namen des
passenden Kernelmoduls. Der Name des Kernelmoduls ist in blauer Schrift.
Notieren Sie sich den Namen des zu ihrer Soundkarte passenden
Kernelmoduls und klicken Sie auf die Schrift. Es erscheint die
Hilfeseite des Kernelmoduls. Bitte lesen Sie die Hilfeseite aufmerksam
durch und testen Sie nachher das Kernelmodul mit:

::

   # kldload <Name des Kernelmoduls>.ko
   # cat /dev/sndstat

Die Ausgabe von ``cat /dev/sndstat`` sollte ähnlich wie das Beispiel:

::

  FreeBSD Audio Driver (newpcm) Installed devices: 
  pcm0: <Intel ICH4 (82801DB)> at io 0xc0000c00, 0xc0000800 irq 11 bufsz 16384 kld 
  **snd_ich** (1p/1r/1v channels duplex default)

aussehen, ansonsten wird die Soundkarte nicht vom verwendeten
Kernelmodul unterstützt.

Unbekannte Soundkarte
~~~~~~~~~~~~~~~~~~~~~

Ich kenne meine Soundkarte nicht, was soll ich tun?

1. Variante => Alle Soundtreiber im laufenden Betrieb ausprobieren

::

   #!/bin/sh
   for module in /boot/kernel/snd_*.ko; do 
     kldload $module 
   done
   cat /dev/sndstat

2. Variante => Den Meta-Treiber snd_driver ausprobieren. Siehe dazu
bitte im
`FreeBSD-Handbuch <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/sound-setup.html>`__
nach!

.. note::

  Nicht mit ``/boot/loader.conf`` herumbasteln!  Eine falsch konfigurierte
  ``/boot/loader.conf`` kann zu einem nicht startenden FreeBSD-System führen!!
  
kein Ton
~~~~~~~~

Hilfe ich höre nichts, obwohl ``cat /dev/sndstat`` ein positive
Rückmeldung ergab.

Kontrolliere zuerst mit dem Befehl:

::

   # mixer

ob alle relevanten Lautstärkeregler stimmen. Besonders der Regler "vol",
"pcm" und "cd" dürfen nicht stummgeschalten (Anzeige: "0:0") sein. Hier
als Beispiel die mixer-Ausgabe meiner Soundkarte:

::

   Mixer vol is currently set to 25:25
   Mixer pcm is currently set to 49:49
   Mixer line is currently set to 75:75
   Mixer mic is currently set to 0:0
   Mixer cd is currently set to 75:75
   Mixer rec is currently set to 0:0
   Mixer ogain is currently set to 50:50
   Mixer line1 is currently set to 75:75
   Mixer phin is currently set to 0:0
   Mixer phout is currently set to 0:0
   Recording source: mic

Die Anzahl der Regler und die jeweilige Reglerbezeichnung variiert je
nach Soundkarte! Wie man die Lautstärkeregler-Einstellung verändern
kann, erläutert:

::

   # man mixer

Öffne nach der ersten Lautstärkeregler-Kontrolle eine csh-Shell:

::

   csh

Tippe mehrmals auf die Tabulator-Taste, erhöhe dabei die Lautstärke so lange,
bis der Beep-Ton hörbar ist.  Früher oder später geht einem das gebeepe dann
wohl doch auf den Keks so lässt es per

::

   # sysctl hw.syscons.bell=0
   # echo hw.syscons.bell=0 >> /etc/sysctl.conf

dauerhaft deaktivieren. Alternativ kann man auch

::

   keybell="visual"

in die /etc/rc.conf eintragen. Hierdurch wird der Piepton deaktiviert
und stattdessen der Bildschirm einmal ausgeleuchtet.

.. note::

  Viele Laptops besitzen eine Hardware-Lautstärkeregelung! Diese
  Hardware-Lautstärkeregelung wird nicht von mixer angezeigt. Die
  Hardware-Lautstärkeregelung kann nur mit einem Rad oder zweier Tasten am
  Laptop verstellt werden! 

Lautstärke-Einstellungen speichern
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Bleiben die Lautstärke-Einstellungen während einem Rechner-Neustart
nicht erhalten, siehe bitte:

http://www.bsdforen.de/showthread.php?t=8247

Weiterführende Literatur
------------------------

-  `FreeBSD-Handbuch <http://www.de.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/sound-setup.html>`__
-  `FreeBSD-Umsteigerhandbuch <http://www.bsdforen.de/showthread.php?t=5900>`__
-  `onboard-nforce-Soundkarte <http://www.bsdforen.de/showthread.php?p=57378>`__
-  `Wechsel von pcm zu sound incl. Installation von notwendigem
   Treiber <http://freebsd-node.de/DEVICE_PCM_to_SOUND_in_FreeBSD_Current>`__
   FIXME
-  `Audigy-Soundkarte <http://chibis.persons.gfk.ru/audigy/>`__
-  `5.1-Surround-Sound <http://wiki.bsd-crew.de/index.php/5.1-Surround-Sound_mit_FreeBSD>`__
-  `ISA-Bus-Soundkarten <http://www.bsdforen.de/showthread.php?t=11739>`__
-  `Intel 82801 <http://www.bsdforen.de/showthread.php?t=12734>`__
-  `Intel High Definition
   Audio <http://www.bsdforen.de/showthread.php?t=13247>`__

* :ref:`genindex`

Zuletzt geändert: |date|

