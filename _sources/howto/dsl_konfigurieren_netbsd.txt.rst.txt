DSL konfigurieren (NetBSD)
==========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-netbsd.png

Da es keine deutsche Übersetzung für die DSL-Konfiguration unter NetBSD gibt
(zumindest fand ich keine), habe ich die Passage aus dem Handbuch übersetzt und
erweitert. Ich hoffe, es hilft einigen NetBSD-Benutzern.

Zuerst bauen wir eine manuelle Testverbindung via DSL auf. Wenn ihr einen neuen
Kernel gebaut habt, überprüft, ob ihr auch die Option ``'pseudo-device pppoe``'
im Kernel habt. (Diese Option ist im Generic-Kernel). Überprüfen könnt ihr es
ganz schnell mit folgendem Kommando:

::

   # ifconfig -C 
   tun pppoe

Wenn ihr als aus Ausgabe ``'tun pppoe``' bekommt, ist alles in Oordnung,
ansonsten heißt es, den Kernel nochmal kompilieren.

Fangen wir also mit der manuellen Testverbindung an (manuell deswegen,
damit wir einen Fehler schneller ausmerzen können, bei einer
automatischen Einwahl ist die Fehlersuche etwas aufwendiger, deswegen
dieser Weg).

Zuerst erstellen wir das pppoe0-Interface und weisen ihr eine Dummy-IP
zu, die später beim Verbindungsaufbau durch die zugewiesene ersetzt
wird. Die Befehle müßt ihr als root ausführen. Das Interface, das bei
mir zum DSL-Modem geht, heißt ``'ne0``', das interne, das ins LAN geht,
ist ``'rtk0``'.

::

   # ifconfig pppoe0 create
   # ifconfig pppoe0 inet 0.0.0.0 0.0.0.1 down

Als nächtes wird dem pppoe0-Interface die NIC, die zum DSL-Modem geht,
zugewiesen (hier ne0)

::

   # ifconfig ne0 up
   # pppoectl -e ne0 pppoe0

Nun geben wir unseren Loginnamen und das Passwort an (bei myauthname
kommt der Login hin, bei t-online die T-Onlinenummer, bei myauthsecret
das Passwort, der rest bleibt so wie er ist).

Syntax für myauthname bei T-Online:

::

   anschlusskennung+t-online-nr+#0001@t-online.de

wenn die Anschlusskennung aus über 12 Zeichen besteht, entfällt die
Raute.

::

   # pppoectl pppoe0 myauthproto=pap 'myauthname=000123456789123456789123#0001@t-online.de'<br /> 'myauthsecret=01234567' hisauthproto=none

Jetzt setzen wir die Anzahl der Fehlerversuche auf eins (sonst versucht
er ständig, sich einzuwählen mit eventuell falschen Login-Daten. Wenn er
das sieben mal macht, wird euer Anschluss bis Mitternacht gesperrt)

::

   # pppoectl pppoe0 max-auth-failure=1

Probieren wir jetzt den Verbindungsaufbau:

::

   # ifconfig pppoe0 up

Falls ein Ping auf eine Webseite noch nicht geht, müßt ihr wohl noch die
``'/etc/resolv.conf``' editieren und Nameserver eintragen, hier seht ihr
meine ``'/etc/resolv.conf``':

::

   # cat /etc/resolv.conf 
   nameserver 194.152.64.35 
   nameserver 194.25.2.132

Wenn ihr nun noch folgende Ausgabe bekommt:

::

   # pppoectl -d pppoe0
   pppoe0: state = session
   Session ID: 0x91a
   PADI retries: 0
   PADR retries: 0

das heißt, ``'state = session``', dann habt ihr eine Verbindung via DSL
mit eurer NetBSD-Kiste, was ihr nun auch mit ifconfig sehen solltet:

::

   # ifconfig -a
   rtk0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
   address: 00:50:fc:ca:da:df
   media: Ethernet none (none)
   inet 192.168.1.200 netmask 0xffffff00 broadcast 192.168.1.255
   ne0: flags=8863<UP,BROADCAST,NOTRAILERS,RUNNING,SIMPLEX,MULTICAST> mtu 1500
   address: 00:40:95:45:4a:a2
   media: Ethernet manual
   lo0: flags=8009<UP,LOOPBACK,MULTICAST> mtu 33220
   inet 127.0.0.1 netmask 0xff000000
   pppoe0: flags=8851<UP,POINTOPOINT,RUNNING,SIMPLEX,MULTICAST> mtu 1492
   inet 217.230.135.213 -> 217.5.98.56 netmask 0xff000000

Wenn ein Ping funktioniert, und ihr keine Webseiten erreichen könnt,
habt ihr ein DNS-Problem, das heißt, ihr habt vergessen, einen
Nameserver in der ``'/etc/resolv.conf``' einzutragen; wenn ein Ping
nicht geht (no route to host), aber ihr trotzdem eine IP zugewiesen
bekommt und ``'"state = session"``' erscheint, habt ihr ein
Routing-Problem; Per ``'"netstat -r"``' oder ``'"route show"``' seht ihr
eure Default-Route bzw. Gateways. Der Default-Gateway muß auf die
interne IP zeigen, in diesem Fall 217.230.135.213); (bei der
DSL-Verbindung werden zwei IPs angzeigt, eine lokale und eine von der
Gegenstelle, da DSL eine Punkt-zu-Punkt-Verbindung ist. Hier in meinem
Fall ist meine IP, unter der ich auch erreichbar bin, die
217.230.135.213).

Mit ``'route show``' seht ihr eure IP:

::

   # route show
   Routing tables
   Internet:
   Destination Gateway Flags 
   default 217.230.135.213 UG 
   loopback 127.0.0.1 UGR 
   localhost 127.0.0.1 UH 
   192.168.1.0 link#1 U 
   192.168.1.5 00:10:5a:d8:39:ee UH 
   217.5.98.56 217.230.135.213 UH

und bei ``'netstat -r``' den DNS-Namen (hier pD9E687D5.dip.t-di...)

::

   # netstat -r
   Routing tables 
   Internet:
   Destination Gateway Flags Refs Use Mtu Interface
   default pD9E687D5.dip.t-di UGS 0 34 - pppoe0
   loopback localhost UGRS 0 0 33220 lo0
   localhost localhost UH 1 0 33220 lo0
   192.168.1 link#1 UC 1 0 - rtk0
   192.168.1.5 00:10:5a:d8:39:ee UHLc 1 1302 - rtk0
   217.5.98.56 pD9E687D5.dip.t-di UH 0 0 - pppoe0 <br />
   AppleTalk:
   Destination Gateway Flags Refs Use Mtu Interface

So sollte es aussehen. Ich nehme an, das sind die häufigsten Fehler, die
auftauchen, neben dem vertippen. ;-)

Nachdem die manuelle Verbindung nun hoffentlich funktioniert, sollten
wir eine automatische Verbindung erstellen lassen, da wir uns ansonsten
jeden Tag neu einwählen müßten. Dieses Skript wird immer, wenn der
Server hochfährt, bzw. wenn die Verbindung gekappt wird, den Server neu
einwählen. Zuerst erstellen wir die ``'/etc/ifconfig.pppoe0``' Sie
sollte so aussehen:

::

   # cat /etc/ifconfig.pppoe0 
   create
   # Mark the physical interface used by this PPPoE interface up
   ! /sbin/ifconfig ne0 up
   # Let $int use ne0 as its Ethernet interface
   ! /sbin/pppoectl -e ne0 $int
   # Configure authentication
   ! /sbin/pppoectl $int myauthproto=pap 'myauthname=000123456789320012345678#0001@t-online.de' 'myauthsecret=01234567' hisauthproto=none
   # Configure the PPPoE interface itself. These addresses are magic
   # meaning we don't care about either address and let the remote
   # ppp choose them.
   0.0.0.0 0.0.0.1 up

