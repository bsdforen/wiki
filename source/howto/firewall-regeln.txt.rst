Firewall-Regeln
===============


.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png

.. warning::

  Da ich, elwood (der Wikimigrator 2), keine Firewall auf dem System, sondern
  auf dem Router habe, kann nicht beurteilen, ob die Infos hier noch up-to-date
  sind. Bitte um Überprüfung!

IPFW (FreeBSD)
--------------

Ein einfacher Client
~~~~~~~~~~~~~~~~~~~~

::

   #!/bin/sh
   fwcmd="/sbin/ipfw -q"
    
   ${fwcmd} -f flush
     
   # Alles auf dem Loopback Interface erlauben
   ${fwcmd} add 00100 allow ip from any to any via lo0
    
   ${fwcmd} add 00500 check-state
    
   # TCP
    
   # Ausgehenden Traffic erlauben, Zustand (State) merken
   ${fwcmd} add 01000 allow tcp from me to any setup keep-state
    
   # Muli und ICQ eingehend erlauben
   ${fwcmd} add 01100 allow tcp from any to me 5662,60000-60001 setup keep-state
    
   # UDP
    
   # Ausgehenden Traffic erlauben, Zustand (State) merken
   ${fwcmd} add 02000 allow udp from me to any keep-state
    
   # Muli eingehend erlauben
   ${fwcmd} add 02100 allow udp from any to me 5665
    
   # ICMP
    
   # ping out erlauben
   ${fwcmd} add 03000 allow icmp from me to any icmptypes 8 keep-state
    
   # die sind wichtig, hab vergessen was genau das jetzt ist
   ${fwcmd} add 03100 allow icmp from any to me icmptypes 3,4,11
    
   # Alles andere verbieten
   ${fwcmd} add 60000 deny ip from any to any

Ein einfacher Router/Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Bei diesem Regelwerk ist keine Regel für NAT enthalten, da dies der
gemeine DSL-Benutzer mit ppp(8) macht.

$inet ist das lokale Netz

$tcp_server sind die Ports die auf dem Router selber nach draussen offen
sein sollen

$client ist ein Client der ein paar Ports weitergeleitet bekommt

$*_forward sind die weitergeleiteten Ports für den Client

::

   #!/bin/sh
   fwcmd="/sbin/ipfw -q"
   inet="192.168.0.0/24"
   client="192.168.0.2"
   tcp_server="22, 80"
   tcp_forward="5662, 60000-60004"
   udp_forward="5665"
    
   ${fwcmd} -f flush
    
   # Alles auf dem Loopback Interface erlauben
   ${fwcmd} add 00100 allow ip from any to any via lo0
    
   ${fwcmd} add 00500 check-state
    
   # TCP
    
   # Traffic von $inet oder dem Router/Server nach aussen erlauben
   ${fwcmd} add 01000 allow tcp from { me or ${inet} } to any setup keep-state
    
   # Traffic von aussen an die Ports $tcp_server des Routers/Servers reinlassen
   ${fwcmd} add 01100 allow tcp from any to me ${tcp_server} setup keep-state
    
   # Traffic von aussen an $client weiterleiten
   ${fwcmd} add 01200 allow tcp from any to ${client} ${tcp_forward} setup keep-state
    
   # UDP
    
   # $inet oder Router/Server Traffic rauslassen
   ${fwcmd} add 02000 allow udp from { me or ${inet} } to any keep-state
    
   # Traffic an $client weiterleiten
   ${fwcmd} add 02100 allow udp from any to ${client} ${udp_forward} keep-state
    
   # ICMP
    
   ${fwcmd} add 03000 allow icmp from any to { me or ${inet} } icmptypes 3
   ${fwcmd} add 03100 allow icmp from any to { me or ${inet} } icmptypes 4
   ${fwcmd} add 03200 allow icmp from { me or ${inet} } to any icmptypes 8 keep-state
   ${fwcmd} add 03300 allow icmp from any to { me or ${inet} } icmptypes 11
    
   # Alles andere verbieten
   ${fwcmd} add 65000 reset ip from any to any

IPFilter (FreeBSD)
------------------

Einfache Regeln mit IPFilter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

IPFilter-Regelwerk für einen alleinstehenden Rechner mit direkter
Internetanbindung. IP-Adressen wurden bewusst weggelassen, da der
Rechner eine dynamische IP-Adresse (DHCP) erhält. Generell wird alles
hinaus gelassen, hinein darf ausser SSH nichts. SSH dient als Beispiel
für Serverdienste.

