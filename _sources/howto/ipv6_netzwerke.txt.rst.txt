IPv6-Netzwerke mit FreeBSD als Router
=====================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

.. note::

  Dieser Artikel ist ursprünglich auf http://wiki.yamagi.org erschienen und war
  dort unter die Creative Commons Attribution-Noncommercial-No Derivative Works
  2.0 Germany Lizenz gestellt. Am 23. Februar 2008 habe ich (Yamagi) als Autor
  diesen Artikel relizensiert und unter den Bedingungungen des BSDForen.de-Wiki
  (http://wiki.bsdforen.de/wiki:lizenz) hier eingestellt. Die Weiterentwicklung
  findet ausschließlich hier statt.

Seit langer Zeit geistert IPv6 - früher IPng - durch die Medien und wird
als nächster großer Sprung im Internet angepriesen. Dies ist sicherlich
nicht ganz falsch, aber derzeit noch eine gewagte Aussage. Noch ist nur
ein kleiner Teil des Netzes über IPv6 zu erreichen. Die meisten LANs
sind sogar nur IPv4-Netze. Trotzdem kann es schon jetzt interessant
sein, seine eigenen Netze auf IPv6 zu migrieren und sich per IPv6 ans
Internet anzubinden.

Vorteile von IPv6 sind:

-  Öffentliche IP-Adressen in nahezu unbegrenzter Anzahl sind völlig
   kostenlos verfügbar. So bekommt jeder Interessierte ein /48er Subnetz
   mit milliarden Adressen, welche von jedem Ort der Welt aus erriechbar
   sind. Dies bedeutet, dass jeder Rechner im LAN eine öffentliche IP
   bekommen kann, was Dinge wie NAT und Portforwarding überflüssig
   macht.
-  Vollautomatische Konfiguration der Clients. Was bei IPv4 nur per DHCP
   gelöst werden kann, wodurch man sich einen fehleranfälligen
   Mechanismus ins Netz holt, ist bei IPv6 direkt in das Protokoll
   implementiert. Der alte Traum vom "Einschalten und geht" wird damit
   endlich wahr.
-  Viel größere Subnetze können einen Großteil der eigenen Router
   überflüssig machen. Sicherheit kann stattdessen viel eleganter
   mittels Brücken oder VLans erreicht werden.
-  IPSec ist in das Protokoll integriert und kein später eingefügter
   Aufsatz - man könnte auch "Hack" sagen - wie bei IPv4

In diesem Artikel wird daher kurz erläutert, wie man ein vorhandenes LAN
von IPv4 auf IPv6 umstellen kann. Themen sind dabei auch Dinge wie IPv4
-> IPv6 Übersetzung und Firewalls. Dieser Artikel setzt gute Kenntnisse
über Netze im Allgemeinen und über FreeBSD (eine Umsetzung auf ein
anderes BSD oder Linux dürfte trivial sein) vorraus. Auch sollte man
bereit sein, Doku und RFCs [1]_ zu lesen sowie ein wenig selbst zu
denken.

Anmerkungen zu Beginn
---------------------

Zu Beginn müssen noch einmal zwei Dinge deutlich gemacht werden:

#. Auch wenn die Entwicklung von IPv6 bereits in den 90ern begann und
   die RFCs längst in der finalen Version vorliegen, leidet das
   Protokoll immer noch unter Kinderkrankheiten. So sind einige
   Infrastrukturdaemons fehlerhaft, es wurde erst vor kurzem eine
   kritische Lücke im Protokoll gefunden und einige Fragen sind noch
   nicht geklärt. Dennoch kann IPv6 ohne große Bedenken eingesetzt
   werden. Wenn man weiß, wo die Probleme liegen, kann man auf sie
   Rücksicht nehmen. Bei nicht geklärten Dingen kann man improvisieren.
   Vor allem wird sich an den RFCs nichts bedeutendes mehr ändern.
   Pionierarbeit ist nicht immer angenehm, aber man hat hinterher das
   Gefühl etwas für die Welt getan zu haben. Und vor allem, wenn niemand
   mit etwas anfängt, kann sich auch nie etwas neues durchsetzen. Ganz
   entscheidend: Wer sich bereits jetzt mit IPv6 auseinandersetzt, muss
   zwar ein wenig Arbeit investieren, hat aber dafür ein modernes und
   stabiles Netz. Wenn die anderen in einigen Jahren mit IPv6 anfangen,
   es gehypt und zu einem "must have" wird, kann man sich selbst in Ruhe
   zurücklehnen und milde über deren Fehler lächeln :)
