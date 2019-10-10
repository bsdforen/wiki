VPN (OpenBSD)
=============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

Das folgende HOWTO beschreibt einige wesentliche Elemente der
Konfiguration eines VPN-Routers. Das Gerät ist lüfterlos und bootet von
einer CF-Karte. Das System läuft im RAM-Drive. Die Anbindung an DAS
INTERNET erfolgt unter Verwendung eines marktüblichen ADSL-Anschlusses
(natürlich flat...). Die dynamisch zugeteilte Adresse wird per DYNDNS
kanonisch auflösbar gemacht. Als VPN-Applikation wird ISAKMPD verwendet.

Ziel war es, viele kleine Standorte kostengünstig mittels ADSL an ein
zentrales LAN via VPN anzubinden. Die VPN-Gateways an den Standorten
sollten wartungsarm sein, d.h. über keine bewegten Teile verfügen.
Weiterhin mussten sie schnell updatebar sein. Aus diesem Grund wurde
eine zentrale Masterstation (baugleich mit den Stationen an den
Standorten) aufgesetzt, die die CF-Karten bespielt, mit denen die
VPN-Gateways an den Standorten bestückt werden.

DSL-Einwahl
-----------

/etc/rc.local
~~~~~~~~~~~~~

::

   ...other stuff
   # PPPOE
   /sbin/ifconfig re0 up && /usr/sbin/ppp -ddial pppoe
   ...other stuff

*Kommentar:*

-  re0 ist in diesem Fall das zu benutzende Interface.
-  Das Interface muss vor der Einwahl auf UP gesetzt werden.

/etc/ppp/ppp.conf
~~~~~~~~~~~~~~~~~

::

   default:
    set log Phase Chat LCP IPCP CCP tun command
    set redial 10 0
    set reconnect 10 10000
   pppoe:
    set device "!/usr/sbin/pppoe -i re0"
    disable acfcomp protocomp
    deny acfcomp
    set mtu 1484
    set mru 1484
    set mtu maximum 1492
    set mru maximum 1492
    set speed sync
    disable ipv6 ipv6cp
    set authname "xxxxxxxxxxxxxxxxxxxxxxxxxxxx@t-online.de"
    set authkey "yyyyyyyy"
    enable mssfixup
    enable lqr
    add default HISADDR

*Kommentar:*

-  re0 ist in diesem Fall das zu benutzende Interface.
-  authname und authkey muessen natürlich die entsprechenden Werte
   enthalten.

/etc/ppp/ppp.linkup
~~~~~~~~~~~~~~~~~~~

::

   pppoe:
    ! sh -c "/sbin/pfctl -F all -f /etc/ppp/pf.conf -D EXT0IF=INTERFACE"
    !bg sh -c "/usr/local/sbin/ddclient -daemon=0 -syslog -quiet retry -mail root"
    !bg /sbin/isakmpd

*Kommentar:*

-  Die Konfigurationen der Firewall,des dd-Klienten sowie des ISAKMPD
   werden später erklärt.
-  pfctl flushed die aktuellen und lädt die neuen Firewallregeln. Es
   unterstützt die Übergabe von Variablenwerten in der Kommandozeile,
   d.h. EXT0IF taucht später in der pf.conf wieder auf. Es wird hier auf
   den Wert des von PPP verwendeten Interfaces, meist tun0, gesetzt.
-  Der ddclient erledigt die Aktualisierung des bei ``dyndns.org``
   registrierten kanonischen Names anhand der dynamisch vergebenen
   IP-Adresse.
-  Der ISAKMPD stellt die VPN-Verbindung zum VPN-Gateway her.

/etc/ppp/ppp.linkdown
~~~~~~~~~~~~~~~~~~~~~

::

   MYADDR:
    !bg sh -c "/bin/kill -KILL `/bin/cat /var/run/isakmpd.pid`"
    ! sh -c "/sbin/pfctl -F all -f /etc/pf.conf"

*Kommentar:*

-  Der ISAKMPD wird gestoppt.
-  Die Standard-Firewallregeln werden geladen.

Tägliche Neuanmeldung
~~~~~~~~~~~~~~~~~~~~~

Die DTAG erzwingt alle >24h eine Abmeldung der bestehenden
DSL-Verbindung. Um diesen Reset zu einem von uns bestimmten Zeitpunkt
kontrolliert zu gestalten, melden wir uns einmal alle 24h ab und wieder
an.

crontab
^^^^^^^