::

   # /etc/ipf.rules
   # FreeBSD-ipfilter-Regelwerk
    
   # Platzhalter definieren
   #-------------------------------------------------------------------------
   out_if = "fxp0"            # Name des Netzwerkinterface
    
   # Regeln
   #-------------------------------------------------------------------------

   # Standartmaessig alles verbieten
   block in on $out_if
   block out on $out_if

   # Alles auf dem (internen) Loopback-Interface erlauben
   pass in quick on lo0 
   pass out quick on lo0

   # Blockiere den externen Verkehr mit privaten Adressen
   block in quick on $out_if from 10.0.0.0/8 to any
   block in quick on $out_if from 192.168.0.0/16 to any
   block in quick on $out_if from 172.16.0.0/12 to any
   block in quick on $out_if from 127.0.0.0/8 to any
   block out quick on $out_if from any to 10.0.0.0/8
   block out quick on $out_if from any to 192.168.0.0/16
   block out quick on $out_if from any to 172.16.0.0/12
   block out quick on $out_if from any to 127.0.0.0/8

   # TCP, UDP und ICMP hinauslassen
   pass out on $out_if proto tcp from any to any flags S keep state keep frags
   pass out on $out_if proto udp from any to any keep state
   pass out on $out_if proto icmp from any to any keep state

   # Server-Ports freigeben
   pass in on $out_if proto tcp from any to any port = ssh flags S/SA keep state keep frags

PF (Free-, Open- und NetBSD)
----------------------------

Einfacher Client mit SSH-Zugriff
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sehr einfaches Regelwerk: Alles raus, nichts ausser SSH rein. (Quelle:
`OpenBSD PF <http://www.openbsd.org/faq/pf/example1.html>`__)

.. note::

  pf ist unter `NetBSD <NetBSD>`__ (noch) nicht im vollen Umfang nutzbar, mehr
  Infos dazu gibt es
  [http://www.netbsd.org/Documentation/network/pf.html#differences
  hier].

::

   # Makros
   #------------------------------------------------------------------------
   ext_if="fxp0"   # durch das eigentliche externe Interface ersetzen, z.B. dc0
    
   # Private Netzwerke
   priv_nets = "{ 127.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12, 10.0.0.0/8 }"

   # Regeln
   #-------------------------------------------------------------------------
   # Keine Rueckmeldung bei Blockierungen
   set block-policy drop

   # Normalisierung
   scrub in all
    
   # Standardmäßig alles verbieten
   block drop in all
   block drop out all

   # Über Loopback alles durchlassen
   pass quick on lo0 all
    
   # Blockiere den externen Verkehr mit privaten Adressen
   block drop in  log quick on $ext_if from $priv_nets to any
   block drop out log quick on $ext_if from any to $priv_nets
    
   # Schutz gegen gefälschte Adressen für interne Interfaces aktivieren
   antispoof log quick for $ext_if inet
    
   # TCP, UDP und ICMP über das externe Interface erlauben und
   # Zustand von UDP bzw. ICMP und modulierten Zustand von TCP merken
   pass out on $ext_if proto tcp all modulate state flags S/SA
   pass out on $ext_if proto { udp, icmp } all keep state

   # Öffnen des SSH-Ports und protokollieren der Verbindungen
   pass in log on $ext_if proto tcp from any to ($ext_if) port ssh flags S/SA synproxy state

Der SSH Port ist nun geöffnet. Informationen zum Schutz vor Bruteforce
Attacken gibt es `hier <Ssh-blocker>`__.

ALTQ und andere Goodies
~~~~~~~~~~~~~~~~~~~~~~~

Regeln mit altq, transparentem Squid usw.

