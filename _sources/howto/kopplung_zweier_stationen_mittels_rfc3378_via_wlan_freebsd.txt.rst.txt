Kopplung zweier Stationen mittels RFC3378 via WLAN (FreeBSD)
============================================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Ein weiterer Anwendungsfall von ipsec(4) eröffnet sich u. a. zum
Absichern von Kommunikationsbeziehungen, welche auf dem in
`RFC3378 <http://tools.ietf.org/html/rfc3378>`__ beschriebenen
EtherIP-Protokoll basieren.

Ein EtherIP-Tunnel wird konfiguriert, wenn bspw. jeweils ein Ethernet
Interface mit einem
`gif(4) <http://www.freebsd.org/cgi/man.cgi?query=gif&sektion=4&apropos=0&manpath=FreeBSD+8.2-RELEASE>`__
Interface mittels
`if_bridge(4) <http://www.FreeBSD.org/cgi/man.cgi?query=if_bridge&apropos=0&sektion=0&manpath=FreeBSD+8.2-RELEASE&format=html>`__
auf zwei (oder mehreren) miteinander über
`ip(4) <http://www.freebsd.org/cgi/man.cgi?query=ip&apropos=0&sektion=4&manpath=FreeBSD+8.2-RELEASE&format=html>`__
kommunizierenden Stationen gekoppelt werden, wie in if_bridge(4)
beschrieben:

::

   ifconfig gif0 create
   ifconfig gif0 tunnel 1.2.3.4 5.6.7.8 up
   ifconfig bridge0 create
   ifconfig bridge0 addm fxp0 addm gif0 up

Folgende Situation ist gegeben: ein DSL-Router soll via WLAN mit einer
Brücke bzw. mit einem Switch per EtherIP-Tunnel kommunizieren. Das
Switch ist u. a. mit einem DSL-Modem verbunden und soll ergänzend
Ethernet Rahmen umsetzen, die dem in
`RFC2516 <http://tools.ietf.org/html/rfc2516>`__ spezifizierten
Protokoll entsprechen. Beide Stationen besitzen jeweils mindestens ein
`wlan(4) <http://www.freebsd.org/cgi/man.cgi?query=wlan&apropos=0&sektion=0&manpath=FreeBSD+8.2-RELEASE&format=html>`__
bzw.
`ath(4) <http://www.freebsd.org/cgi/man.cgi?query=ath&apropos=0&sektion=0&manpath=FreeBSD+8.2-RELEASE&format=html>`__
Interface und bilden gemeinsam ein
`RFC1918 <http://tools.ietf.org/html/rfc1918>`__ konformes
`/30 <http://de.wikipedia.org/wiki/CIDR>`__ Ad-hoc-Netz.

============== =============== ============ ===============
**DSL-Router**                 **Switch**  
============== =============== ============ ===============
Netzadresse:   192.168.0.1     Netzadresse: 192.168.0.2
Netzmaske:     255.255.255.252 Netzmaske:   255.255.255.252
ssid:          tiamat          ssid:        tiamat
============== =============== ============ ===============

Die Konfiguration eines DSL-Routers würde den Rahmen dieses Artikels
sprengen. Daher verweise ich auf

-  `Kapitel
   28 <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/ppp-and-slip.html>`__
   des FreeBSD-Handbuches,
-  alternativ auf den Artikel
   `anschluss_an_das_internet_mit_pppoe_mpd_und_firewall_pf </howto/anschluss_an_das_internet_mit_pppoe_mpd_und_firewall_pf>`__
-  sowie `Kapitel
   32.2 <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/network-routing.html>`__
   des FreeBSD-Handbuches,
-  ferner `Kapitel
   32.3 <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/network-wireless.html>`__
   des FreeBSD-Handbuches
-  und diesen
   `Artikel <http://www.freebsd.org/doc/en_US.ISO8859-1/articles/dialup-firewall/article.html>`__.

