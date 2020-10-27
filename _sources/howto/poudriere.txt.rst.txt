Poudriere
=========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Unter FreeBSD besteht die Möglichkeit, sich sein eigenen PKG Repository aus den
Ports zu bauen. Der Vorteil besteht darin, dass man sich auf den produktiven
Systemen nicht mit dem kompilieren der Software befassen muss, da dies
vorgängig auf einem dritten System erledigt werden kann. Ein weiterer Vorteil
besteht darin, dass das einspielen der fertigen PKG Pakete angenehmer ist. Der
gesamte Updateprozess wird dadurch schneller und die Downtime der Systeme
kürzer. Poudriere hat im Moment noch ein paar Schwierigkeiten, wenn es in einer
Jail betrieben wird. Daher ist es ratsam, Poudriere auf einer physikalische
Maschine zu installieren. Dies kann im Notfall auch eine VM in Virtualbox oder
Co. sein. Ich habe keine Probleme diesbezüglich festgestellt.

Systemvorraussetzungen
----------------------

Poudriere benötigt ausser einer guten CPU mit genügend RAM folgende
Punkte:

-  FreeBSD >= 8.3
-  "root" Zugriff
-  ZFS (kein Muss!)

In der Regel buildet Poudriere "PKG" (pkgng) Pakete. Es ist aber auch
möglich die alte Variante zu bauen. Dies sollte man aber vermeiden, da
in Zukunft nur noch "PKG" unterstützt wird (FreeBSD >=10.0).

Installation
------------

Folgende Schritte sind für die Installation erfolderlich:
`ports-mgmt/poudriere <https://www.google.com/search?q=ports-mgmt/poudriere&btnI=lucky>`__
(FreeBSD Ports)::

  % cd /usr/ports/ports-mgmt/poudriere 
  % make install clean 
  % cp /usr/local/etc/poudriere.conf.sample /usr/local/etc/poudriere.conf 

oder::

  % portmaster ports-mgmt/poudriere 
  % cp /usr/local/etc/poudriere.conf.sample /usr/local/etc/poudriere.conf

Konfiguration und Anpassungen
-----------------------------

Poudriere Einstellungen
.......................

In der Datei "poudriere.conf" findet man sämtliche Einstellungen. Meist sind
diese gut gewählt. Ein paar Einstellungen müssen aber manuell gesetzt werden.

::

   ZPOOL=rpool
   ZROOTFS=/poudriere
   FREEBSD_HOST=http://ftp.ch.freebsd.org/
   RESOLV_CONF=/etc/resolv.conf
   BASEFS=/poudriere/base
   POUDRIERE_DATA=${BASEFS}/data
   USE_PORTLINT=no
   DISTFILES_CACHE=/poudriere/distfiles
   CHECK_CHANGED_OPTIONS=verbose
   CHECK_CHANGED_DEPS=yes
   WRKDIR_ARCHIVE_FORMAT=txz
   NOLINUX=yes

"ccache" und "tmpfs" können zum schnelleren bauen der Pakete verwendet
werden. Bei "ccache" ist darauf zu achten, dass es schon auf dem System
eingerichtet ist! Bei "tmpfs" ist dies nicht nötig.

::

   CCACHE_DIR=/ccache
   USE_TMPFS=yes

Möchte man ZFS nicht verwenden, dann ist folgende Zeile wichtig:

::

   NO_ZFS=yes

Existiert der ZFS Pfad noch nicht, muss dieser erstellt werden::

  % zfs create -o mountpoint=/poudriere rpool/poudriere

Poudriere verwendet dieses Verzeichnis als "Working Folder". Sämtliche Pakete,
Jails und Logs werden hier erstellt und auch wieder gelöscht. Ein weiteres
Verzeichnis welches verwendet wird ist: **/usr/local/etc/poudriere.d**. Hier
sind die Einstellungen der Ports hinterlegt, genauer im Verzeichnis
**/usr/local/etc/poudriere.d/options/<PORT>**. 

Portstree
.........

Als
erstes checken wir uns einen aktuellen Portstree aus::

  poudriere ports -c 

Jail erstellen
..............

Dann können wir verschiedene Versionen von FreeBSD in den Jails erstellen z.B.
mit FreeBSD 10.0-RELEASE oder 9.2-RELEASE::

  poudriere jail -c -j 100_x64 -v 10.0-RELEASE -a amd64 
  poudriere jail -c -j 92_x64 -v 9.2-RELEASE -a amd64 

Der Name (100_x64) kann frei gewählt werden, sollte aber aussagekräftig sein.

Anpassungen "make.conf"
-----------------------

