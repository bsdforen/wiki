Systemüberwachung
=================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Das System läßt sich vielfältig überwachen. Eine praktische Variante
ist, die gesammelten Informationen grafisch darzustellen. Dadurch läßt
sich ein guter Überblick gewinnen und es sieht auch noch gut aus. Es
gibt diverse Programme, die diese Aufgabe erfüllen und weit mehr
Funktionalität bieten, als hier erwähnt. Für ein paar Infos reichen aber
durchaus einige Shellskripte aus. Folgende Informationen sollen hier
erfaßt und grafisch dargestellt werden.

-  CPU Temperatur
-  Festplatten Temperatur
-  Netzwerkauslastung

Das Ganze wird mit Hilfe von Shellskripten periodisch ermittelt und dann
mit RRDTool aufbereitet und grafisch dargestellt. RRDTool findet sich in
den Ports unter *net/rrdtool*.

RRDTool ist für solche Aufgaben hervorragend geeignet, da es die Daten
in einer Round Robin Datenbank ablegt. Dies bedeutet, es wird je
Zeitschlitz nur eine festgelegte Anzahl an Werten gespeichert.
Zusätzlich werden dann die Werte über einen größeren Zeitraum
konsolidiert und in den nächsten Zeitschlitz geschrieben. Durch dieses
Verfahren ist zum einem die Größe der Datenbank festgelegt und ändert
sich auch nicht, zum anderen können so die Daten über einen großen
Zeitraum gespeichert werden.

Anlegen der RRDTool Datenbank
-----------------------------

Als erstes muß die Datenbank angelegt werden. Dazu muß angegeben werden,
wie die Daten organisiert und welche Konsolidierungsfunktionen verwendet
werden sollen.

Datenorganistion
----------------

RRDTool muß wissen, wieviele Zeitschlitze vorhanden sein und wann diese
Daten konsolidiert werden sollen.

Jede Datenbank enthält mindestens eine Data Source (DS), die angibt von
welchem Typ die erhobenen Daten sind.

-  GAUGE, z.B. Temperatur Daten
-  COUNTER, z.B. Bytes/s Netzwerkdurchsatz
-  ABSOLUT Werte, die nach dem Lesen zurückgesetzt werden oder schnell
   zu einem Überlauf führen
-  DERIVE z.B. um den Anteil der Leute festzustellen, die einen Raum
   betreten/verlassen

(Die Beispiele sind aus der man-page.)

Dann enthält jede Datenbank noch verschiedene Round Robin Archive (RRA),
die festlegen, wieviele Daten für den jeweiligen Zeitschlitz
konsolidiert werden sollen.

Temperatur Datenbank erstellen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Um die Temperatur zu messen, eignet sich eine Data Source vom Typ GAUGE
sehr gut, da hier keine Durchschnittswerte für das jeweilige
Zeitintervall gebildet werden. Nun muß noch festgelegt werden, in
welchem Intervall die Daten erhoben und konsolidiert werden. Am besten
läßt sich dies an realen Werten nachvollziehen.

Für 1 Woche (7 Tage) sollen die Daten in 5-Minute Intervallen vorliegen,
d.h es müssen 2016 Werte gespeichert werden.

::

   2016 Werte / 7 Tage = 288 Werte pro Tag
   288 Werte pro Tag / 24 Stunden = 12 Werte pro Stunde
   12 Werte pro Stunde = alle 5 Minuten 1 Wert

Für dieses Archiv brauchen wir alle 5 Minuten 1 Wert.

Da damit aber die Daten für maximal eine Woche gespeichert werden und
das etwas wenig ist, sollen für 38 Tage (1 Monat und 1 Woche) die Daten
konsolidiert und in 30 Minuten Intervallen gespeichert werden. Dafür
werden 1824 Werte benötigt.

::

   1824 Werte / 38 Tage = 48 Werte pro Tag
   48 Werte pro Tag / 24 Stunden = 2 Werte pro Stunde (alle 30 Minuten 1 Wert)

Nur mit diesen beiden Archiven würden die Daten für ca. 1 Monat und 1
Woche vorliegen.

Entsprechend setzt sich die Berechnung fort. Im Überblick sieht das
Ganze dann so aus:

========== ============ ==================================
Intervall  Anzahl Werte Dauer
========== ============ ==================================
5 Minuten  2016         7 Tage
30 Minuten 1824         31 Tage (+ 7 Tage)
1 Stunde   1656         31 Tage (+ 31 Tage + 7 Tage)
2 Stunden  1176         60 Tage (+ 31 Tage + 7 Tage)
1 Tag      830          732 (+ 60 Tage + 31 Tage + 7 Tage)
========== ============ ==================================

Ob diese Einteilung sinnvoll ist oder nicht, ist ein anderes Thema.
Sicherlich könnte man zwischen den Werten für einem Monat und 2 Jahren
die Daten noch monatsweise konsolidieren und das Archive mit den
Durchschnittswerten im 2 Stunden Intervall entfernen.

Per Default ist der zeiltliche Abstand zwischen 2 Werten 300 Sekunden,
d.h. die Daten werden so konsolidiert, als wären sie alle 5 Minuten
eingefügt worden.

Die Datenbank wird nun mit folgendem Befehl angelegt:

::

   rrdtool create temperature.rrd                                       \
     DS:cpu_temp:GAUGE:600:0:U                                          \
     DS:hdd_temp:GAUGE:600:0:U                                          \
     RRA:AVERAGE:0.5:1:2016                                             \
     RRA:AVERAGE:0.5:6:1824                                             \
     RRA:AVERAGE:0.5:12:1656                                            \
     RRA:AVERAGE:0.5:24:1176                                            \
     RRA:AVERAGE:0.5:288:830                                            \
     RRA:MIN:0.5:1:2016                                                 \
     RRA:MIN:0.5:6:1824                                                 \
     RRA:MIN:0.5:12:1656                                                \
     RRA:MIN:0.5:24:1176                                                \
     RRA:MIN:0.5:288:830                                                \
     RRA:MAX:0.5:1:2016                                                 \
     RRA:MAX:0.5:6:1824                                                 \
     RRA:MAX:0.5:12:1656                                                \
     RRA:MAX:0.5:24:1176                                                \
     RRA:MAX:0.5:288:830

Noch ein paar Informationen zu den Parametern:

-  **DS** gibt die Data Source an

   -  der Name kann frei gewählt werden
   -  GAUGE ist der Typ
   -  600 gibt an, daß maximal 2 Werte (10 Minuten = 600 Sekunden) kein
      Wert geschrieben werden darf, bevor ein Wert von UNKNOWN
      angenommen wird
   -  0 und *U* sind Minimum bzw. Maximum Werte; *U* steht hier für
      UNKNOWN und entspricht keiner Obergrenze

Nach den Data Sources kommen die Archive mit den
Konsolidierungsfunktionen, denen die oben erklärten Berechnungen zu
Grunde liegen.

-  **RRA** gibt das Round Robin Archive an

   -  AVERAGE ist die Konsolidierungsfunktion
   -  1 gibt an, wieviele Werte zur Konsolidierung benötigt werden
   -  2016 gibt an wieviele Werte gespeichert werden

In diesem Beispiel werden sowohl Mittelwert, Minimun und Maximum Werte
gespeichert. Daneben gibt es noch die "Konsolidierungsfunktion" LAST,
die den letzten Wert angibt. Diese "Funktion" steht aber immer zur
Verfügung und muß nicht angegeben werden.

Über den Data Source Namen kann später auf die gespeicherten
Konsolidierungsfunktionen zugegriffen werden, um die Werte zu
visualisieren.

Nun ist die Datenbank angelegt und kann gefüllt werden.

Temperatur Protokollierung
~~~~~~~~~~~~~~~~~~~~~~~~~~

Eine nützliche Information ist die Temperatur der CPU und der
Festplatte. Die CPU Temperatur läßt sich unter FreeBSD praktisch mittels
*sysctl* auslesen. Bei den anderen \*BSDs hab ich das nicht getestet,
dort sollte aber ähnliches funktionieren. Es wäre gut, wenn diejenigen,
die dort Informationen haben, diese hier ergänzen.

Unter FreeBSD liest folgendes kleines Shellskript die CPU Temperatur und
die Festplatten Temperatur aus. **Wichtig:** Das Auslesen der
Festplatten Temperatur hängt davon ab, ob *smartctl* mit der Festplatte
funktioniert. Die ID zum Auslesen der Temperatur Information variiert
von Hersteller zu Hersteller. Es sollte in jeden Fall auf der Seite der
*smartmontools* und des Herstellers nachgeschaut werden. *smartctl -A
/dev/ad0* listet alle hersteller-spezifischen Informationen zur ersten
IDE Platte auf. Damit das Skript als Nicht-root Benutzer ausgeführt
werden kann, muß der jeweilige Benutzer Mitglied der Gruppe *operator*
sein, da dieser Lesezugriff auf */dev/ad0* hat.

