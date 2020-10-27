Strato MR2 (OpenBSD)
====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

Hier wird die Installation von OpenBSD auf einem Strato MR2 Root Server
beschrieben. Es wird ein Image, welches den Ramdisk Kernel beinhaltet erstellt
und auf den Server geladen. Nach einem Reboot des Servers wird automatisch die
Installationsroutine gestartet. Wichtig ist, dass zu Beginn der Installation
noch keine Netzwerkverbindung besteht, der Zugriff über eine Remote Console ist
unbedingt nötig.

Wer einen älteren Strato Server besitzt und nur per SSH auf den Server
zugreifen kann, sollte die Variante von
`dettus <http://www.dettus.net>`__ benutzen. Der MR2 kann damit nur
bearbeitet werden, wenn das Floppy Image von dettus mit den current srcs
selber erstellt wurde. Also ist meine Variante eigentlich überflüssig...
aber Quantität ist bekanntlich besser als Qualität und eine Methode mehr
schadet nicht. Die Images von dettus sind alle mit Releases gebaut, die
Festplatten können jedoch nur von Current oder dem kommenden Release 4.0
erkannt werden.

Es kann sein, dass einige der hier genannten Punkte überflüssig bzw.
umständlich sind. Nochmal: Das ist meine Variante.

Vorbereitung
------------

Bevor der Server bsdfiziert wird, benötigst du die wichtigsten Daten um
später das Netzwerk einzurichten:

::

   IP Adresse :                   ifconfig
   Hostname:                      cat /etc/hostname
   Domainname und DNS:            cat /etc/resolv.conf
   Default Route:                 route -n

Erstelle ein Image des MBR und lade es auf deinen lokalen Rechner:

::

   # dd if=/dev/sda of=mbr.img bs=1k count=1024 *
   * wir wollen ja nicht kleinlich sein

Erstellen des Installationsimages
---------------------------------

Erstelle ein leeres Images:

::

   # dd if=/dev/zero of=empty.img bs=* count=*
   * 10Mb sollten locker ausreichen 

Erstelle ein neues Image aus mbr.img und empty.img:

::

   # cat mbr.img empty.img > install.img

Lade dir einen Snapshot von bsd.rd vom FTP Server deiner Wahl, z.B.
`cd39.iso <ftp://openbsd.informatik.uni-erlangen.de/pub/OpenBSD/snapshots/i386/cd39.iso>`__
auf deinen lokalen Rechner und Mounte das ISO Image oder brenne es auf
CD (Verschwendung).

Starte qemu mit dem zusammengebastelten Image und dem gemounteten ISO
Image bzw. der gemounteten CD:

::

   # qemu -hda install.img -cdrom *Mountpunkt des ISO Images oder CDLW* -boot d

Bei der Installation musst du beachten, dass du bei den Sets nur bsd.rd
auswählst. Die Zeitzone und alles, was nach dem Installieren der Sets
abgefragt wird, muss nicht auswählt werden, da kein /etc installiert
wird und somit nichts gespeichert werden kann.

Nachdem die Installationsroutine beendet ist, erstelle /etc und
/etc/boot.conf von Hand um sämtliche Ausgaben auf die serielle Konsole
auszugeben:

::

   # mkdir /mnt/etc
   # cat "set tty com0" > /mnt/etc/boot.conf
   # cat "stty com0 57600" >> /mnt/etc/boot.conf

Starte qemu nach der Installation nochmals und boote vom Image:

::

   # qemu -hda install.img -boot c

Schalte gleich um auf die serielle Konsole von qemu:

::

   Strg + Alt + 3

Sollte hier keine Ausgabe zu sehen sein hast du bestimmt etwas falsch
gemacht.... Im Normalfall siehst du jetzt ein "boot>" als Ausgabe. Boote
jetzt den Ramdisk Kernel:

::

   boot> boot bsd.rd 

Wenn die Installationsroutine startet, hat ja alles geklappt.

Installation
------------

Nachdem das Image erstellt ist, gibt es mehrere Möglichkeiten das Image
auf den Server zu bekommen, die Einfachste ist vom lokalen Rechner das
Image über SSH auf den Server zu kopieren:

::

   # bzip2 -c9 install.img | ssh -l root@*server* "bzip2 -cd | dd of=/dev/sda"

