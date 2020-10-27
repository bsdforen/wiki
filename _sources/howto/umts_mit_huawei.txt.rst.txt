UMTS mit HUAWEI
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Diese Anleitung beschreibt die Benutzung des HUAWEI E160E USB-Stick,
welcher als N24- und Pro7-Stick vertrieben wird. Der Provider ist
Vodafone. Der Autor weiss nicht, ob die Einstellung im ppp.conf optimal
sind. Dies müsste überprüft und ggf. verbessert werden!

Im Kernel muß das Modul u3g einkompiliert bzw. das Modul mittels kldload
geladen sein. Sollte der Stick nicht von u3g erkannt werden, muss evtl.
auf FreeBSD 8.x umgestiegen werden.

Schritt 1
---------

Den Stick in einen USB-Port stecken. dmesg(1) sollte in etwa Folgendes
ausgeben:

::

   u3g_huawei_init:272:
   usb_alloc_device:1817: Found Huawei auto-install disk!
   ugen4.2: <HUAWEI Technology> at usbus4
   ugen4.2: <HUAWEI Technology> at usbus4 (disconnected)
   ugen4.2: <HUAWEI Technology> at usbus4
   u3g0: <HUAWEI Technology HUAWEI Mobile, class 0/0, rev 2.00/0.00, addr 2> on usbus4
   u3g0: Found 2 ports.
   umass0: <HUAWEI Technology HUAWEI Mobile, class 0/0, rev 2.00/0.00, addr 2> on usbus4
   umass0:  SCSI over Bulk-Only; quirks = 0x0000
   umass0:1:0:-1: Attached to scbus1
   umass1: <HUAWEI Technology HUAWEI Mobile, class 0/0, rev 2.00/0.00, addr 2> on usbus4
   umass1:  SCSI over Bulk-Only; quirks = 0x0000
   (probe0:umass-sim0:0:0:0): TEST UNIT READY. CDB: 0 0 0 0 0 0
   (probe0:umass-sim0:0:0:0): CAM Status: SCSI Status Error
   (probe0:umass-sim0:0:0:0): SCSI Status: Check Condition
   (probe0:umass-sim0:0:0:0): NOT READY asc:3a,0
   (probe0:umass-sim0:0:0:0): Medium not present
   (probe0:umass-sim0:0:0:0): Unretryable error
   cd1 at umass-sim0 bus 0 scbus1 target 0 lun 0
   cd1: <HUAWEI Mass Storage 2.31> Removable CD-ROM SCSI-2 device
   cd1: 40.000MB/s transfers
   cd1: Attempt to query device size failed: NOT READY, Medium not present
   umass1:2:1:-1: Attached to scbus2
   da0 at umass-sim1 bus 1 scbus2 target 0 lun 0
   da0: <HUAWEI MMC Storage 2.31> Removable Direct Access SCSI-2 device
   da0: 40.000MB/s transfers
   da0: 488MB (1000448 512 byte sectors: 64H 32S/T 488C)

Im obigen Beispiel ist dabei noch eine MicroSD-Karte im Kartenleser des
UMTS-Sticks.

In der Regel muss erst eine PIN an die SIM-Karte im Stick gesendet
werden. Dazu kann das folgende Perl-Skript benutzt werden, wobei
``'XXX``' durch die zu sendende PIN ersetzt werden muss. Der Aufruf
erfolgt mit

::

   # umts_stick -a

Achtung: Wird der PIN 3x falsch eingegeben, muss er mittels der PUK
unter Windows wieder freigeschaltet werden! Wenn dem Stick die korrekte
PIN geschickt wurde, nimmt er keine weiteren PINs mehr an und quittiert
weitere Versuche mit ERROR.

