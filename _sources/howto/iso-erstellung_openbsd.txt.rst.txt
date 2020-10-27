ISO-Erstellung (OpenBSD)
========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

Hier werden Möglichkeiten beschrieben, eine OpenBSD ISO zu erstellen bzw.
allgemein sich eine ISO herunterzuladen.

Script
------

Hier ein kleines Perlscript, das automatisch die benötigten Quellen fuer
OpenBSD herunterlädt und daraus eine ISO baut.

Benötigte Programme
~~~~~~~~~~~~~~~~~~~

Folgenge Pakete werden benötigt:

-  `net/wget <https://www.google.com/search?q=net/wget&btnI=lucky>`__
-  `sysutils/cdrtools <https://www.google.com/search?q=sysutils/cdrtools&btnI=lucky>`__
   (enthält ``mkisofs``)

Für die Erstellung der ISO werden ca. 350 MB benötigt.

Vorzunehmende Einstellungen
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ihr solltet ``$mirror`` in den gewünschten
`Mirror <http://openbsd.org/ftp.html>`__ umändern.

``$release`` ist das Releaseverzeichnis auf dem Mirror.

``$workdir`` sollte passende Rechte haben.

``$arch`` ist die Prozessorarchitektur, diese sollte ggf. auch angepasst
werden.

Das Script
~~~~~~~~~~

.. code:: perl

   #!/usr/pkg/bin/perl
   use strict;
    
   # Mirror hier eintragen. Kein Slash am Ende.
   my $mirror = "ftp://ftp.calyx.nl/pub/OpenBSD";
     
   # Die Releasenummer
   my $release = "4.3";
   my $ver = "43";
   # Die Architektur
   my $arch = "i386";
      
   # Das Arbeitsverzeichnis. Kein Slash am Ende.
   my $workdir = "/home/sierrax/openbsd";
       
   if(-d $workdir && -W $workdir){
     chdir $workdir || die "Falsche Rechte fuer das Arbeitsverzeichnis.\n";
     } else {
     mkdir $workdir || die "Schreibrechte fuer das Arbeitsverzeichnis fehlen.\n";
     chdir $workdir || die "Falsche Rechte fuer das Arbeitsverzeichnis.\n";
     }
   system "wget -c -N -P $workdir/$release/doc $mirror/doc/obsd-faq-de.pdf";
   system "wget -c -N -P $workdir/$release/doc $mirror/doc/obsd-faq.txt";
   system "wget -c -N -P $workdir/$release/doc $mirror/doc/pf-faq.txt";
   system "wget -c -N -P $workdir/$release/doc $mirror/doc/pf-faq.pdf";
                            
   my $ver = $release;
   $ver =~ s/\.//;
   system "wget -c -N -P $workdir/$release $mirror/songs/song$ver.mp3";
                             
   system "wget -c -N -r -nd -P $workdir/$release/$arch $mirror/$release/$arch";
                              
   system "mkisofs -quiet -l -J -r -o ../openbsd-$release.iso -c boot.catalog -b $release/$arch/floppy$ver.fs -V \"OpenBSD-$release\" $workdir";
   print "\nopenbsd-$release.iso wurde erstellt.\n\n";

ISO via BitTorrent
------------------

Alternativ können die ISOs auch mittels
`BitTorrent </anwendungen/BitTorrent>`__
`hier <http://openbsd.somedomain.net/>`__ heruntergeladen werden. Dazu
wird ein BitTorrent Client benötigt.

Anmerkung
---------

Man sollte, wenn die Möglichkeit besteht, die CDs von OpenBSD kaufen
oder etwas Geld spenden, da sich das Projekt hauptsächlich über den
Verkauf von CDs finanziert.

Weblinks
--------

Original erstellt von
`"sebbo" <http://www.bsdforen.de/showthread.php?p=31501>`__

OpenBSD ISO Torrents: http://openbsd.somedomain.net/

* :ref:`genindex`

Zuletzt geändert: |date|