Desweiteren sind Informationen bzgl. Installation und Konfiguration von
`ipsec(4) <http://www.freebsd.org/cgi/man.cgi?query=ipsec&apropos=0&sektion=4&manpath=FreeBSD+8.2-RELEASE&format=html>`__
und `racoon(8) <http://www.cl.cam.ac.uk/cgi-bin/manpage?8+racoon>`__ im
Kontext zu dem Artikel `ipsec-vpn </howto/ipsec-vpn>`__ und `Kapitel
15.9 <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/ipsec.html>`__
des FreeBSD-Benutzerhandbuches zu betrachten.

Aufbau des EtherIP-Tunnels beim DSL-Router
------------------------------------------

Zuallererst sollte ein unbenutztes wlan(4) Interface konfiguriert
werden::

  # ifconfig wlan0 create wlandev ath0 wlanmode adhoc 
  # ifconfig wlan0 inet 192.168.0.1 netmask 255.255.255.252 ssid tiamat up

Wie in `Kapitel
28.5 <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/pppoe.html>`__
des FreeBSD-Handbuches beschrieben, wird für PPPoE ein Ethernet
Interface benötigt. Ein wlan(4) Interface (direkt) für diesen Zweck zu
verwenden, ist nicht zu empfehlen. Daher muss ein virtuelles Ethernet
Interface konfiguriert werden.

Es ist nicht (wirklich) möglich
`ng_eiface(4) <http://www.FreeBSD.org/cgi/man.cgi?query=ng_eiface&sektion=4&apropos=0&manpath=FreeBSD+8.2-RELEASE>`__
als Schnittstelle für PPPoE (direkt) zu verwenden, denn
`ng_pppoe(4) <http://www.freebsd.org/cgi/man.cgi?query=ng_pppoe&manpath=FreeBSD+9.1-RELEASE>`__
interagiert mittels
`ng_ether(4) <http://www.freebsd.org/cgi/man.cgi?query=ng_ether&manpath=FreeBSD+9.1-RELEASE>`__
mit (physischen) Ethernet Interfaces.

Als nächstes wird ngeth0 mit einem gif(4) Interface verbrückt::

  # ifconfig gif0 create # ifconfig gif0 tunnel 192.168.0.1 192.168.0.2 mtu 1500 up

  # sysctl net.link.gif.parallel_tunnels=1

  # ifconfig bridge0 create # ifconfig bridge0 addm ngeth0 addm gif0 up

/etc/rc.conf
~~~~~~~~~~~~

Wird ergänzt mit:

::

   wlans_ath0="wlan0"
   create_args_wlan0="wlanmode adhoc"
   ifconfig_wlan0="inet 192.168.0.1 netmask 255.255.255.252 ssid tiamat"

   gif_interfaces="gif0"
   gifconfig_gif0="192.168.0.1 192.168.0.2"
   ifconfig_gif0="mtu 1500 up"

   cloned_interfaces="bridge0"
   ifconfig_bridge0="addm ngeth0 addm gif0 up"

/etc/sysctl.conf
~~~~~~~~~~~~~~~~

Wird ergänzt mit:

::

   net.link.gif.parallel_tunnels=1

/etc/ppp/ppp.conf
~~~~~~~~~~~~~~~~~

Wird ergänzt mit:

::

   net.link.gif.parallel_tunnels=1

Aufbau des EtherIP-Tunnels beim Switch
--------------------------------------

Das Switch besitzt (hier) drei Ethernet Interfaces (fxp0, fxp1, fxp2)
und ein wlan(4) Interface, wobei fxp0 mit dem DSL-Modem verbunden werden
sollte. Die Betrachtung des Switches wird zunächst auf die einer
Netzwerkbrücke reduziert::

  # ifconfig wlan0 create wlandev ath0 wlanmode adhoc 
  # ifconfig wlan0 inet 192.168.0.2 netmask 255.255.255.252 ssid tiamat up

  # ifconfig gif0 create # ifconfig gif0 tunnel 192.168.0.2 192.168.0.1 mtu 1500 up

  # sysctl net.link.gif.parallel_tunnels=1

  # ifconfig bridge0 create # ifconfig bridge0 addm fxp0 addm gif0 up