::

   #!/usr/bin/perl
   #
   # Copyright (C) 2009 Lars Engels <lars@0x20.net>
   # All rights reserved. Standard 2-clause BSD license and disclaimer applies.

   # umts_stick


   use strict;
   use warnings;

   use Getopt::Std;

   my $VERSION = 0.1;

   my $modem = "/dev/cuaU0.1";     # Change to your device name
   my $pin = "XXXX";               # Insert your device's PIN

   my $rc = 0;
   my %options=();

   $SIG{ALRM} = sub {
           die("Timeout: no response from modem $modem\n");
   };

   sub query_pin_status;
   sub query_provider;
   sub query_signal;
   sub send_pin;
   sub usage;

   usage if ((@ARGV <= 0) || (@ARGV >= 2));

   getopts("ahsp", \%options) or usage;
   usage if defined $options{h};

   open(MODEM, "+<", $modem) or die("can't open modem $modem: $!");
   alarm(10);

   send_pin if defined $options{a};
   if (defined $options{s}) {
           query_provider;
           sleep(1);
           query_signal;
   }
   query_pin_status if defined $options{p};


   close(MODEM) or die "Cannot close modem: $!";
   exit $rc;



   #### SUBS ####
   sub query_pin_status {
           my $query_pin_cmd = "AT+CPIN?\n\r";
           print MODEM  $query_pin_cmd;
           while (<MODEM>) {
                   if (/READY/) {
                           print "Ready: PIN already accepted or no PIN needed.\n";
                           last;
                   } elsif (/SIM\ PIN/) {
                           print "Waiting for PIN.\n";
                           last;
                   } elsif (/SIM\ PUK/) {
                           print "Waiting for PUK.\n";
                           last;
                   } elsif (/^(OK)|$/) {
                           next;
                   } else {
                           print "Unknown PIN status!\n";
                           $rc = 1;
                   }
           }
           $rc = 0;
   }

   sub query_provider {
           my $query_prov_cmd = "AT+COPS?\n\r";
           my @conn_types = ('GSM', 'Compact GSM', 'UMTS', 'EDGE', 'UMTS with HSDPA', 'UMTS with HSUPA', 'UMTS with HSDPA and HSUPA');
           print MODEM $query_prov_cmd;
           while (<MODEM>) {
                   if (/^\+COPS/) {
                           my (undef, undef, $provider, $conn_type) = split /,/;
                           printf "Provider: %s\nConnection type: %s\n", $provider, $conn_types[$conn_type];
                           $rc = 0;
                           last;
                   }
           }
   }

   sub query_signal {
           my $query_signal_cmd = "AT+CSQ\n\r";
           print MODEM $query_signal_cmd;
           while (<MODEM>) {
                   if (/^\+CSQ:\ (\d+),\d+/) {
                           my $rssi = $1;
                           printf "Signal strength: %d dBm\n", ($rssi * 2) - 113;
                           $rc = 0;
                           last;
                   }
           }
   }

   sub send_pin {
           my $send_pin_cmd = "AT+CPIN=\"$pin\"\n\r";
           print MODEM $send_pin_cmd;
           while (<MODEM>) {
                   if (/OK/) {
                           print "PIN accepted\n" ;
                           $rc = 0;
                           last;
                   }
                   if (/ERROR/) {
                           print "PIN rejected\n" ;
                           $rc = 1;
                           last;
                   }
           }
   }

   sub usage {
           print "Usage: $0 [-a] [-h] [-s] [-p]\n";
           print "Options:\n";
           print "    -a     send PIN to device\n";
           print "    -s     print current provider, connection type and signal quality\n";
           print "    -p     get current PIN status\n";
           print "    -h     this help message\n";
           exit -1;
   }

Schritt 2
---------

Ist das Senden des PINs erfolgreich gewesen, kann man sich die manuelle
Arbeit ersparen und devd das Senden automatisch übernehmen lassen. Dazu
legt man die Datei ``'/usr/local/etc/devd/umts.conf``' mit folgendem
Inhalt an, ggf. muss man die ``'product``'-ID auf den eigenen Stick
angepasst werden.

::

   attach 100 {
           device-name "u3g[0-9]+";
           match "vendor" "0x12d1";
           match "product" "0x1003";
           action "/home/lars/bin/umts_stick -a";
   };

Danach mit

::

   # /etc/rc.d/devd restart

den devd neu starten und den Stick erneut einstecken.

Schritt 3
---------

Nun muß die /etc/ppp/ppp.conf wie folgt erweitert werden:

::

   default:
    set log Phase Chat LCP IPCP CCP tun command +connect
   n24:
    set device /dev/cuaU0.0
    set speed 460800
    set phone *99***1\#
    set authname vodafone
    set authkey
    set dial "ABORT BUSY ABORT NO\\sCARRIER TIMEOUT 5 \
            \"\" \
            AT OK-AT-OK \
            AT+CFUN=1 OK-AT-OK \
            AT+CSQ OK \
            AT+CGDCONT=1,\\\"IP\\\",\\\"event.vodafone.de\\\" OK \
            AT+CGACT? OK-AT-OK \
            AT+CGATT? OK \
            AT+COPS? OK \
            ATD*99***1# CONNECT"
    set timeout 180 # 3 minute idle timer (the default)
    set ifaddr 10.0.0.1/0 10.0.0.2/0 255.255.255.0 0.0.0.0
    set vj slotcomp off
    set crtscts on
    add default HISADDR

Dieses Beispiel kann für N24 benutzt werden. Bei anderen Anbietern ist
z.B. event.vodafone.de durch die vom Anbieter vorgeschriebenen Angaben
zu ersetzen.

Fertig
------

Nun wird ppp wie folgt gestartet:

::

   # ppp n24

und danach

::

   # dial

Wenn drei große 'P' also

::

   PPP ON "systemname">

erscheinen, ist der Aufbau der Verbindung erfolgreich gewesen. Soll die
Verbindung automatisch aufgebaut werden, verwendet man den Aufruf

::

   # ppp -ddial n24

Ob die Verbindung erfolgreich zustande gekommen ist, kann man sehen,
wenn die LED am Stick konstant blau leuchtet. Übrigens kann man dann
immer noch keinen Server anpingen oder etwas anderes machen als einen
Browser zu öffnen, der einen auf die N24 bzw. Pro7 Seite umleitet, wo
man sich entscheiden kann, ob man sich ein Zeitkontingent kaufen möchte.
Das obige ``'umts_stick``'-Skript kann auch dazu benutzt werden, die
aktuellen Verbindungsparameter anzuzeigen.

::

   # umts_stick -s

zeigt an, mit welchem Mobilfunkprovider man verbunden ist, mit welcher
Verbindungsart und wie gut das Empfangssignal ist.

Viel Spass

Siehe auch
----------

-  `UMTS <UMTS>`__
-  `UMTS mit Sunrise und Swisscom <UMTS mit Sunrise und Swisscom>`__
   (Diente als Vorlage für diesen Artikel)

* :ref:`genindex`

Zuletzt geändert: |date|

