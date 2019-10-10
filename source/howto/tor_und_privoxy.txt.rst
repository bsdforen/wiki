Tor und Privoxy
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png

Dieser Artikel beschäftigt sich mit der Installation von `Tor
<http://tor.eff.org/index.html.de>`__ und `Privoxy
<http://www.privoxy.org/>`__.

Installation
------------

Anmerkung: Privoxy wird nur gebraucht, wenn man Tor auch als Client
nutzen will. Wer das nicht nicht vor hat, kann das Programm einfach
weglassen.

FreeBSD
~~~~~~~

Zunächst müssen wir Tor und Privoxy installieren. Dies machen wir am
besten mit dem Port-System oder laden uns die Pakete herunter:

::

   # cd /usr/ports/security/tor && make install clean
   # cd /usr/ports/www/privoxy && make install clean

Um Tor und Privoxy beim Booten zu starten, müssen wir in der
/etc/rc.conf noch zwei Zeilen hinzufügen:

::

   privoxy_enable="YES"
   tor_enable="YES"

OpenBSD
~~~~~~~

Die einfachste Möglichkeit die benötigten Pakte zu installieren ist
pkg_add:

::

   # pkg_add tor privoxy

Konfiguration Tor
-----------------

Allgemein: Welche Änderungen vorgenommen werden müssen, hängt von
Version und Betriebssystem ab.

Es muss in der Konfigurationsdatei /usr/local/etc/tor/torrc (FreeBSD)
bzw. /etc/tor/torrc (OpenBSD) folgendes hinzugefügt bzw. auskommentiert
werden:

::

   RunAsDaemon 1

Falls kein Datenverzeichnis (in der Konfigurationsdatei und im
Verzeichnisbaum) existiert, muss es hinzugefügt werden:

::

   DataDirectory /var/tor

in die Konfigurationsdatei schreiben und dann mit

::

   # mkdir /var/tor
   # chmod 0700 /var/tor
   # chown _tor:_tor /var/tor

das Verzeichnis anlegen.

Um die Sicherheit zu erhöhen, empfiehlt es sich Tor unter dem
Benutzerkonto *\_tor* laufen zu lassen:

::

   User _tor
   Group _tor

Wer Tor-Meldungen in einer separaten Logdatei haben will, muss dies in
der Konfigurationsdatei bei der Variablen "Log" angeben und eine Datei
mit entsprechenden Rechten anlegen.

.. _openbsd-1:

OpenBSD
~~~~~~~

Damit Tor bei jedem Systemstart startet, muss die **/etc/rc.local**
editiert werden:

::

   ...

   if [ -x /usr/local/bin/tor ];
   then
       echo -n ' tor';
       /usr/local/bin/tor -f /etc/tor/torrc
   fi

Die Standard-Konfigurationsdatei sollte schon lauffähig sein.

Konfiguration Privoxy
---------------------

Damit Privoxy die Anfragen auch an Tor weiterleitet, müssen wir
/usr/local/etc/privoxy/config (FreeBSD) bzw. /etc/privoxy/config
(OpenBSD) ändern. Wir müssen die beiden Einträge

::

   jarfile jarfile
   logfile logfile

auskommentieren:

::

   #jarfile jarfile
   #logfile logfile

<note important> Wenn ihr diese Änderungen in der Privoxy-Konfiguration
nicht vornehmt, dann logt Privoxy mit und speichert Cookies. </note>

Nun müssen wir noch einstellen, dass Privoxy auch alle Anfragen an Tor
weiterleitet:

::

   forward-socks5 / localhost:9050 .

Der Punkt am Ende ist auch wichtig.

Soll Tor auch anderen Rechnern im lokalen Netzwerk zur Verfügung stehen,
muss noch

::

   listen-address lokale_IP-Adresse:8118

hinzugefügt werden.

.. _openbsd-2:

OpenBSD
~~~~~~~

Soll Privoxy beim Booten schon starten, muss folgender Abschnitt der
rc.local hinzugefügt werden:

::

   if [ -x /usr/local/sbin/privoxy ]; then
               echo -n ' privoxy';
               /usr/local/sbin/privoxy --user _privoxy._privoxy \
                       /etc/privoxy/config
   fi

Tor testen
----------

Wir sind jetzt mit der Konfiguration von Tor und Privoxy fertig und
können einen Test wagen. Zunächst muss Tor und Privoxy gestarten werden
und dann noch im Browser eingetragen werden:

::

   HTTP: localhost/IP-Adresse:8118
   SSL:  localhost/IP-Adresse:8118
   SOCKSv5: localhost/IP-Adresse:9050