Sollten alle anderen Ethernet NICs miteinbezogen werden, so rückt
`netgraph(4) <http://www.freebsd.org/cgi/man.cgi?query=netgraph&apropos=0&sektion=4&manpath=FreeBSD+8.2-RELEASE&format=html>`__
bzw.
`ng_bridge(4) <http://www.freebsd.org/cgi/man.cgi?query=ng_bridge&apropos=0&sektion=4&manpath=FreeBSD+8.2-RELEASE&format=html>`__
in das Zentrum der Betrachtung:

::

  # ifconfig bridge0 destroy
  
  # ifconfig fxp1 up
  # ifconfig fxp2 up
  
  # kldload netgraph
  # kldload ng_ether
  # kldload ng_eiface
  # kldload ng_bridge
  
  # ngctl mkpeer . eiface hook ether
  # ifconfig ngeth0 ether $mac_adresse up
  
  # ngctl mkpeer fxp0: bridge lower link0
  # ngctl name fxp0:lower relay
  # ngctl connect fxp1: relay: lower link1
  # ngctl connect fxp2: relay: lower link2
  # ngctl connect ngeth0: relay: ether link3
  
  # ngctl msg fxp0: setpromsisc 1
  # ngctl msg fxp0: setautosrc 0
  # ngctl msg fxp1: setpromsisc 1
  # ngctl msg fxp1: setautosrc 0
  # ngctl msg fxp2: setpromsisc 1
  # ngctl msg fxp2: setautosrc 0
  
  # ifconfig bridge0 create
  # ifconfig bridge0 addm ngeth0 addm gif0 up

Damit ist ein EtherIP-Tunnel zwischen beiden Stationen etabliert.

.. warning::

  Es ist unbedingt abzuraten, momentan eine PPPoE-Sitzung auszulösen, da bei
  PPP-Authentifizierungsphase, im Hinblick auf das in `RFC1334
  <http://tools.ietf.org/html/rfc1334>`__ beschriebenen Password Authentication
  Protocol, die Benutzerkennung und das Passwort im Klartext übertragen werden.
  Ferner sollte das DSL-Modem noch nicht aktiviert oder mit dem Switch
  gekoppelt sein.

Daher sind die Tools
`ping(8) <http://www.freebsd.org/cgi/man.cgi?query=ping&apropos=0&sektion=0&manpath=FreeBSD+8.2-RELEASE&format=html>`__,
`tcpdump(8) <http://www.freebsd.org/cgi/man.cgi?query=tcpdump&apropos=0&sektion=0&manpath=FreeBSD+8.2-RELEASE&format=html>`__
und das
`bpf(4) <http://www.freebsd.org/cgi/man.cgi?query=bpf&apropos=0&sektion=0&manpath=FreeBSD+8.2-RELEASE&format=html>`__
Interface geeignet zum Überprüfen des Verbindungsstatus.

/etc/start_if.wlan0
~~~~~~~~~~~~~~~~~~~

::

   ngctl mkpeer . eiface hook ether
   ifconfig ngeth0 ether $mac_adresse up

.. _etcrc.conf-3:

/etc/rc.conf
~~~~~~~~~~~~

Wird ergänzt mit:

::

   wlans_ath0="wlan0"
   create_args_wlan0="wlanmode adhoc"
   ifconfig_wlan0="inet 192.168.0.2 netmask 255.255.255.252 ssid tiamat"

   gif_interfaces="gif0"
   gifconfig_gif0="192.168.0.2 192.168.0.1"
   ifconfig_gif0="mtu 1500 up"

   cloned_interfaces="bridge0"
   ifconfig_bridge0="addm ngeth0 addm gif0 up"

.. _etcsysctl.conf-3:

/etc/sysctl.conf
~~~~~~~~~~~~~~~~

Wird ergänzt mit:

::

   net.link.gif.parallel_tunnels=1

Wie wird jetzt mit ng_bridge(4) umgegangen?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Das Shellscript /usr/share/examples/netgraph/ether.bridge sollte
angepasst werden:

.. code:: sh

   #!/bin/sh
   # $FreeBSD: src/share/examples/netgraph/ether.bridge,v 1.5.10.1.4.1 2010/06/14 02:09:06 kensmith Exp $
   # This script sets up an Ethernet bridging network across multiple
   # Ethernet interfaces using the ng_bridge(4) and ng_ether(4) ng_eiface(4) netgraph
   # node types.
   #
   # To use this script:
   #
   # 0. Make your own copy of this example script.
   #
   # 1. Give your bridging network a name by editing the definition of
   #    ${BRIDGE_NAME} below. It must be a valid netgraph node name.
   #
   # 2. Edit the definitions of ${PHYSICAL_IFACES} and ${VIRTUAL_IFACES}
   #    as described below to define your bridging interfaces.
   #
   # 3. Run this script with "start" as the command line argument.
   #
   # 4. Examine bridging statistics by running this script with "stats"
   #    as the command line argument.
   #
   # 5. Stop bridging by running this script with "stop" as the
   #    command line argument.
   #
   # To run multiple independent bridging networks, create multiple
   # copies of this script with different variable definitions.
   #

   # Give each bridging network a unique name here.

   BRIDGE_NAME="relay"

   VIRTUAL_IFACES="ngeth0"
   PHYSICAL_IFACES="fxp0 fxp1 fxp2"

   #### Modefied by user.                                          
       
   # Routine to verify node's existence.
   bridge_verify() {
       ngctl info ${BRIDGE_NAME}: >/dev/null 2>&1
       if [ $? -ne 0 ]; then
           echo "${BRIDGE_NAME}: bridge network not found"
           exit 1
       fi
   }

   # Routine to get and display link stats.
   bridge_linkstats() {
       STATS=`ngctl msg ${BRIDGE_NAME}: getstats $1`
       if [ $? -ne 0 ]; then
           exit 1
       fi
       echo "${STATS}" | fmt 2 | awk '/=/ { fl=index($0, "="); \
           printf "%20s = %s\n", substr($0, 0, fl - 1), substr($0, fl + 1); }'
   }

   # Start/restart routine.
   bridge_start() {

       # Load netgraph KLD's as necessary.
       for KLD in ng_ether ng_bridge ng_eiface; do
           if ! kldstat -v | grep -qw ${KLD}; then
               echo -n "Loading ${KLD}.ko... "
               kldload ${KLD} || exit 1
               echo "done"
           fi
       done

       # Reset all interfaces.
       bridge_stop

       # Verify all interfaces exist.
       for ETHER in ${PHYSICAL_IFACES} ${VIRTUAL_IFACES}; do
           if ! ngctl info ${ETHER}: >/dev/null 2>&1; then
               echo "Error: interface ${ETHER} does not exist"
               exit 1
           fi
           ifconfig ${ETHER} up || exit 1
       done

       # Create new ng_bridge(4) node, attached to the first virtual interface.
       FIRSTIF=`echo ${VIRTUAL_IFACES} | awk '{ print $1 }'`
       ngctl mkpeer ${FIRSTIF}: bridge ether link0 || exit 1
       ngctl name ${FIRSTIF}:ether ${BRIDGE_NAME} || exit 1

       # Attach other virtual interfaces as well.
       LINKNUM=1
           VIRTUAL_IFACES=`echo ${VIRTUAL_IFACES} | cut -d" " -f2-`
       if [ "${VIRTUAL_IFACES}" != "${FIRSTIF}" ];
       then
           for ETHER in ${VIRTUAL_IFACES}; do
               ngctl connect ${ETHER}: ${BRIDGE_NAME}: \
                       ether link${LINKNUM} || exit 1
                       
               LINKNUM=`expr ${LINKNUM} + 1`
           done
       fi
       
       # Attach other physical interfaces as well.
       for ETHER in ${PHYSICAL_IFACES}; do
               ngctl connect ${ETHER}: ${BRIDGE_NAME}: \
                   lower link${LINKNUM} || exit 1
           LINKNUM=`expr ${LINKNUM} + 1`
       done

   # Set all physical interfaces in promiscuous mode and don't overwrite src addr.
       for ETHER in ${PHYSICAL_IFACES}; do
           ngctl msg ${ETHER}: setpromisc 1 || exit 1
           ngctl msg ${ETHER}: setautosrc 0 || exit 1
       done
   }

   # Stop routine.
   bridge_stop() {
       ngctl kill ${BRIDGE_NAME}: >/dev/null 2>&1
       for ETHER in ${PHYSICAL_IFACES} ${VIRTUAL_IFACES}; do
           ngctl kill ${ETHER}: >/dev/null 2>&1
       done
   }

   # Stats routine.
   bridge_stats() {

       # Make sure node exists.
       bridge_verify

       echo ""
       echo "Statistics for bridging network ${BRIDGE_NAME}:"
       echo ""
       LINKNUM=0
       for VIRTUAL_IFACE in ${VIRTUAL_IFACES}; do
           echo "Virtual interface ${VIRTUAL_IFACE}:"
           bridge_linkstats ${LINKNUM}
           LINKNUM=`expr ${LINKNUM} + 1`
       done
       for ETHER in ${PHYSICAL_IFACES}; do
           echo "Network interface ${ETHER}:"
           bridge_linkstats ${LINKNUM}
           LINKNUM=`expr ${LINKNUM} + 1`
       done    
   }

   # Main entry point.
   case $1 in
       start)
           bridge_start
           ;;
       stats)
           bridge_verify
           bridge_stats
           ;;
       stop)
           bridge_verify
           bridge_stop
           ;;
       *)
           echo "usage: $0 [ start | stop | stats ]"
           exit 1
   esac

