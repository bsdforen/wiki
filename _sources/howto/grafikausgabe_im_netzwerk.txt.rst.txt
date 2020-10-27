Grafikausgabe im Netzwerk: X11, XDM und XDMCP
=============================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

X11 ist ein Display-Protokoll, das ursprünglich für die Grafikausgabe im
Netzwerk konzipiert worden ist. Die Netzwerkfähigkeit von X11, die es so
interessant und vielfältig einsetzbar macht, stammt jedoch aus einer Zeit, als
Netzwerk-Sicherheit noch keine grosse Rolle spielte.

<note important> Deshalb sollten die im folgenden Artikel gezeigten
Konfigurationen nur in lokalen, gut abgesicherten Netzen genutzt werden.
Das Tunneln einer XDMCP-Verbindung durch OpenSSH ist nach meinem
Kenntnisstand nicht möglich - stattdessen sollte z.B. ein OpenVPN-Tunnel
zur Absicherung eingesetzt werden.</note>

Modell & Praxis
---------------

Die Anwenderin sitzt vor einem Computer mit Bildschirm. Auf diesem läuft
als Programm ein lokaler *X-Server*. In der Praxis werden diese Computer
- je nach Ausstattung und Anwendungszweck - *X-Terminals* oder
*X-Workstations* genannt.

Beim Start des X-Servers sucht dieser im Netzwerk nach Computern auf
denen die sogenannten *X-Displaymanager* (XDM) laufen. Über diese auch
*XDM-Server* genannten Computer kann sich die Anwenderin dann auf wieder
anderen Computern anmelden und die dort installierten Programme
benutzen. Auf diesen entfernten Computern - den *X-Clients* - arbeitet
die Anwenderin dann mit Programmen wie OpenOffice oder Konqueror,
angezeigt bekommt sie dies aber von dem lokalen Computer, auf dem der
X-Server läuft.

Soweit das Modell, das aus X-Server, XDM-Server und X-Client besteht
(also drei miteinander vernetzten Computern). Es handelt sich hierbei um
eine XDMCP-Verbdingung über den UDP-Port 177 (*X Display Manager
Configuration Protocol*). In der Praxis läuft der X-Displaymanager meist
direkt auf dem X-Client. Und bei den unix- oder linux-basierten
Desktoprechnern dieser Tage, laufen X-Server, Displaymanager und
Anwendungen gemeinsam auf einem Rechner.

In diesem Fall erscheint die Netzwerkfähigkeit natürlich als Overkill.
Andererseits wird der Einsatz von Virtuellen Maschinen populärer, auch
bei Anwendern im privaten Bereich. Mit dem gleichzeitigen Einsatz von
mehreren Unix -und Linux-Systemen auf einem einzelnen Rechner kann (und
muss!) die Netzwerkfähigkeit von X11 wieder sinnvoll genutzt werden.

XDM konfigurieren
-----------------

Die folgende Konfiguration beschränkt sich derzeit auf den
Displaymanager xdm. Die Displaymanager von GNOME oder KDE, gdm bzw. kdm
unterscheiden sich in der Konfiguration dazu geringfügig bis stark.
Entsprechende Abschnitte dazu fehlen noch.

Zunächst wird das System konfiguriert, auf dem der X-Displaymanager und
die Programme laufen sollen. Die Beispiel-IP-Adresse dieses Systems ist
192.168.1.3

Es müssen zumindest die Dateien **xdm-config**, **Xaccess** und
**Xservers** angepasst werden. Sie befinden sich unter NetBSD 3.1 in dem
Verzeichnis **/etc/X11/xdm/**

::

   # cd /etc/X11/xdm/
   # vi xdm-config

In der Datei **xdm-config** findet sich die folgende Zeile:

::

   DisplayManager.requestPort: 0

Die Zeile muss zur Aktivierung von XDMCP-Verbindungen auskommentiert
werden:

::

   !DisplayManager.requestPort: 0

In der Datei **Xaccess** sind normalerweise die folgenden beiden Zeilen
so auskommentiert:

::

   #*                             #any host can get a login window
   #*     CHOOSER BROADCAST       #any indirect host can get a chooser

Diese beiden Optionen werden benötigt. Die Kommentarzeichen müssen
entfernt werden:

::

   *                             #any host can get a login window
   *     CHOOSER BROADCAST       #any indirect host can get a chooser

In der Datei **Xservers** befindet sich standardmässig die folgende
Zeile:

::

   :0 local /usr/local/bin/X vt05 -nolisten tcp

Mit dieser Option würde der Displaymanager einen lokalen X-Server
starten. Das ist aber nomalerweise überflüssig oder sinnlos, weil zur
Grafikausgabe ja ein anderer X-Server auf einem anderen System genutzt
wird. Diese Zeile muss also mit einem Kommentarzeichen deaktiviert
werden:

::

   #:0 local /usr/local/bin/X vt05 -nolisten tcp

Nun muss noch der Displaymanager gestartet werden. Damit dies
automatisch nach dem Systemstart geschieht, trägt man den xdm als Option
in die Datei **/etc/rc.conf** ein:

::

    xdm=YES xdm_flags=""

Um das System nicht neu booten zu müssen, wird der xdm dieses eine mal
noch manuell von der Kommandozeile aus gestartet:

::

   # xdm &

Den X-Server starten
--------------------

Nun wird auf dem Computer mit Grafikkarte und angeschlossenem Bildschirm
der X-Server gestartet. Der X-Server sollte natürlich über die Datei
**/etc/X11/XF86Config** bereits korrekt konfiguriert sein. Von der
Kommandozeile aus wird der X-Server gestartet und dabei angewiesen, den
Displaymanager auf dem Rechner ``192.168.1.3`` anzuzeigen:

::

   # X -query 192.168.1.3 &

Falls bereits ein X-Server laufen sollte, wird einfach ein zusätzlicher
gestartet. In diesem Fall sieht der Aufruf dann wie folgt aus:

::

   # X -query 192.168.1.3 :1 &

Nach kurzer Zeit sollte ein kleines Login-Fenster erscheinen, das die
Anwenderin auf dem Rechner 192.168.1.3 begrüßt.

Sichere Alternative
'''''''''''''''''''

Wenn auf das grafische Einloggen mit dem xdm verzichtet werden kann,
bietet es sich an, die grafischen Programme auf dem entfernten Rechner
über eine sichere OpenSSH-Verbindung zu starten (*Tunneln*). Das bietet
den Vorteil, auf die notwendigen Konfigurationschritte von xdm, OpenVPN
und pf verzichten zu können, so dass zusätzliche Fehlerquellen vermieden
werden.

Auf dem entfernten Rechner, auf dem der Desktop oder andere grafische
Programme gestartet werden sollen, muss sich die folgende Zeile in der
Datei **/etc/ssh/sshd_config** befinden:

::

   X11Forwarding yes

In der zentralen Konfigurationsdatei **/etc/rc.conf** muss eventuell
noch der ssh-Daemon eingetragen werden:

::

   sshd_enable="YES"

Zum Abschluss wird der sshd gestartet:

::

   # /etc/rc.d/sshd restart

Auf dem lokalen Rechner wird nun als normale Anwenderin der X-Server
gestartet, nur mit einem xterm - aber ohne Windowmanager! Das kann mit
der Datei **.xinitrc** im Home-Verzeichnis der Anwenderin konfiguriert
werden: einfach nur 'xterm' dort eintragen. Mit 'startx' wird der
X-Server gestartet und im xterm die ssh Verbindung hergestellt, um
schliesslich auf dem entfernten Rechner, den KDE-Desktop zu starten:

::

   local$ ssh -Y user@192.168.0.2
   ...
   ...
   remote$ startkde

Verweise
--------

-  `Linux
   XDMCP-HOWTO <http://www.tldp.org/HOWTO/XDMCP-HOWTO/index.html>`__
-  `For a full list of where to stop X from listening to port
   6000 <http://groups.google.com/group/alt.os.linux.slackware/msg/983b7ab4cb74db5c>`__
-  `XDMCP oder Linux-PCs als
   X-Terminals <http://www.pl-berichte.de/t_netzwerk/xdmcp.html>`__
-  `X-Display
   umleiten <http://www.internetmagazin.de/praxis/linux/a/X_Display_umleiten>`__
-  `GDM
   Security <http://www.gnome.org/projects/gdm/docs/2.14/security.html>`__
-  `Remote X Apps
   mini-HOWTO <http://www.faqs.org/docs/Linux-mini/Remote-X-Apps.html>`__
-  `XDM and X Terminal
   mini-HOWTO <http://www.tldp.org/HOWTO/XDM-Xterm/index.html>`__
-  `X Window System
   Security <http://bau2.uibk.ac.at/matic/xsecur.htm>`__
-  `Crash Course in X Windows
   Security <http://www.uwsg.iu.edu/usail/external/recommended/Xsecure.html>`__
-  `Secure Computing: Notes on X Window
   Security <http://www.stanford.edu/group/security/securecomputing/x-window/index.html>`__
-  `Using
   X-Windows <http://pangea.stanford.edu/computerinfo/unix/xterminal/index.html>`__

* :ref:`genindex`

Zuletzt geändert: |date|

