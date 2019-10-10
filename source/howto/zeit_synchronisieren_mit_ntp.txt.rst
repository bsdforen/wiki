Zeit synchronisieren mit ntp
============================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Echtzeituhren eines modernen PCs haben teilweise eine geringere Genauigkeit als
beispielsweise eine Werbegeschenk-Armbanduhr.  Dieser Artikel beschreibt den
automatischen Abgleich der Systemuhr.

Einleitung
----------

Anstatt die Zeit beim Starten des Rechners oder via eines Cron-Jobs
einzustellen, hat die hier beschriebene Lösung folgende Vorteile:

-  ntpd und ntpdate sind im Gegensatz z.B. zu rdate Basisfunktionen von
   FreeBSD
-  ntp basiert auf RFC958 (und neuere), was sich anscheinend im
   Gegensatz zu RFC868 (rdate) durchzusetzen scheint.
-  ntp berücksichtigt die Laufzeitwege der IP-Pakete und korrigiert
   diese
-  ntpd muss nicht via Cron aufgerufen werden und kümmert sich selbst um
   alles
-  ntpd nutzt spezielle Mechanismen von FreeBSD wie Software-PLL, womit
   sogar die Geschwindigkeit der System-Uhr korrigiert wird. Ein ntp
   synchronisierter Rechner ist daher auch dann noch genau, auch wenn er
   keinen Kontakt mehr zum Zeitquellenserver hat.
-  ntp 4 hat eine Genauigkeit von 10 Millisekunden über das öffentliche
   Internet und bis zu 200 Mikrosekunden über ein lokales Netzwerk

Bei der Verwendung von ntpd entscheidet ntpd selbst, wann und wie oft er
sich gegen den Zeitserver synchronisieren muss. Da ntpd selbst den
einstellbaren Zeitgeber (Software-PLL) überwacht und korrigiert, kann er
selbst anhand des gemessenen Fehlers abschätzen, wann es notwendig ist,
die Uhrzeit zu synchronisieren. Ausserdem geschieht das Verändern der
Systemzeit nur in notwendigen Fällen und sanft.

Voraussetzung
-------------

Gerade durch die (sinnlose) Uhrzeitumstellung auf Sommer- oder
Winterzeit kommt es vor, dass ntpd (FreeBSD ntpd und OpenNTPD)
durcheinanderkommt. In vielen FreeBSD-Installationen läuft die CMOS-Uhr
des Rechners auf lokaler Uhrzeit (Standard bei der Installation) und
nicht auf UTC. Dieser Modus ist eigentlich nur dazu da, wenn man noch
ein Windows-Betriebssystem parallel auf dem gleichen Rechner verwendet.
(Hintergrundinformationen sind im Anhang dieses Artikels zu finden)

Während bei der UTC-Einstellung FreeBSD immer davon ausgehen kann, dass
die Uhr auf UTC läuft und keine Sommerzeitumstellungen mitgemacht hat,
ist es sich bei dem anderen Modus nicht sicher. Es scheint sogar davon
auszugehen, dass Windows die Zeitumstellung vornimmt. Gerade bei
Maschinen, die nicht während der Zeitumstellung laufen - also morgens
eingeschaltet werden - führt das zu Problemen.

Bei Systemen, bei denen nur FreeBSD (oder Linux, Solaris im UTC-Modus)
installiert ist, empfiehlt es sich die CMOS-Uhr auf UTC laufen zu
lassen, um solchen Problemem schon im Vorfeld aus dem Weg zu gehen.

CMOS-Uhr auf UTC-Modus stellen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Das Betriebssystem erkennt an dem Vorhandensein der Datei
``/etc/wall_cmos_clock``, dass die CMOS-Uhr auf lokaler Zeit läuft.
Daher wird diese Datei gelöscht, damit die CMOS-Uhr danach auf UTC
läuft.

::

   rm /etc/wall_cmos_clock