und in das Verzeichnis /usr/local/etc/rc.d als ether.bridge.0.sh kopiert
werden. Tiefergehende Informationen bzgl. rc.d scripting sind in diesem
`Artikel <http://www.FreeBSD.org/doc/en_US.ISO8859-1/articles/rc-scripting/index.html>`__
beschrieben.

Absichern des EtherIP-Tunnels mittels ipsec(4)
----------------------------------------------

Hierbei wird vorausgesetzt, dass ipsec(4) im Kernel aktiviert und
racoon(8) auf beiden Stationen installiert ist.

/etc/ipsec.conf
~~~~~~~~~~~~~~~

Im gegebenen Kontext müsste der DSL-Router wie folgt konfiguriert
werden:

::

   spdadd 192.168.0.1 192.168.0.2 any -P out \
     ipsec esp/transport/192.168.0.1 192.168.0.2/require;
   spdadd 192.168.0.2 192.168.0.1 any -P in \
     ipsec esp/transport/192.168.0.2 192.168.0.1/require;

Entsprechend auf dem Switch:

::

   spdadd 192.168.0.2 192.168.0.1 any -P out \
     ipsec esp/transport/192.168.0.2 192.168.0.1/require;
   spdadd 192.168.0.1 192.168.0.2 any -P in \
     ipsec esp/transport/192.168.0.1 192.168.0.2/require;

/usr/local/etc/racoon/racoon.conf
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sowie auf beiden Stationen:

::

   path pre_shared_key "/usr/local/etc/racoon/psk.txt" ;
   log debug;

   padding
   {
           maximum_length 20;      
           randomize off;         
           strict_check off;       
           exclusive_tail off;     
   }

   timer
   {
           counter 5;              
           interval 40 sec;      
           persend 1;             
           phase1 50 sec;
           phase2 30 sec;
   }

   listen
   {
           isakmp $ip_von_wlan_interface_dieser_station [500];
   }

   remote $ip_von_wlan_interface_der_gegenstelle [500]
   {
           exchange_mode main;
           doi ipsec_doi;
           situation identity_only;
           nonce_size 16;
           
           my_identifier address $ip_von_wlan_interface_dieser_station;
           peers_identifier adress $ip_von_wlan_interface_der_gegenstelle;
           
           lifetime time 1 hour;   
           passive off;
           
           proposal_check strict;    

           proposal {
                   encryption_algorithm 3des;
                   hash_algorithm sha256;
                   authentication_method pre_shared_key;
                   dh_group 1;
           }
   }

   sainfo anonymous
   {
           pfs_group 1;
           lifetime time 30 sec;
           encryption_algorithm 3des;
           authentication_algorithm hmac_sha256;
           compression_algorithm deflate;
   }