Hier nun das Skript:

::

   #!/bin/sh

   cpu_temp="hw.acpi.thermal.tz0.temperature"
   disk="/dev/ad0"

   smartctl="/usr/local/sbin/smartctl"
   smartarg="-A"
   option="194"  # smart option for temp
   cmd="/sbin/sysctl"
   rrdtool="/usr/local/bin/rrdtool"
   rrdfile="temperature.rrd"

   h=`$smartctl $smartarg $disk | grep $option | awk '{print $10}'`
   t=`$cmd $cpu_temp | awk '{print $2}' | sed -e s/C//`

   # RRD Datenbank aktualisieren
   $rrdtool update $rrdfile N:$t:$h

In der letzten Zeile werden die Werte in die RRD Datenbank geschrieben.
Die Konsolidierung übernimmt RRDTool. Mit dem Parameter **N** übernimmt
RRDTool die aktuelle Zeit und schreibt die Daten in den aktuellen
passenen Zeitschlitz. Alternativ könnte man auch anstelle von **N** die
aktuelle Zeit in Seconds since Epoch angeben.

Netzwerk Datenbank erstellen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Der Netzwerkdurchsatz soll von RRDTool analog behandelt werden, d.h. es
werden zwar andere Typen für die Data Sources verwendet, die Auflösung
und die Konsolidierungsfunktionen bleiben aber gleich. Hier nun der
Aufruf, um die Datenbank zu erzeugen:

::

   rrdtool create network.rrd                                           \
     DS:inbytes:COUNTER:600:0:U                                         \
     DS:outbytes:COUNTER:600:0:U                                        \
     RRA:AVERAGE:0.5:1:2016                                             \
     RRA:AVERAGE:0.5:6:1824                                             \
     RRA:AVERAGE:0.5:12:1656                                            \
     RRA:AVERAGE:0.5:24:1176                                            \
     RRA:AVERAGE:0.5:288:830                                            \
     RRA:MIN:0.5:1:2016                                                 \
     RRA:MIN:0.5:6:1824                                                 \
     RRA:MIN:0.5:12:1656                                                \
     RRA:MIN:0.5:24:1176                                                \
     RRA:MIN:0.5:288:830                                                \
     RRA:MAX:0.5:1:2016                                                 \
     RRA:MAX:0.5:6:1824                                                 \
     RRA:MAX:0.5:12:1656                                                \
     RRA:MAX:0.5:24:1176                                                \
     RRA:MAX:0.5:288:830

Hier wird der Typ COUNTER verwendet, der keine absoluten Werte
speichert, sondern in diesem Fall die Anzahl der Bytes, die im Mittel in
den letzten 5 Minuten gesendet/empfangen wurden. Ein Overflow der Bytes,
die meistens ja in 32bit Integern gezählt werden, erkennt RRDTool
automatisch. Sollten 64bit Integer benutzt werden, kommt RRDTool auch
damit klar und erkennt den Overflow.

Netzwerk Durchsatz protokollieren
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mit dem Programm ``statgrab`` (in den Ports unter deve/libstatgrab zu
finden) lassen sich sehr bequem die benötigten Werte ermitteln.

Zum einen gibt es ein curses-basiertes Tool (``saidar``), um die Daten
live anzuzeigen, zum anderen das Tool ``statgrab``, das die Daten
ähnlich wie ``sysctl`` ausgibt. Dadurch wird die Ermittlung der
gewünschten Daten stark vereinfacht.

Um beispielsweise für die Interfaces ``tun0``, ``ste0`` und ``ste1`` die
Daten zu sammeln und in die Datenbank einzutragen, kann folgendes
Shellscript verwendet werden.

::

   #!/bin/sh

   file="/path/to/rrdfiles/hostname_IF.rrd"
   statgrab="/usr/local/bin/statgrab"
   rrdtool="/usr/local/bin/rrdtool"
   ifaces="tun0 ste0 ste1"

   for i in ${ifaces}; do \
       ibts=`${statgrab} -u net.${i}.rx`; \
       obts=`${statgrab} -u net.${i}.tx`; \
       rrdfile=`echo ${file} | sed -e "s/IF/${i}/"`; \
       call="${rrdtool} update ${rrdfile} N:${ibts}:${obts}"; \
       `${call}`
   done

