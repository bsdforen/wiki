Samsung SMT-7020S
=================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

.. note::

  In diesem Artikel möchte ich im Laufe der Zeit Informationen
  zusammenschreiben, die den Aufbau eines FreeBSD-Systems auf einem Samsung SMT
  7020s Satelittenreceiver beinhaltet. Erstmal werde ich diesen Artikel nicht
  im Wiki verlinken, vorerst schreibe ich nur zusammen, um am Ende ein
  vollständiges Howto zu haben, welches man auch anderen Nutzern zur Verfügung
  stellen kann. 

Voraussetzungen
---------------

-  Ein Fernseher sollte vorhanden sein, denn der einfachste Anschluss an
   einen Bildschirm ist wohl der über das mitgelieferte SCART-Kabel an
   ein TV.
-  Um die Box steuern zu können, ist eine USB-Tastatur notwendig. Das
   BIOS erreicht man bei geschwindem drücken auf <F2> während des
   Willkommens-Bildschirms (nur für den Fall der Fälle, aber reinschauen
   will da wohl jeder mal).
-  Eine Festplatte, möglichst 2,5"
-  Ein `IDE-Kabel
   40-Pin <https://de.wikipedia.org/wiki/Bild:IDE_cable_40_pin_%26_80_pin.jpg>`__
-  Ein `IDE-Adapter 3,5" auf 2,5" mit
   Stromanschluß <https://de.wikipedia.org/wiki/Bild:Adapter_IDE_to_Notebook-HDD.jpg>`__
-  Ein Lötkolben
-  Ein `Molex-Stecker <https://de.wikipedia.org/wiki/Molex>`__ mit etwas
   Kabel (z.B. aus einem ausgeschlachtetem Netzteil)

.. note::

  -  Durch die Öffnung und Veränderung des Gerätes verfällt der
     Gewährleistungsanspruch!
  -  Es ist zu beachten, daß im Gehäuse offen 230V Wechselspannung
     anliegen und somit die Gefahr eines Stromschlages - auch bei
     vermeintlich abgeschaltetem Gerät - besteht!
  -  Der ATX-Stromanschluß auf dem Mainboard entspricht nicht der Norm!
     (er ist gespiegelt angebracht). Daher NIEMALS ein Standard
     ATX-Netzteil am Mainboard anschließen!
  -  Alle Angaben erfolgen ohne Gewähr!

Erste Schritte
--------------

Festplatte
~~~~~~~~~~

Aufgund der Platzverhältnisse bietet sich der Einbau von
2,5"-PATA-Festplatten an. 3,5"-Festplatten bekommt man nur mit einigen
Schwierigkeiten ins Gehäuse.

