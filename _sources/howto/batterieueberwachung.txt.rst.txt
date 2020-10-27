Batterieüberwachung
===================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Mittels `cron
<http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/configtuning-cron.html>`__
und einem Shellskript soll bei leerer Batterie der Laptop automatisch
ausgeschaltet werden. Zwei Minuten bevor der Laptop heruntergefahren wird,
erhält der Benutzer eine Warnmeldung auf der Konsole und auf dem
X11-Bildschirm.

Voraussetzung
-------------

ACPI (Advanced Configuration and Power Interface) ist aktiviert, näheres
zu ACPI unter: `Energie- und Ressourcenverwaltung / Was ist
ACPI? <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/acpi-overview.html>`__
und ggf.:
`ACPI-Fehlersuche <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/acpi-debug.html>`__

Batteriestatus abfragen
-----------------------

Grundsätzlich gibt es 2 Möglichkeiten, um unter FreeBSD den
Batteriestatus eines Laptops abzufragen:

**Möglichkeit 1:** 

::

  $ /usr/sbin/acpiconf -i batt

Ausgabe (Beispiel):

::

   Design capacity:        4000 mAh
   Last full capacity:     4000 mAh
   Technology:     secondary (rechargeable)
   Design voltage:         14800 mV
   Capacity (warn):        420 mAh
   Capacity (low):         156 mAh
   Low/warn granularity:   264 mAh
   Warn/full granularity:  3780 mAh
   Model number:   PA3395U
   Serial number:  3658Q
   Type:           Li-Ion
   OEM info:       TOSHIBA
   State:          discharging
   Remaining capacity:     23%
   Remaining time:         unknown
   Present rate:   0 mA
   Voltage:        14334 mV

**Möglichkeit 2:** 

::

  $ /sbin/sysctl hw.acpi.battery

Ausgabe (Beispiel):

::

   hw.acpi.battery.life: 23
   hw.acpi.battery.time: 15
   hw.acpi.battery.state: 1
   hw.acpi.battery.units: 1
   hw.acpi.battery.info_expire: 5

Möglich ist hierbei auch:

::

  $ /sbin/sysctl hw.acpi.battery.life 
  $ /sbin/sysctlhw.acpi.battery.time

Ausgaben dazu (Beispiele):

::

   hw.acpi.battery.life: 23
   hw.acpi.battery.time: 15

bzw.:

::

  $ /sbin/sysctl -n hw.acpi.battery.life 
  $ /sbin/sysctl -n hw.acpi.battery.time

Ausgaben dazu (Beispiele):

::

   23
   15

Die o.g. Befehle kann man verwenden, um per Shellskript den
Batteriestatus zu überwachen.

.. warning::

  Es kann vorkommen, daß bestimmte Laptops die ``hw.acpi.battery.time``
  Information nicht liefern, deshalb ist es ratsam vor der Erstellung eines
  Überwachungsskriptes zu testen (im Batteriebetrieb!), welche Ausgaben die
  o.g. Befehle produzieren.

Shellskript
-----------

Das Shellskript ``/usr/local/sbin/batterietest`` kontrolliert die
Stromversorgung und den Batteriestand des Laptops. Ist die Batterie
leer, löst das Shellskript das Herunterfahren des Laptops aus.

``/usr/local/sbin/batterietest``

.. code:: bash

   #!/bin/csh
   # /usr/local/sbin/batterietest
   # Akkustand kontrollieren

   set restminuten=`/sbin/sysctl -n hw.acpi.battery.time`
   set modus=`/sbin/sysctl -n hw.acpi.battery.state`

   # Wird der Laptop nur von der Batterie mit Strom versorgt?
   if ( "${modus}" == "1" ) then

     # Reicht die Batterie nicht mehr fuer 7 Minuten?
     if ( "${restminuten}" < "7" ) then
       shutdown -p +2 "Die Batterie ist leer! Der Computer wird in 2 Minuten heruntergefahren!" &
     endif

   endif

Eine Alternative (falls das o.g. Skript nicht funktionieren sollte, weil
z.B. \`/sbin/sysctl -n hw.acpi.battery.time\` keine/falsche Ergebnisse
liefert):

.. code:: bash

   #!/bin/csh
   # /usr/local/sbin/batterietest
   # Akkustand kontrollieren

   set restprozent=`/sbin/sysctl -n hw.acpi.battery.life`
   set modus=`/sbin/sysctl -n hw.acpi.battery.state`

   # Wird der Laptop nur von der Batterie mit Strom versorgt?
   if ( "${modus}" == "1" ) then

     # Batteriestand unter 15%?
     if ( "${restprozent}" < "15" ) then
       shutdown -p +2 "Die Batterie ist leer! Der Computer wird in 2 Minuten heruntergefahren!" &
     endif

   endif

Nicht vergessen: Das Shellskript ausführbar machen mit:

::

  # chown root /usr/local/sbin/batterietest 
  # chmod u+x /usr/local/sbin/batterietest

root-Crontab
------------

Jetzt benötigt man noch einen Eintrag in der root-Crontab, damit alle 3
Minuten das Shellskript ``/usr/local/sbin/batterietest`` aufgerufen
wird:

::

  # su # crontab -e

Jetzt folgende Zeilen eingeben:

.. code:: bash

   # root-crontab
   #
   SHELL=/bin/sh
   PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin
   #
   #minute hour mday month wday command
   #
   # Batteriestand alle 3 Minuten kontrollieren
   */3 * * * * /usr/local/sbin/batterietest

die neuen Zeilen speichern und den Editor verlassen.

* :ref:`genindex`

Zuletzt geändert: |date|