#. Wer mit IPv6 arbeitet, ist meist sehr schnell von dem Protokoll
   überzeugt. Die oben unter 1. gewachsenen Zweifel weichen schnell
   Begeisterung, man möchte am liebsten das komplette Netz so schnell
   wie möglich migrieren. Leider ist dies nicht so einfach. Auch wenn
   man IPv4 weitgehend ersetzen kann, ist dies leider noch nicht
   komplett möglich:

   -  Zwar ist die meiste Open Source Software inzwischen
      IPv6-kompatibel, nur muss man eventuell komplexe Systeme auf neue
      Versionen umstellen um sie nutzen zu können. Dies ist nicht immer
      zeitnah möglich, sondern eine Sache die sich über Jahre hinziehen
      kann.
   -  Bei proprietärer Software sieht es ähnlich aus. Hier kommen noch
      die Kosten für aktuelle Lizenzen hinzu, die man nicht selten
      meiden möchte. Mittelfristig wäre die Umstellung auf Open Source -
      soweit möglich - eine Lösung, andernfalls muss gewartet werden,
      bis die Software im normalen Lebenszyklus getauscht wird.
   -  Nicht jedes Betriebssystem unterstützt IPv6 (vollständig). Während
      die Unterstützung bei den BSDs sehr gut ist auch Linux dort
      inzwischen recht ausgereift, hat Windows erst vor kurzem den
      Sprung geschafft. [2]_
   -  Zu den BSDs sei gesagt, dass nur relativ alte Versionen kein IPv6
      können. Hier wäre allgemein über ein Update nachzudenken. Weitere
      Informationen finden sich in der Doku.
   -  Bei Linux ist IPv6 sehr von der Distribution abhängig. Einige
      hatten sehr früh IPv6, als es noch nicht einmal offizieller
      Bestandteil des Kernels war, andere kommen bis heute ohne daher.
      Hier muss im Einzelfall geprüft und gegebenenfalls die
      Distribution oder der Kernel aktualisiert werden. [3]_
   -  Bei Windows war IPv6 lange eine Baustelle. Für Windows 2000 und XP
      gibt es experimentelle Implementierungen. [4]_ Diese sind jedoch
      weit von der benötigten Stabilität entfernt und zudem
      unvollständig. Jeder muss für sich selbst entscheiden, in wie weit
      er sie einsetzen möchte. Aber durch die immer noch sehr hohe
      Verbreitung dieser beiden Systeme liegt hier sicher eines der
      Haupthemmnisse für IPv6. Windows 2003 und Windows XP x64 bringen
      beide eine Implementation mit, die von Microsoft als "final"
      gekennzeichnet ist. Sie funktioniert stabil, implementiert
      allerdings nur vorläufige Versionen der RFCs. Dies kann unter
      Umständen Probleme machen. Auch wenn man hier wieder abwägen muss,
      kann man dennoch sagen, dass diese Betriebssyteme meist ohne große
      Probleme mit IPv6 eingesetzt werden können. Erst Windows Vista und
      der kommende Longhorn Server implementieren IPv6 wirklich
      komplett. [5]_ Hier steht der Migration nichts mehr im Wege.
   -  Nicht alle Hardware im Netzwerk (Layer 3 Switches, Firewalls,
      Router) kann IPv6. Da man diese nicht mal schnell austauschen
      kann, liegt hier ein weiterer großer Stein im Weg.