::

   2 * * * /path/to/restart.sh

restart.sh
^^^^^^^^^^

::

   #!/bin/sh
   PPPID=`ps wax | awk '{if ($5 ~ "ppp" && $6 == "-ddial") {print $1}}'`
   if [ $PPPID -gt 0 ]; then
       kill -INT $PPPID
   else
       echo "$(basename $0): $PPPID is not a number"
       echo "$(basename $0): can not terminate ppp"
       exit 1
   fi
   exit 0

Firewall
--------

Die Firewall sollte beim Systemstart aktiviert werden. Dazu muss in der
``/etc/rc.conf`` ``PF=YES`` gesetzt werden. Es existieren zwei
``pf.conf``. Die eine, ``/etc/pf.conf``, wird im Zustand ohne
DSL-Anbindung, die andere ``/etc/ppp/pf.conf`` wird nach dem Aufbau der
DSL-Verbindung geladen.

/etc/pf.conf
~~~~~~~~~~~~

::

   INT0IF="rl0"
   set skip on lo0
   scrub in
   block in
   pass out keep state
   antispoof quick for { lo0 $INT0IF }

/etc/ppp/pf.conf
^^^^^^^^^^^^^^^^

::

   INT0IF="rl0"
   EXT0IF="tun0"
   VPNGATE="aaa.bbb.ccc.ddd/xx"
   ADMINNET="aaa.bbb.ccc.ddd/xx"
   SERVERNET="aaa.bbb.ccc.ddd/xx"
   HOMENET="aaa.bbb.ccc.ddd/xx"
   DNSSERVER="aaa.bbb.ccc.ddd"
   HTTPPROXYSERVER="aaa.bbb.ccc.ddd"
   FTPPROXYSERVER="aaa.bbb.ccc.ddd"
   IMAPSERVER="aaa.bbb.ccc.ddd"
   SMTPSERVER="aaa.bbb.ccc.ddd"
   OFFICESERVER="aaa.bbb.ccc.ddd"
   MYSQLSERVER="aaa.bbb.ccc.ddd"
   INFORMIXSERVER="aaa.bbb.ccc.ddd"
   WSUSSERVER="aaa.bbb.ccc.ddd"
   AVGSERVER="aaa.bbb.ccc.ddd"
   NTPSERVER="aaa.bbb.ccc.ddd"
   table <VPN_T> \
       { $VPNGATE }
   set skip on { lo0 enc0 }
   scrub all
   block in log all
   pass out keep state
   block out log on $EXT0IF from $HOMENET to any
   # FROM HERE TO VPNGATE #################################################
   # ICMP
   pass in on $INT0IF inet proto icmp all icmp-type 8 code 0 keep state
   # DNS
   pass in on $INT0IF proto udp from $HOMENET to $DNSSERVER port 53 keep state
   # HTTP ########################################################################
   pass in on $INT0IF proto tcp from $HOMENET to $HTTPPROXYSERVER port 3128 keep state
   pass in on $INT0IF proto udp from $HOMENET to $HTTPPROXYSERVER port 3128 keep state
   # IMAP ########################################################################
   pass in on $INT0IF proto tcp from $HOMENET to $IMAPSERVER port { 993 } keep state
   pass in on $INT0IF proto udp from $HOMENET to $IMAPSERVER port { 993 } keep state
   # SMTP ########################################################################
   pass in on $INT0IF proto tcp from $HOMENET to $IMAPSERVER port 25 keep state
   # OFFICE SAMBA ########################################################################
   pass in on $INT0IF proto { tcp udp } from $HOMENET to $OFFICESERVER port { 137 138 139 445 } keep state
   # MYSQL ################################################################
   pass in on $INT0IF proto { tcp udp } from $HOMENET to $MYSQLSERVER port { 1433 } keep state
   # INFORMIX ################################################################
   pass in on $INT0IF proto { tcp udp } from $HOMENET to $INFORMIXSERVER port { 1526 } keep state
   # WSUS ################################################################
   pass in on $INT0IF proto { tcp udp } from $HOMENET to $WSUSSERVER port { 8530 } keep state
   # AVG ################################################################
   pass in on $INT0IF proto { tcp udp } from $HOMENET to $AVGSERVER port { 135 137 139 445 4156 } keep state
   # NTP ################################################################
   pass in on $INT0IF proto { tcp udp } from $HOMENET to $NTPSERVER port { 123 } keep state
   # VNC ################################################################
   pass in on $INT0IF proto { tcp udp } from $HOMENET to  { $ADMINNET $SERVERNET } port { 5800 5900 } keep state
   # PRINT ################################################################
   pass in on $INT0IF proto { tcp udp } from $HOMENET to  { $SERVERNET } port { 9100 } keep state
   # FROM VPNGATE TO HERE #################################################
   # ICMP ########################################################################
   pass in on enc0 from { $ADMINNET $SERVERNET } to $HOMENET keep state
   # SSH #########################################################################
   pass in on $EXT0IF proto tcp from { $VPNGATE } to ($EXT0IF) port 22 keep state
   # CRYPT #######################################################################
   pass out quick proto esp from any to <VPN_T>
   pass in quick proto esp from <VPN_T> to any
   pass out quick on enc0 from any to <VPN_T>
   pass in quick on enc0 from <VPN_T> to any
   pass out quick on $EXT0IF proto udp from any to <VPN_T> port = 500
   pass in quick on $EXT0IF proto udp from <VPN_T> to any port = 500
   ###############################################################################

