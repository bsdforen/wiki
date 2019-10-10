Anschluss an das Internet mit PPPoe (mpd) und Firewall (pf)
===========================================================

.. date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Die meisten Rechner sind via PPPoe mit dem Internet verbunden. Diese Anleitung
beschreibt das Herstellen einer Internetverbindung mit mpd4 und der Absicherung
mit der Firewall pf.

ipfw vs. pf
-----------

<<Vorteile von PF>>

User-ppp vs. mpd
----------------

Auch im FreeBSD-Handbuch wird auf die Konfiguration von User-PPP
eingegangen, welche sich im wesentlichen in ``/etc/ppp`` abspielt. Der
Grund mag sein, dass User-ppp ist im Gegensatz zu mpd Teil des
Betriebssystems ist.

User-ppp hat jedoch den entscheidenden Nachteil, dass dieses
unkompatibel zu der Firewall PF ist. Dieser Fehler äußert sich erst nach
der 24-Stunden-Trennung des Providers, bei der User-ppp es nicht mehr
schafft sich neu zu verbinden:

::

   deflink: Already in NETWORK phase

Multilink PPP Daemon (mpd) funktioniert hingegen mit PF und hat
ausserdem noch den Vorteil, dass es durch die Verwendung von netgraph
deutlich weniger CPU-Last erzeugt, da weniger Kontext-Switche anfallen.

mpd einrichten
--------------

Installation von mpd
~~~~~~~~~~~~~~~~~~~~

Da mpd nicht Teil des Betriebssystems ist, muss dieses über sysinstall
von CD oder z.B. über die Ports wie folgt installiert werden: <xterm> #
cd /usr/ports/net/mpd4 # make install clean </xterm>

aktivieren von netgraph
~~~~~~~~~~~~~~~~~~~~~~~

mpd benötigt Netgraph, welches ein Kernel-Modul ist und durch die
Ergänzung der folgenden Zeile in die Datei ``/boot/loader.conf`` geladen
wird.

::

   # Datei /boot/loader.conf
   ng_ether_load="YES"

Konfiguration von mpd
~~~~~~~~~~~~~~~~~~~~~

Die gesamte Konfirugation von mpd befindet sich in
``/usr/local/etc/mpd4``. Es ist zu beachten, dass die Einrückungen mit
<tab> gemacht werden, denn zumindest User-ppp war in der Hinsicht sehr
wählerisch.

**/usr/local/etc/mpd4/mpd.conf**

An der rot markierten Stelle muss der Benutzer, den Sie für den
DSL-Zugang vom Provider bekommen haben, eingesetzt werden. **Hinweis: Es
ist wichtig, dass in den folgenden Konfigurationsdateien die Abschnitte
mit exakt einem TAB eingerückt werden. Ansonsten werden die
Konfigurationsdateien fehlerhaft interpretiert**

::

    default:
           load meinprovider
    meinprovider:
           new -i ng0 meinprovider PPPoE
           set iface addrs 10.2.1.1 10.2.1.2
           set iface route default
           set iface enable on-demand
           set iface idle 0
           set bundle disable multilink
           set auth authname USERNAME
           set link no acfcomp protocomp
           set link disable pap chap
           set link accept pap chap
           set link mtu 1456
           set link keep-alive 10 60
           set link max-redial 0
           set bundle no noretry
           set ipcp yes vjcomp
           set ipcp ranges 0.0.0.0/0 0.0.0.0/0
           log +link
           log +lcp
           log +auth
           set ipcp enable req-pri-dns
           set ipcp enable req-sec-dns
           set iface up-script /usr/local/etc/mpd4/mpd_dsl.linkup
           open

**/usr/local/etc/mpd4/mpd.links**

Hier ist zu beachten, dass das Netzwerkinterface eingetragen wird, an
dem das DSL-Modem direkt über Ethernet angeschlossen ist.

::

    PPPoE:
           set link type pppoe
           set pppoe iface fxp1
           set pppoe service "foo"
           set pppoe disable incoming
           set pppoe enable originate

Es kann sein, dass bei wenigen Providern bei pppoe service etwas
bestimmtes eingetragen werden muss. Dies ist aber z.B. bei T-Online und
GMX nicht der Fall.

**/usr/local/etc/mpd4/mpd.secret**