/usr/local/etc/racoon/psk.txt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   $ip_von_wlan_interface_der_gegenstelle $ich_bin_das_selbe_geheime_passwort

<note warning> Diese Datei muss mit den Benutzerrechten 0600 und in
Besitz stehend von dem User mit UID 0 auf jeder Station angelegt werden.
</note>

.. _etcrc.conf-2:

/etc/rc.conf
~~~~~~~~~~~~

Die auf ipsec(4) und racoon(8) verweisenden Einträge sollten auf beiden
Stationen ergänzt werden:

::

   ipsec_enable="YES"
   ipsec_file="/etc/ipsec.conf" 
   racoon_enable="yes"

/etc/rc.firewall
~~~~~~~~~~~~~~~~

Könnte sich auf dem Switch wie folgt gestalten:

.. code:: sh

   #!/bin/sh

   fwcmd="/sbin/ipfw"

   wlanif="wlan0"
   intrfc_table="lo0 fxp0 fxp1 fxp2 ngeth0 gif0"

   $fwcmd -f flush

   $fwcmd add deny log ip from any to any in via $wlanif not verrevpath

   for intrfc in $intrfc_table
   do
       $fwcmd add allow all from any to any via $intrfc
   done
       
   $fwcmd add deny all from any to 127.0.0.0/8
   $fwcmd add deny ip from 127.0.0.0/8 to any

   $fwcmd add allow esp from 192.168.0.2 to 192.168.0.1 out via $wlanif
   $fwcmd add allow udp from 192.168.0.2 500 to 192.168.0.1 500 out via $wlanif

   $fwcmd add allow esp from 192.168.0.1 to 192.168.0.2 in via $wlanif
   $fwcmd add allow udp from 192.168.0.1 500 to 192.168.0.2 500 in via $wlanif

   $fwcmd add deny log ip from any to any

sowie bspw. beim DSL-Router um folgende Zeilen ergänzt werden:

::

   $fwcmd add deny log ip from any to any in via $wlanif not verrevpath    
   $fwcmd add allow esp from 192.168.0.1 to 192.168.0.2 out via $wlanif
   $fwcmd add allow udp from 192.168.0.1 500 to 192.168.0.2 500 out via $wlanif

   $fwcmd add allow esp from 192.168.0.2 to 192.168.0.1 in via $wlanif
   $fwcmd add allow udp from 192.168.0.2 500 to 192.168.0.1 500 in via $wlanif

Absichern des EtherIP-Tunnels ohne Neustart
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ist ein Neustart beider Stationen nicht erwünscht, kann die
Verschlüsselung des EtherIP-Tunnels wie folgt "manuell" auf dem Router
aktiviert werden:

::

  # setkey -FP
  # setkey -F
  # setkey -c << EOF
  # spdadd 192.168.0.1 192.168.0.2 any -P out ipsec esp/transport/192.168.0.1 192.168.0.2/require;
  # spdadd 192.168.0.2 192.168.0.1 any -P in ipsec esp/transport/192.168.0.2 192.168.0.1/require;
  # EOF
  
  # racoon -f /usr/local/etc/racoon/racoon.conf -l /var/log/racoon.log

  # setkey -FP # setkey -F # setkey -c << EOF #

sowie beim Switch: 

::

  # setkey -FP
  # setkey -F
  # setkey -c << EOF
  # spdadd 192.168.0.2 192.168.0.1 any -P out ipsec esp/transport/192.168.0.2 192.168.0.1/require;
  # spdadd 192.168.0.1 192.168.0.2 any -P in ipsec esp/transport/192.168.0.1 192.168.0.2/require;
  # EOF
  
  # racoon -f /usr/local/etc/racoon/racoon.conf -l /var/log/racoon.log

.. note::

  Zum detaillierten Überprüfen, ob ipsec(4) (auch wirklich) funktioniert, sei
  dem Leser empfohlen sich mit diesem `Artikel
  <http://www.freebsd.org/doc/en_US.ISO8859-1/articles/ipsec-must/article.html>`__
  näher zu beschäftigen.

* :ref:`genindex`

Zuletzt geändert: |date|