Die realistische Einschätzung ist daher, dass IPv4 und IPv6 noch über
viele Jahre koexistieren werden. Die Frage für das eigene Netz ist, wie
dies geschehen soll.

IPv4 und IPv6 im gleichen Netz
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Beides sind Protokolle auf der dritten Schicht des OSI-Modells [6]_. Sie
können ohne Probleme koexistieren und sich Schicht 1 und 2 teilen. Bevor
man sich über eine Methode der Koexistenz nähere Gedanken macht, sollte
man erst einmal überprüfen, wie viel IPv4 denn im eigenen Netz
verbleiben muss:

-  Wie viele Dienste unterstützen ausschließlich IPv4?
-  Welche Betriebssysteme setze ich ein und unterstützen sie IPv6?
-  Habe ich FreeBSD Jails, die derzeit nur IPv4 können?
-  Habe ich auf IPv4 beschränkte Hardware?

Nach dieser Bestandsaufnahme gibt es zwei Möglichkeiten:

#. Dualstack. Die Maschinen mit IPv4-Bedarf haben zusätzlich zu der
   IPv6-Anbindung eine IPv4-Adresse. Dies ist mit allen
   Implementierungen problemlos möglich. So können diese IPv6-fähige
   Dienste auch hierüber anbieten, der Rest erfolgt über das klassische
   IPv4.
#. Getrenntes Netz. Alle IPv4-Dienste kommen in einen eigenen
   Netzbereich, in welchem nur IPv4 verwendet wird. Dies ist sinnvoll,
   falls es sich um sehr viele Maschinen handelt. Von Nachteil ist
   jedoch, dass man hier später nur schwerlich weitere Migrationen
   vornehmen kann.

In beiden Fällen erfolgt eine Übersetzung von IPv4 -> IPv6, wodurch ein
(großer) Teil des Netzes ausschließlich IPv6 nutzen kann.

Voraussetzungen
~~~~~~~~~~~~~~~

Um IPv6 wie hier vorgestellt nutzen zu können, sind einige
Voraussetzungen zu erfüllen:

-  Für die Verwaltung des Netzwerks und die Übersetzung von IPv4 -> IPv6
   wird eine dedizierte Maschine benötigt. Diese darf keine Dienste
   anbieten und muss das Internet per IPv4 erreichen können. Falls
   vorhanden eignet sich hierfür ein Router auf der Basis eines
   entsprechend Betriebssystems. Gerade wenn große Mengen Daten von IPv6
   zu IPv4 übersetzt werden sollen, muss jene entsprechende Leistung
   bieten.
-  Ein Tunnel ins IPv6-Netz und ein IPv6-Subnetz. Beides wird z.B.
   kostenlos von Sixxs [7]_ angeboten, an dessen Beispiel hier
   vorgegangen wird.
-  Ein DNS-Server. Er kann auch von dem eigenen IPv4-Provider genommen
   werden, wobei ein eigener für eine eigene lokale Zone sehr zum
   empfehlen ist. Dazu später mehr.

Der Plan
--------

Hier wird gezeigt werden, wie man einen IPv6-Router für das eigene Netz
einrichtet. Als Betriebssystem kommt FreeBSD 6.2 zum Einsatz sowie
einige Programme aus den Ports. Als Hardware für dieses Beispiel wurde
ein Server mit AMD Sempron 2000+ (K7 Klasse), 512 MB RAM und 3Com
Gigabit NIC genutzt. Eine deutlich kleinere Maschine ist meist
ausreichend. Zuerst wird ein Tunnel ins IPv6-Netz aufgebaut. Nun kann
man vom eigenen Server aus das IPv6-Netz erreichen. Im zweiten Schritt
wird dieser als lokaler Router mit Advertisement-Daemon eingerichtet,
wodurch man sein lokales Netz ans IPv6-Netz anbindet. Zuletzt folgt die
Übersetzung von IPv6 zu IPv4 inklusive DNS-Proxy.