Für jede Jail kann entweder eine eigene "make.conf" erstellt oder eine
globale, welche für alle Jails gültig ist verwendet werden. Jeweils der
Name ist dafür ausschlaggebend. Bei den oben erstellten Jails würde der
Name "100_x64-make.conf" lauten. Die Datei muss im Ordner
"/usr/local/etc/poudriere.d" gespeichert werden. Im folgendem Beispiel
werden sämtliche Pakete ohne "X" gebaut.

::

   # PKG
   WITH_PKGNG=yes

   # build ports allways without X 
   WITHOUT_X11=yes

   # default options
   OPTIONS_UNSET= BONJOUR DEBUG X11

Der Eintrag **WITH_PKGNG=yes** wird nur für FreeBSD 8.X und 9.X benötig.
Ab FreeBSD 10.0 wird dieser automatisch gesetzt.

Liste der Pakete
----------------

Damit Poudriere auch weiss welche Pakete es bauen soll, erstellt man
eine Datei poudriere-list und speichert diese im Ordner
"/usr/local/etc/". Der Name der Datei ist nicht zwingend und kann
beliebig gewählt werden. Der Inhalt der Datei sieht z.B. so aus:

::

   shells/zsh
   sysutils/beadm
   sysutils/bsdinfo
   sysutils/smartmontools
   sysutils/tmux
   sysutils/zfs-stats

Optionen pro Port setzen
........................

Poudriere bietet auch die Möglichkeit, spezielle Anpassungen pro Port
einzustellen. Damit können die Ports dann individuell konfiguriert werden::

  % poudriere options -c shells/zsh 

Optionen löschen
................

Die gewählten Optionen kann man von einem Port auch wieder löschen::

  % poudriere options -r shells/zsh

Dies sollte auch möglich sein, indem man den Ordner des Ports im Ordner
"/usr/local/etc/poudriere.d/options" löscht.

Kompilieren der Ports
---------------------

Das kompilieren muss separat pro Jail gestartet werden. Poudriere kann Ports
parallel bauen. Gibt man keine speziellen Optionen in der "poudriere.conf" mit,
so entspricht dies der Anzahl an Cores auf der Hostmaschine. ::

  % poudriere bulk -f /usr/local/etc/poudriere-list -j 100_x64 
  % poudriere bulk -f /usr/local/etc/poudriere-list -j 92_x64

Möchte man alle Ports (Achtung über 24'000!) bauen kann man dies natürlich auch
machen::

  % poudriere bulk -a -j 100_x64 poudriere bulk -a -j 92_x64

Update und tägliche Aufrufe
---------------------------

Der Portstree sollte immer aktuell gehalten werden, damit Poudriere die Pakete
auch aktualisieren und bauen kann::

  % poudriere ports -u

Weiter ist es sinnvoll, auch die Jail's mit Updates (freebsd-update) zu
versorgen::

  % poudriere jail -u -j 100_x64
  % poudriere jail -u -j 92_x64

Stehen Updates zur Verfügung, muss das kompilieren im Anschluss angestossen
werden::

  % poudriere bulk -f /usr/local/etc/poudriere-list -j 100_x64 
  % poudriere bulk -f /usr/local/etc/poudriere-list -j 92_x64

Status abfragen
---------------

Webfrontend
...........

Im Verzeichnis **/poudriere/base/data/logs/bulk/<JAIL>/latest** befindet sich
ein JSON basiertes Webfrontend, welches eine schnelle Übersicht über die
aktuelle Lage liefert. Dieses Verzeichnis könnte man jetzt über einen Webserver
wie z.B. "nginx" verfügbar machen. Für eine "Quick-and-dirty" Lösung ist aber
auch "python" nützlich. Dazu wechselt man in dieses Verzeichnis und startet
"python" mit dem Modul "SimpleHTTPServer"::

  % cd /poudriere/base/data/logs/bulk/100_x64-default 
  % python2 -m SimpleHTTPServer 8080 

  % python3 -m http.server 8080

Im Browser öffnet man jetzt die Adresse "<HOSTNAME>:8080" und klickt auf den
Namen "latest@". Hier sollte sich dann das WebGUI befinden. 

CLI
...

Auf der Konsole kann man jederzeit, solange der Buildprozess noch läuft "CTRL +
t" für einen Statusabfrage drücken.

Die fertigen Pakete
-------------------

Sind alle Ports gebaut so liegen die fertigen Pakete im Verzeichnis
**/poudriere/base/data/packages/<JAIL>/ALL**. Wie man diese in das "PKG"
Repository des jeweiligen Server's einbindet steht im Artikel

* :ref:`genindex`

Zuletzt geändert: |date|

