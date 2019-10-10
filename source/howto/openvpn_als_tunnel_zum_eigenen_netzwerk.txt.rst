OpenVPN als Tunnel zum eigenen Netzwerk
=======================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Mit Hilfe eines VPNs kann man bequem auf Resourcen des eigenen Netzes zugreifen
und sogar seinen kompletten Traffic über den Tunnel routen, so dass man
Portsperren in öffentlichen Netzen (zum Beispiel im Uninetz) „umgehen“ kann. In
diesem HowTo wird beschrieben, wie man dies mit Hilfe von OpenVPN tun kann.

Installation
------------

Es ist OpenVPN (server- und clientseitig), sowie die Bash Shell (nur
serverseitig) zu installieren.

-  `security/openvpn <https://www.google.com/search?q=security/openvpn&btnI=lucky>`__
   (FreeBSD Ports)
-  `shells/bash <https://www.google.com/search?q=shells/bash&btnI=lucky>`__
   (FreeBSD Ports)

Gearbeitet wird in dem gesamten HowTo mit der Bash Shell, da die
easy-rsa Skripte diese verlangen. Falls keine dauerhafte Umstellung der
Shell gewünscht ist, wechselt man die aktuelle Shell mittels::

  # bash

Erstellen der Zertifikate
~~~~~~~~~~~~~~~~~~~~~~~~~

Nun muss für den Server, sowie für jeden Client jeweils ein Zertifikat
erstellt werden. Generiert werden diese auf dem Server. Für die
Zertifikatserstellung nutzen wir easy-rsa. Easy-rsa ist eine Sammlung
von Skripten, welche die Erstellung der Zertifikate um einiges
erleichtern. Diese Skripte müssen zuerst noch installiert werden. Unter
DESTDIR ist das gewünschte Installationsverzeichnis anzugeben::

  # cd /usr/local/share/doc/openvpn/easy-rsa/2.0 
  # make install DESTDIR=/root/easy-rsaVPNgw/ 
  # cd /root/easy-rsaVPNgw 

Nun bearbeiten wir das Skript vars. Dieses Skript enthält Variablen, welche zur
Erstellung der Zertifikate nötig sind. Die Schlüsselgröße passen wir noch an
indem wir sie von 1024bit auf 2048bit heraufsetzen.

::

   export KEY_SIZE=2048

Danach ersetzten wir noch die Werte

::

   export KEY_COUNTRY="US"
   export KEY_PROVINCE="CA"
   export KEY_CITY="SanFrancisco"
   export KEY_ORG="Fort-Funston"
   export KEY_EMAIL="me@myhost.mydomain"

durch geeignetere. Zum Beispiel:

::

   export KEY_COUNTRY="DE"
   export KEY_PROVINCE="HESSEN"
   export KEY_CITY="Frankfurt"
   export KEY_ORG="SecretVPN"
   export KEY_EMAIL="support@SecretVPN.de"

Erstellen einer eigenen CA
~~~~~~~~~~~~~~~~~~~~~~~~~~

Zuerst müssen wir eine eigene CA erstellen. Dies wird mit den folgenden
Befehlen gemacht::

  # source vars 
  # ./clean-all 
  # ./build-ca 
  # ./build-dh

Server Zertifikat
~~~~~~~~~~~~~~~~~

Nun können wir das Server Zertifikat erstellen. Dieses Zertifikat wird
nur einmal erstellt und dem OpenVPN Server übergeben::

  # source vars 
  # ./build-key-server VPNgwServer

Client Zertifikat erstellen
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Jetzt wird pro Client ein Client Zertifikat erstellt. Hier für den
Client „fridolin“::

  # source vars 
  # ./build-key fridolin

Client Zertifikat sperren
~~~~~~~~~~~~~~~~~~~~~~~~~

Falls ein erstelltes Zertifikat nicht mehr benötigt wird oder abhanden
gekommen ist kann dieses mittels easy-rsa auch gesperrt werden. Hier für
das Zertifikat von „fridolin“::

  # source vars 
  # ./revoke-full fridolin

Konfiguration von OpenVPN
-------------------------

Nach der Erstellung der Zertifikate kann mit der Konfiguration von
OpenVPN begonnen werden.

Server Konfiguration
~~~~~~~~~~~~~~~~~~~~

Damit Portsperren in fremden Netzen relativ einfach zu handhaben sind
wird der OpenVPN Server auf dem Port 80 lauschen und das TCP Protokoll
nutzen. Damit sollte eine Verbindung in jedem Netz zu stande kommen, in
dem das Surfen erlaubt ist. Dafür erstellen wir eine Konfigurationsdatei
mit einem beliebigen Namen (hier: /root/VPNgwServer.conf).

Darauf zu achten ist, dass vor dem Erstellen der Konfigurationsdatei ein
Zertifikat mit dem oben beschriebenen Befehl (./revoke-full) widerrufen
wird, da ansonsten die Datei crl.pem nicht erstellt wird und OpenVPN
dadurch nicht startet.