*Kommentar:* Der CRYPT-Abschnitt sollte analog in der pf.conf des
VPN-Gateways auftauchen.

ddclient
--------

Der ``ddclient`` aktualisiert einen zuvor unter
`www.dyndns.org <www.dyndns.org>`__ angelegten Dynamischen DNS Host mit
dem Namen ``myvpnrouter.homeunix.net``.

/etc/ddclient.conf
~~~~~~~~~~~~~~~~~~

::

   daemon=300                              # check every 300 seconds
   syslog=yes                              # log update msgs to syslog
   mail=root                               # mail all msgs to root
   mail-failure=root                       # mail failed update msgs to root
   pid=/var/run/ddclient.pid               # record PID in file.
   use=web                                 # via web
   protocol=dyndns2                        # default protocol
   server=members.dyndns.org               # default server
   login=xxxxxxxxx                         # default login
   password=yyyyyyyyy                      # default password
   wildcard=yes                            # add wildcard CNAME?
   server=members.dyndns.org,              \
   protocol=dyndns2                        \
   myvpnrouter.homeunix.net

ISAKMPD
-------

Der ISAKMPD wird benutzt, um eine VPN-Verbindung zum Haupt-LAN
herzustellen. Zur Authentifizierung werden x509-Zertifikate verwendet.
Die Maschine die den Zugang zum Haupt-LAN verwaltet nennen wir
**myvpngate**. Unseren VPN-Router nennen wir **myvpnrouter00**.

/etc/isakmpd/isakmpd.policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   Keynote-version:2
   Authorizer: "POLICY"
   Conditions: app_domain == "IPsec policy" &&
               esp_present == "yes" &&
               esp_enc_alg != "null" -> "true";

*Kommentar*:

-  Diese Datei ist auf beiden Maschinen gleich.

VPN-Gateway
~~~~~~~~~~~

Die Gegenstelle unseres VPN-Routers ist ein VPN-Gateway, der Zugang zum
eigentlichen LAN. Dieses wird wie folgt konfiguriert um mit mehreren
VPN-Routern mit dynamischen Adressen jeweils einen VPN-Tunnel aufbauen
zu können:

/etc/isakmpd/isakmpd.conf
^^^^^^^^^^^^^^^^^^^^^^^^^

