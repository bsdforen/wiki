KDM
===

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

.. warning::

  Die Inhalte sind arg veraltet!

Vielen Personen bereitet die Installation und Konfiguration des grafischen
Login-Manager KDM Mühe. Diese Anleitung soll diesem Übel entgegen treten. Im
Artikel `KDM </anwendungen/KDM>`__ ist das Programm beschrieben.

Bitte kontrollieren Sie auch die
`DNS-Namensauflösung <Namensauflösung>`__ Ihres Systems.

X11-Server
----------

Bevor der KDM genutzt werden kann, muss der X11-Server sauber laufen. Es
wird davon ausgegangen, dass der Xorg-7 Zweig verwendet wird.

Installation
~~~~~~~~~~~~

Zur Installation von Xorg wird am besten der Metaport
`x11/xorg <https://www.google.com/search?q=x11/xorg&btnI=lucky>`__
verwendet. Lesen Sie bitte den Artikel `Paketsysteme <Paketsysteme>`__
um zu erfahren, wie das Paket installiert wird.

3D-Grafik-Beschleunigung
~~~~~~~~~~~~~~~~~~~~~~~~

Eine Anleitung zur Einrichtung der 3D-Grafik-Beschleunigung finden Sie
unter `3D-Grafik-Beschleunigung <3D-Grafik-Beschleunigung>`__.

Konfiguration
~~~~~~~~~~~~~

Anders als bei Linux-Distributionen muss bei FreeBSD die
X11-Server-Konfiguration (``/usr/local/etc/X11/xorg.conf`` oder
``/etc/X11/xorg.conf``) selber gemacht werden! Eine sehr gute Anleitung
finden Sie im Wiki unter
`X11-Serverkonfiguration <X11-Serverkonfiguration>`__.

NVidia-Binärtreiber
~~~~~~~~~~~~~~~~~~~

Der NVidia-Binärformat-Grafikkartentreiber steht nur als Port
`x11/nvidia-driver <https://www.google.com/search?q=x11/nvidia-driver&btnI=lucky>`__
zur Verfügung. Für mehr Informationen zum
NVidia-Binärformat-Grafikkartentreiber siehe bitte:
http://www.bsdforen.de/showthread.php?t=8124&highlight=nvidia

Test
~~~~

Testen Sie mit folgenden Befehlen die korrekte Konfiguration des
X11-Servers, es darf bei der Befehlseingabe kein X11-Server laufen!

::

   $ su
   # X

Den X11-Server kann man mit <Ctrl-Alt-Backspace> beenden, die
Backspace-Taste ist die "Pfeil nach Links"-Taste oberhalb der
Return-Taste. Wenn die Konfiguration einigermassen in Ordnung ist,
sollten Sie nun auf dem Monitor ein gerastertes Hintergrundbild sehen.
Das ist sozusagen das "Testbild". Stellen Sie Ihren Monitor so ein, daß
er das Bild vollständig darstellt. Bei einem Flachbildschirm darf die
Darstellung nicht flimmern und muss über die ganze Bildschirmfläche
gleichmässig aussehen, wenn nicht, müssen Sie Phase und Pitch anpassen.

Vorzugsweise wählen Sie bei einem TFT Bildschirm die automatische
Justierung aus, dies erzeugt meist das beste Ergebnis. Nähere
Informationen zur korrekten Einstellung Ihres Monitors entnehmen Sie
bitte dem Monitor-Handbuch.

Kontrolle
~~~~~~~~~

Kontrollieren Sie mit:

::

   # more /var/log/Xorg.0.log |grep EE

ob der X11-Server eine Fehlermeldung ausgab. Wenn keine Fehlermeldung
vorhanden ist, sieht es so aus:

::

   (WW) warning, (EE) error, (NI) not implemented, (??) unknown.
   (II) Loading extension MIT-SCREEN-SAVER

startx
~~~~~~

Um den X11-Server von der Konsole aus starten zu können, benötigt man
startx. Bevor Sie startx zum ersten Mal starten, geben Sie in der Datei
``~/.xinitrc`` an, welchen Window-Manager Sie starten möchten. Hier als
Beispiel KDE:

::

   exec startkde

Los gehts, es darf kein X11-Server bei der Befehlseingabe laufen!

::

   $ su
   # startx

Kontrollieren Sie die Ausgaben in den Log-Dateien:

-  ``/var/log/Xorg.0.log``
-  ``/root/.xsession-errors``

KDM
---

Wenn startx ohne Probleme funktioniert, können Sie sich an die
Installation und Konfiguration des grafischen Login-Manager KDM
heranwagen.

Installation
~~~~~~~~~~~~

Der KDM ist Teil von KDE, genauer von x11/kdebase3 und wird mit: #
pkg_add -r kdebase installiert.

``'Beachten Sie bitte die Angaben in der Datei /usr/ports/UPDATING!``'

Konfiguration
~~~~~~~~~~~~~

Existieren im Verzeichnis ``/usr/local/share/config/kdm`` die Dateien:
``Xaccess``, ``Xreset``, ``Xservers``, ``Xsession``, ``Xsetup``,
``Xstartup``, ``Xwilling`` und ``kdmrc``. Wenn eine fehlt, müssen die
fehlenden Dateien mit:

::

   # genkdmconf

erstellt werden!

Test
~~~~

Testen Sie nun KDM, es darf kein X11-Server bei der Befehlseingabe
laufen!

::

   $ su
   # kdm

Kontrollieren Sie die Ausgaben in den Log-Dateien:

-  ``/var/log/kdm.log``
-  ``/var/log/Xorg.0.log``
-  ``/root/.xsession-errors``

Startskript (Ohne rc.conf)
~~~~~~~~~~~~~~~~~~~~~~~~~~

Falls der KDM einwandfrei beim Test funktionierte, können Sie ein
Startskript unter ``/usr/local/etc/rc.d/kdm.sh`` für den automatischen
Start des KDM beim Computerstart erstellen:

::

   #!/bin/sh
   PREFIX=/usr/local

   case "$1" in
    start)
      mkdir -p /tmp/.ICE-unix && chmod 1777 /tmp/.ICE-unix && chown root:wheel /tmp/.ICE-unix
      ${PREFIX}/bin/kdm
      ;;

    stop)
      /usr/bin/killall -m kdm 2>/dev/null
      ;;

    *)
      echo "Usage: `basename $0` start | stop"
      exit 64
      ;;
   esac
   exit 0

Mit:

::

   # /usr/local/etc/rc.d/kdm.sh stop
   # /usr/local/etc/rc.d/kdm.sh start

können Sie die gesamte graphische Oberfläche beenden und neustarten.

Startskript (Mit rc.conf)
~~~~~~~~~~~~~~~~~~~~~~~~~

Dies Startskript respektiert die Einstellungen in der ``rc.conf``. Damit
kann man den KDM recht einfach aktivieren oder wieder deaktivieren.

Falls der KDM einwandfrei beim Test funktionierte, können Sie ein
Startskript unter ``/usr/local/etc/rc.d/kdm`` für den automatischen
Start des KDM beim Computerstart erstellen:

::

   #!/bin/sh
   #

   # PROVIDE: kdm
   # REQUIRE: DAEMON LOGIN moused

   # The following variables are provided to control startup of KDM in
   # rc configuration file (eg /etc/rc.conf):
   # kdm_enable (bool):    Set to "NO" by default.
   #                       Set it to "YES" to enable KDM
   #
   # kdm_wait:             Waiting time to start KDM.
   #                       Default is 5 seconds.
   #                       If KDM starts too fast, keyboard is disabled.
   #
   # Please see kdm(1), rc.conf(5) and rc(8) for further details.

   . /etc/rc.subr

   name="kdm"
   rcvar=`set_rcvar`
   pidfile="/var/run/${name}.pid"
   stop_cmd="kdm_stop"
   start_cmd="kdm_start"

   PREFIX=/usr/local

   # Set defaults
   [ -z "$kdm_enable" ] && kdm_enable="NO"
   [ -z "$kdm_wait" ] && kdm_wait="5"

   kdm_start()
   {
           echo "Starting kdm."
           (/bin/sleep ${kdm_wait}; ${PREFIX}/bin/kdm 2>/dev/null) &
   }

   kdm_stop()
   {
           echo "Stopping kdm."
           /usr/bin/killall -m kdm-bin 2>/dev/null
   }

   load_rc_config $name

   run_rc_command "$1"

Wenn folgendes in der ``/etc/rc.conf`` steht, startet KDM bereits bei
booten, bzw. ist generell startbar:

::

   kdm_enable="YES"

Mit:

::

   # /usr/local/etc/rc.d/kdm stop
   # /usr/local/etc/rc.d/kdm start

können Sie die gesamte graphische Oberfläche beenden und neustarten.

Starten als Terminal
~~~~~~~~~~~~~~~~~~~~

Alternativ kann kdm auch einfach als Terminal gestartet werden. Dazu
ändert man die Zeile **ttyv8** in der Datei ``/etc/ttys``
folgendermaßen:

::

   ttyv8 "/usr/local/bin/kdm -nodaemon"  xterm   on  secure

Fehlerbehebung
--------------

Xorg-6.8.1
~~~~~~~~~~

Falls Sie den Xorg-Server 6.8.1 einsetzen und KDE nicht mehr starten
will, lesen Sie bitte folgenden Eintrag in der Datei
/usr/ports/UPDATING:

::

   20041229:
     AFFECTS: users of x11/kdebase3, x11-servers/xorg-server
     AUTHOR: lofi@freebsd.org

     If KDE does not start anymore after upgrading Xorg to version 6.8.1
     (X restarts when the KDE splash screen has reached the third icon),
     please check whether the directory /tmp/.ICE-unix exists, is owned by root
     and has permissions 1777 (read/write/access for everybody + sticky bit).

     To make sure everything is in working order, do (as root):
     mkdir -p /tmp/.ICE-unix && chmod 1777 /tmp/.ICE-unix &&
     chown root:wheel /tmp/.ICE-unix

     Also, make sure you do NOT have clear_tmp_enable="YES" set in /etc/rc.conf,
     as it will remove the directory on every reboot and applications will re-
     create it with the wrong ownership.

     Users of daily_clean_tmps_enable in /etc/periodic.conf should make sure
     daily_clean_tmps_ignore contains /tmp/.ICE-unix.

Tipps
-----

FontConfig
~~~~~~~~~~

Oft beschleunigt ein:

::

   # fc-cache -f -v

den Start von KDE. Der obengenannte Befehl aktualisiert die Cachedateien
von FontConfig, was bis zu 6 Sekunden Zeitgewinn beim KDM-Start bringen
kann! Mehr Informationen zu FontConfig finden Sie unter:
`Bessere Schriften#FontConfig <Bessere Schriften#FontConfig>`__

* :ref:`genindex`

Zuletzt geändert: |date|

