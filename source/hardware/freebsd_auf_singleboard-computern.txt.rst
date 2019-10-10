FreeBSD auf Singleboard-Computern
=================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

FreeBSD kann man problemlos auf sogenannten Appliances (kleinen,
engeriesparenden Rechnern ohne Bildschirm, Maus oder Tastatur)
einsetzen. Bisher habe ich das mit einer Soekris net4801 und einer PC
Engines wrap 2e.1 ausprobiert.

Hardware-Beschreibung
---------------------

PC-Engines WRAP
---------------

Die `Wireless Router Application
Platform <http://www.pcengines.ch/wrap.htm>`__ (kurz: WRAP) ist ein
Produkt der schweizer Firma
`http://www.pcengines.ch/PC Engines <http://www.pcengines.ch/PC Engines>`__.
Sie ist in mehreren Ausführungen mit bis zu drei Netzwerk-, einer
seriellen und bis zu zwei Mini-PCI-Schnittstellen erhältlich. Tastatur-,
Maus- und Videoschnittstellen fehlen und lassen sich auch nicht
nachrüsten. Eine USB-Schnittstelle ist optinal erhältlich und muss
selbst eingelötet werden. Die Geode-CPU ist mit 233 MHz ausreichend
schnell, um Router jeder Art zu bauen. Da sowohl IDE- als auch
SCSI-Schnittstellen fehlen, eignet sich die Hardware nicht für
Fileserver.

Soekris net4801
---------------

Die `net4801 <http://www.soekris.com/net4801.htm>`__ ist ein Produkt der
amerikanischen Firma `Soekris <http://www.soekris.com/>`__. Auch hier
stehen drei Netzwerk-, eine serielle und ein Mini-PCI-Schnittstelle zur
Verfügung; Tastatur-, Maus- und Video-Schnittstellen fehlen. Darüber
hinaus bietet die Soekris serienmässig USB und eine zweite interne
serielle Schnittstelle sowie einen IDE-Notebook-Stecker, über den
2,5"-Platten auch mit Strom versorgt werden.

FreeBSD-Unterstützung
---------------------

Seit FreeBSD 5.4 unterstützt FreeBSD mit der Kernel-Option ``CPU_GEODE``
die in beiden Appliances verbaute CPU. Für Soekris-Hardware gibt es
zusätzlich die Option ``CPU_SOEKRIS``, die die Ansteuerung der
Error-Leuchtdiode und des Temperatursensors ermöglicht. Im
GENERIC-Kernel fehlt die Unterstützung für diese Hardware, deshalb
benötigt man auf jeden Fall einen selbst gebauten Kernel. Da viele
SBC-Anwendungen kein vollständiges FreeBSD-System benötigen, baut man
ein massgeschneidertes Betriebssystem nur für die geplante Anwendung. Da
die Einstellungen am Build-System auch das Host-FreeBSD betreffen, kann
man diese Arbeiten auch in einem `Jail </howto/Jails>`__ oder einer
``chroot-Umgebung`` durchführen; z.B. mittels
`MiniBSD <http://www.minibsd.org/>`__.

System an SBCs mit GEODE-Prozessor anpassen
-------------------------------------------

Die SBC-Hardware unterscheidet sich in einigen Punkten von Standard-PCs,
insbesondere muss man die CPU-Optimierung an den Prozessor anpassen und
sich um einen Hardware-Bug im Timer der CPU herumprogrammieren.

Kernel-Konfigurationsdatei
--------------------------

Folgende Konfigurationsdatei ist sowohl für Soekris als auch WRAP
einsetzbar. Optionale Teile sind unterhalb der Konfigurationsdatei
beschrieben, auskommentiert und können bei Bedarf durch entfernen der
Kommentarzeichen wieder aktiviert werden. Der Schönheit halber sollte
man im Folgenden eventuell ``WRAP`` durch ``SOEKRIS`` ersetzen.

``File:/usr/src/sys/i386/conf/WRAP``:

::

    # SBC -- Hardware specific kernel configuration file for FreeBSD/i386 on
    # Soekris net4801
    #        or PC Engines Wrap
    #
    # $cvs: src/sys/i386/conf/WRAP,v 1.2 2007/01/20 23:21:50 soc Exp $

    machine         i386
    cpu             I586_CPU         # I486_CPU und I686_CPU werden nicht gebraucht
    options         CPU_GEODE        # This option is necessary because the i8254 timecounter is toast.
    ident           SBC
    options         SCHED_4BSD       # 4BSD scheduler
    options         PREEMPTION       # Enable kernel thread preemption
    options         INET             # InterNETworking
    #1# options     INET6            # IPv6

IPv6-Unterstützung benötigt man nur dann, wenn man tatsächlich an einem
solchen Netzwerk teilnimmt.

::

    options         FFS              # Berkeley Fast Filesystem
    options         SOFTUPDATES      # soft updates brauchen kaum Platz, also
    'rein damit
    #2# options     UFS_ACL          # Access Control Lists

