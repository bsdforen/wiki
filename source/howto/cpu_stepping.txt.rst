CPU Stepping (cpufreq, powerd)
==============================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel erklärt wie ein `FreeBSD </FreeBSD>`__ Rechner konfiguriert wird
um den CPU Takt je nach Last anzupassen.

Einleitung
----------

CPU Stepping ist der Vorgang den Takt einer CPU dynamisch an den Bedarf
anzupassen. Dafür gibt es verschiedene Techniken, je nach Baureihe und
Hersteller der CPU. In der Vergangenheit gab es dafür unter FreeBSD
entweder keine oder nur eine proprietäre Unterstützung, wie z.B. est mit
estctrl für Intel Enhanced SpeedStep, das zum Zeitpunkt dieses Artikels
nur auf Pentium-M Systemen zum Einsatz kommt.

Inzwischen gibt es aber das Kernel Modul cpufreq, das die
Implementierungen für mehrere CPUs zusammenfasst und ein einheitliches
Kernel Interface, sowie sysctl Variablen zur Steuerung der
CPU-Geschwindigkeit anbietet. Darauf setzt powerd, das inzwischen auch
in Releng_5 enthalten ist auf. Powerd steuert die Geschwindgkeit der CPU
lastbedingt über das sysctl Interface.

cpufreq
-------

Cpufreq unterstützt die Steuerung verschiedener CPU-Typen auf x86 und
x64 Plattformen. Welche Stepping-Technik ihre CPU unterstützt entnimmt
man am Besten einem Datenblatt oder der Homepage des Herstellers. Ob
diese Technik von cpufreq unterstützt wird erfährt man am einfachsten
auf der Manpage **cpufreq(4)**.

Die CPU-Geschwindigkeit kann mit dem Befehl:

::

   # sysctl dev.cpu.0.freq=600

manuell eingestellt werden. Die Geschqwindigkeit muss jedoch unter
**dev.cpu.0.freq_levels** gelistet sein. Falls mehrere CPUs oder
CPU-Kerne im Rechner vorhanden sind, können diese mit dev.cpu.0 bis
dev.cpu.(n-1) angesprochen werden.

In manchen Fällen ist es sinnvoll eine Mindestgeschwindigkeit anzugeben.
Der Autor hat Stromverbrauchsmessungen an einem Notebook mit einem
Pemtium-M 1300 durchgeführt, die darauf schließen lassen, dass der
Stromverbrauch sich von 75 bis 600MHz nicht verändert. Auch wirkt das
System recht träge wenn es immer wieder so weit heruntergetaktet wird.
Um dem Rechnung zu tragen kann in der **/etc/sysctl.conf** folgender
Eintrag gemacht werden:

::

   debug.cpufreq.lowest=600

Im Allgemeinen wird für Intel-Prozessoren empfohlen, die minimale
Taktrate auf die Hälfte der Maximalgeschwindigkeit zu setzen, darunter
gibt es meistens keine Energieersparnis.

Bei einigen Rechnern scheint es nötig zu sein das cpufreq Modul zu
laden. Bei anderen funktioniert es einfach so, obwohl es nicht einmal
von kldstat gelistet wird. Falls der Verdacht besteht, dass die unten
beschriebene Einrichtung von powerd keinen Effekt hat, sollte in der
Datei **/boot/loader.conf**:

::

   cpufreq_load="YES"

eingetragen werden. Um das Modul ohne Neustart zu laden, genügt die
Eingabe von:

::

   # kldload cpufreq

powerd
------

Powerd nimmt einem die lästige Arbeit ab die CPU Geschwindigkeit selbst
anzupassen. Dazu ermittelt es 2 mal pro Sekunde die Last des Systems.
Aktiviert wird powerd mit dem Eintrag:

::

   powerd_enable="YES"

in der Datei **/etc/rc.conf**. Zusätzlich reagiert powerd auf diverse
Parameter die ebenfalls in der **/etc/rc.conf** eingetragen werden. Eine
nützliche Einstellung für Notebook Besitzer wäre z.B.:

::

   powerd_flags="-a max -b adaptive"

In diesem Fall läuft die CPU immer auf voller Leistung wenn das Notebook
an einem Netzteil betrieben wird. Sonst wird die Taktfrequenz dynamisch
angepasst. Eine Einstellung die in jedem Fall auf Energieverbrauch
optimiert (und somit auch bei Betrieb am Netzteil weniger Wärme
produziert), könnte so aussehen:

::

   powerd_flags="-a adaptive -b min"

Um powerd ohne Neustart zu aktivieren sollte der Befehl:

::

   # /etc/rc.d/powerd start

ausgeführt werden. Eine Beschreibung der Parameter ist in der Manpage
**powerd(8)**.

Temperatur
----------

Bei Notebooks kann in der Regel direkt über ACPI auf die
Temperatursensoren zugegriffen werden:

::

   > sysctl hw.acpi.thermal | grep temperature
   hw.acpi.thermal.tz0.temperature: 43.0C
   hw.acpi.thermal.tz1.temperature: 29.2C
   hw.acpi.thermal.tz2.temperature: 0.0C
   hw.acpi.thermal.tz3.temperature: 45.0C
   hw.acpi.thermal.tz4.temperature: 44.0C

Leider funktioniert das nicht mit jedem Mainboard und BIOS so gut.
Deshalb gibt es auch die Möglichkeit, zumindest bei einigen CPUs die
Temperatur direkt abzufragen. Dazu muss ein weiteres Kernelmodul geladen
werden. Das Modul heißt ``coretemp`` für Intel Core\* CPUs und
``amdtemp`` für AMD CPUs. Die Temperaturwerte finden sich dann direkt
bei den CPU Daten.

::

   # kldload coretemp
   # sysctl dev.cpu | grep temperature
   dev.cpu.0.temperature: 45.0C
   dev.cpu.1.temperature: 46.0C

Um das entsprechende Modul bei jedem Start automatisch zu laden sollte
es in der Datei ``/boot/loader.conf`` eingetragen werden.

::

   # Intel CPU temperature reporting
   coretemp_load="YES"

   # AMD CPU temperature reporting
   amdtemp_load="YES"

Natürlich wird nur einer dieser Einträge benötigt.

Verweise
--------

-  Die cpufreq Manpage **cpufreq(4)**.
-  Die powerd Manpage **powerd(8)**.
-  Die coretemp Manpage **coretemp(4)**.
-  Die amdtemp Manpage **amdtemp(4)**.

* :ref:`genindex`

Zuletzt geändert: |date|