Dann noch ein

::

   # chmod 700 ifconfig.pppoe0

Nun erstellen wir das PPP-Verzeichnis in ``'/etc/``':

::

   # mkdir /etc/ppp

Die beiden Skripte ``'ip-up``' und ``'ip-down``' kommen in das
``'/etc/ppp/``'-Verzeichnis:

::

   # cat /etc/ppp/ip-up 

::

   #! /bin/sh
   /sbin/route add default $5

Mit dem ip-up wird beim Aufbauen der Verbindung die Default-Route auf
die interne IP der DSL-Verbindung gesetzt, dies muß jedesmal gemacht
werden, da bei jedem neuen Verbindungsaufbau eine neue IP zugewiesen
wird;

::

   # cat /etc/ppp/ip-down

::

   #! /bin/sh
   /sbin/route delete default $5

Hier wird die Route beim Verbindungsabbau wieder gelöscht;

Jetzt muß sie nur noch für root ausführbar gemacht werden:

::

   # chmod 700 ip-down ip-up

Am Ende noch ein ``'ifwatchd=YES``' in die ``'/etc/rc.conf``' einfügen
und fertig. Jetzt sollte sich der Server nach einem Reboot oder beim
Verbindungsabbau wieder einwählen; ifwatchd ist im übrigen dafür
zuständig, daß die ip-up und -down-Skripte ausgeführt werden.

Nun solltet ihr ohne Probleme mit dem Router Online gehen können,
einwaehlen sollte er sich automatisch; als nächstes konfigurieren wir
NAT, damit auch alle Rechner hinter dem Router Online gehen können.

Zuerst setzen wir in der ``'/etc/sysctl.conf``' folgenden Wert ein:

::

   # Obey interface MTUs when calculating MSS
   net.inet.tcp.mss_ifmtu = 1
   net.inet.ip.forwarding = 1

und passen danach noch die ``'/etc/ipnat.conf``' an bzw. erstellen sie
mit folgenden Werten:

::

   map pppoe0 192.168.1.0/24 -> 0/32 portmap tcp/udp 44000:49999 mssclamp 1440
   map pppoe0 192.168.1.0/24 -> 0/32 mssclamp 1440

Meine IP-Range daheim ist 192.168.1.0, mit der Netmask 255.255.255.0
bzw. /24, diese Werte müßt ihr natürlich an eure anpassen.

Nun erstellen wir noch die ``'/etc/ipf.conf``' und zum testen lassen wir
alle Pakete durch den Paketfilter passieren:

::

   pass in from any to any
   pass out from any to any

In die ``'/etc/rc.conf``' müßt ihr noch folgendes eintragen, damit die
Firewall und NAT immer gestartet werden:

::

   ipfilter=YES
   ipnat=YES

Nach einem Neustart oder einem ``'/etc/rc.d/ipfilter restart``' und
``'/etc/rc.d/ipnat restart``', sollten die Clients Online gehen können.
Ihr müßt natürlich noch den Paketfilter anpassen, sonst ist euer Router
nach außen offen wie ein Scheunentor.

Dial on Demand
--------------

Zwar findet man in der PPPoE Dokumentation von NetBSD die Angabe, daß
für PPPoE ein Dial on Demand möglich ist, allerdings scheint es nicht
ohne weiteres möglich zu sein, dieses umzusetzen. Vergleiche hierzu den
[http://www.bsdforen.de/showthread.php?p=66477 Thread in BSDForen.de].
<br />Stand dieser Information: Januar 2005

Weblinks
--------

-  http://www.netbsd.org/Documentation/network/pppoe -
   PPPoE-Dokumentation von NetBSD
-  http://www.dslweb.de

* :ref:`genindex`

Zuletzt geändert: |date|