ACLs benötigt man nur für erweiterte Zugriffsrechteverwaltung, z.B. für
Windows-Fileserver

::

    #3# options     UFS_DIRHASH      # Große Directories

Unterstützung für Directories mit sehr vielen Einträgen: nur für
Fileserver

::

    #4# options     MD_ROOT          # MD als root device unterstützen

Nur für Systeme, deren Root-Filesystem in einer RAMdisk liegen soll

::

    #5# options     NFSCLIENT        # Network Filesystem Client
    #5# options     NFSSERVER        # Network Filesystem Server
    #6# options     NFS_ROOT         # NFS als root device unterstützen
    #(benötigt NFSCLIENT)

NFS wird nur für Fileserver mit Unix-Clients benötigt. NFS_ROOT lädt das
Root-Filesystem über NFS. Das ist hilfreich beim Entwicklung eigener
Erweiterungen, die nur auf der SBC-Hardware getestet werden können.

::

    #7# options     MSDOSFS          # MSDOS Filesystem
    #7# options     CD9660           # ISO 9660 Filesystem -- nur für CD-Server

MSDOSFS und CD9660 benötigt man nur, wenn man solche Dateisysteme
tatsächlich ansprechen will, z.B. auf USB-Sticks oder für CD-Server.

::

    options         PROCFS           # Process filesystem (requires PSEUDOFS)
    options         PSEUDOFS         # Pseudo-filesystem framework
    options         GEOM_GPT         # GUID Partition Tables.
    options         COMPAT_43        # Compatible with BSD 4.3 [[KEEP|THIS!]]
    #8# options     COMPAT_FREEBSD4  # Compatible with FreeBSD4
    #8# options     COMPAT_FREEBSD5  # Compatible with FreeBSD5
   Kompatibilität zu alten FreeBSD-Systemen benötigt man nur, wenn dies eine Anwendung verlangt
   <code>
    options         SCSI_DELAY=3000  #9# Delay (in ms) before probing SCSI

Das SCSI-Delay ist verkürzt, da nur USB-Geräte unterstützt werden.

::

    options         SYSVSHM          # SYSV-style shared memory
    options         SYSVMSG          # SYSV-style message queues
    options         SYSVSEM          # SYSV-style semaphores
    options         _KPOSIX_PRIORITY_SCHEDULING # POSIX P1003_1B real-time
    extensions
    options         ADAPTIVE_GIANT   # Giant mutex is adaptive.

    device          apic             # I/O APIC

    # Bus support.
    device          pci

    # ATA and ATAPI devices
    device          ata
    options         ATA_STATIC_ID   # Static device numbering
    device          atadisk         # ATA disk drives
    #a# device      ataraid         # ATA RAID drives
    #a# device      atapicd         # ATAPI CDROM drives
    #a# device      atapifd         # ATAPI floppy drives
    #a# device      atapist         # ATAPI tape drives

Im ATA-Abschnitt sollten nur die Geräte aktiviert werden, die
tatsächlich benötigt werden

::

    # Add suspend/resume support for the i8254.
    device          pmtimer

    # Serial (COM) ports
    device          sio             # 8250, 16[[45]]50 based serial ports

    # PCI Ethernet NICs that use the common MII bus controller code.
    device          miibus          # MII bus support
    device          sis             # Silicon Integrated Systems SiS 900/SiS 7016

    # Pseudo devices.
    device          loop            # Network loopback
    device          random          # Entropy device
    device          ether           # Ethernet support
    #n# device      sl              # Kernel SLIP
    #n# device      ppp             # Kernel PPP

``ppp`` bzw. ``sl`` nur dann, falls das Gerät PPP- bzw.
SLIP-Funktionalität benötigt

::

    device          tun             # Packet tunnel.
    device          pty             # Pseudo-ttys (telnet etc)
    device          md              # Memory "disks"
    device          gif             # IPv6 and IPv4 tunneling
    device          faith           # IPv6-to-IPv4 relaying (translation)

    ### device      bpf             # Berkeley packet filter

    # USB support (Nur für Soekris oder WRAP mit USB-Erweiterung notwendig)
    #u# device      uhci            # UHCI PCI->USB interface
    #u# device      ohci            # OHCI PCI->USB interface
    #u# device      usb             # USB Bus (required)
    #u# device      ugen            # Generic
    #u# device      umass           # Disks/Mass storage - Requires scbus and
    #da
    #u# device      scbus           # SCSI bus (required for SCSI)
    #u# device      da              # Direct Access (disks)

Der USB-Abschnitt kann vollständig entfallen, wenn keine USB-Geräte
unterstützt werden sollen. Vorsicht: umass (zur Unterstützung von
USB-Sticks und USB-Festplatten) benötigt die SCSI-Einträge ``scbus`` und
``da``.