Alternativ kann die zweite Festplatte oder der Backupserver von Strato
als Aufbewahrungsort für das Image dienen, falls man es ein zweites Mal
hochladen muss. Danach den Server im normalen Modus starten. Wenn alles
richtig durchgeführt wurde siehst du dort wieder boot> und kannst mit

::

   boot> boot bsd.rd

die Installationsroutine starten.

Bei der Installation halte dich an die `OpenBSD
FAQ <http://www.openbsd.org/faq/faq4.html#Install>`__. Zu beachten gibt
es nur wenige Dinge

-  kann bei der Installation nicht der komplette Platz der Festplatte
   ausgenutzt werden, beende die Installation mit Strg+c und "leere" die
   Festplatte komplett:

::

   # dd if=/dev/zero of=/dev/wd0c *
   * bs und count bleibt euch überlassen, die Platte sollte halt einfach ganz leer sein.

Danach musst du die Installationsroutine erneut starten:

::

   # install

-  bei der Netzmaske musste ich 255.255.0.0 wählen, auch wenn das ein
   oder andere Tutorial 255.255.255.0 oder 255.255.255.255 angibt,
   vergleiche einfach die eigene IP und die Default Route, dann kommst
   du schon drauf.
-  bei der Auswahl der Sets muss ein -current System gewählt werden,
   verwende nicht den vorgegebenen Pfad auf dem FTP Server
   pub/OpenBSD/snapshots/i386/ nicht pub/OpenBSD/3.9/i386
-  die Ausgabe der Konsole auf com0 mit "yes" bestätigen

Ein Neustart und alles sollte funktionieren. Hier noch meine dmesg:

::

   OpenBSD 3.9-current (GENERIC) #874: Fri Jun  9 13:40:50 MDT 2006
       deraadt@i386.openbsd.org:/usr/src/sys/arch/i386/compile/GENERIC
   RTC BIOS diagnostic error ce<clock_battery,ROM_cksum,fixed_disk,invalid_time>
   cpu0: AMD Opteron(tm) Processor 146 ("AuthenticAMD" 686-class, 1024KB L2 cache) 2 GHz
   cpu0: FPU,V86,DE,PSE,TSC,MSR,PAE,MCE,CX8,APIC,SEP,MTRR,PGE,MCA,CMOV,PAT,PSE36,CFLUSH,MMX,FXSR,SSE,SSE2,SSE3
   cpu0: Cool`n'Quiet K8 1996 Mhz: speeds: 2000 1800 1000 Mhz
   real mem  = 1073246208 (1048092K)
   avail mem = 971403264 (948636K)
   using 4256 buffers containing 53764096 bytes (52504K) of memory
   RTC BIOS diagnostic error ce<clock_battery,ROM_cksum,fixed_disk,invalid_time>
   mainbus0 (root)
   bios0 at mainbus0: AT/286+(00) BIOS, date 01/11/06, BIOS32 rev. 0 @ 0xf0010, SMB IOS rev. 2.3 @ 0xf8dc0 (61 entries)
   bios0: Supermicro H8SSL
   pcibios0 at bios0: rev 2.1 @ 0xf0000/0x10000
   pcibios0: PCI IRQ Routing Table rev 1.0 @ 0xf4f50/160 (8 entries)
   pcibios0: no compatible PCI ICU found: ICU vendor 0x1166 product 0x0205
   pcibios0: PCI bus #2 is the last bus
   bios0: ROM list: 0xc0000/0x8000 0xc8000/0x2000! 0xca000/0x1800 0xcb800/0x1800 0xcd000/0x1000
   ipmi at mainbus0 not configured
   cpu0 at mainbus0
   pci0 at mainbus0 bus 0: configuration mode 1 (no bios)
   ppb0 at pci0 dev 1 function 0 "ServerWorks HT-1000 PCI" rev 0x00
   pci1 at ppb0 bus 1
   ppb1 at pci1 dev 13 function 0 "ServerWorks HT-1000 PCIX" rev 0xb2
   pci2 at ppb1 bus 2
   bge0 at pci2 dev 3 function 0 "Broadcom BCM5704C" rev 0x10, BCM5704 B0 (0x2100): irq 9, address 00:30:48:58:25:78
   brgphy0 at bge0 phy 1: BCM5704 10/100/1000baseT PHY, rev. 0
   bge1 at pci2 dev 3 function 1 "Broadcom BCM5704C" rev 0x10, BCM5704 B0 (0x2100): irq 5, address 00:30:48:58:25:79
   brgphy1 at bge1 phy 1: BCM5704 10/100/1000baseT PHY, rev. 0
   pciide0 at pci1 dev 14 function 0 "ServerWorks K2 SATA" rev 0x00: DMA
   pciide0: using irq 11 for native-PCI interrupt
   pciide0: port 0: device present, speed: 1.5Gb/s
   wd0 at pciide0 channel 0 drive 0: <HDT722516DLA380>
   wd0: 16-sector PIO, LBA48, 157066MB, 321672960 sectors
   wd0(pciide0:0:0): using PIO mode 4, Ultra-DMA mode 5
   pciide0: port 1: device present, speed: 1.5Gb/s
   wd1 at pciide0 channel 1 drive 0: <HDT722516DLA380>
   wd1: 16-sector PIO, LBA48, 157066MB, 321672960 sectors
   wd1(pciide0:1:0): using PIO mode 4, Ultra-DMA mode 5
   pciide0: port 2: PHY offline
   pciide0: port 3: PHY offline
   pchb0 at pci0 dev 2 function 0 "ServerWorks HT-1000" rev 0x00
   pcib0 at pci0 dev 2 function 2 "ServerWorks HT-1000 LPC" rev 0x00
   ohci0 at pci0 dev 3 function 0 "ServerWorks HT-1000 USB" rev 0x01: irq 10, version 1.0, legacy support
   usb0 at ohci0: USB revision 1.0
   uhub0 at usb0
   uhub0: ServerWorks OHCI root hub, rev 1.00/1.00, addr 1
   uhub0: 2 ports with 2 removable, self powered
   ohci1 at pci0 dev 3 function 1 "ServerWorks HT-1000 USB" rev 0x01: irq 10, version 1.0, legacy support
   usb1 at ohci1: USB revision 1.0
   uhub1 at usb1
   uhub1: ServerWorks OHCI root hub, rev 1.00/1.00, addr 1
   uhub1: 2 ports with 2 removable, self powered
   ehci0 at pci0 dev 3 function 2 "ServerWorks HT-1000 USB" rev 0x01: irq 10
   usb2 at ehci0: USB revision 2.0
   uhub2 at usb2
   uhub2: ServerWorks EHCI root hub, rev 2.00/1.00, addr 1
   uhub2: 4 ports with 4 removable, self powered
   vga1 at pci0 dev 5 function 0 "ATI Rage XL" rev 0x27
   wsdisplay0 at vga1 mux 1: console (80x25, vt100 emulation)
   wsdisplay0: screen 1-5 added (80x25, vt100 emulation)
   pchb1 at pci0 dev 24 function 0 "AMD AMD64 HyperTransport" rev 0x00
   pchb2 at pci0 dev 24 function 1 "AMD AMD64 Address Map" rev 0x00
   pchb3 at pci0 dev 24 function 2 "AMD AMD64 DRAM Cfg" rev 0x00
   pchb4 at pci0 dev 24 function 3 "AMD AMD64 Misc Cfg" rev 0x00
   isa0 at pcib0
   isadma0 at isa0
   pckbc0 at isa0 port 0x60/5
   pckbd0 at pckbc0 (kbd slot)
   pckbc0: using irq 1 for kbd slot
   wskbd0 at pckbd0: console keyboard, using wsdisplay0
   pcppi0 at isa0 port 0x61
   midi0 at pcppi0: <PC speaker>
   spkr0 at pcppi0
   npx0 at isa0 port 0xf0/16: using exception 16
   pccom0 at isa0 port 0x3f8/8 irq 4: ns16550a, 16 byte fifo
   pccom0: console
   pccom1 at isa0 port 0x2f8/8 irq 3: ns16550a, 16 byte fifo
   biomask fdc5 netmask ffe5 ttymask ffe7
   pctr: user-level cycle counter enabled
   nvram: invalid checksum
   dkcsum: wd0 matches BIOS drive 0x80
   wd1: no disk label
   dkcsum: wd1 matches BIOS drive 0x81
   root on wd0a
   rootdev=0x0 rrootdev=0x300 rawdev=0x302
   clock: unknown CMOS layout 

Danke an sbeh, dettus und alle anderen, die mir bei meinen Problemchen
geholfen haben.

Quellen
-------

-  http://www.dettus.net/openbsd_at_strato.txt
-  http://www.openbsd.org

* :ref:`genindex`

Zuletzt geändert: |date|

