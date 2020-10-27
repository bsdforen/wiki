UMTS mit Sunrise und Swisscom
=============================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png

Sunrise und Swisscom sind in der Schweiz Providers welche günstig einen UMTS
Service anbietet. Die PCMCIA Karte ist eine Sierra Wireless AC850. Unter
OpenBSD wird die Karte unterstützt unter FreeBSD muss man den Kernel patchen
(zumindest noch in der Version 6.2).  Dieser Artikel beschreibt, wie man diese
Karte zum Laufen bekommt.  ``**Ein NetBSD patch ist in Arbeit (vom CCCZH und
Chaostreff Bern).**`` Bei fragen steht Pascal Vizeli gerne zur verfügung:
pvizeli AT yahoo DOT de

FreeBSD Kernel Patch
--------------------

**2007-10-12**: Der Patch wurde in current aufgenommen und wird in der
Version 6.3 vorhanden sein.

Den Patch kannst du `im
Forum <http://www.bsdforen.de/showpost.php?p=154567&postcount=9>`__
downloaden. Den Patch einspielen und den Kernel neue kompilieren.
Darüber kannst du hier mehr lesen:

-  `Kernel kompilieren unter FreeBSD </freebsd/kernel_erstellen>`__

Schritt 1
---------

Stecke die Karte in den Slot. Du musst nun den PIN der Karte mitteilen.
Dazu kannst du diesen Perl Script nehmen. ``'XXXX``' durch deinen PIN
ersetzen.

::

   #!/usr/bin/perl

   use strict;
   use warnings;

   my $modem = "/dev/cuad4";
   my $pin = "XXXX";

   $SIG{ALRM} = sub {
     die("timeout: no response from modem $modem\n");
   };

   open(MODEM, "+<", $modem) or die("can't open modem $modem");
   alarm(10);

   print(MODEM "AT+CPIN=\"$pin\"\n\r");
   while (<MODEM>) {
     
     if (m/OK/) {
       close(MODEM);
       print("PIN accepted\n");
       exit(0);
     }

     if (m/ERROR/) {
       close(MODEM);
       print("PIN rejected\n");
       exit(1);
     }
   }

Signal abfrage
~~~~~~~~~~~~~~

Mit diesem Perlscript kannst du die Signalstärke abfragen. Aber nur
solange das ppp noch nicht aktive ist.

::

   #!/usr/bin/perl

   use strict;

   my $modem = "/dev/ttyd4";
   my $antwort = "";

   open(MODEM, "+<", $modem) or die("can't open modem $modem");

   print(MODEM "AT+CSQ\n\r");
   while(<MODEM>){

     if(/OK/m) { exit(1); }
     if(/CSQ:/s) { print $_; }
   }

Schritt 2
---------

Nun musst du das ppp konfigurieren. Mit pppd hab ich es nie zum laufen
bekommen, das funktioniert wohl nur unter Linux.

**/etc/ppp/ppp.conf**

::

   default:
    set log Phase Chat LCP IPCP CCP tun command
    ident user-ppp VERSION (built COMPILATIONDATE)

    set device /dev/cuad4
    set speed 57600
    set dial "ABORT BUSY ABORT NO\\sCARRIER TIMEOUT 5 \
              \"\" AT OK \\dATD\\T TIMEOUT 40 CONNECT"
    set timeout 180
    enable dns

   ac850:
    set log local chat error warning connect
    set ifaddr 10.0.0.1/0 10.0.0.2/0 255.255.255.255 0.0.0.0
    set crtscts on
    set vj slotcomp off
    set phone "*99#"
    add default HISADDR

Fertig
------

Jetzt musst du dich nur noch einwählen wenn du Surfen willst.

::

   # ppp ac850

und danach

::

   # dial

Viel Spass

Siehe auch
----------

-  `UMTS <UMTS>`__
-  `UMTS mit Huawei </howto/UMTS mit Huawei>`__

* :ref:`genindex`

Zuletzt geändert: |date|

