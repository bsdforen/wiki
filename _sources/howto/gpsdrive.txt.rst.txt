GPSDrive
========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Hier erfahrt Ihr, wie Ihr
`GPSdrive <http://gpsdrive.kraftvoll.at/index.shtml>`__ mit einem
Bluetooth-Gerät benutzen könnt. Ihr müsst vorher natürlich Bluetooth
installiert und Konfiguriert haben. Das ist
`hier <http://wiki.bsdforen.de/Bluetooth>`__ perfekt beschrieben. Als
GPS Gerät benutze ich ein GPSlim 236 von HOLUX.

Installation
------------

Als nächstes müssen wir GPSdrive aus den Ports installieren::

  % cd /usr/ports/astro/gpsdrive/ && make install clean

Serielle Verbindung aufbauen
----------------------------

Damit wir GPSdrive benutzen können, müssen wir uns eine Serielle
Schnittstelle schaffen.

::

  % rfcomm_sppd -a GPS_NAME -t /dev/ttyp7

GPS_NAME ist der Name von dem Bluetooth Gerät, welches ihr in der
**/etc/bluetooth/hosts** gesetzt habt. Ihr müsst auch nicht ttyp7 benutzen, das
sollte aber meistens frei sein.

gpsd Starten
------------

Jetzt müssen wir ``gpsd`` noch sagen wo er das GPS Gerät finden kann und
starten es so::

  % gpsd -p /dev/ttyp7

GPSDrive Starten
----------------

Was soll ich sagen, einfach starten::

  % gpsdrive

Tipps
-----

gpsdrive bringt nur eine kleine abgespeckte Version von gpsd mit und
beinhaltet nicht das Programm xgps, zum Beobachten der Satelliten . Am
besten installiert man sich noch gpsd aus den Ports::

  % cd /usr/ports/astro/gpsd/ && make install clean

* :ref:`genindex`

Zuletzt geändert: |date|

