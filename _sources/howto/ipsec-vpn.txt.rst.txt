IPSEC-VPN
=========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel ist noch recht jung und vermutlich sehr auf den Aufbau des
Autors zugeschnitten, trotzdem hoffe ich, dass dieser Artikel jemandem die
nötigen Denkanstösse liefern kann.

Versuchsaufbau
--------------

In diesem Artikel wird davon ausgegangen, dass auf beiden Seiten fixe
IP-Adressen verfügbar sind. In beiden Netzen steht ein Gateway Rechner,
der auf einem Interface mit einer öffentlichen IP-Adresse mit dem
Internet verbunden ist und mit einem zweiten Interface mit einer
RFC1918-Adresse mit dem internen Netz verbunden ist. Diese Rechner sind
auf beiden Seiten auch für die Paketfilterung und für NAT zuständig.

================================ ============== ================================ ==============
**Netz A:**                                     **Netz B:**                     
================================ ============== ================================ ==============
Interne Netzadresse:             192.168.0.0/24 Interne Netzadresse:             192.168.1.0/24
Interne IP-Adresse des Gateways: 192.168.0.1    Interne IP-Adresse des Gateways: 192.168.1.1
Externe IP-Adresse des Gateways: 194.1.1.1      Externe IP-Adresse des Gateways: 62.2.2.2
================================ ============== ================================ ==============

IPSEC-Einrichtung
-----------------

Aktivieren von IPSec im Kernel
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Zuerst sollte man ueberprüfen, ob IPSec im Kernel aktiviert ist. Das
kann man mit Hilfe von sysctl ueberprüfen. Folgendes sollte bei
folgender Eingabe erscheinen:

::

  $ sysctl net.inet.ipsec

  net.inet.ipsec.esp_trans_deflev: 1 net.inet.ipsec.esp_net_deflev: 1
  net.inet.ipsec.ah_trans_deflev: 1 net.inet.ipsec.ah_net_deflev: 1
  net.inet.ipsec.ah_cleartos: 1 net.inet.ipsec.ah_offsetmask: 0
  net.inet.ipsec.dfbit: 0 net.inet.ipsec.ecn: 0 net.inet.ipsec.debug: 0
  net.inet.ipsec.esp_randpad: -1

Bekommt man nur die Meldung **sysctl: unknown oid net.inet.ipsec** und
bringt ein Aufruf von **setkey -v** ein **pfkey_open: Protocol not
Supported**, dann ist IPSec nicht im Kernel aktiviert.

Hierzu muss man folgendes im Kernel hinzufügen:

::

   options   IPSEC        #IP security
   options   IPSEC_ESP    #IP security (crypto; define w/ IPSEC)

Optional kann man fuers Debuggen noch folgendes im Kernel hinzufuegen:

::

   options   IPSEC_DEBUG  #debug for IP security

Nach dem Installieren des Kernels und Restarten des Systems sollte IPSec
im Kernel aktiviert sein.

Weitere Informationen findet man im FreeBSD Handbuch
http://www.de.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/ipsec.html

.. warning::

  **FreeBSD 6.1**: Leider ist IPSec weder im GENERIC Kernel aktiviert, noch
  gibt es einen Hinweis in der Datei **src/sys/i386/conf/NOTES**. Selbst ein
  **man ipsec** läßt sich nicht über die Kernel-Optionen IPSEC aus. Es ist aber
  wichtig das diese im Kernel hinzugefügt werden.

Einstellungen im Basissytem
---------------------------

/etc/rc.conf
~~~~~~~~~~~~

Auf dem Gateway von Netz A (192.168.0.0/24):

::

   gif_interfaces="gif0"
   gifconfig_gif0="194.1.1.1 62.2.2.2"
   ifconfig_gif0="inet 192.168.0.1 192.168.1.1 netmask 0xffffffff"
   ipsec_enable="YES"              # Enable IPSEC at system start
   ipsec_file="/etc/ipsec.conf"    # Policy rules for IPSEC
   static_routes="ipsectob"        # Static routes for IPSEC
   route_ipsectob="-net 192.168.1.0/24 192.168.1.1"

Auf dem Gateway von Netz B (192.168.1.0/24):

::

   gif_interfaces="gif0"
   gifconfig_gif0="62.2.2.2 194.1.1.1"
   ifconfig_gif0="inet 192.168.1.1 192.168.0.1 netmask 0xffffffff"
   ipsec_enable="YES"              # Enable IPSEC at system start
   ipsec_file="/etc/ipsec.conf"    # Policy rules for IPSEC
   static_routes="ipsectoa"        # Static routes for IPSEC
   route_ipsectoa="-net 192.168.0.0/24 192.168.0.1"