Die ``for``-Schleife iteriert über die Liste der Interfaces, liest die
Werte aus, bildet den Namen der RRD Datei und schreibt die Werte.
Vermutlich lässt sich das Ganze auch einfacher gestalten, aber so
funktioniert es erstmal. ;-)

für Faule
^^^^^^^^^

``statgrab`` unterstützt auch MRTG. Mit ``statgrab-make-mrtg-config``
kann man sich eine entsprechende Konfiguration erzeugen und
``statgrab-make-mrtg-index`` hilft bei der Erstellung der Index HTML
Seite. Verwendet man diese Tools, hat man eine Übersicht der Last, der
Netzwerkauslastung, der Dateisystembelegegung und noch einige weitere
Details.

Crontab
-------

Um diese Skripte nun alle 5 Minuten auch auszuführen und die Datenbank
zu aktualisieren, sind folgende 2 Zeilen in der *crontab* notwendig.

::

   */5 * * * * temperature.sh
   */5 * * * * network.sh

Grafische Auswertung
--------------------

Es gibt verschiedene Möglichkeiten, die Daten grafisch darzustellen.

-  Die Shellskripte, die alle 5 Minute laufen, erzeugen jeweils die
   Grafiken und stellen sie zur Verfügung (z.B. über einen Webserver).
-  Auf einem Webserver liegt ein CGI Skript, daß die Graphen erzeugt.
   Dies geht z.B. sehr praktisch über Perl mit RRDs oder RRDTool::OO.

Shellskript
~~~~~~~~~~~

Für das Shellskript muß *rrdtool graph* aufgerufen werden.

::

   rrdtool graph temperature.png                                \
    --start -1d                                                 \ # die Daten des letzten Tages
    --title "CPU/HDD Temperatur"                                \
    --vertical-label "°C"                                       \
    --width 400 --height 150                                    \
    --alt-y-grid                                                \ # bessere Y-Achsen Skalierung
    --interlaced                                                \ # interlaced PNG Image
    'DEF:cpu_temp=temperature.rrd:cpu_temp:AVERAGE'             \ # CPU Temperatur im Mittel
    'DEF:hdd_temp=temperature.rrd:hdd_temp:AVERAGE'             \ # HDD Temperatur im Mittel
    'LINE:cpu_temp#00ff00:CPU Temperatur'                       \
    'LINE:hdd_temp#0000ff:HDD Temperatur'

Perl
~~~~

RRDTool::OO findet sich in der Ports unter *devel/p5-RRDTool-OO*.

::

   #!/usr/bin/perl -w

   use strict;
   use RRDTool:OO;

   my $rrd = RRDTool::OO->new(file => "network.rrd");

   $rrd->graph("network.png",
               title          => "Traffic ste0",
               interlaced     => undef,
               vertical_label => "kBytes/s",
               units_exponent => 3,
               start          => "-1d",
               width          => 400,
               height         => 150,
               draw           => { dsname => "inbytes",
                                   cfunc  => "AVERAGE",
                                   type   => "area",
                                   color  => "008800",
                                   legend => "Incoming Bytes/s"
               },
               draw           => { dsname => "outbytes",
                                   cfunc  => "AVERAGE",
                                   type   => "line",
                                   color  => "55cc02",
                                   legend => "Outgoing Bytes/s"
               }
              );

Zusammenfassung
---------------

Diese Seite deckt sicherlich nur einen Teil der Möglichkeiten von
RRDTool ab. Ein Link zur RRDTool Seite findet sich am Ende der Seite.
Dort finden sich eine gute Dokumentation und eine Gallerie, um zu sehen,
was andere Leute so überwachen.

Es gibt viele Tools, die viel mehr Informationen sammeln und abdecken,
von der Inode Nutzung bis zur Apache Auslastung. Dennoch gibt es
Informationen, die diese nicht abdecken, oder man muß diesen Tools ein
eigenes Skript unterschieben.

Verweise
--------

-  `RRDTool <http://people.ee.ethz.ch/~oetiker/webtools/rrdtool/>`__
-  `RRDTool::OO <http://search.cpan.org/~mschilli/RRDTool-OO-0.13/>`__

* :ref:`genindex`

Zuletzt geändert: |date|