Aus Sicherheitsgründen wird Benutzername und Passwort getrennt zur
Konfiguration eingetragen. Die Datei kann mit weiteren
Dateiberechtigungen versehen werden. Dies ist so zu verstehen, dass in
der mpd.conf| nur der Benutzername steht und diesem hiermit das
Passwort zugeordnet wird. Aus diesem Grund muss der USERNAME hier erneut
angegeben werden.

::

   USERNAME    PASSWORT

**/usr/local/etc/mpd4/mpd_dsl.linkup**

Nach dem Aufbauen einer PPPoe-Verbindung soll ein Reset von pf
sicherstellen, dass pf auf die veränderte Umgebung wie z.B. eine neue IP
am DSL-Interface reagieren kann. Gerade beim Reconnect nach der
24-Stunden-Trennung des Providers ist eine kritische Situation. Dieses
Skript wird automatisch nach jedem Verbindungsaufbau aufgerufen:

::

   #!/bin/sh
   # PF resetten
   /sbin/pfctl -Fa -e -f /etc/pf.conf
   # Nameserver uebenehmen
   echo nameserver $7 >  /etc/resolv.conf
   echo nameserver $9 >> /etc/resolv.conf

aktivieren von mpd4
-------------------

mpd wird durch das Hinzufügen der folgenden Zeile aktiviert:

**/etc/rc.conf**

::

   mpd_enable="YES"

Hierdurch wird die Verbindung direkt nach dem Booten geöffnet und immer
offen gehalten, daher ist diese Konfiguration nur sinnvoll, wenn man
einen Pauschaltarif (neudeutsch Flat-Rate) hat.

syslog die Log-Inforationen ausgeben lassen
-------------------------------------------

mpd schreibt nach syslog, jedoch werden die Dateien bei der
Standardkonfiguration nicht in eine Datei geschrieben.

-  Das wird wie folgt geändert in der Datei ``/etc/syslog.conf``:

::

   !mpd
   *.*                                             /var/log/mpd.log
   #Wichtig, das muss vor der folgenden Zeile eingefügt werden.
   !*

-  Anlegen einer leeren Datei, da sonst syslog nicht in diese schreibt:

.. code:: bash

   touch /var/log/mpd.log

-  syslog die Konfiguration erneut lesen lassen

.. code:: bash

   /etc/rc.d/syslogd reload

mpd4 Webinterface
-----------------

Wenn man das Webinterface von mpd4 benutzen will, dann muss man
folgendes in der mpd.conf| hinzufügen:

::

    startup:
           set web ip span xxx.xxx.xxx.xxx
           set web port 4444
           set web user username passwort
           set web open

Probleme beim Setzen der Default-Route
--------------------------------------

mpd schafft es nicht die Default-Route über die PPPoe-Verbindung zu
setzen, wenn bereits eine Systemweite Default-Route definiert ist. Daher
sollte diese in der /etc/rc.conf| auskommentiert werden

::

   #defaultrouter="192.168.32.4"

pf einrichten
-------------

pf einschalten
--------------

Seit FreeBSD 5.3 kann pf als Modul geladen werden, wozu ein Eintrag in
der /etc/rc.conf| genügt.

.. _etcrc.conf-1:

**/etc/rc.conf**

::

   pf_enable="YES" 

Regeln konfigurieren
--------------------

Die hier vorgestellten Beispiel-Regeln blockieren alle Verbindungen vom
Internet auf den lokalen Rechner, lassen aber alle Verbindungen von dem
Rechner ins Internet zu:

**/etc/pf.conf**

::

   # Definition der Interfaces
   ext_if="ng0"
   dsl_if="fxp1"
   int_if="fxp0"
   # zunaechst alles oeffnen
   pass out all keep state
   pass in all keep state
   # pppoe-Device dicht machen
   pass out on $ext_if all keep state
   block in on $ext_if all

Sollen jetzt einzelne Ports wie **ssh** und **https** geöffnet werden,
dann können folgende Zeilen **unterhalb** o.g. Konfiguration angehängt
werden:

::

   pass in on $ext_if proto tcp from any to any port **ssh** keep state
   pass in on $ext_if proto tcp from any to any port **https** keep state

Weitere Rechner hinter diesem ins Internet bringen
--------------------------------------------------

Bisher wurde dieser eine Rechner mit dem Internet verbunden und durch
eine Firewall geschützt. In den meisten Fällen existieren jedoch noch
viel mehr Rechner, die die Internetverbindung mitverwenden sollen.

