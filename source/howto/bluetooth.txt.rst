Bluetooth
=========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dies ist eine kurze Anleitung, wie man Bluetooth ab FreeBSD
5.3 zum Laufen bringt. Es wurde hierzu ein "Mitsumi Bluetooth
WML-C52APR"-Adapter benutzt, welcher vom ng_ubt-Treiber unterstützt wird. Die
erste wichtige Anlaufstelle ist `FreeBSD-Handbuch - Bluetooth
<http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/network-bluetooth.html>`__.

Den richtigen Treiber laden
---------------------------

::

  # kldload ng_ubt 

oder in ``/boot/loader.conf`` folgende Zeile hinzufügen:

::

   ng_ubt_load="YES"

Bluetooth-Stack starten
-----------------------

::

  # sudo cp /usr/share/examples/netgraph/bluetooth/rc.bluetooth /etc
  # sudo chmod 770 /etc/rc.bluetooth
  # /etc/rc.bluetooth start ubt0 (startet Stack)
  # /etc/rc.bluetooth stop ubt0 (stopt Stack)

Welche Geräte sind in der Nähe?
-------------------------------

::

  # hccontrol -n ubt0hci inquiry 
  Inquiry result, num_responses=1
  Inquiry result #0
       __BD_ADDR__: trebroN
       Page Scan Rep. Mode: 0x1
       Page Scan Period Mode: 00
       Page Scan Mode: 00
       Class: 52:02:04
       Clock offset: 0xd4a

BD_ADDR in Klarschrift
----------------------

trebroN steht unter 3 in der BD_ADDR nur, weil in
``/etc/bluetooth/hosts`` der Eintrag:

::

   00:0e:04:85:16:0c trebroN

vorgenommen wurde

Ping zum Host "trebroN"
-----------------------

::

  # l2ping -a trebroN

Datenübertragung
----------------

Installieren der notwendigen Anwendung
`comms/obexapp <https://www.google.com/search?q=comms/obexapp&btnI=lucky>`__:

::

  # cd /usr/ports/comms/obexapp 
  # sudo make install clean
  # obexapp -a trebroN -C OPUSH 

baut Verbindung zum Telefon auf.

Befehle: CApability, CD, DElete, DIsconnect, Empty, Get, Ls, Mkdir, Put?
(Extrem rudimentäre Applikation zum Übertragen von Daten; vgl.
commandline von ftp.)

Zum Übertragen von Daten vom Telefon auf den Rechner habe ich ein
kleines Ruby-Skript geschrieben. Dieses "automatisiert" das Übertragen
von Bildern via obexapp; damit wird die Anwendung "ein bißchen"
angenehmer. Dokumentation befindet sich
`hier <http://foobla.wigbels.de/2005/08/17/freebsd-fotos-vom-handy-via-bluetooth-ubertragen/>`__.

* :ref:`genindex`

Zuletzt geändert: |date|