::

   ### VARIABLEN ###

   Ext="tun0"                # Device an dem das Internet angeschlossen ist
   Int="fxp0"                # Device an dem das interne Netz haengt
   IntNet="192.168.1.0/24"   # Adressraum des internen Netzes
   RouterIP="192.168.1.254"  # IP Adresse des Routers
   Loop="lo0"                # Loopback Device
   Server="192.168.1.2"     # Interner server fuer mail usw...

   # Adressen die auf dem externen Device nicht geroutet werden
   # (Adressbereich des internen Netzes muss man wegen der Weiterleitungen zulassen)
   table <NoRoute> { 127.0.0.1/8, 172.16.0.0/12, 192.168.0.0/16, !$IntNet, 10.0.0.0/8, 255.255.255.255/32 }

   # Ports die geoeffnet werden sollen
   InServicesTCP="{ ssh, smtp, ftp, auth, 222, https, imaps }"
   InServicesUDP="{ imaps }"

    
   ### OPTIONS ###

   # Macht Statistiken fuer die DSL-Verbindung (pfctl -s info)
   set loginterface $Ext
    
   # Beendet inaktive Verbindungen schneller - geringerer Speicherverbrauch.
   set optimization aggressive

   # Fragmentierte Pakete saeubern
   scrub on $Ext all fragment reassemble random-id

   # Queueing
   altq on $Ext cbq bandwidth 128Kb queue { std_out, ack_out, ssh_out, ssh_ack_out }

   queue std_out                cbq(red default)
   queue ack_out     priority 2 cbq(red)
   queue ssh_out     priority 4 cbq(red)
   queue ssh_ack_out priority 6 cbq(red)


   ### NAT & FORWARD ###

   # NAT aktivieren (unter Linux als Masquerading bekannt)
   nat on $Ext from $IntNet to any -> $Ext static-port

   # Active FTP - Umleitung zu unserem ftp-proxy
   rdr on $Int proto tcp from !$RouterIP to !$IntNet port 21 -> 127.0.0.1 port 8021

   # WWW - Umleitung zu squid
   rdr on $Int proto tcp from !$RouterIP to !$IntNet port 80 -> 127.0.0.1 port 8080

   # SMTP - Umleitung zu -> $server
   rdr on $Ext proto tcp from !$IntNet to any port smtp -> $Server port smtp

   # https - Umleitung zu -> $server
   rdr on $Ext proto tcp from !$IntNet to any port https -> $Server port https

   # imaps - Umleitung zu -> $server
   rdr on $Ext proto tcp from !$IntNet to any port imaps -> $Server port imaps
   rdr on $Ext proto udp from !$IntNet to any port imaps -> $Server port imaps

   # ssh - Umleitung zu -> bull
   rdr on $Ext proto tcp from !$IntNet to any port ssh -> $Server port ssh

   rdr-anchor redirect


   ### FILTER ###

   # Generelle Block Regel
   block on $Ext

   # Freiwillig machen wir keinen mucks ;)
   block return log on $Ext

   # Wir wollen kein IPv6.0
   block quick inet6

   # Loopback Device darf alles
   pass quick on $Loop

   # Erschwert scannen mit nmap und co.
   block in log quick on $Ext inet proto tcp from any to any flags FUP/FUP
   block in log quick on $Ext inet proto tcp from any to any flags SF/SFRA
   block in log quick on $Ext inet proto tcp from any to any flags /SFRA
   block in log quick on $Ext os NMAP

   # IP Spoofing verhindern
   block in log quick on $Ext inet from <NoRoute> to any
   block in log quick on $Ext inet from any to <NoRoute>

   # MS-Geroedel blocken
   block in quick on $Int inet proto tcp from $IntNet to any port { 134 >< 140, 445 }
   block in quick on $Int inet proto udp from $IntNet to any port { 134 >< 140, 445 }

   # Active FTP erlauben
   pass in quick on $Ext inet proto tcp from any to any port > 49151 user proxy flags S/SAFR keep state

   # Ping akzeptieren (ablehnen ist uebrigends wenig sinnvoll)
   pass in quick on $Ext inet proto icmp all icmp-type 8 code 0 keep state

   # Ports nach aussen oeffnen
   pass in quick on $Ext inet proto tcp from any to any port $InServicesTCP flags S/SAFR keep state label ServicesTCP
   pass in quick on $Ext inet proto udp from any to any port $InServicesUDP keep state label ServicesUDP

   anchor passin

   # Raus darf (fast) alles
   #pass out quick on $Ext keep state queue (q_def,q_pri)

   # Outgoing SSH always gets the highest priority.
   pass out quick on $Ext proto tcp from any to   any port ssh keep state  queue ( ssh_out, ssh_ack_out )

   # outgoing TCP
   pass out quick on $Ext proto tcp all keep state queue ( std_out, ack_out )

   # outgoing UDP
   pass out quick on $Ext proto udp all keep state queue std_out

* :ref:`genindex`

Zuletzt geändert: |date|