Vorbereitung der Maschine
-------------------------

Zuerst wird auf dem Rechner schlicht FreeBSD 6.2 installiert. Ich
empfehle die Option "Nutzer" aka "User", da diese ein schlankes System
bestehend aus der Base und der Doku installiert. Nach der Installation
wird die übliche Konfiguration vergenommen. Ein eigener Kernel ist nicht
erforderlich, falls dennoch einer gebaut werden soll, müssen folgende
Dinge enthalten sein oder später per Modul geladen werden:

-  IPv4
-  IPv6
-  if_tun
-  faith
-  pf

Nun darf die Maschine keinen eigenen Dienste nach außen anbieten, wenn
diese auf einem Port liegen, der später für die IPv6 -> IPv6 Übersetzung
benötigt wird. Wenn SSH später übersetzt werden soll, muss man den SSHd
von Port 22 auf einen andere verlegen - beispielsweise Port 2222. Ebenso
kann ein lokaler DNS-Server z.B. mit BIND9 eingerichtet werden. Dieser
darf aber nicht auf Port 53 lauschen. Stattdessen kann man ihn z.B. auf
Port 5353 umbiegen.

Als Abschluss der Vorbereitungen muss der Rechner das klassische
IPv4-Internet erreichen können. Hierzu sollte eine feste IP Adresse
vergeben und die sonstigen Einstellungen wie immer gemacht werden. Zudem
muss ein NTP-Service laufen, da der Tunnel eine sehr genau gehende Uhr
benötigt [8]_.

Aufbau des Tunnels zum IPv6-Netz
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Der Tunnel zum IPv6-Netz ist dank guter Werkzeuge schnell aufgebaut.
Hierzu installiert man aus den Ports ``net/sixxs-aiccu``. Aiccu ist die
beste Lösung. Alternativ kann der nicht mehr weiterentwickelte
sixxs-heartbeatd eingesetzt werden, dieser ist jedoch veraltet. Zudem
ist er schwer durch Firewalls und NAT hindurch einzurichten. Ein
statischer Tunnel empfiehlt sich nur, wenn man selbst nach außen hin
eine statische IPv4-Adresse hat, was in den seltensten Fällen gegeben
sein dürfte.

Nach der Installation findet man unter ``/usr/local/etc/`` die
``aiccu.conf``. Diese wird kurz angepasst. Eingetragen werden der eigene
Nutzername und das Passwort. Automatischer Login sollte aktiviert
werden, sonst ist das Starten per RC-Skript nicht möglich. Das Interface
ist entgegen aller Doku seitens Sixxs tunX und nicht gifX! Nun wird der
Tunnel mit ``sixxs-aiccu start /usr/local/etc/aiccu.conf`` gestartet. Er
sollte sich anmelden und einen Tunnel aufbauen. Zur Kontrolle wird
versucht www.kame.net per ping6 zu pingen. Bei Erfolg ist alles ok. Nun
noch ein ``sixxs_aiccu_enable="YES"`` in die rc.conf, Aiccu wieder per
strg-c beenden und durch sein RC-Skript neu starten. Noch ein weiterer
Test per ping und alles ist OK.

Kurze Anmerkungen zur Firewall
------------------------------

Der Server ist nun über eine statische IPv6-Adresse heraus aus dem
Internet erreichbar. Später wird dies für jeden Rechner im ganzen Netz
gelten. Eine gute Firewall ist daher wichtig und sollte an dieser Stelle
eingerichtet werden. Sinnvoll ist dabei PF zu nutzen, da dies sehr gut
mit IPv6 umgehen kann. [9]_ Alternativ ist eventuell IPFW möglich.

