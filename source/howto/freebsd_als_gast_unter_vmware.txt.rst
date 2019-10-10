FreeBSD als Gast unter VMware
-----------------------------

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

.. note::

  Dieser Artikel gibt Tipps für die Installation von FreeBSD als
  Gast-Betriebssystem unter der kostenlosen Virtuellen Maschine VMware Server
  1.0.

Zuerst sollte das FreeBSD ISO-Image (CD 1) von der gewünschten zu
installierenden Architektur (i386/amd64) auf dem Hostsystem, auf dem
VMware Server läuft, liegen. Dieses wird für die Installation von
FreeBSD benötigt. Als nächstes startet man die VMware Server Console und
erzeugt mit "File" -> "New" -> "Virtual Machine" eine neue VM-Instanz
mit folgenden Einstellungen:

-  Als Guest operating system sollte "FreeBSD" (Other -> FreeBSD)
   eingestellt werden.
-  Beim Erzeugen der VM sollte nur eine CPU für die VM konfiguriert
   werden, da momentan (Ende 2007) der Support für FreeBSD in VMware
   Server 1.0.x noch "experimental" ist und es mit 2 CPUs (SMP) zu
   Abstürzen, Speicherfehlern etc. im Gastsystem kommt.
-  Der LSI Logic virtual SCSI adapter sollte konfiguriert werden,
   *nicht* der Buslogic SCSI-Adapter.

Im nächsten Schritt sollte der von VMware Server emulierte
Ethernet-Netzwerkadapter geändert werden, da der Standard-Adapter ein
"AMD Lance/PCnet" ist, der von FreeBSD 6.x mit GENERIC-Kernel von lnc(4)
getrieben wird. Diese Kombination aus VMware-Emulation und lnc(4)
arbeitet jedoch nicht zufriedenstellend und führt unter höherer Last
reproduzierbar zu verlorenen Paketen. Es ist daher anzuraten, eine nicht
dokumentierte Option zu verwenden, um auf eine Intel PRO/1000-Emulation
umzuschalten, welche vom in FreeBSD sehr gut unterstützten em(4)
getrieben wird. Dazu wird im Installationsverzeichnis der gerade
erzeugten VM in die **\*.vmx**-Datei folgender Eintrag hinzugefügt:

::

   Ethernet0.virtualDev="e1000"

Nachdem die VM erzeugt wurde, wird das Installations-Image für FreeBSD
in das virtuelle Laufwerk gemounted mit "Edit virtual machine settings"
-> "CD-ROM (IDE 1:0)" -> "Use ISO image:". Nun kann man die virtuelle
Maschine starten und FreeBSD wie gewohnt installieren.

Nach der Installation der /boot/loader.conf hinzufügen, um die
Häufigkeit der Timer-Interrupts zu senken und die Uhr richtig laufen zu
lassen:

::

   kern.hz="100"
   hint.apic.0.disabled=1

VMware-Tools installieren
~~~~~~~~~~~~~~~~~~~~~~~~~

Nach der Installation des Gast-Systems sollten auf diesem die
VMware-Tools installiert werden, um die VM vom Host aus zu steuern
(herunterfahren mit Gast-Unterstützung), die Zeit des Gastsystems mit
dem Host zu synchronisieren etc.

::

   cd /usr/ports/emulators/vmware-guestd6/ && make install clean

Während des Installationsprozesses muss das VMware-Tools CD-Image in das
virtuelle CD-Laufwerk der VM gemounted werden. Dazu wird in der VMware
GUI-Konsole der Menüpunkt "VM" -> "Install VMware Tools..." aufgerufen,
während die FreeBSD-VM läuft. Nach abgeschlossener Installation kann der
vmware-guest Daemon gestartet werden mit

::

   /usr/local/etc/rc.d/vmware-guestd start

* :ref:`genindex`

Zuletzt geändert: |date|