**gif_interfaces**, **gifconfig_gif0** und **ifconfig_gif0** erzeugen
beim Systemstart die nötigen ifconfig-Aufrufe, die den Tunnel festlegen.
Die ``ipsec_*``-Einträge sorgen dafür, daß die nötigen Kommandos
ausgeführt werden, die den Tunnel-Traffic auch verschlüsseln (spadd) und
die Einträge für die Routen erzeugen beim Systemstart die nötige
statische Route zum jeweils anderen Tunnel-Endpunkt.

/etc/ipsec.conf
~~~~~~~~~~~~~~~

Die Datei **/etc/ipsec.conf** konfiguriert den Tunnel und legt fest,
welcher Verkehr verschlüsselt durch den Tunnel muß:

Auf dem Gateway von Netz A (192.168.0.0/24):

::

   spdadd 192.168.0.0/24 192.168.1.0/24 any -P out \
     ipsec esp/tunnel/194.1.1.1-62.2.2.2/require;
   spdadd 192.168.1.0/24 192.168.0.0/24 any -P in \
     ipsec esp/tunnel/62.2.2.2-194.1.1.1/require;

Auf dem Gateway von Netz B (192.168.1.0/24):

::

   spdadd 192.168.1.0/24 192.168.0.0/24 any -P out \
     ipsec esp/tunnel/62.2.2.2-194.1.1.1/require;
   spdadd 192.168.0.0/24 192.168.1.0/24 any -P in \
     ipsec esp/tunnel/194.1.1.1-62.2.2.2/require;

Racoon
~~~~~~

Racoon ist im Port
`security/ipsec-tools <https://www.google.com/search?q=security/ipsec-tools&btnI=lucky>`__
enthalten und handelt zwischen den beteiligten Gateways die nötigen
Informationen für die Authentifizierung und Verschlüsselung aus. In
diesem Beispiel wird ein pre-shared-key verwendet. Im
**Listen**-Abschnitt muß man in der Zeile mit **isakmp $Öffentliche-IP
[500];** anstelle von **$Öffentliche-IP** wirklich die IP-Adresse
einsetzen und die **\*_identifier**-Einträge sollten jeweils die E-Mail
Adresse einer Kontaktperson enthalten.

/usr/local/etc/racoon/racoon.conf
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   # $KAME: racoon.conf.in,v 1.18 2001/08/16 06:33:40 itojun Exp $

   # "path" must be placed before it should be used.
   # You can overwrite which you defined, but it should not use due to confusing.
   path include "/usr/local/etc/racoon" ;
   #include "remote.conf" ;

   # search this file for pre_shared_key with various ID key.
   path pre_shared_key "/usr/local/etc/racoon/psk.txt" ;

   # racoon will look for certificate file in the directory,
   # if the certificate/certificate request payload is received.
   path certificate "/usr/local/etc/cert" ;

   # "padding" defines some parameter of padding.  You should not touch these.
   padding
   {
           maximum_length 20;      # maximum padding length.
           randomize off;          # enable randomize length.
           strict_check off;       # enable strict check.
           exclusive_tail off;     # extract last one octet.
   }

   # if no listen directive is specified, racoon will listen to all
   # available interface addresses.
   listen
   {
           #isakmp ::1 [7000];
           #isakmp 202.249.11.124 [500];
           #admin [7002];          # administrative's port by kmpstat.
           #strict_address;        # required all addresses must be bound.
           isakmp $Öffentliche-IP [500];
   }

   # Specification of default various timer.
   timer
   {
           # These value can be changed per remote node.
           counter 5;              # maximum trying count to send.
           interval 40 sec;        # maximum interval to resend.
           persend 1;              # the number of packets per a send.

           # timer for waiting to complete each phase.
           phase1 50 sec;
           phase2 30 sec;
   }

   remote anonymous
   {
           #exchange_mode main,aggressive;
           exchange_mode aggressive,main;
           doi ipsec_doi;
           situation identity_only;

           #my_identifier address;
           my_identifier user_fqdn "kontaktperson@netza.local";
           peers_identifier user_fqdn "kontaktperson@netza.local";
           #certificate_type x509 "mycert" "mypriv";

           nonce_size 16;
           lifetime time 1 hour;   # sec,min,hour
           initial_contact on;
           #support_mip6 on;
           support_proxy on;
           proposal_check obey;    # obey, strict or claim

           proposal {
                   encryption_algorithm 3des;
                   hash_algorithm sha1;
                   authentication_method pre_shared_key ;
                   dh_group 2 ;
           }
   }

   sainfo anonymous
   {
           pfs_group 1;
           lifetime time 30 sec;
           encryption_algorithm 3des ;
           authentication_algorithm hmac_sha1;
           compression_algorithm deflate ;
   }

/usr/local/etc/racoon/psk.txt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Auf dem Gateway von Netz A (192.168.0.0/24):

::

   62.2.2.2   sfkjh%I-ZO§gLKabf

Auf dem Gateway von Netz B (192.168.1.0/24):

