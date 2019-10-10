DHCP-Server einrichten
======================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-netbsd.png

Dieser Artikel erklärt wie unter NetBSD ein DHCP-Server eingerichtet
wird.

.. warning::

  Dieser Artikel stammt von **[moR-pH-euS]** und wurde zuerst `im Forum
  <http://www.bsdforen.de/showthread.php?t=4737>`__ veröffentlicht und von
  steinlaus ins alte Wiki übertragen.

Zuerst muss die Datei **dhcpd.conf** unter **/etc** erstellt werden.
Diese Konfigurationsdatei ist ziemlich selbsterklärend.

**/etc/dhcpd.conf**

::

   # global dhcpd parameters
   deny unknown-clients;                   #allow unknown connections
   ddns-update-style none;                 #disallow dynamic DNS updates

   #which network interface the server will listen on
   subnet 192.168.1.0 netmask 255.255.255.0 { #the zeros indicate which range


   range 192.168.1.5 192.168.1.10;

   }                                                  #of addresses are allowed to connect

    #set of parameters common to all clients
     group {
       option broadcast-address 192.168.1.255;
       option domain-name "home";
       option domain-name-servers 194.152.64.35, 194.25.2.132;
       option routers 192.168.1.254;
       option subnet-mask 255.255.255.0;

     #set of parameters specific to one particular host
       host freebsd-wk {
        hardware ethernet <die-entsprechende-mac-adresse>;
        fixed-address 192.168.1.5;
        option host-name "freebsd-wk";              #name of the host (if the fixed address
                                                         #doesn't resolve to a simple name)
    }

     #set of parameters specific to one particular host
        host w2-moni {
         hardware ethernet <die-entsprechende-mac-adresse>;
         fixed-address 192.168.1.6;
         option host-name "w2-moni";              #name of the host (if the fixed address
                                                           #doesn't resolve to a simple name)
    }

       #set of parameters specific to one particular host
          host netbsd-sparc {
           hardware ethernet <die-entsprechende-mac-adresse>;
           fixed-address 192.168.1.7;
           option host-name "netbsd-sparc"; #name of the host (if the fixed address
                                                              #doesn't resolve to a simple name)
   }

     #set of parameters specific to one particular host
        host debian-gameserver {
         hardware ethernet <die-entsprechende-mac-adresse>;
         fixed-address 192.168.1.9;
         option host-name "debian-gameserver";              #name of the host (if the fixed address
                                                          #doesn't resolve to a simple name)
    }


   }

So sieht sie bei mir aus. Die Kommentare erklären alles, oben legt ihr
die globalen Werte fest und die spezifischen Host-Namen der Maschinen
(die MAC-Adressen müsst ihr noch auf eure anpassen, außerdem noch die
Hostnamen und IPs)

In der ``'/etc/rc.conf``' tragt ihr folgendes ein:

::

   dhcpd=YES
   dhcpd_flags="-q rtk0"

dabei ist rtk0 das interne Interface. Dann könnt ihr den Server auch
schon starten (der Eintrag in der rc.conf ist ja nur dafür, daß beim
Start des Servers der Dienst auch gestartet wird, aber wir müssen ja
nicht immer booten ;-)

::

   # /etc/rc.d/dhcpd start

komischerweise wollte der Server bei mir erst starten, als ich eine
Datei angelegt habe:

::

   # touch /var/db/dhcpd.leases

danach lief er ohne Probleme. Nun müsst ihr nur noch auf den Clients
DHCP einstellen.

* :ref:`genindex`

Zuletzt geändert: |date|

