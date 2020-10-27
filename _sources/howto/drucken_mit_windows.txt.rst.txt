Drucken mit Windows
===================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Statt `Samba </howto/Samba>`__ zum Drucken über das Netzwerk zu verwenden, kann
man auch dies auch einfach unter Verwendung von IPP und CUPS.

Das Ziel lautet: **Mit einem Windows-Client auf einem CUPS-SERVER
drucken**.

Voraussetzungen
---------------

-  CUPS muß schon konfiguriert sein und man kann schon auf dem
   \*BSD/Linux-Rechner drucken. Wie man das macht, ist immer noch am
   besten auf `CUPS-Anleitung </howto/cups>`__ erklärt.
-  Den Druckertreiber für das entsprechende Windows.
-  Für Windows 95 + 98 braucht man noch das folgende Tool von Microsoft:
   `Windows
   95 <http://www.microsoft.com/Windows95/downloads/contents/WUPreviews/IPP>`__,
   `Windows
   98 <http://www.microsoft.com/Windows98/downloads/contents/WUPreviews/IPP>`__
-  Alternativ kann auch der `ShinePrint IPP
   Client <http://www.shinesoft.com/shineprint/spclient.html>`__
   verwendet werden, der unter Win9x sowie Windows NT funktioniert
-  ab Windows 2000 ist IPP schon dabei.
-  Außerdem sollte das Netzwerk schon eingerichtet sein.

Schreiten wir zur Tat
---------------------

-  ``/usr/local/etc/cups/cupsd.conf`` bearbeiten. Hier wird eingestellt,
   daß die Windows-Kisten mit der IP 192.168.123.2 und 192.168.123.3 auf
   dem CUPSSERVER alle Drucker benutzen dürfen:

::

   <Location /printers>
   Order Deny,Allow
   Deny From All
   Allow From 192.168.123.2
   Allow From 192.168.123.3
   < /Location>

-  Port 631 bei der Firewall aufmachen.
-  IPP bei Windows 98, 95 oder NT installieren
-  Drucker auf den Windows-Kisten nach der Anleitung unter
   `installieren <http://www.owlfish.com/thoughts/winipp-cups-2003-07-20.html>`__.
   Der Druckerpfad wird nach folgendem Schema gebildet:
   **http://IPdesCUPSSERVERS:631/printers/DRUCKERNAME**
-  Drucken und freuen!

* :ref:`genindex`

Zuletzt geändert: |date|