::

   194.1.1.1   sfkjh%I-ZO§gLKabf

Paketfilter-Konfiguration
-------------------------

Im Beispiel wird als Paketfilter IPFW mit divert (natd) verwendet. Es
ist unbedingt darauf zu achten, dass Traffic für den IPSEC-Tunnel
**nicht** auf eine divert-Regel (NAT) treffen darf (jedenfalls gelang es
dem Autor nicht, Traffic durch den Tunnel zu bekommen, wenn die Pakete
auf die divert-Regel trafen).

Einstellungen im Basissystem
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _etcrc.conf-1:

/etc/rc.conf
~~~~~~~~~~~~

::

   natd_enable="YES"               # Enable natd for Port Adress Translation
   natd_interface="tun0"
   natd_flags="-f /etc/natd.conf"
   firewall_enable="YES"           # Enable packet filtering
   firewall_script="/etc/ipfw.conf"
   gateway_enable="YES"            # Enable routing

/etc/natd.conf
~~~~~~~~~~~~~~

::

   use_sockets yes
   same_ports yes
   unregistered_only yes
   dynamic yes
   interface tun0

/etc/ipfw.conf
~~~~~~~~~~~~~~

.. warning::

  Bitte beachten, daß dieses Regelwerk keinesfalls vollständig ist und unter
  keinen Umständen in genau dieser Form verwendet werden soll. Es soll nur
  verdeutlichen, welche Regeln man wo setzen muß, damit der IPSEC-Traffic
  ordnungsgemäß durch den Tunnel geleitet wird.

::

   #!/bin/sh

   # Define the firewall command
   fwcmd="/sbin/ipfw"
   # Define variables
   dsl="tun0"
   extif="ed0"
   intif="rl0"
   myip="192.168.0.1/32"
   trusted="192.168.0.0/24"
   trustedremote="192.168.1.0/24"

   # Force a flushing of the current rules before we reload
   ${fwcmd} -f flush

   # Allow IPSEC
   ${fwcmd} add allow udp from any to any isakmp via ${dsl} // ISAKMP
   ${fwcmd} add allow esp from any to any via ${dsl} // ESP

   # IPSEC tunnel traffic
   ${fwcmd} add allow icmp from ${trusted} to ${trustedremote}
   ${fwcmd} add allow icmp from ${trustedremote} to ${trusted}
   ${fwcmd} add reset all from ${trustedremote} to ${trusted} dst-port 135,137,138,139,445 // block SMB
   ${fwcmd} add reset all from ${trusted} to ${trustedremote} dst-port 135,137,138,139,445 // block SMB
   ${fwcmd} add allow tcp from ${trusted} to ${trustedremote} dst-port telnet // allow telnet
   ${fwcmd} add allow tcp from ${trustedremote} to ${trusted} src-port telnet // allow telnet
   ${fwcmd} add allow tcp from ${trusted} to ${trustedremote} dst-port rdp // allow RDP
   ${fwcmd} add allow tcp from ${trustedremote} to ${trusted} src-port rdp // allow RDP
   # Block the rest, be polite
   ${fwcmd} add unreach port udp from ${trustedremote} to ${trusted}
   ${fwcmd} add unreach port udp from ${trusted} to ${trustedremote}
   ${fwcmd} add reset all from ${trustedremote} to ${trusted}
   ${fwcmd} add reset all from ${trusted} to ${trustedremote}

   # divert for NAT
   ${fwcmd} add divert natd all from any to any via ${dsl}

   # check state for dynamic rules
   ${fwcmd} add check-state

   # external interface (extif)
   # allow everything between external interface and modem
   ${fwcmd} add allow all from any to any via ${extif}

   # PPPoE interface (ext)
   # allow icmp
   ${fwcmd} add allow icmp from any to any via ${dsl}

   # reject critical Microsoft ports
   ${fwcmd} add reset all from any to me dst-port 135,137,138,139,445,5000 in recv ${dsl}

   # allow special protocols in
   ${fwcmd} add allow tcp from any to me dst-port https in via ${dsl} keep-state // https

   # reject NetBIOS traffic trying to leave my network
   ${fwcmd} add reset all from any to any src-port 135,137,138,139,445 out xmit ${dsl}

   # let everything else out
   ${fwcmd} add allow all from any to any out xmit ${dsl} keep-state

   # reject the rest
   ${fwcmd} add reset all from any to me in via ${dsl} setup keep-state

   # internal interface (int)
   # allow everything from internal network
   ${fwcmd} add allow all from any to any via ${intif} keep-state

   # loopback interface
   # allow everything
   ${fwcmd} add allow all from any to any via lo0

   # default: reject everything
   ${fwcmd} add unreach port udp from any to any
   ${fwcmd} add reset all from any to any

* :ref:`genindex`

Zuletzt geändert: |date|