In diesem Fall wird eine 2,5"-Festplatte verbaut. Die Verkabelung mit
dem Mainboard wird mit einem 40-adrigen DMA-Kabel realisiert. Dabei wird
die 2,5"-Platte mittels eines Adapter von ATA40 (3,5") auf ATA44 (2,5")
am Kabel angeschlossen. Dieser Adapter dient neben dem Anschluß der
Datenleitungen auch zur Stromversorgung der Platte. Ein
Standard-Festplattenanschluß (Molex) kann am Netzteil, am besten an der
Unterseite, angelötet werden und mit dem Anschluß des Adapters verbunden
werden.

Installation
~~~~~~~~~~~~

Bei der Installation sollte darauf geachtet werden, dass ein User
angelegt wird, der auch noch Mitglied der Gruppe "wheel" sein sollte,
damit späteres einloggen per ssh sofort problemfrei funktioniert.

FIXME vielleicht auch ändern in gescheite ssh-config von Anfang an. ach
ich weiß auch nicht, sicherheitsgedanken nicht vergessen, du heini! ma
kuckn...

TODO

Probleme beim Booten
~~~~~~~~~~~~~~~~~~~~

Beim Booten gab das schlechte Ergebnisse, die in einem Bootabbruch
gipfelten. Zunächste wurde der MBR verdammt langsam gelesen, dann brach
der Bootvorgang am Ende des Kernelladens ab. Die Meldung beinhaltete
Information der folgenden Art:

::

   read_dma status=51 error=84

Erst ein Umstellen der Festplatteneinstellungen im BIOS der Box auf
"User" mit den Werten "Transfer Mode" "Standard" und "UDMA-Mode"
"Disabled" überredeten die Box zum booten.

Wenn danach noch Probleme mit dem Zugriff auf die Festplatte bestehen
(beispielsweise durch eine Systeminstallation per USB), dann steht man
an einer Meldung mit dem Inhalt:

::

  mountroot

Hier soll man einen anderen Eintrag, als den bei USB-Installation
gesetzten ``ufs:da0s1a`` angeben. Das ist auch klar, denn da die
Festplatte nun in der Box verbaut ist und direkt am IDE-Port hängt, ist
der Zugriff nicht mehr über USB zu bewältigen. Der Zugriff ändert sich
also zu

::

  mountroot> ufs:ad0s1a

Endlich am login-Prompt angelangt, loggen wir uns ein und ändern eben
diese Informationen hinsichtlich der Festplatte in der ``/etc/fstab``
ab. Diese Datei sollte dann etwa wie folgt aussehen (natürlich
variierend je nach der beim Installationsvorgang gewählten
Partitionstabelle):

::

   # Device                Mountpoint      FStype  Options         Dump    Pass
   /dev/ad0s2b             none            swap    sw              0       0
   /dev/ad0s1a             /               ufs     rw              1       1
   /dev/acd0               /cdrom          cd9660  ro,noauto       0       0

Netzwerk
--------

Da beim Anschluss an einen Fernseher über das Scart-Kabel an der
Rückseite der Box das Bild leicht nach links verschoben ist, sieht man
die ersten 5 Zeichen nicht. So kann es Probleme geben, das
Netzwerkinterface zu konfigurieren, da man dessen Namen ja nicht kennt.
So sei hier gesagt, dass sich ein Intel DA82562ET-Netzwerkcontroller auf
dem Board befindet, der auf das Kürzel **fxp0** reagiert. Ist ein
DHCP-Server angeschlossen, so bekommt man mit

::

  # dhclient fxp0

einen Netzwerklink. Um das ganze Spiel schon beim Booten automatisiert
zur Verfügung zu haben, sollte man sich die Datei ``/etc/rc.conf`` zur
Brust nehmen. Hier gehört der Eintrag

::

   ifconfig_fxp0="DHCP"

rein. Wer lieber eine statische Konfiguration der Box wünscht, der trägt
anstelle dessen etwas dieser Art ein:

::

   ifconfig_fxp0="inet 192.168.1.2 netmask 255.255.255.0"
   defaultrouter="192.168.1.1"

sshd(8)
~~~~~~~

Von nun an kann man sich NICHT per **ssh(2)** verbinden. Dafür muss erst
der defaultmäßig deaktivierte **sshd(8)** gestartet werden.

Das geschieht fürs nächste Booten mit folgendem Eintrag in der
``/etc/rc.conf``

::

   sshd_enable="YES"

Für das aktuell gestartete System eignet sich der Befehl

::

  # /etc/rc.d/sshd start

Jetzt ist kein root-login möglich, das ist per default verboten. Wer
aber clever war, hat beim Installieren des Systems direkt einen User
angelegt, mit dem man sich nun einloggen kann. Wurde dieser auch noch
als Member der Gruppe "wheel" vermerkt, kann man nach einem Login als
dieser User mit einem

::

  # su

root-Rechte verleihen, die in diesem Anfangsstadium der Konfiguration
sicher wichtig sind...

Infrarot
--------

Der nächste Schritt muss MythTV und vor allem Infrarot heißen. Zunächst
wird mal der ``lirc`` installiert, weil es außer diesem nur noch einen
``uird`` gibt, der aber wohl nur mit einem ganz bestimmten
Infrarot-Gerät zusammenarbeitet. Also los:

::

  # pkg_add -r lirc

Eine wichtige Info zu diesem Zeitpunkt ist wohl diese hier:

::

     **********************************************************************
     This port does not contain any FreeBSD device drivers for infrared
     devices. This port installs the LIRC daemons and tools for interacting
     with drivers that support the LIRC device interface.

     You will need to obtain third party device drivers or port the Linux
     drivers in ${WRKSRC}/drivers to use the LIRC port.
     **********************************************************************

MythTV
------

Ein ``pkg_add`` geht bei **MythTV** zum Zeitpunkt der Erstellung dieses
Artikel nicht, weil der Port als "broken" markiert ist und deshalb nicht
als Package zu Verfügung gestellt wird. Dummerweise funktionierte der
Port zum genannten Zeitpunkt nicht (nicht nur, weil er "broken" war,
sondern auch weil die Quellen nicht verfügbar waren), weshalb nun alles
aus den Quellen kompiliert wird.

Damit aber nicht alle Abhängigkeiten auch noch als Port installiert
werden, nehmen wir diese Dinge selbst in die Hand und installieren
möglichst viel per ``pkg_add``:

::

  # pkg_add -r qt

``(NICHT qt4!! laut mythtv.org läuft mythtv nur mit qt3)``

Leider gibt es Lizenzprobleme hinsichtlich ``'lame``', weshalb kein
Package zur Verfügung gestellt wird. Das ist uns natürlich egal, wir
installieren einfach den Port... Dafür wird zuerstmal der Portstree mit
Hilfe von ``portsnap`` geholt. Wer FreeBSD vor Version 6.0-RELEASE
benutzt, ist selbst schuld. Der muss ``portsnap`` nämlich erstmal
installieren. Das würde dann so gehen:

::

  # pkg_add -r portsnap

Das eigentliche Installieren des Portstrees geschieht dann mit folgendem
Befehl:

::

  # portsnap fetch extract

Wer wirklich ausführliche Informationen zu diesem Thema sucht, der ist
wohl bei den `paketsysteme <paketsysteme>`__\ n und artverwandten
Artikeln richtig aufgehoben!

Jetzt ist endlich ``'lame``' dran.

::

  # cd /usr/ports/audio/lame 
  # make install clean

Und die übrigen Abhängigkeiten:

::

  # pkg_add -r wget 
  # pkg_add -r qt-mysql-plugin 
  # pkg_add -r p5-xmltv 
  # pkg_add -r freetype2 
  # pkg_add -r mysql50-client

mySQL
-----

Wir benötigen auf diesem Gerät auch noch einen mySQL-Server, so denn
kein andere im Netzwerk verfügbar ist. Der einfachheit halber gehe ich
aber davon, dass sowas nicht da ist und wir betreiben sowohl den Server,
als auch den Client direkt auf der Samsung-Box:

::

  # pkg_add -r mysql50-server

mySQL muss noch kurz konfiguriert werden:

::

  # echo 'mysql_enable="YES"' >> /etc/rc.conf

Zum einmaligen Starten von mySQL ist das hier notwendig:

::

  # /usr/local/etc/rc.d/mysql-server start

Dann muss noch das initiale Passwort für den Nutzer "root" gesetzt
werden:

::

  # mysql --user=root

  Welcome to the MySQL monitor. Commands end with ; or \\g. Your MySQL
  connection id is 12 to server version: 5.0.27

  Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

  mysql> update mysql.user set
  Password=PASSWORD('**hier_das_neue_passwort**') where User='root';
  mysql> flush privileges; mysql> quit;

Jetzt ist das Passwort für ``root`` gesetzt und man kann die
MythTV-Datenbank einpflegen. Hierbei ist darauf aufzupassen, dass der
benutzte Pfad auch dem Pfad zur Datei ``mc.sql`` entspricht. Auf meinem
System hab ich die Datei nur ein einziges Mal gefunden:

::

  # mysql -uroot -p < /root/mythtv-0.20.2/database/mc.sql

Sound
-----

Wer Interesse hat, kann zunächst mal testen, welche Karte überhaupt in
der Box steckt, das geht am einfachsten mit dem bei FreeBSD
mitgelieferten Meta-Treiber ``snd_driver``. Danach guckt man sich den
Inhalt von ``/dev/sndstat`` an und weiß, welche Karte eingebaut ist:

::

  videoonkel# kldload snd_driver
  videoonkel# cat /dev/sndstat
  FreeBSD Audio Driver (newpcm)
  Installed devices:
  pcm0: <Intel ICH2 (82801BA)> at io 0x1c00, 0x1880 irq 5 bufsz 16384 kld snd_ich (1p/1r/0v channels duplex default)

Wir haben also eine Intel ICH2 an Board. Damit die immer beim Booten
verfügbar ist, fügen wir den Aufruf des Treibers in das System ein:

::

  # echo snd_ich_load="NO" # Intel ICH >> /boot/loader.conf

Um sofort die Soundkarte zu testen, machen wir einfach das hier:

::

  # kldload snd_ich
  # cd
  # cat mythtv-0.20.2.tar.bz2 > /dev/dsp

Jetzt sollte ein eifriges Rauschen aus dem Fernseher zu hören sein. Die
Soundkarte funktioniert.

Windowmanager
-------------

Da stolper ich doch gerade noch darüber, dass man einen Windowmanager
braucht. Es wird wohl ein "Leichtgewicht" genügen, also machen wir mal
schnell das hier:

::

  # pkg_add -r ratpoison

Jetzt kommt also MythTV an die Reihe:

Um also nun MythTV SELBST ZU KOMPILIEREN (ARGH!, es ist ne 700MHz CPU,
dankeschön, FreeBSD!), benötigen wir erstmal das Quellpaket von MythTV.
Dasfür benötigen wir wget.

Das wird sich fein von der
`MythTV-Seite <http://www.mythtv.org/modules.php?name=Downloads&d_op=viewdownload&cid=1>`__
runtergeladen und dann per scp auf die Box kopiert.

Ich muss es selbst bauen, aber es fehlen ja noch Dependencies fürs
Bauen:

::

  # pkg_add -r qmake 
  # pkg_add -r p5-XML-SAX-Expat

Entpacken des Pakets:

::

  # tar -xvjf mythtv-0.20.tar.bz2

So, jetzt kann also endlich das Kompilieren von MythTV beginnen. Also
hinein in das Verzeichnis, in das das mythtv-Paket entpackt wurde und ab
geht die Luzie, allerdings ist zuvor noch eine kleine Variable zu
setzen, und zwar die, in der die spezifikationen für qmake drin stehen.
Soweit ich das mit meinem kleinen Hirn nachvollziehen kann, stehen in
dieser Datei viele Informationen drin, wo qmake plattformabhängig
verschiedene Dine findet, Da wir hier ein FreeBSD vor der Nase haben,
müssen wir in dem "mkspecs"-Verzeichnis ein FreeBSD-Unterverzeichnis
auswählen. Es gibt derer drei, eines ist freebsd-g++, welchen ein
kompilieren mit dem GNU-C Compiler durchführt, dann gibt es noch
freebsd-g++34 für version 3.4 selbigen Compilers und es gibt noch
freebsd-icc für den Intel Compiler. Wir wollen ersteren. Also los:

::

  # setenv QMAKESPEC /usr/local/share/qt/mkspecs/freebsd-g++ 
  # setenv QTDIR /usr/X11R6/

Komischerweise wird trotz der abgeschlossenen qt-Installation die
entsprechende Bibliothek nicht verlinkt. Wieso das so ist, kann ich
nicht sagen. Es kotzt mich mal wieder nur an. Eine Stunde mit Suchen
verbracht. Naja, dafür ist die Musik aus den Boxen gut. Also weiter,
legen wir den link von Hand an (und neben diesem noch eine Reihe
andere...):

::

  # /sbin/ldconfig
  # ln -s /usr/X11R6/lib/libqt-mt.so /usr/lib/libqt-mt.so
  # ln -s /usr/X11R6/lib/libXext.so /usr/lib/libXext.so
  # ln -s /usr/X11R6/lib/libX11.so /usr/lib/libX11.so

Dann muss malloc.c in der Datei
/root/mythtv-0.20.2/libs/libmythtv/videoout_xv.cpp geändert werden in
<stdlib.h>

Beim Kompilieren will Onkel Dittmeier dann auch noch nach ``/src/moc``
suchen, dabei ist ``moc`` lange kompilierterweise installiert
(``pkg_add -r qt`` hatten wir ja durchgeführt). Also auskommentiert den
Dreck:

::

  # vi $mythtv-Entpackverzeichnis/libs/libmyth/Makefile

Die Zeile mit ``/src/moc`` und dem anschließenden ``make`` gehört
auskommentiert (mit einer # am Zeilenanfang!). Möglicherweise muss auch
noch ein symbolischer Link namens /bin/moc auf /usr/X11R6/bin/moc
erstellt werden.

Dann gibt es bestimmt noch Meckereien wegen LONG_LONG_MAX. In der
entsprechenden Datei umändern in LLONG_MAX. Puh!

Jetzt endlich können wir ein make im Entpackverzeichnis von mythtv
anschieben. Meine Fräse, ey!

::

  # make
  # make install
  # make clean

Xorg
----

Zunächst mal war man sicher clever genug, in der Startinstallation des
Betreibssystems, Xorg in die Installation einzuschließen. Nun wollen wir
diesen X-Server aber endlich auch mal starten, dafür brauchen wir aber
eine default-Konfiguration, mit der wir dann mal starten können.

::

  # Xorg -configure

Das erstellt uns eine xorg.conf.new in /root, die flugs ausprobiert wird mit

::

  # Xorg -config /root/xorg.conf.new

Weil das so gut funktioniert, nehmen wir die Datei gleich als default:

::

  # mv /root/xorg.conf.new /etc/X11/xorg.conf

Naja, jetzt wollen wir auch noch den Mode des X-Servers auf den Fernseher
einstellen. Dafür muss die Sektion Monitor in der xorg.conf durch diese hier
ersetzt werden:

::

   Section "Monitor"
       Identifier   "Glotze"
       ModeLine "768x576" 50.00 768 832 846 1000 576 590 595 630
   EndSection

Die gesamte Datei sieht bei mir so aus:

::

   Section "ServerLayout"
       Identifier     "X.org Configured"
       Screen      0  "Screen0" 0 0
       InputDevice    "Mouse0" "CorePointer"
       InputDevice    "Keyboard0" "CoreKeyboard"
   EndSection

   Section "Files"
       RgbPath      "/usr/X11R6/lib/X11/rgb"
       ModulePath   "/usr/X11R6/lib/modules"
       FontPath     "/usr/X11R6/lib/X11/fonts/misc/"
       FontPath     "/usr/X11R6/lib/X11/fonts/TTF/"
       FontPath     "/usr/X11R6/lib/X11/fonts/Type1/"
       FontPath     "/usr/X11R6/lib/X11/fonts/CID/"
       FontPath     "/usr/X11R6/lib/X11/fonts/75dpi/"
       FontPath     "/usr/X11R6/lib/X11/fonts/100dpi/"
   EndSection

   Section "Module"
       Load  "dbe"
       Load  "dri"
       Load  "extmod"
       Load  "glx"
       Load  "record"
       Load  "xtrap"
   #   Load  "freetype"
       Load  "type1"
   EndSection

   Section "InputDevice"
       Identifier  "Keyboard0"
       Driver      "kbd"
   EndSection

   Section "InputDevice"
       Identifier  "Mouse0"
       Driver      "mouse"
       Option      "Protocol" "auto"
       Option      "Device" "/dev/sysmouse"
       Option      "ZAxisMapping" "4 5 6 7"
   EndSection

   Section "Monitor"
           Identifier   "Glotze"
       VertRefresh 48-61
       HorizSync   27-96
     #     ModeLine "768x576" 50.00 768 832 846 1000 576 590 595 630
       ModeLine "720x576" 27.5 720 744 800 880  576 581 583 625
       DisplaySize 300 225
   EndSection

   Section "Device"
           ### Available Driver options are:-
           ### Values: <i>: integer, <f>: float, <bool>: "True"/"False",
           ### <string>: "String", <freq>: "<f> Hz/kHz/MHz"
           ### [arg]: arg optional
           #Option     "NoAccel"               # [<bool>]
           #Option     "SWcursor"              # [<bool>]
           #Option     "ColorKey"              # <i>
           #Option     "CacheLines"            # <i>
           #Option     "Dac6Bit"               # [<bool>]
           #Option     "DRI"                   # [<bool>]
           #Option     "NoDDC"                 # [<bool>]
           #Option     "ShowCache"             # [<bool>]
           #Option     "XvMCSurfaces"          # <i>
           #Option     "PageFlip"              # [<bool>]
       Identifier  "Card0"
       Driver      "i810"
       VendorName  "Intel Corporation"
       BoardName   "82815 CGC [Chipset Graphics Controller]"
   #   BusID       "PCI:0:2:0"
       BusID       "0000:02:00"
       Screen      0
       VideoRam    4096
       Option      "DCC"   "off"
       Option      "ColorKey"  "0"
       Option      "BlankTime"     "0"
   EndSection

   Section "Screen"
           Identifier "Screen0"
           Device     "Card0"
           Monitor    "Glotze"
       DefaultDepth    24
       SubSection  "Display"
           Modes   "640x480"
           Viewport    0 0
           Depth   24
       EndSubSection
   #        SubSection "Display"
   #                Depth 24
   #       Modes "768x576"
   #        EndSubSection
   EndSection

   Section "DRI"
       Mode 0666
   EndSection

Schwups - schon läuft der Onkel. Jetzt fehlt ja noch die Einbindung von
unserem "ratpoison":

::

  # echo /usr/X11R6/bin/ratpoison > ~/.xinitrc

Weblinks
--------

-  http://www.vdr-wiki.de/wiki/index.php/Samsung_SMT-7020S

* :ref:`genindex`

Zuletzt geändert: |date|