Interne Netzwerkkarte einrichten
--------------------------------

Hierzu muss der Rechner eine zweite Netzwerkkarte(hier fxp0) besitzen,
welche mit dem lokalen Netzwerk (z.B. ein Switch) verbunden ist, in dem
auch die anderen Rechner hängen. Diese muss z.B. folgende statische IP
zugewiesen bekommen:

.. _etcrc.conf-2:

**/etc/rc.conf**

::

   ifconfig_fxp0="inet 192.168.32.99  netmask 255.255.255.0"

IP-Forward aktivieren
---------------------

Damit der Rechner überhaupt erst Pakete zwischen internem Netz und
Internet weiterleitet, muss dies wie folgt aktiviert werden:

.. _etcrc.conf-3:

**/etc/rc.conf**

::

   gateway_enable="YES"

NAT aktivieren
--------------

Da die Rechner im internen Netzwerk in den meisten Fällen keine
offizielle IP-Adresse besitzen können, besitzt nur die PPPoe-Seite
dieses Rechneres eine offizielle IP-Adresse. In diesem Fall müssen alle
internen Adressen durch den Rechner so umgesetzt werden, dass es
aussieht, als kämen der gesamte Verkehr nur von diesem mit der
offiziellen Adresse. Hierzu verwendet man sog. NAT (eigentlich NAPT).
Diese hier gewählte Umsetzung hat zusätzlich den Vorteil, dass alle
Rechner von innen beliebige Verbindungen ins Internet aufbauen können,
jedoch kein Rechner eine Verbindung zu den internen Rechnern initiieren
kann. Somit sind die Rechner von aussen geschützt.

Unter FreeBSD gibt es viele Möglichkeiten NAT zu aktivieren (natd,
ppp-NAT, pf-nat). Da hier PF sowieso als Firewall verwendet wird, bietet
es sich hier an, PF auch für NAT zu verwenden:

.. _etcpf.conf-1:

**/etc/pf.conf**

::

   # um NAT erweiterte Version 
   # ext_if="tun0"
   ext_if="ng0"
   dsl_if="fxp1"
   int_if="fxp0"
   internal_net="192.168.0.0/16"
   # nat
   nat on $ext_if from $internal_net to any -> ($ext_if)
   # zunaechst alles oeffnen
   pass out all keep state
   pass in all keep state
   # interne devices explizit durchgaengig machen
   pass quick on lo0 all
   pass quick on $int_if all
   pass quick on $dsl_if all
   # pppoe-Device dicht machen
   pass out on $ext_if all keep state
   block in on $ext_if all

Maximale Segment Größe
~~~~~~~~~~~~~~~~~~~~~~

Falls Probleme beim Aufrufen bestimmter Seiten auftreten, kann das an
der Paketgröße liegen (Stichwort MTU, Maximum Transmission Uni). Statt
bei jedem Client (bzw. jedem Netzwerkinterface) jeweils die MTU auf den
passenden Wert zu setzen, kann man PF mit Hilfe des Scrubbing beibringen
die Pakete gleich so umzupacken, dass sie die MTU nicht überschreiben
und so nicht verworfen werden.

::

   ...
   scrub in on $int_if fragment reassemble max-mss 1452

   # nat
   nat on $ext_if from $internal_net to any -> ($ext_if)
   ...

Scrubbing kann unter Umständen Probleme mit z.B. NFS bringen laut
`OpenBSD PF FAQ <http://www.openbsd.org/faq/pf/de/scrub.html>`__, bei
mir hat die oben genannte Scrubregel allerdings keine Auswirkung auf
NFS.

Das Festsetzen einer Maximalen Segment Größe ist laut `Wikipedia
(MSS) <http://de.wikipedia.org/wiki/Maximum_Segment_Size>`__ allerdings
nicht sehr sauber. Nunja, NAT ist ja auch nicht das Gelbe vom Ei,
demnach...

Nameserver und DHCP-Server einrichten
-------------------------------------