::

    # Wireless NIC cards
    #w# device          wlan            # 802.11 support
    #w# device          wlan_wep        # 802.11 WEP support
    #w# device          wlan_ccmp       # 802.11 CCMP support
    #w# device          wlan_tkip       # 802.11 TKIP support
    #w# device          an              # Aironet 4500/4800 802.11 wireless
    #NICs.
    #w# device          ath             # Atheros pci/cardbus NIC's
    #w# device          ath_hal         # Atheros HAL (Hardware Access Layer)
    #w# device          ath_rate_sample # SampleRate tx rate control for ath
    #w# device          awi             # BayStack 660 and others
    #w# device          ral             # Ralink Technology RT2500 wireless
    #NICs.
    #w# device          wi              # WaveLAN/Intersil/Symbol 802.11
    #wireless NICs.
    #w# #device         wl              # Older non 802.11 Wavelan wireless
    #NIC.

Der WLAN-Abschnitt kann vollständig entfallen, wenn keine WLAN-Geräte
unterstützt werden sollen. Andernfalls aktiviert man ausschliesslich die
benötigten Treiber.

Betriebssystem an SBCs anpassen
-------------------------------

In ``/etc/make.conf`` schaltet man all die Teile des Betriebssystems
aus, die man auf Appliances nicht benötigt.

File:/etc/make.conf
-------------------

::

    KERNCONF=           WRAP
    CPUTYPE?=           i586/mmx
    CFLAGS=             -O -pipe
    COPTFLAGS=          -O -pipe
    MAKE_SHELL?=        sh
    COMPAT4X=           YES
    BOOTWAIT=           1000
    NO_ACPI=            true
    NO_CVS=             true
    NO_CXX=             true
    NO_DICT=            true
    NO_DYNAMICROOT=     true
    NO_GDB=             true
    NO_GPIB=            true
    NO_I4B=             true
    NO_LPR=             true
    NO_ATM=             true
    NO_GAMES=           true

    # Beispiele und man-Pages kann man auf SBCs getrost weglassen:
    NO_SHARE=           true # /usr/share/examples ncht einbinden
    NO_SHAREDOCS=       true # /usr/share/docs nicht einbinden
    NO_INFO=            true # GNU Info-Pages weglassen
    NO_MAN=             true # man-Pages weglassen

    # Entwicklungsumgebungen werden auf einem Embedded-System nie
    # benötigt: weg damit.
    NO_FORTRAN=         true #
    NO_OBJC=            true #
    NO_GDB=             true #
    NO_TOOLCHAIN=       true #
    NO_PROFILE=         true #

    # Threads und rekursive Libraries werden nur von wenigen Anwendungen
    # benötigt.
    NO_LIBTHR=          true # ...falls keine Anwendung diese benötigt
    NO_LIBC_R=          true # rekursive C-Lib weglassen

Ab hier beginnen die optionalen Komponenten. Was genau die einzelnen
Schalter bewirken wird in ``make.conf`` genau beschrieben.

::

    NO_BLUETOOTH=       true # Bluetooth-Unterstützung weglassen
    NO_IPFILTER=        true # IPFilter weglassen
    NO_PF=              true # PF weglassen
    NO_AUTHPF=          true #
    NO_INET6=           true # IPv6-Unterstützugn wegassen
    NO_KERBEROS=        true # Kerberos weglassen

    NO_USB=             true # USB-Unterstützung entfällt bei Geräten ohne USB-Schnittstelle (Wrap)
                             # Vorsicht: umass benötigt da scbus im Kernel!

    NO_VINUM=           true # Nur für Fileserver, wird unter 6.x nicht mehr vollständig unterstützt

    # BIND wird nur für DNS benötigt, nicht benutzte Komponenten kann man
    # weglassen
    NO_BIND_DNSSEC=     true # DNSSEC-Unterstützung (dnssec-keygen(8) und dnssec-signzone(8)) weglassen
    NO_BIND_ETC=        true # Default-Dateien in /var/named/etc/namedb weglassen
    NO_BIND_LIBS_LWRES= true # Resolver-Library weglassen (nur für offline-Rechner!)
    NO_BIND_MTREE=      true #
    NO_BIND_NAMED=      true #
    WITH_BIND_LIBS=     true #
    NO_RCMDS=           true # rsh, rcp... sind böse!
    NO_MODULES=         true #

Quellen
-------

Grosse Teile dieser Artikels beruhen auf dem Artikel
"`https://www.bsdforen.de/threads/freebsd-auf-pc-engines-wrap.7987/ FreeBSD auf
PC-Engines WRAP
<https://www.bsdforen.de/threads/freebsd-auf-pc-engines-wrap.7987/ FreeBSD auf
PC-Engines WRAP>`__" von `Elessar
<https://www.bsdforen.de/members/elessar.115/>`__.

* :ref:`genindex`

Zuletzt geändert: |date| 
