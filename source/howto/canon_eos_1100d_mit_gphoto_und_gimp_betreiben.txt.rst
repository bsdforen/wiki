Verwendung einer Canon EOS 1100D mit Gimp im Tethering Mode
===========================================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieses HowTo erklärt wie man Tethering mit einer Canon EOS 1100D und
Gimp einrichtet. Tethering bedeutet in diesem Zusammenhang dass die
Kamera mittels USB an den Computer angeschlossen ist und der Fotograf
das Bild direkt nach dem Auslösen auf dem Bildschirm sieht. Somit kann
ein Bild natürlich besser beurteilt werden als wenn man es nur auf einem
kleinen Kamerabildschirm betrachtet.

Installation der benötigten Ports
---------------------------------

Folgende Ports benötigt ihr:

-  `graphics/gimp <https://www.google.com/search?q=graphics/gimp&btnI=lucky>`__
-  `graphics/gphoto2 <https://www.google.com/search?q=graphics/gphoto2&btnI=lucky>`__

Optional als Adobe Lightroom Ersatz:

-  `graphics/digikam <https://www.google.com/search?q=graphics/digikam&btnI=lucky>`__

Konfigurieren der USB Kamera im devfs
-------------------------------------

Damit man nicht jedesmal root sein muss um die Kamera zu verwenden muss
folgendes in seine devfs.rules eintragen:

::

   [localrules=10]
   add path 'ugen*' mode 0770 group wheel
   add path 'usb/*' mode 0770 group wheel

Damit diese Rollen beim Start geladen werden müssen die localrules noch
in die rc.conf eingetragen werden.

::

   devfs_system_ruleset="localrules"

Ein erster Test mit gphoto2:

::

  carbuncle@carbuncle-quad:/etc % gphoto2 --auto-detect 
  Modell Port
  --------------
  Canon EOS 1100D usb:/dev/usb,/dev/ugen2.2

Die Kamera ist also unter /dev/ugen2.2 eingebunden worden. FreeBSD
besitzt für die Kameras keinen Kerneltreiber, deshalb wird sie einfach
als generisches USB Gerät eingehängt. gPhoto2 kann die Kamera dann
verwenden.

Einrichten des Hook Skripts für gPhoto
--------------------------------------

Damit die Bilder die man mit der Kamera fotografiert direkt in Gimp
geladen werden können, muss man gphoto2 ein sogenanntes Hook Skript
übergeben. Dieses Hook Skript deckt 4 verschiedene Zustände ab (Init,
Start, Download, Stop). Uns interessiert eigentlich nur der Download
Zustand.

::

   #! /bin/sh

   self=`basename $0`

   case "$ACTION" in
       init)
           echo "$self: INIT"
           #exit 1 # non-null exit to make gphoto2 call fail
           ;;
       start)
           echo "$self: START"
           ;;
       download)
           echo "$self: DOWNLOAD to $ARGUMENT"
           /usr/local/bin/gimp $ARGUMENT &
           ;;
       stop)
           echo "$self: STOP"
           #/usr/local/bin/gimp $ARGUMENT
           ;;
       *)
           echo "$self: Unknown action: $ACTION"
           ;;
   esac

   exit 0

Das oben stehende Skript kann als gphoto-hook.sh abgespeichert werden.
Danach muss das Skript noch ausführbar gemacht werden mit

::

  chmod +x gphoto-hook.sh

Die Models zum Shooting antanzen lassen
---------------------------------------

Mit der folgenden Zeile rufen wir gPhoto2 auf.

::

  gphoto2 --auto-detect --capture-tethered --hook-script=./ghoto-hook.sh --force-overwrite

gPhoto2 führt jetzt nach jedem Druck auf den Kameraauslöser das Hook
Skript aus und startet Gimp mit dem neuen heruntergeladenen Foto. Da
Gimp als Standard nur eine Instanz zulässt, landen die Bilder in der
gleichen Gimp Applikation. Wenn mehrere Kameras angeschlossen sind, muss
natürlich direkt die richtige Kamera angegeben werden.

Anscheinend gibt es eine Acquire Funktion in Gimp, welche Image Daten
von verschiedenen Geräten (wie z.B. Scanner) einlesen kann. Allerdings
taucht diese Funktion bei mir in Gimp nicht auf.

* :ref:`genindex`

Zuletzt geändert: |date|