Auf dem Tunnelinterface sollte eingehend alles gesperrt werden. Dies
gilt vor allem für Port 53 - dazu später mehr. Auf jeden Fall muss
jedoch ICMP6 an alle Ziele hineingelassen werden, da das Netz sonst
nicht funktioniert und Sixxs den Tunnel eventuell sogar sperrt. Ob der
eigene Server per IPv6 pingbar ist, kann von http://www.ipv6tools.com/
aus geprüft werden.

Einrichten des lokalen Routers
------------------------------

Nachdem der Server nun mit dem IPv6-Netz verbunden ist, soll dieser zum
lokalen Router ausgebaut werden. Hierzu wird der NIC des Rechners erst
einmal per rc.conf eine feste IPv6-Adresse zugewiesen. Zwar wurde von
Sixxs ein /48er Präfix zugewiesen, trotzdem wird hier dringend geraten
ein /64er Subnetz abzutrennen und nur mit diesem zu arbeiten. Alles
andere kann später sehr großen Ärger machen. Die erste Adresse im
Subnetz sollte der NIC zugewiesen werden. Zudem wird in der rc.conf der
IPv6 Gateway eingeschaltet.

Nach diesen Vorarbeiten wird nun der Router Advertisement Daemon
eingerichtet. Dieser verteilt das Präfix und die Defaultroute an die
Clients im Netz, sodass diese sich vollautomatisch eine passende
IPv6-Adresse generieren können und ans Netz angebunden sind. FreeBSD
nutzt hierfür den Daemon des KAME-Projektes [10]_: rtadvd. Leider ist
dieser in einigen Belangen fehlerhaft oder unausgereift - je nach
Sichtweise. Dennoch funktioniert er gut. Alternativ kann aus den Ports
der von Linux stammende Daemon radvd eingesetzt werden.

Es muss zuerst einmal die ``/etc/rtadvd.conf`` eingerichtet werden.
Diese ist im legendären Termcap-Format aufgebaut. Hier wird die NIC des
Routers eingetragen und ihr Präfix definiert. Dies sieht dann in etwa so
aus:

::

   sk0:\
       :addrs#1:addr="xxxx:xxxx:xxxx:xxxx::":prefixlen#64:tc=ether:

Der NIC sk0 wird das Präfix xxxx:xxxx:xxxx:xxxx:: zugewiesen. Die Länge
ist 64. Diese Konfiguration ist ausreichend, auch wenn noch deutlich
mehr Parameter definiert werden können. Siehe dazu die Manpage zu
rtadvd.conf. Nun wird der Daemon in der rc.conf aktiviert und - nicht
nötig aber sehr wichtig um fehler zu vermeiden - seine Interfaces
angegeben. Dies wäre im Beispiel "sk0". Nun den Daemon starten und
fertig ist der Router.

Hier noch einige Anmerkungen zu den Macken von rtadvd:

-  Wenn der Dienst anmerkt, dass er eine Netzwerkkarte in der
   Konfiguration nicht finden kann, aber korrekte Präfixe verteilt, ist
   diese Angabe schlicht falsch.
-  Das Erkennen echter Interfaces klappt nur selten. Daher müssen die
   Interfaces manuell angegeben werden.
-  Die NIC muss eine IP-Adresse mit dem Präfix haben, das sie verteilen
   soll.
-  Wenn alle Stricke reißen und das Ding sich weigert mitzuarbeiten,
   kann manchmal ein Neustart des Rechners helfen.

**Hinweis des Bummibären:** aus den Ports /usr/ports/net/radvd
verwenden, der tut deutlich besser.

Konfiguration der Clients
-------------------------

