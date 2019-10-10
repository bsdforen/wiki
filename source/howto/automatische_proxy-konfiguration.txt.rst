Automatische Proxy-Konfiguration
================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Dies ist eine Beschreibung, wie man verschiedenen Browsern ermöglicht,
verschiedene Proxies zu verwenden, ohne daß diese explizit konfiguriert werden
müssen.

Einleitung
----------

Bisher habe ich das unter Debian GNU/Linux und FreeBSD 6.0 getestet. Da
der Apache, der Nameserver und der DHCP Server auch unter den anderen
\*BSDs verfügbar ist, sollte es dort auch laufen. Verwendet habe ich
folgende Versionen:

-  Apache 2.0.x
-  ISC Bind 9.x
-  ISC DHCP Server 3.x

Prinzipiell sollte jeder Webserver funktionieren, da ja nur eine
statische Seite ausgeliefert wird.

Man kann mittels
`WPAD <http://www.web-cache.com/Writings/Internet-Drafts/draft-ietf-wrec-wpad-01.txt>`__
die URL für das automatische Konfigurieren des Proxies bekannt machen.
Wie die automatische Proxy Config auszusehen hat, steht
`hier <http://wp.netscape.com/eng/mozilla/2.0/relnotes/demo/proxy-live.html>`__
bzw. unter `wpad.dat
erstellen <Automatische Proxy-Konfiguration#wpad.dat_erstellen>`__. Der
Vorteil besteht darin, daß die User nur noch einstellen müssen, dass
eine automatische Config verwendet wird. Die URL dieser Config ermittelt
der Browser aber selbständig. Natürlich kann auch die URL des Proxy
Config Scripts angegeben werden.

Webserver Konfiguration
-----------------------

Dem Webserver muß der MIME Typ für die Datei **wpad.dat** bekannt
gemacht werden. Beim Apache 2 kann man mittels *AddType* neue Typen
definieren
(`Hier <http://httpd.apache.org/docs/2.0/mod/mod_mime.html#addtype>`__
nachzulesen). In diesem Fall sollte folgende Zeile eingetragen werden:

::

   AddType application/x-ns-proxy-autoconfig wpad.dat

DHCP Server
-----------

Beim ISC DHCP Server muß als erstes eine neue Option eingetragen werden.
Dies geschieht durch folgenden Eintrag in der **/etc/dhcp3/dhcpd.conf**.

::

   option wpad code 252 = text;

Danach kann die Option im jeweiligen Subnetz verwendet werden. Der
Eintrag dafür lautet:

::

   option wpad "http://10.0.0.1/wpad.dat";

Wobei 10.0.0.1 die IP Adresse des Webservers ist.

DNS Server
----------

Bei ISC Nameserver Bind sind nun noch zwei Anpassungen notwendig, um die
unterschiedlichen Methoden der Browser zu unterstützen.

-  Es wird ein A Record erzeugt, der den Hostnamen wpad auf den
   Webservernamen auflöst.

::

   wpad          IN A 10.0.0.1

-  Es wird ein Service Eintrag ergänzt, der die URL bekannt macht.

::

   wpad.tcp      IN SRV 0 0 80   10.0.0.1
                 IN TXT "service: wpad:http://10.0.0.1/wpad.dat"

Danach sollten der DHCP Server und der DNS Server ihre Konfiguration neu
einlesen. Ab jetzt können die Browser auf automatische
Proxykonfiguration umgestellt werden. Bei Firefox gibt es beispielsweise
eine Auswahl *Auto-detect proxy settings*, die gewählt werden sollte.

wpad.dat erstellen
------------------

Die **wpad.dat** enthält lediglich die Java Script Funktion
*FindProxyForURL(url, host)*. *url* enthält den komplette URL, die
Angefragt wird und *host* nur den Hostnamen. Um für das lokale Netz
keinen Proxy und für alle anderen Rechner einen Proxy zu verwenden, ist
beispielsweise folgende Konfiguration möglich.

::

   function FindProxyForURL(url, host) {
    if(isPlainHostName(host)) {
     return ("DIRECT");
    }
    else if(isInNet(host, "10.0.0.0", "255.0.0.0")) {
     return ("DIRECT");
    }
    return ("PROXY 10.0.0.2:3128");
   }

Welche Funktionen verwendet werden können und was dabei u. U. zu
beachten ist, kann unter `Netscape Proxy
Config <http://wp.netscape.com/eng/mozilla/2.0/relnotes/demo/proxy-live.html>`__
nachgelesen werden.

Wer möchte kann die Konfiguration natürlich auch dynamisch erzeugen.

Browser
-------

Ein paar Anmerkungen zu Problemen/Besonderheiten, die bei verschiedenen
Browsern zu beachten sind.

Mozilla
-------

Bei Mozilla ist nichts zu beachten, allerdings eignet er sich sehr gut
zum Debuggen des Scripts. Um Probleme zu finden, kann man bei Mozilla
die Java Script Console zu öffnen. Parallel kann man die **wpad.dat**
unter *Edit/Preferences/Advanced/Proxies* neu laden. Falls Fehler in der
**wpad.dat** auftauchen, lassen diese sich dort finden.

Opera & Konqueror
-----------------

Damit das Ganze mit Opera funktioniert, muß dort ein Link **wpad.pac**
(oder ähnlich, wichtig ist die Endung *.pac*) angelegt werden, da der
MIME-Typ, den Opera verwendet, eine **wpat.dat** nicht richtig zuordnet
und entsprechend nicht auswerten kann. Für den Konqueror gilt das
gleiche.

Internet Explorer
-----------------

Bisher keine Probleme, in der Default Einstellung wird die Config
gefunden und eingebunden.

Original Eintrag: 2008/04/27 17:56 (Externe Bearbeitung)

* :ref:`genindex`

Zuletzt geändert: |date|