::

   [Phase 1]
   Default=                myvpnrouters
   [Phase 2]
   Passive-Connections=    VPN-myvpngate-myvpnrouter00,VPN-myvpngate-myvpnrouter01
   [myvpnrouters]
   Phase=                  1
   Transport=              udp
   Configuration=          main-mode
   ID=                     my-ID
   [my-ID]
   ID-type=                FQDN
   Name=                   myvpngate
   [VPN-myvpngate-myvpnrouter00]
   Phase=                  2
   ISAKMP-peer=            myvpnrouter00
   Configuration=          quick-mode
   Local-ID=               myvpngate-internal-network
   Remote-ID=              myvpnrouter00-internal-network
   [VPN-myvpngate-myvpnrouter01]
   Phase=                  2
   ISAKMP-peer=            myvpnrouter01
   Configuration=          quick-mode
   Local-ID=               myvpngate-internal-network
   Remote-ID=              myvpnrouter01-internal-network
   [myvpngate-internal-network]
   ID-type=                IPV4_ADDR_SUBNET
   Network=                192.168.0.0
   Netmask=                255.255.0.0
   [myvpnrouter00-internal-network]
   ID-type=                IPV4_ADDR_SUBNET
   Network=                10.0.0.0
   Netmask=                255.255.255.0
   [myvpnrouter01-internal-network]
   ID-type=                IPV4_ADDR_SUBNET
   Network=                10.0.1.0
   Netmask=                255.255.255.0
   [x509-certificates]
   CA-directory=   /etc/isakmpd/ca/
   Cert-directory= /etc/isakmpd/certs/
   Private-key=    /etc/isakmpd/private/myvpngate.key
   [main-mode]
   DOI=IPSEC
   EXCHANGE_TYPE=ID_PROT
   Transforms=BLF-SHA-RSA_SIG
   [quick-mode]
   DOI=IPSEC
   EXCHANGE_TYPE=QUICK_MODE
   Suites=QM-ESP-AES-SHA-SUITE
   [Default-main-mode]
   DOI=                    IPSEC
   EXCHANGE_TYPE=          ID_PROT
   Transforms=             3DES-SHA,BLF-SHA
   [Default-quick-mode]
   DOI=                    IPSEC
   EXCHANGE_TYPE=          QUICK_MODE
   Suites=                 QM-ESP-3DES-SHA-SUITE
   [PGP-main-mode]
   DOI=                    IPSEC
   EXCHANGE_TYPE=          ID_PROT
   Transforms=             CAST-SHA,3DES-MD5
   [PGP-quick-mode]
   DOI=                    IPSEC
   EXCHANGE_TYPE=          QUICK_MODE
   Suites=                 QM-ESP-CAST-SHA-SUITE,QM-ESP-CAST-MD5-SUITE,QM-ESP-3DES-MD5-SUITE
   [3DES-MD5]
   ENCRYPTION_ALGORITHM=   3DES_CBC
   HASH_ALGORITHM=         MD5
   AUTHENTICATION_METHOD=  PRE_SHARED
   GROUP_DESCRIPTION=      MODP_1024
   Life=                   LIFE_1_DAY
   [CAST-SHA]
   ENCRYPTION_ALGORITHM=   CAST_CBC
   HASH_ALGORITHM=         SHA
   AUTHENTICATION_METHOD=  PRE_SHARED
   GROUP_DESCRIPTION=      MODP_1536
   Life=                   LIFE_1_DAY
   [LIFE_1_DAY]
   LIFE_TYPE=              SECONDS
   LIFE_DURATION=          86400,79200:93600

VPN-Router
~~~~~~~~~~

Diese Sektion beschreibt die Konfiguration unseres VPN-Routers.

.. _etcisakmpdisakmpd.conf-1:

/etc/isakmpd/isakmpd.conf
^^^^^^^^^^^^^^^^^^^^^^^^^

::

   [Phase 1]
   # Statische IP des VPN-Gateways
   aaa.bbb.ccc.ddd=          myvpngate
   [Phase 2]
   Connections=            VPN-myvpnrouter00-myvpngate
   [myvpngate]
   Phase=                  1
   Transport=              udp
   Address=                aaa.bbb.ccc.ddd
   Configuration=          main-mode
   ID=                     my-ID
   [my-ID]
   ID-type=                FQDN
   Name=                   myvpnrouter00
   [VPN-myvpnrouter00-myvpngate]
   Phase=                  2
   ISAKMP-peer=            myvpngate
   Configuration=          quick-mode
   Local-ID=               myvpnrouter00-internal-network
   Remote-ID=              myvpngate-internal-network
   [myvpnrouter00-internal-network]
   ID-type=                IPV4_ADDR_SUBNET
   Network=                10.0.0.0
   Netmask=                255.255.255.0
   [myvpngate-internal-network]
   ID-type=                IPV4_ADDR_SUBNET
   Network=                192.168.0.0
   Netmask=                255.255.0.0
   [x509-certificates]
   CA-directory=   /etc/isakmpd/ca/
   Cert-directory= /etc/isakmpd/certs/
   Private-key=    /etc/isakmpd/private/myvpnrouter00.key
   [main-mode]
   DOI=IPSEC
   EXCHANGE_TYPE=ID_PROT
   Transforms=BLF-SHA-RSA_SIG
   [quick-mode]
   DOI=IPSEC
   EXCHANGE_TYPE=QUICK_MODE
   Suites=QM-ESP-AES-SHA-SUITE
   [Default-main-mode]
   DOI=                    IPSEC
   EXCHANGE_TYPE=          ID_PROT
   Transforms=             3DES-SHA,BLF-SHA
   [Default-quick-mode]
   DOI=                    IPSEC
   EXCHANGE_TYPE=          QUICK_MODE
   Suites=                 QM-ESP-3DES-SHA-SUITE
   [PGP-main-mode]
   DOI=                    IPSEC
   EXCHANGE_TYPE=          ID_PROT
   Transforms=             CAST-SHA,3DES-MD5
   [PGP-quick-mode]
   DOI=                    IPSEC
   EXCHANGE_TYPE=          QUICK_MODE
   Suites=                 QM-ESP-CAST-SHA-SUITE,QM-ESP-CAST-MD5-SUITE,QM-ESP-3DES-MD5-SUITE
   [3DES-MD5]
   ENCRYPTION_ALGORITHM=   3DES_CBC
   HASH_ALGORITHM=         MD5
   AUTHENTICATION_METHOD=  PRE_SHARED
   GROUP_DESCRIPTION=      MODP_1024
   Life=                   LIFE_1_DAY
   [CAST-SHA]
   ENCRYPTION_ALGORITHM=   CAST_CBC
   HASH_ALGORITHM=         SHA
   AUTHENTICATION_METHOD=  PRE_SHARED
   GROUP_DESCRIPTION=      MODP_1536
   Life=                   LIFE_1_DAY
   [LIFE_1_DAY]
   LIFE_TYPE=              SECONDS
   LIFE_DURATION=          86400,79200:93600