Die Konfiguration der Clients ist simpel. Unter FreeBSD muss zusätzlich
oder anstelle - wie gewünscht - der IPv4-Konfiguration
``ipv6_enable="YES"`` in die rc.conf. Beim nächsten Start wird IPv6
automatisch konfiguriert. Manuell kann dies mittels ``rtsol $interface``
gemacht werden. Nun sollte der Client eine IPv6-Adresse mit dem
korrekten Präfix zugewiesen bekommen, die Defaultroute korrekt gesetzt
werden und das IPv6-Internet erreichbar sein.

Noch einige Anmerkungen zu rtsol:

-  IPv6 verhält sich wie IPv4. Eine Maschine darf nur mit einer NIC im
   gleichen Subnetz hängen, niemals mit zwei NICs. Daher kann sich rtsol
   immer nur an eine NIC binden, wenn mehrere vorhanden sind muss diese
   klar definiert werden.
-  Wenn man mehrere NICs hat, die am gleichen Switch oder im gleichen
   physikalischen Netz hängen, wird rtsol nur auf einer von diesen
   funktionieren. Leider ist keine Regelmäßigkeit feststellbar, welche
   dies ist. Herausfinden kann man dies durch Aufruf von "rtsol -D
   $nic". Schlägt der Vorgang fehl, findet sich etwas wie "received RA
   from fe80::20a:5eff:fe53:6487 on an unexpected IF(nfe0)" in der Log.
   Die NIC, die die Antwort gibt, ist das von rtsol zu konfigurierende
   Interface.

Adressübersetzung IPv6 -> IPv4
------------------------------

Mit Adressübersetzung lassen sich IPv4-Only Host über eine IPv6-Adresse
erreichen. Dies geschieht unter FreeBSD mit Hilfe des Pseudointerfaces
"faith0" und dem zugehörigen Daemon faithd. Zudem wird ein DNS-Proxy
benötigt, der die DNS-Anfragen so ändert, dass ein entsprechender
AAAA-Record vorhanden ist. Da faithd auf die Ports binden muss, die er
übersetzen soll, dürfen hier keine Dienste laufen. Daher musste oben SSH
verlegt werden. Für die Adressübersetzung wird ein eigenes Präfix der
Länge 96 benötigt. 96, da 96Bit IPv6-Präfix + 32Bit IPv4-Adresse =
128Bit IPv6-Adresse ergeben [11]_. Dieses Präfix darf auf keinen Fall
einer global erreichbaren Adresse entsprechen, sonst gibt es große
Probleme. Das Präfix von Sixxs darf also nicht genutzt werden. Wir
werden hier den Vorschlag der Manpage zu faithd nutzen:
3ffe:501:4819:ffff:: [12]_

