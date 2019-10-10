VLAN-Konfiguration mit FreeBSD
==============================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel soll eine kurze Einführung in die Einbindung eines
FreeBSD-Rechners in ein Layer-2 VLAN nach IEEE 802.1q geben. Erklärt
wird die Einrichtung von (mehreren) logischen Interfaces auf einer
physikalischen Netzwerkkarte.

Vorwissen
---------

-  IEEE 802.1q VLANs hängen eine Markierung (Englisch "tag") an ein
   Ethernet-Paket an. In diesem ist dann codiert, zu welchem virtellen
   LAN (VLAN) das Paket gehört.
-  Normalerweise wird das gesamte VLAN-Management in den Switchen in
   Hardware erledigt. Für DHCP-Server, Sicherheits-Zwecke etc. kann es
   jedoch sinnvoll sein, einen Rechner an mehreren VLANs gleichzeitig
   teilhaben zu lassen, über eine physikalische Netzwerkkarte. Dies
   erfolgt dann sehr einfach durch Zuordnung des Switchports des
   Rechners zu mehreren tagged VLANs. Die Aufteilung der Pakete auf die
   unterschiedlichen VLANs muss in diesem Fall auf dem Rechner erfolgen.
   Damit befasst sich dieser Artikel.

Notwendige Vorbedingung
-----------------------

Der VLAN-Treiber muss entweder in den Kernel kompiliert sein (ist im
Standard GENERIC-Kernel der Fall). Wenn ein eigener Kernel verwendet
wurde und der VLAN-Treiber fehlt kann das Laden des Moduls nachgeholt
werden über den Eintrag von

::

   if_vlan_load="YES"

in die Datei

::

   /boot/loader.conf

Wenn die Datei fehlt einfach anlegen.

Weiterhin muss die Netzwerkkarte in der Lage sein, tagged 802.1q Pakete
zu verarbeiten. Im Zweifelsfall hilft die Manpage von
`vlan(4) <http://www.freebsd.org/cgi/man.cgi?query=vlan&sektion=4&format=html>`__
weiter.

Einrichtung in FreeBSD
----------------------

Alle notwendige Konfiguration kann am einfachsten in der
Start-Konfigurationsdatei

::

   /etc/rc.conf

erledigt werden.

Zunächst muss für jedes VLAN ein logisches cloned_interface angelegt
werden. Dieses Interface bildet dann einen Teilnehmer an einem tagged
VLAN. Es ist sinnvoll, dem Interface einen leicht zu merkenden Namen zu
geben. In diesem Beispiel wird der Name "vlan8" für das Interface mit
Teilnahme am tagged VLAN mit Tag-Nummer 8 vergeben.

::

   cloned_interfaces="vlan8"

Im nächsten Schritt muss dem cloned_interface wie bei physikalischen
Netzwerkkarten eine IP-Adresse, die IP-Netzmaske, die Tag-Nummer des
VLANs und die physikalische Netzwerkkarte zugewiesen werden. Zum
Beispiel für unser VLAN 8:

::

   ifconfig_vlan8="inet 192.168.8.3 netmask 255.255.255.0 vlan 8 vlandev bge1"

Kurze Erklärung der Parameter:

-  inet 192.168.8.3 - weist die IPv4-Adresse zu
-  netmask 255.255.255.0 - weist die IPv4-Netzmaske zu
-  vlan 8 - setzt die VLAN-Tagnummer, hier auf Tag 8
-  vlandev bge1 - weist dem VLAN-Interface eine physikalische
   Netzwerkkarte zu, auf die VLAN-Frames empfangen und gesendet werden
   sollen. Hier handelt es sich um die Netzwerkkarte /dev/net/bge1

Zum Schluss muss das physikalische Interface noch aktiviert werden. Dies
ist insbesondere nötig, wenn der Netzwerkkarte selbst keine (untagged)
Pakete empfängt und sendet, also selber keine IP-Adresse hat. Daher:

::

   ifconfig_bge1="up"

Für unser Beispiel.

Damit ist das VLAN-Interface angelegt und kann nun ganz normal mit den
üblichen Betriebssystemmitteln wie eine physikalische
Ethernet-Netzwerkkarte verwaltet werden. Für unser Beispiel sieht die
ifconfig-Ausgabe dann so aus:

::

   bge1: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
         options=1b<RXCSUM,TXCSUM,VLAN_MTU,VLAN_HWTAGGING>
         ether 00:14:5e:bc:3f:cd
         media: Ethernet autoselect (none)
         status: no carrier
   vlan8: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
         inet 192.168.8.3 netmask 0xffffff00 broadcast 192.168.8.255
         ether 00:14:5e:bc:3f:cd
         media: Ethernet autoselect (none)
         status: no carrier
         vlan: 8 parent interface: bge1

Leider war beim Schreiben dieses Artikels die Netzwerkkarte wie man an
"media" sehen kann nicht mit einem Switch verbunden. ;-)

Mehrere logische VLAN-Interfaces
--------------------------------

Das Anlegen mehrerer VLAN-Interfaces erfolgt in der Datei /etc/rc.conf
in der cloned_interface Zeile durch Leerschritte getrennt, zum Beispiel:

::

   cloned_interfaces="vlan8 vlan12 vlan117"

Für jedes angelegte VLAN-Interface ist dann noch je eine ifconfig-Zeile
für die IP-Konfiguration nötig, zum Beispiel:

::

   ifconfig_vlan8="inet 192.168.8.3 netmask 255.255.255.0 vlan 8 vlandev bge1"
   ifconfig_vlan12="inet 192.168.12.3 netmask 255.255.255.0 vlan 12 vlandev bge1"
   ifconfig_vlan117="inet 192.168.117.3 netmask 255.255.255.0 vlan 117 vlandev bge1"

* :ref:`genindex`

Zuletzt geändert: |date|

