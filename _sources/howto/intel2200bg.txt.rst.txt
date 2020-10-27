Intel 2200BG
============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Um unter FreeBSD ein Intel PRO/Wireless 2200BG-Interface nutzen zu
können (steckt in einem SIEMENS AMILO 1425) sind nur wenige Schritte
notwendig.

Treiberinstallation
-------------------

Zunächst muss der Treiber per Modul gestartet oder in den Kernel
einkompiliert werden. Das Laden per Modul geschieht via:

::

  # kldload -v if_iwi

Möchte man den Treiber in den Kernel integrieren, so sind zwei
zusätzliche Angaben in der Konfigurationsdatei des Kernels zu machen:

::

   device pci
   device wlan
   device iwi

Ein
`http://wiki.bsdforen.de/index.php/Kernel_kompilieren_%28FreeBSD%29 Kompilieren des Kernels <http://wiki.bsdforen.de/index.php/Kernel_kompilieren_%28FreeBSD%29 Kompilieren des Kernels>`__
folgt diesem und der Treiber ist prinzipiell einsatzfähig. Bei der
Kernel-Methode ist natürlich ein reboot notwendig.

Firmware
--------

Zunächst ist es für Karten von diesem Hersteller üblich, keine on-Chip
gelieferte Firmware zu nutzen, sondern sie beim laden des Treibers
hinzuzuladen. Dafür wird eine Firmware benötigt. Diese lässt sich bequem
per
`http://wiki.bsdforen.de/index.php/FreeBSD_-_Ports FreeBSD-Portsystem <http://wiki.bsdforen.de/index.php/FreeBSD_-_Ports FreeBSD-Portsystem>`__
installieren.

::

  # cd /usr/ports/net/iwi-firmware # make install clean

Anschliessend wird gemeldet, dass die Firmware für verschiedene Modi
nunmehr unter ``/boot/firmware`` zu finden ist. In diesem Verzeichnis
sind nach der Portinstallation folgende Firmwares verfügbar:

::

   iwi-boot.fw           # Boot mode firmware
   iwi-bss.fw            # BSS mode firmware
   iwi-ibss.fw           # IBSS mode firmware
   iwi-sniffer.fw        # Sniffer mode firmware
   iwi-ucode-bss.fw      # BSS mode micro-code
   iwi-ucode-ibss.fw     # IBSS mode micro-code
   iwi-ucode-sniffer.fw  # Sniffer mode micro-code

Für den jeweiligen Einsatz sollte also auch immer der richtige Modus
gewählt werden.

Um den am häufigsten verwendeten Modus BSS (Infrastruktur-Netzwerk)
einzustellen, wird nun mit dem folgenden Befehl firmwaretechnisch
eingestellt:

::

  # iwicontrol -i iwi0 -d /boot/firmware -m bss

Hier kann man nun einfach überprüfen, ob alls korrekt verlaufen ist:

::

  # ifconfig iwi0 
  iwi0: flags=8806<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
     ether xx:xx:xx:xx:xx:xx
     media: IEEE 802.11 Wireless Ethernet autoselect (autoselect)
     status: no carrier
     ssid ""
     channel -1 authmode OPEN powersavemode OFF powersavesleep 100
     rtsthreshold 2312 protmode CTS txpower 100
     wepmode OFF weptxkey 0

Eine permanente Konfiguration der Karte schon beim Booten des Rechners
ist mit dem Eintrag ``iwi_enable="YES"`` in ``/etc/rc.conf`` zu
bewerkstelligen.

Konfiguration der Karte
-----------------------

Nun fehlt nur noch die Assoziierung mit einem Accesspoint. Dies
geschieht mit **ifconfig(8)**. Um herauszubekommen, welche APs überhaupt
verfügbar sind, genügt folgender Befehl:

::

  # ifconfig iwi0 list ap

Nun wird die Karte einer SSID zugeordnet, die man zuvor mit dem
**ifconfig iwi0 list ap**-Befehl herausgefunden hat.

::

  # ifconfig iwi0 ssid <gewählte SSID>

Es fehlt noch das Aktivieren des Interfaces...

::

  # ifconfig iwi0 up

und die Vergabe einer entsprechenden IP-Adresse, was entweder mit einem
**dhclient iwi0** oder der Vergabe einer statischen IP-Adresse
geschieht:

::

  # ifconfig iwi0 192.168.0.2

Jetzt sollte die Karte einsatzfähig sein.

Nutzung
-------

Um das Interface anschließend noch beispielsweise für einen scan zu
nutzen, ist es wichtig, die Karte komplett zu disassoziieren. Das
geschieht mit

::

  # ifconfig iwi0 0.0.0.0 # ifconfig iwi0 ssid "" 
  # ifconfig iwi0A down

Anschließend sind weitere, ungebundene Aktionen (wie beispielsweise das
genannte Scannen) möglich.

Verweise
--------

Die meisten dieser Informationen stammen von `Damien Bergaminis
Homepage <http://damien.bergamini.free.fr/ipw/iwi-freebsd.html>`__.
Sollte etwas nicht klappen, ist das sicher die richtige Anlaufstelle.

* :ref:`genindex`

Zuletzt geändert: |date|