Nach einem Betriebssystemneustart ist diese Einstallung wirksam, wobei
durch die veränderte Interpretation der CMOS-Uhr die Uhrzeit nun um eine
oder zwei Stunden falsch geht. Das kann z.B. mit
``ntpdate de.pool.ntp.org`` schnell korrigiert werden.

ntpd auswählen
--------------

Im ersten Schritt soll abegwägt werden, welche ntpd verwendet werden
soll. Es ist möglich den mitgelieferten ntpd von FreeBSD oder den von
OpenNTP zu nutzen. Dieser Artikel deckt die Konfiguration beider Systeme
ab, es darf aber nur einer installiert werden.

OpenNTP
~~~~~~~

Im Gegensatz zu dem FreeBSD ntpd wurde dieser neu entwickelt. Hier wurde
besonders Wert auf Quelltext-Sauberkeit und Zuverlässigkeit gelegt
gelegt, was vorrang vor dem Erreichen der letzten Mikrosekunde
Genauigkeit hat. In der Praxis ist die Genauigkeit bei normalen
Internetverbindungen (40 ms Latanz) wie auch bei dem FreeBSD ntpd bei
rund einer Millisekunde, was für nahezu alle Anwendungszwecke ausreichen
sollte. Im Gegensatz aber zum FreeBSD verliert OpenNTP seine Zeitservern
nicht auch bei vielen Internetverbindungsproblemen und sorgt stets für
korrekte Zeit. In fast allen Fällen sollte daher OpenNTP die bessere
Wahl sein, auch wenn dieser nicht Teil des Betriebssystems ist und
nachinstalliert werden muss.

FreeBSD ntpd
~~~~~~~~~~~~

Der ntpd von FreeBSD ist über viele Jahre durch viele Personen auf
höchste Genauigkeit optimiert, wodurch die Stabilität und
Quelltextqualität etwas in Mitleidenschaft gezogen wurde. Dieser ntpd
sollte nur verwendet werden, wenn eine ständige Verbindung zu den
ntp-Servern besteht (also keine Wählverbindung wie z.B. DSL oder ein
NAT-Gateway), denn auf Verbindungswechsel reagiert der ntpd nicht
sonderlich gut und verliert seine Verbindungen zu den ntp-Servern
dauerhaft ohne dies ins Syslog zu vermerken. Der Wechsel von
Sommer/Winterzeit ist auch nicht der beste Freund von diesem ntpd. Der
notwendige Workaround ist weiter unten im Artikel beschrieben.

OpenNTPD
--------

Installation von OpenNTPD
~~~~~~~~~~~~~~~~~~~~~~~~~

Über die Ports

::

   cd /usr/ports/net/openntpd
   make install clean

Oder als Package

::

   pkg_add -r openntpd

OpenNTPD hat keine abhängigen Pakete und ist insgesamt sehr sparsam mit
Speicherplatz.

Konfiguration von OpenNTP
~~~~~~~~~~~~~~~~~~~~~~~~~

Die Konfiguration befindet sich in der Datei
``/usr/local/etc/ntpd.conf`` und hat beispielsweise folgenden Inhalt.
Die Auslieferungszustand kann hier z.B. direkt übernommen werden.

::

   # $OpenBSD: ntpd.conf,v 1.7 2004/07/20 17:38:35 henning Exp $
   # sample ntpd configuration file, see ntpd.conf(5)

   # Addresses to listen on (ntpd does not listen by default)
   #listen on *
   #listen on 127.0.0.1
   #listen on ::1

   # sync to a single server
   #server ntp.example.org

   # use a random selection of 8 public stratum 2 servers
   # see http://twiki.ntp.org/bin/view/Servers/NTPPoolServers
   servers pool.ntp.org