Jetzt können wir auf
`serifos.eecs.harvard.edu <http://serifos.eecs.harvard.edu/cgi-bin/ipaddr.pl?tor=1>`__,
auf `proxydetect.com <http://www.proxydetect.com/>`__ oder auf
`heise.de/ip <http://www.heise.de/netze/tools/ip>`__ schauen, ob wir
auch wirklich mit Tor surfen.

Tor-Server betreiben
--------------------

In der torrc:

::

   Nickname ''ididnteditheconfig''
   ORPort 9001
   DirPort 9030 #falls man Verzeichnisse von anderen spiegeln will
   ContactInfo ''human@example.com''
   #BandwidthRate 20 kB
   #BandwidthBurst 30 kB

Die Bandbreitenangabe ist optional. Erforderlich für einen Tor-Server
sind mindest 20kB in jede Richtung. Bei asymmetrischen Leitungen ist die
Richtung mit der geringen Bandbreite ausschlaggebend (meist der
Upstream). Die Burstrate gibt an, wieviel Bandbreite kurzfristig zur
Verfügung gestellt werden kann.

Wer keinen exit node betreiben will, also einen Server, der das Ende der
Kaskade bildet und sich somit direkt zu den Zielhosts verbindet, muss
die letzte Zeile auskommentieren:

::

   ExitPolicy reject *:* # middleman only -- no exits allowed

Für jene, die eine hinter einem Router sitzen bzw. dessen Rechner ihre
Internet-IP nicht kennen, empfielt es sich, ein DynDNS-Konto (bspw. bei
`no-ip <http://www.no-ip.com/services/managed_dns/free_dynamic_dns.html>`__
oder `dyndns <http://www.dyndns.com/services/dns/dyndns/>`__) zu
registrieren und nach erfolgreicher Inbetriebnahme der Domain

::

   Address torserver.no-ip.com

in die torrc hinzuzufügen.

Ebenfalls für Leute hinter einem NAT ist Portforwarding der
entsprechenden Ports zum Server ebenfalls notwendig.

Wer seinen Tor-Server auch Leuten zur Verfügung stellen will, die hinter
einer restriktiven Firewall sitzen, ändert folgende Werte in der torrc:

::

   orport 443 
   orlistenaddress 0.0.0.0:9001
   # für dirports
   dirport 80
   dirlistenaddress 0.0.0.0:9030"

Dadurch lauscht der Server auf Port 9001 bzw. 9030, ist aber für andere
unter Port 443 (https) bzw. 80 (http) erreichbar. Danach leitet man die
Ports mit pf einfach um:

::

   rdr on $ext_if inet proto tcp from any to $IP_vom_tor_server port 443 -> $IP_vom_tor_server port 9001
   rdr on $ext_if inet proto tcp from any to $IP_vom_tor_server port 80 -> $IP_vom_tor_server port 9030

Ob der Tor-Server erfolgreich läuft, kann man an folgender Meldung in
der Logdatei sehen:

::

   [notice] router_orport_found_reachable(): Self-testing indicates your ORPort is reachable from the outside. Excellent. Publishing server descriptor.

Tipps
-----

Damit man nicht immer im Firefox die Proxy-Einstellungen mühselig ändern
muss empfehle ich den
`Turbotton <https://addons.mozilla.org/firefox/2275/>`__. Der
funktioniert aber nur, wenn der Proxy auf dem gleichen Hostläuft wie der
Browser. Falls das nicht der Fall ist, ist `SwitchProxy
Tool <https://addons.mozilla.org/firefox/125>`__ eine gute Alternative.

SILC mit OpenBSD
~~~~~~~~~~~~~~~~

Um z.B. anonym mit `SILC <SILC>`__ unterwegs zu sein, brauchen wir
vorher noch das Packet **dsocks**:

::

   # pkg_add dsocks

Wenn wir SILC nun wie folgt starten:

::

   $ dsocks-torify.sh silc

wird SILC über Tor eine Verbindung zum SILC-Server aufbauen.

Vidalia
-------

`Vidalia <http://vidalia-project.net/>`__ ist ein grafisches Interface
für Tor auf QT4. Es hilft beim Verwalten der Tor-Server und man kann
sich verschiedene Einstellungen und Optionen anschauen lassen.
Installieren kann man es nun über die Ports.

::

   # cd /usr/ports/net-mgmt/vidalia/ && make install clean

Weiterführende Literatur
------------------------

-  http://wiki.noreply.org/noreply/TheOnionRouter
-  http://wiki.noreply.org/noreply/TheOnionRouter/TorFAQ
-  http://tor.eff.org/docs/tor-doc-server.html
-  http://6sxoyfb3h2nvok2d.onion/ - Tor Hidden Wiki (nur mit Tor
   erreichbar)

* :ref:`genindex`

Zuletzt geändert: |date|