x509-Zertifikate
----------------

Auf die Funktionsweise von Zertifikaten wird an dieser Stelle nicht
weiter eingegangen. Um unsere Zertifikate erstellen zu können, müssen
wir uns von einer autorisierten Stelle zertifizieren lassen: Das kostet
Geld. Als echte BSDler verabscheuen wir den Kommerz und zertifizieren
uns selbst: Erst erzeugen wir uns für unsere eigene CA einen privaten
Schlüssel,

::

   # openssl genrsa -des3 -out $CA_PRIVATEKEY $KEYLENGTH

generieren eine Zertifikatsanfrage

::

   # openssl req -new -key $CA_PRIVATEKEY -out $CA_CERT_REQ

und zertifizieren uns selbst:

::

   # openssl x509 -req -days $(($VALIDYEARS * 365)) -in $CA_CERT_REQ -signkey $CA_PRIVATEKEY \
      -extfile $SSLDIR/x509v3.cnf -extensions x509v3_CA -out $CA_CERT

Eine Kopie von $CA_CERT wird auf der jeweiligen Gegenseite in dem
Verzeichnis abgelegt, das in der gegnerischen ``isakmpd.conf`` als
CA-directory eingetragen ist. In der Regel ist das ``/etc/isakmpd/ca``.

Jetzt können wir unseren privaten Schlüssel erzeugen,

::

   # openssl genrsa -out $PRIVATEKEY $KEYLENGTH

eine Zertifikatsanforderung erstellen,

::

   # openssl req -new -key $PRIVATEKEY -out $CERTREQ

ein Zertifikat erstellen,

::

   # export CERTFQDN=$FQDN
   # openssl x509 -req -days $VALIDDAYS -in $CERTREQ -CA $CA_CERT -CAkey $CA_PRIVATEKEY \
      -CAcreateserial -extfile $SSLDIR/x509v3.cnf -extensions x509v3_FQDN -out $CERT

und zuletzt das Zertifikat verifizieren.

::

   # openssl verify -CAfile $CA_CERT $CERT 

.. note::

  Der Name der Organisation sollte sich von dem der Zertifizierungsstelle
  unterscheiden!

Jetzt können wir das Zertifikat umbenennen

::

   # mv $CERT $(dirname "$CERT")/$FQDN.crt

und die Rechte auf den privaten Schlüssel einschränken:

::

   # chmod 600 $CFGDIR/private/*

$FQDN sollte sich mit den Einträgen in der ``isakmpd.conf`` decken, also
hier **myvpngate** bzw. **myvpnrouter00**. $FQDN.crt muss sowohl auf der
eigenen, als auch auf der gegnerischen Maschine abgelegt werden und zwar
jeweils dort, wo in der jeweiligen ``isakmpd.conf`` unter
**Cert-directory** angegeben. $PRIVATEKEY bleibt logischerweise auf der
eigenen Maschine und wird so platziert, wie in der lokalen
``isakmpd.conf`` unter **Private-key** eingetragen.

* :ref:`genindex`

Zuletzt geändert: |date|