<note important> Es kursieren einige Beispielkonfigurationen mit festen
IPs oder Adressen von ntp-Servern. Diese bitte auf keinen Fall benutzen.
Teilweise kann es sich um Server handeln, die für die öffentliche
Nutzung nicht vorgesehen sind. Es gibt alternative Pool-Adressen. Dort
wird ein legaler Server dem System zugewiesen. Dies sind z.B.
**pool.ntp.org**, aber auch **de.pool.ntp.org**. </note>

Konfiguration von OpenNTP als Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Um den OpenNTPD als Zeit-Server im netzwerk zur verfügung zu stellen,
muß lediglich die zeile ``listen on *`` in der ntpd.conf auskommentiert
werden. Anschließend nicht vergessen den dienst neu zu starten. Soll der
openntpd dienst nur auf einem bestimmten Interface zur verfügung stehen,
die entsprechende IP angeben.

::

   # $OpenBSD: ntpd.conf,v 1.7 2004/07/20 17:38:35 henning Exp $
   # sample ntpd configuration file, see ntpd.conf(5)

   # Addresses to listen on (ntpd does not listen by default)
   listen on *
   #listen on 127.0.0.1
   #listen on ::1

   # sync to a single server
   #server ntp.example.org

   # use a random selection of 8 public stratum 2 servers
   # see http://twiki.ntp.org/bin/view/Servers/NTPPoolServers
   servers pool.ntp.org

Aktivieren von OpenNTP
~~~~~~~~~~~~~~~~~~~~~~

Aktiviert wird OpenNTPD wie bei FreeBSD typisch mit folgendem Eintrag in
der Datei **``/etc/rc.conf.local``**:

::

   openntpd_enable="YES"
   openntpd_flags="-s"

Gerade bei Rechnern, die nicht rund um die Uhr laufen, ist es sinnvoll
größere Abweichungen beim Systemstart sofort per Sprung setzen zu
lassen, da eine fließende Korrektur mit *adjtime()* Stunden dauern kann.
Die Option *"-s"* sorgt dafür, dass Abweichungen größer als 180 Sekunden
sofort korrigiert werden beim Systemstart - danach wird wieder
*adjtime()* verwendet.

.. _freebsd-ntpd-1:

FreeBSD ntpd
------------

Konfiguration von ntpd
~~~~~~~~~~~~~~~~~~~~~~

Die Konfiguration befindet sich in der Datei ``/etc/ntp.conf`` und hat
beispielsweise folgenden Inhalt

::

   server ptbtime1.ptb.de prefer
   server ptbtime2.ptb.de
   server de.pool.ntp.org
   tinker panic 4000
   driftfile /var/db/ntp.drift

In diesem Fall wird der erste Zeitserver der Physikalisch Technischen
Bundesanstalt (ptbtime1) als glaubwürdigste Quelle deklariert für den
Fall, falls nptd selbst nicht wegen starker Abweichung aus den drei
angegebenen Zeitquellen die verlässlichste ermitteln kann. Die PTB
übermittelt die "gesetzliche Zeit".

Normalerweise setzt ntpd die lokale Uhrzeit nur, wenn diese weniger als
1000 Sekunden von der Uhrzeit der Zeitserver abweicht. Die Option
"tinker panic 4000" setzt den Standardwert auf 4000 Sekunden hoch, was
etwas mehr als eine Stunde ist. Dies korrigiert das Problem, dass wenn
ntpd nicht während der Zeitumstellung (Winter-/Sommerzeit) aktiv war,
danach das Setzen der Uhrzeit verweigert, da er einen Unterschied von
großer als 1000 Sekunden erkennen würde.

Also für alle Rechner, die nachts evtl. auch mal ausgeschaltet sein
können, ist diese Einstellung ratsam, da ntpd damit leider sonst nicht
klarkommt und die Uhrzeit nicht mehr setzt, obwohl es in dem Augenblick
wirklich sinnvoll wäre.

Die Fehlermeldung sieht so aus:

::

   Oct 31 14:46:24 box ntpd[67238]: ntpd 4.2.0-a Tue Jul 11 12:50:49 CEST 2006 (1)
   Oct 31 14:58:14 box ntpd[67238]: time correction of -3599 seconds exceeds sanity limit (1000); set clock manually to the correct UTC time.

Aktivieren von ntpd
^^^^^^^^^^^^^^^^^^^

In die Datei /etc/rc.conf muss folgende Zeile hinzugefügt werden
ntpd_enable="YES"

Anmerkungen
-----------

Setzen einer ganz falsch gehenden Systemuhr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Dieses Kapitel bespricht den Sonderfall, wenn die Systemuhr des Rechners
absolut falsche Werte liefert. Die Ursache dafuer kann z.B. eine leere
oder nicht vorhandenen CMOS-Pufferbatterien wie z.B. enbedded Systemen
sein. Hierzu kann ntpd mit der folgenden Option in der **/etc/rc.conf**
dazu veranlasst werden, dass beim ersten Start in jedem Fall die lokale
Urzeit gesetzt wird und erst nach dem ersten Setzen die weiter oben
vorgestellte Panic-Zeit beachtet wird.

::

   ntpd_flags="-g"

``Anmerkung:`` Die "-g"-Option kann dazu führen, dass ntpd die
Systemzeit nur beim ersten Mal setzt oder gar nicht mehr. ntpd hat
hiermit ein etwas unvorhersagbares und der Dokumentation
widersprechendes Verhalten, daher sollte das möglichst nicht verwendet
werden.

Die rc.conf-Einstellung ntpd_sync_on_start="YES" soll den gleichen Zweck
haben, funktioniert aber nicht (getestet FreeBSD 5.5 und FreeBSD
Prerelease 6.2).

Fehlersuche bei ntpd
--------------------

Gerade der ntpd von FreeBSD neigt dazu, nach einiger Zeit bei nicht
immer vorhandener Verbindung zu den Zeit-Servern die Verbindung zu
verlieren. Mit folgendem Befehl können Informationen abgefragt werden:

::

   a@host:[~]% ntpq -p
        remote           refid      st t when poll reach   delay   offset  jitter
   ==============================================================================
   *ptbtime1.ptb.de .PTB.            1 u  300 1024  377   67.547   -1.374   3.255
   +ptbtime2.ptb.de .PTB.            1 u  271 1024  377   65.997   -1.313   1.541
   +draisine.zs64.n 192.53.103.104   2 u  426 1024  377   58.800    2.449   3.648

Hier werden alle Zeit-Server aufgelistet, zu denen Kontakt besteht. Ist
diese Liste beim FreeBSD-ntpd mehrfach leer, sollte auf OpenNTPD
gewechselt werden.

Die Parameter haben folgende Bedeutung:
---------------------------------------

-  ``Remote``: Name oder IP des entfernten Zeit-Servers
-  ``refid``: Die Quelle, von dem der entfernte Zeit-Server selbst seine
   Zeit bezieht. ".PTB." ist direkt die Atomuhr, während in dem Beispiel
   der letzte Zeit-Server seine Zeit über Netzwerk holt.
-  ``st``: Stratum. Das beschreibt die Hierarchieebene des Zeit-Servers,
   d.h. über wie viele Zeitserver wird die Zeitinformation bis zu diesem
   Verbreitet. Stratum 1 bedeutet hier z.B., dass der Zeit-Server seine
   Zeit direkt von der Atomuhr holt, während Stratum 2 (draisine...)
   seine Zeit erst von einem weiteren Zeit-Server mit Verbindung zur
   Atomuhr holt.
-  ``when``: Letzte Abfrage des Zeitservers in Sekunden
-  ``poll``: Von ntpd selbst gewähltes Intervall, in dem
   Zeitinformationen abgeholt werden. Steigt das Vertrauen von ntpd in
   die Systemuhr, deren Drift ntpd selbst korrigiert, dann ist es nicht
   notwendig so oft den Zeitserver zu befragen.