Zuerst wird der DNS-Proxy eingerichtet. Hierzu kommt der vom NetBSD
entwickelte totd zum Einsatz. Er wird über die Ports aus \`dns/totd\`
installiert. Hinterher wird die /usr/local/etc/totd.conf angepasst:

::

   forwarder 192.168.0.1 port 53
   prefix  3ffe:501:4819:ffff::
   interfaces sk0
   retry 300

Eigentlich ist dies selbsterklärend. "forwarder" ist der reale
DNS-Server, "interfaces" gibt die NICs an, auf der der Daemon lauschen
soll. Nachdem dies passiert ist, wird noch schnell ``totd_enable="YES"``
in die rc.conf eingetragen und dann der Daemon gestartet. Nun kann man
ihn unter der IPv4 und IPv6-Adresse des Routers erreichen. Eine Anfrage
per dig zeigt uns, dass vorhandene AAAA-Records übergeben werden, dies
ist z.B. bei www.kame.net der Fall. Sind keine vorhanden, wird ein
Eintrag aus dem Präfix und dem A-Record erzeugt.

Nun noch diesen DNS in die Namensauflösung der Clients eintragen und
alles ist OK. Allerdings dürfen seine Rückmeldungen auf gar keinen Fall
in die globale DNS-Wolke gelangen. Daher wird hier zu Sicherheit noch
einmal nachdrücklich empfolen:

-  totd darf nicht auf dem Tunnelinterface lauschen
-  Auf dem Tunnel sollte Port 53 eingehend blockiert werden

Nun wird faithd konfiguriert. Hierzu muss ein Eintrag in die rc.conf
vorgenommen werden: ``ipv6_faith_prefix="3ffe:501:4819:ffff::"`` Dies
legt beim Systemstart die benötigten Routen und setzt die Sysctl
entsprechend. Nun sollte ein Neustart des Systems gemacht werden,
alternativ kann auch nach der Manpage von faithd vorgegangen werden um
die Routes zu setzen.

Zuletzt noch faithd-Daemons für die zu übersetzenden Dienste starten:
``faithd 80`` übersetzt z.B. alles auf Port 80 (HTTP). Dies Daemons
müssen für jeden Service manuell gestartet werden und forken in den
Hintergrund. Automatisiert kann dies z.B. durch Einträge in die rc.local
werden oder durch ein eigenes RC-Skript.

Es lassen sich eigentlich alle Dienste erstaunlich gut übersetzen. Dies
gilt auch für fast alle Internetseiten. Einige Dinge machen jedoch
Probleme:

-  Einige Internetseiten, wie z.B. http://www.tvtv.de oder Teile von
   Ebay (kann durch das Erlauben von DNS über TCP behoben werden)
-  SSH kann selten IPv4 <-> IPv6 Probleme bekommen (nicht mehr ab
   FreeBSD 6.3)
-  NFS zickt (ab FreeBSD 6.3 behoben)
-  NTP funktioniert nicht. Dies kann durch NTP over IPv6 umgangen
   werden.

Abschließende Anmerkungen
-------------------------

Nach dem Abschluss der obigen Arbeiten sind im LAN IPv6-Only Hosts
möglich, die dennoch IPv4-Hosts erreichen können. Damit dies konfortabel
funktioniert, bietet sich eine eigene lokale Zone im DNS an. Diese kann
z.B. "yamagi.lan" heißen. Hier trägt man die IPv4-Hosts ein und den
IPv6-Router. Es bleiben jedoch 2 Probleme, die jedoch grundsätzlich
lösbar sind:

-  IPv6-Hosts können von IPv4-Hosts aus nicht erreicht werden. Dies kann
   man durch eine entsprechende Netzstruktur oder IPv4 -> IPv6
   Übersetzung umgehen, was beides jedoch sehr aufwändig ist.
-  IPv6-Hosts mit dynamischer IP sind nur schlecht zu erreichen. Eine
   Möglichkeit wären Skripte, die vom Client aus dem DNS die Adresse
   mitteilen. Nur dies ist sicherheitstechnisch indiskutabel und
   außerdem fehleranfällig. Eine gute Lösung hierzu gibt es abgesehen
   von DHCPv6 [13]_ nicht, was wiederrum ein übelster Hack ist.

.. [1]
   http://tools.ietf.org/html/rfc2460

.. [2]
   http://www.ipv6.org/impl/windows.html

.. [3]
   http://mirrors.bieringer.de/Linux+IPv6-HOWTO-de/

.. [4]
   http://research.microsoft.com/msripv6/

.. [5]
   http://www.microsoft.com/technet/network/ipv6/default.mspx

.. [6]
   http://www.its05.de/html/osi-modell.html

.. [7]
   http://www.sixxs.net/

.. [8]
   http://www.sixxs.net/faq/connectivity/?faq=heartbeat

.. [9]
   http://www.openbsd.org/faq/pf/de/

.. [10]
   http://www.kame.net

.. [11]
   http://tools.ietf.org/html/rfc3056

.. [12]
   http://www.freebsd.org/cgi/man.cgi?query=faithd&apropos=0&sektion=0&manpath=FreeBSD+6.2-RELEASE&format=html

.. [13]
   http://tools.ietf.org/html/rfc3315

* :ref:`genindex`

Zuletzt geändert: |date|