Mit der bisherigen Konfiguration können alle internen Rechner bereits
das Internet erreichen. Um Namen wie "www.google.de" zu IP-Adressen
aufzulösen, müssen sie jedoch einen Nameserver befragen. Sie könnten
z.B. direkt einen Nameserver im Internet befragen, jedoch müsste der auf
jedem Rechner gepflegt werden. Eine bessere Lösung ist es, wenn dieser
Rechner die Namensanfragen engegennimmt, denn er kennt immer einen
funktionierenden zuständigen Nameserver. Bei jedem
PPPoe-Verbindungsaufbau werden aktuelle Nameserver vom Provider
mitgeteilt. Um diese Funktionalität zu erreichen, muss ein Nameserver
installiert werden, welcher die Anfragen der internen Rechner
entgegennimmt, die Anfrage an die aktuellen Nameserver im Internet
weiterleitet und dann auch intern beantwortet. Es gibt eine Menge von
Nameservern, welche z.B. schon im Betriebssystem integriert sind (bind)
oder für sehr komplexe und große Netzwerke geeignet sind. Für den hier
dargestellten Einsatzzweck eigent sich **dnsmasq** da dieser einfach zu
konfigurieren ist, wenig Speicher verbraucht, auf das
wählverbindungsbedingte Wechseln der Internet-Nameserver vorbereitet ist
und noch einen DHCP-Server mitbringt.

dnsmasq installieren
~~~~~~~~~~~~~~~~~~~~

dnsmasq kann z.B. wie folgt über die Ports installiert werden: <xterm> #
cd /usr/ports/dns/dnsmasq # make install clean </file>

dnsmasq konfigurieren
~~~~~~~~~~~~~~~~~~~~~

Die Datei /usr/local/etc/dnsmasq.conf.sample| enthält eine
Beispielkonfiguration, welche man als Schablone verwenden kann.

**/usr/local/etc/dnsmasq.conf**

::

   # Never forward plain names (without a dot or domain part)
   domain-needed
   # If you want dnsmasq to listen for DHCP and DNS requests only on
   # specified interfaces (and the loopback) give the name of the
   # interface (eg eth0) here.
   # Repeat the line for more than one interface.
   interface=fxp0
   # Name der lokalen Domaene 
   domain=heimnetz
   # Add local-only domains here, queries in these domains are answered
   # from /etc/hosts or DHCP only.
   local=/heimnetz/
   # Range der zu vergeben Adressen des DHCP-Server
   #(von 192.168.32.100 bis 192.168.32.200 mit einer Lease-Time von 24 Stunden)
   dhcp-range=192.168.32.100,192.168.32.200,24h
   # Speicherort, an dem sich der DHCP-Server seine vergebenen Adressen merkt
   # Default funktioniert unter FreeBSD nicht, daher auf folgendes Verzeichnis umstellen
   dhcp-leasefile=/var/db/dnsmasq.leases

dnsmasq automatisch mitstarten
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Damit der dnsmasq-Dienst direkt bei jedem Rechnerstart automatisch
mitgestartet wird, muss muss die /etc/rc.conf.local| um folgende Zeile
erweitert werden:

**/etc/rc.conf.local**

::

   dnsmasq_enable="YES"

dnsmasq Zusammenfassung
~~~~~~~~~~~~~~~~~~~~~~~

An diesen Rechner können jetzt immer Namensanfragen der internen Rechner
gestellt werden, welche er stets mit Hilfe der aktuellen
Internet-Nameserver beantworten kann. Bei den internen Rechnern mit
fester IP-Adresse kann dieser Rechner als Nameserver und als
Default-Gateway fest eingetragen werden. Da dnsmasq nicht nur die
Funktionalität eines DNS-Servers sondern auch eines DHCP-Servers bietet,
können die internen Rechner auch automatisch über DHCP konfiguriert
werden. Dabei bekommen sie automatisch von diesem Rechner eine
IP-Adresse, die Routing-Einstellungen und korrekte
Namensauflösungsparameter mitgeteilt. Da die internen mit DHCP
konfigurierten Rechner sich beim Bezug der DHCP-Daten auch ihren
Hostname an den DHCP-Server weitergaben, meldet dnsmasq den Namen und
die IP gleich auch an den internen DNS-Teil weiter, so dass andere
interne Rechner diesen jetzt sogar unter seinem Namen erreichen können.
Wenn man die verbleibenden lokale Rechner mit statischer IP in die
/etc/hosts dieses Rechners einträgt, dann sind die für andere Rechner
nun auch über dnsmasq auflösbar. Dies sind ein sehr praktische
Nebeneffekte. FreeBSD|

* :ref:`genindex`

Zuletzt geändert: date|