-  ``reach``: Erreichbarkeit des Zeitservers. Hier wird im Grund eine 1
   Durchgeshiftet. 377 bedeutet, dass der Timeserver immer erreichbar
   war.
-  ``delay``: Ist die durchschnittliche Paketlaufzeit zum Zeit-Server.
-  ``offset``: Differenz zwischen der lokalen Uhr und dem Zeit-Server.
   Trotz Paketlaufzeiten von 60ms werden durch das intelligente
   Protokoll Geneuigkeiten von ein paar Millisekunden erreicht.
-  ``jitter``: Laufzeitdifferenzen der Anfragen an den Zeitserver. Im
   Gegensatz zu den Laufzeiten stört dieser Faktor die Genauigkeit der
   Uhrzeitsynchronisation. Eine ständig stark wechselnd belastete
   Internetverbindung verschlechtert diesen Wert z.B. .

Hintergrundinformationen: Echtzeituhren im Betriebssystem
---------------------------------------------------------

In jedem Computer gibt es zwei Arten von Echtzeituhren:

CMOS-Uhr
~~~~~~~~

Die CMOS-Uhr (aka Hardware-Clock), die batteriegepuffert auch im
ausgeschaltetem Zustand mehr oder weniger richtig läuft, ist auf dem
Mainboard in der Hardware integriert. Wenn die Batterie (früher Akku)
leer ist oder wie z.B. bei Enbedded-Geräten ganz fehlt, dann läuft die
Uhr im ausgeschaltetem Zustand gar nicht weiter und beim booten hat man
immer ein Standard-Datum, das das BIOS-Post beim zurücksetzen des CMOS
dort hineinschreibt (weg. CMOS Checksum Error). Die Zeitdarstellung
dieser Uhr ist ähnlich einem klassischem Datum aufgebaut (z.B. 19.10.06
22:36:22) und die kleinste Einheit ist die Sekunden. Die CMOS-Uhr hat
einen eigenen Kalender.

Betriebssystem-Uhr
~~~~~~~~~~~~~~~~~~

| Fast alle Betriebssysteme haben eine interne Software-Echtzeituhr, die
  das Betriebssystem beim Systemstart nach der CMOS-Uhr stellt. Danach
  läuft die Betriebssystem-Uhr unabhängig weiter, denn die Zeit wird vom
  Betriebssystem weitergezählt, wobei bei einfachen Betriebssystemen der
  Timer-Interrupt den Takt angibt. Das Betriebssystem ermöglicht
  komplexen Zugriff auf die interne Uhr unter berücksichtugung eines
  Betriebssystem-Kalenders und der Zeitzone.
| Bei FreeBSD läuft die interne Kernel-Time immer auf UTC, erst die
  Zugriffe über die Systemcalls rechnen die Zeit in die in der Datei
  ``/etc/localtime`` eingestellten Zeitzone um.
| Die Auflösung der Betriebssystemuhr ist weit unter einer Sekunde im
  Gegensatz zur CMOS-Uhr. FreeBSD hat eine sehr weit entwickelte
  hochauflösende Systemuhr, bei der sogar die Taktgeberfrequenz im
  Betrieb korrigiert werden kann (Software-PLL). ntpd nutzt diese
  Eigenschaft aus und korrigiert neben der Uhrzeit auch die
  Taktgeberfrequenz, so dass die Uhr sogar deutlich exakter als sonst
  weiterlaufen würde, selbst wenn nptd diese nicht mehr synchronisieren
  kann (z.B. wegen fehlender Internetverbindung). Ausserdem scheint
  FreeBSD nicht nur den Timer-Interrupt als Taktgeber zu nutzen, sondern
  auch diverse andere Interrupts, wobei er deren Qualität
  berücksichtigt.


* :ref:`genindex`

Zuletzt geändert: |date|