::

   #Autentifizierungsmethode
   auth RSA-SHA1-2
   #Verschlüsselungsalgorithmus
   cipher AES-256-CBC
   #IP auf der gelauscht wird
   local $IP_DES_SERVERS_IM_LAN
   #Port auf dem gelauscht wird
   port 80
   proto tcp-server

   mode server
   tls-server
   dev tap

   client-to-client
   float
   keepalive 10 60

   #IP des Servers im VPN
   ifconfig 10.8.1.1 255.255.255.0
   #IP Pool, der an Clients vergeben wird
   ifconfig-pool 10.8.1.2 10.8.1.20 
   #IP leases speichern
   ifconfig-pool-persist ipp.txt

   #sende folgende Optionen zum Client
   push "route-gateway 10.8.1.1"
   push "redirect-gateway"
   push "dhcp-option DNS $DNS_SERVER_DES_HEIMNETZES"

   #Zertifikate
   ca /root/easy-rsaVPNgw/keys/ca.crt
   key /root/easy-rsaVPNgw/keys/VPNgwServer.key
   cert /root/easy-rsaVPNgw/keys/VPNgwServer.crt
   dh /root/easy-rsaVPNgw/keys/dh2048.pem
   crl-verify /root/easy-rsaVPNgw/keys/crl.pem

   #Starte Management Console auf Port 7506
   management localhost 7506
   #Nutze Komprimierung
   comp-lzo

   verb 4
   mute 50

Nun muss noch FreeBSD eingerichtet werden. Hier ist in der
/boot/loader.conf die Zeile

::

   if_tap_load="YES"

einzufügen. Für einen automatischen Start beim Booten ist die
/etc/rc.conf um die Zielen

::

   openvpn_enable="YES"
   openvpn_configfile="/root/VPNgwServer.conf"
   openvpn_dir="/root/"

zu erweitern. Zum Schluss sind noch die richtigen Routen einzustellen,
um den Traffic in und aus dem VPN Netz ins heimische Netzwerk zu leiten.
Dafür aktivieren wir das IP Forwarding indem wir in die /etc/rc.conf die
Zeile

::

   gateway_enable="YES"

hinzufügen und am Router des Heimnetzwerkes eine Route hinzufügen, die
als Router für den Netzwerkverkehr des VPN Netzes unseren Server angibt
(Destination IP: 10.8.1.0, Gateway: $IP_DES_SERVERS_IM_LAN). Zusätzlich
kann der Netzwerkverkehr mit einem Paketfilter wie pf eingeschränkt und
kontrolliert werden.

Client Konfiguration
~~~~~~~~~~~~~~~~~~~~

Nun brauchen wir nur noch den Client einzurichten und der ersten
Verbindung steht nichts mehr im Wege. Auf dem Client müssen wir zuerst
die Konfigurationsdatei von OpenVPN erstellen. Diese legen wir der
Übersichtlichkeit halber in einen extra Ordner im Heimverzeichnis des
Nutzers (hier: /home/fridolin/VPNgw/VPNgw.conf).

::

   #Verschluesselungsalgorithmus         
   cipher AES-256-CBC                    
   #Autentifizierungsmethode             
   auth RSA-SHA1-2                       
   #IP/DNS und Port des VPN Servers           
   remote $IP/DNS_DES_SERVERS                  
   port 80                             
   proto tcp-client                      

   dev tap
   tls-client

   #Zertifikate
   ca ca.crt
   key fridolin.key
   cert fridolin.crt

   #Server ueberprueft Zertifikate auf Gueltigkeit
   ns-cert-type server

   #hole Configurationsoptionen vom Server
   pull

   #Nutze Komprimierung
   comp-lzo

   verb 4
   mute 50

Danach kopieren wir die 3 Dateien ca.crt, fridolin.key und fridolin.crt
vom Server auf den Client (hier wurden die Dateien in das selbe
Verzeichnis wie die Konfigurationsdatei kopiert). Nun erstellen wir eine
ausführbare Datei mit der wir die VPN Verbindung starten können. Da bei
FreeBSD der DNS Server nicht verändert wird, obwohl es in der
Konfiguration angegeben ist wird dieser in dem Startskript verändert.

::

   #!/bin/sh
   #set nameserver
   nsset=$(cat /etc/resolv.conf | grep "nameserver $DNS_SERVER_DES_HEIMNETZES" | wc -l)
   if [ $nsset -le 0 ]
           then
           cp /etc/resolv.conf /etc/resolv.conf.temp
           echo "nameserver $DNS_SERVER_DES_HEIMNETZES" > /etc/resolv.conf
           cat /etc/resolv.conf.temp >> /etc/resolv.conf
   fi

   #start OpenVPN
   cd /home/fridolin/VPNgw
   openvpn --config VPNgw.conf

   #unset nameserver
   mv /etc/resolv.conf.temp /etc/resolv.conf

Test
----

Nun können wir den Tunnel das erste Mal testen. Dazu starten wir den OpenVPN
Server und danach den OpenVPN Client auf der Konsole. Server::

  # cd /root
  # openvpn --config VPNgwServer.conf

Wenn folgende Zeile erscheint ist das Server erfolgreich gestartet::

  Sat Apr 17 11:43:08 2010 us=822164 Initialization Sequence Completed

Nun wird der Client gestartet::

  # cd /home/fridolin/VPN/
  # openvpn --config VPNgw.conf

Eine erfolgreiche Verbindung des Clients kann man auf der Konsole am Server
verfolgen. Der Client gibt mit folgender Zeile ebenfalls eine erfolgreiche
Verbindung an::

  Sat Apr 17 11:44:29 2010 us=21798 Initialization Sequence Completed

Mittels netstat -r und tcpdump -vvi tap0 kann man nun prüfen, ob die Routen
richtig gesetzt wurden und der Traffic nun über das TAP-Device anstatt der
normalen (W)LAN Karte läuft. Ist dies der Fall kann das VPN in Betrieb genommen
werden. 


* :ref:`genindex`

Zuletzt geändert: |date|

