CrunchBang Desktop
==================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Der CrunchBang Linux Desktop besteht aus dem Fenstermanager Openbox, dem Panel
Tint2, einem eigenen Theme und ein paar Scripten. Er erfordert nur wenig
Abhängigkeiten und läuft auch auf älteren Rechnern sehr gut.

Verwandte Artikel
-----------------

Es empfiehlt sich zu diesem Thema auch folgende Artikel gelesen zu
haben.

-  `pkg </howto/pkg>`__

Voraussetzungen
---------------

Diese Anleitung setzt voraus dass X11 entsprechend dem FreeBSD Handbuch
eingerichtet wurde, und nach Eingabe von "startx" mindestens ein
Terminal erscheint, von dem aus wir die Einrichtung starten können.

Alle benötigten Scripte, Themen, Konfigurationsdateien usw. befinden
sich in einem Archiv welches `hier
heruntergeladen <http://www.fahrner.name/public/CrunchBSD.txz>`__ werden
kann. Das Archiv enthält alle benötigten Dateien in einem
Verzeichnisbaum der dem entsprechenden Zielort entspricht. Am besten
entpackt man das Archiv in seinem Homeverzeichnis und kopiert die
Dateien/Ordner an ihren Bestimmungsort. Achtung: das Archiv enthält auch
versteckte Dateien/Ordner wie z.B. .config. Diese dürfen nicht übersehen
werden.

Dateien die in das Homeverzeichnis des Benutzers gehören befinden sich
unter /usr/local/share/crunchbang/skel. Diese werden beim Start über
Slim automatisch angelegt. Andernfalls kann man das Script
"/usr/local/share/crunchbang/cb-user-setup benutzername" manuell
ausführen.

Manche Verzeichnisse sind als .tgz Archive gepackt, da sie symbolische
Links enthalten können. Diese müssen am Zielort noch mit tar ausgepackt
werden.

Desweiteren enthält das Archiv ein Verzeichnis /PATCHES. Hierin befinden
sich Patches und gepatchte Scripte für fehlerhafte Originalpakete die
wir ersetzen müssen. Z.B. müssen wir eine Komponente des Openbox
Menüeditors wegen eines Fehlers ersetzen. Jeder Patch hat ein Readme das
beschreibt wo die gepatchte Datei hin kopiert werden muss. Updates
könnten diese Dateien später wieder überschreiben, aber ich hoffe mal
dass Updates auch die Fehler beheben.

Installation der Grundpakete
----------------------------

Als erstes installieren wir die zwingend benötigten Grundpakete, wie
z.B. den Fenstermanager oder das Panel. Wenn nicht anders angegeben
können die Pakete mittels pkg binär installiert werden, oder mittels
portmaster aus den Ports.

Diese Pakete benötigen wir für den ersten Start:

-  `x11-wm/openbox <https://www.google.com/search?q=x11-wm/openbox&btnI=lucky>`__
   Der Openbox Fenstermanager
-  `x11-wm/obmenu <https://www.google.com/search?q=x11-wm/obmenu&btnI=lucky>`__
   Openbox Menüeditor (hat einen Bug, siehe Hinweis unten)
-  `x11-wm/obconf <https://www.google.com/search?q=x11-wm/obconf&btnI=lucky>`__
   Openbox Konfigurationstool
-  `x11/tint-devel <https://www.google.com/search?q=x11/tint-devel&btnI=lucky>`__
   Das Tint2 Panel (Entwicklerversion)
-  `x11-wm/compton <https://www.google.com/search?q=x11-wm/compton&btnI=lucky>`__
   Compositing Manager für Transparenz und Schatten
-  `security/sudo <https://www.google.com/search?q=security/sudo&btnI=lucky>`__
   sudo. Nach der Installation mit "visudo" der Gruppe wheel alle
   Kommandos erlauben, und euren User zur Gruppe wheel hinzufügen.
-  `x11-themes/gtk-murrine-engine <https://www.google.com/search?q=x11-themes/gtk-murrine-engine&btnI=lucky>`__
   Die Theme-Engine
-  `x11-themes/lxappearance <https://www.google.com/search?q=x11-themes/lxappearance&btnI=lucky>`__
   Konfigurationstool für Themes
-  `sysutils/lxterminal <https://www.google.com/search?q=sysutils/lxterminal&btnI=lucky>`__
   Das Terminal
-  `x11-fm/pcmanfm <https://www.google.com/search?q=x11-fm/pcmanfm&btnI=lucky>`__
   Der Dateimanager
-  `shells/bash <https://www.google.com/search?q=shells/bash&btnI=lucky>`__
   Bourne Again Shell (wird von vielen Scripten gebraucht)
-  `graphics/hsetroot <https://www.google.com/search?q=graphics/hsetroot&btnI=lucky>`__
   Hintergrund setzen
-  `graphics/feh <https://www.google.com/search?q=graphics/feh&btnI=lucky>`__
   Grafikbetrachter zum setzen des Bildschirmhintergrunds
-  `graphics/viewnior <https://www.google.com/search?q=graphics/viewnior&btnI=lucky>`__
   Weiterer Grafikbetrachter
-  `graphics/scrot <https://www.google.com/search?q=graphics/scrot&btnI=lucky>`__
   Screenshot-Programm
-  `graphics/evince <https://www.google.com/search?q=graphics/evince&btnI=lucky>`__
   PDF-Viewer
-  `misc/rpl <https://www.google.com/search?q=misc/rpl&btnI=lucky>`__
   Strings in Dateien ersetzen
-  `sysutils/automount <https://www.google.com/search?q=sysutils/automount&btnI=lucky>`__
   Wechselmedien automatisch einhängen
-  `x11/gmrun <https://www.google.com/search?q=x11/gmrun&btnI=lucky>`__
   Programme ausführen
-  `devel/geany <https://www.google.com/search?q=devel/geany&btnI=lucky>`__
   Texteditor
-  `graphics/mesa-demos <https://www.google.com/search?q=graphics/mesa-demos&btnI=lucky>`__
   Enthält das benötigte glxinfo
-  `math/galculator <https://www.google.com/search?q=math/galculator&btnI=lucky>`__
   Taschenrechner
-  `x11/lxmenu-data <https://www.google.com/search?q=x11/lxmenu-data&btnI=lucky>`__
   XDG Menü

Optional:

-  `security/py-paramiko <https://www.google.com/search?q=security/py-paramiko&btnI=lucky>`__
   Für das Openbox Network/SSH Menü
-  `sysutils/fusefs-sshfs <https://www.google.com/search?q=sysutils/fusefs-sshfs&btnI=lucky>`__
   Mounten via SSHFS für Network/SSH Menü
-  `x11/xscreensaver <https://www.google.com/search?q=x11/xscreensaver&btnI=lucky>`__
   Bildschirmschoner/-locker
-  `net-mgmt/wifimgr <https://www.google.com/search?q=net-mgmt/wifimgr&btnI=lucky>`__
   WLAN-Manager
-  `audio/mixmos <https://www.google.com/search?q=audio/mixmos&btnI=lucky>`__
   Audio Mixer

Wer optionale Software nicht installiert, kann sie mit dem Menü-Editor
aus seinem Openbox Menü entfernen.

Compilieren des Qt Kerns
~~~~~~~~~~~~~~~~~~~~~~~~

Damit Programme, welche die Qt Grafikbibliotheken verwenden, sich
optisch besser in die GTK-Umgebung einfügen, sollten wir gleich zu
Beginn 2 Pakete aus den Ports compilieren. Wir fügen in die Datei
/etc/make.conf folgende Zeile ein:

::

   QT4_OPTIONS=QGTKSTYLE

Anschliessend installieren wir diese 2 Pakete mittels portmaster aus den
Ports:

-  `devel/qt4-corelib <https://www.google.com/search?q=devel/qt4-corelib&btnI=lucky>`__
   Qt 4 Kernbibliothek
-  `x11-toolkits/qt4-gui <https://www.google.com/search?q=x11-toolkits/qt4-gui&btnI=lucky>`__
   Qt 4 GUI

Jetzt noch das Paket
`misc/qt4-qtconfig <https://www.google.com/search?q=misc/qt4-qtconfig&btnI=lucky>`__
installieren, und mittels Kommando qtconfig-qt4 den GTK+ Stil auswählen.
Um zu verhindern dass die aus den Ports compilierten Pakete durch "pkg
upgrade" überschrieben werden:

::

   pkg lock devel/qt4-corelib
   pkg lock x11-toolkits/qt4-gui

Konfiguration
-------------

Die Openbox Session wird durch .xinitrc gestartet (Slim) oder durch
.xsession (xdm). Am einfachsten legt man eine .xinitrc an und macht
einen symlink auf .xsession. Beispiel für .xinitrc:

::

   LANG=de_DE.UTF-8; export LANG
   CHARSET=UTF-8; export CHARSET
   BROWSER=firefox; export BROWSER
   XDG_CONFIG_DIRS=/usr/local/etc/xdg; export XDG_CONFIG_DIRS
   XDG_MENU_PREFIX='lxde-'; export XDG_MENU_PREFIX
   exec openbox-session

Beim ersten Start über Slim wird das Konfigurationsscript
/usr/local/share/crunchbang/cb-user-setup ausgeführt, welches alle
benötigten Verzeichnisse und Konfigurationsdateien anlegt. Beim Start
über xdm muss dieses Script einmalig manuell ausgeführt werden:

::

   /usr/local/share/crunchbang/cb-user-setup "benutzername"

Das Exit-Applet muss die Befehle acpiconf und shutdown als root
ausführen können. Damit das jeder Benutzer ohne Passwort machen kann,
fügen wir mittels "visudo" am Ende folgende Zeile an:

::

   # Allow any user to run acpiconf and shutdown as root without password
   ALL ALL=(root) NOPASSWD: /usr/sbin/acpiconf, /sbin/shutdown

Themes
~~~~~~

Wurde das Setup-Script korrekt ausgeführt, sollten alle Einstellungen
passen. Zur Kontrolle:

-  lxappearance starten. Als Fenster-Thema "waldorf" auswählen, als
   Symbol-Thema CrunchBang-Dark.
-  obconf starten. Als Thema "waldorf" auswählen.

Icon Theme
~~~~~~~~~~

Das CrunchBang Icon Theme baut auf dem Gnome Icon Theme auf
(`misc/gnome-icon-theme <https://www.google.com/search?q=misc/gnome-icon-theme&btnI=lucky>`__),
und zwar auf einem älteren, in dem viele Icons eine gräuliche Färbung
besitzen. Im aktuellen Gnome Icon Theme sind diese bräunlich eingefärbt,
was nicht so gut zum restlichen Theme passt. Das ältere (gräuliche) ist
im Archiv mit enthalten. Wer lieber das neuere (bräunliche) mag, der
kann das aus dem Archiv weglassen und stattdessen das Paket
`misc/gnome-icon-theme <https://www.google.com/search?q=misc/gnome-icon-theme&btnI=lucky>`__
installieren.

Alternative Themes
~~~~~~~~~~~~~~~~~~

Was auch sehr gut aussieht ist Theme `"Equinox
Evolution" <http://gnome-look.org/content/show.php?content=140449>`__
mit zugehörigem Openbox Theme "Equinox Evolution" (befindet sich im
Archiv unter EXTRAS). Dazu muss das Paket
`x11-themes/gtk-equinox-engine <https://www.google.com/search?q=x11-themes/gtk-equinox-engine&btnI=lucky>`__
installiert werden. Dazu passt auch das bräunliche Gnome Icon Theme ganz
gut.

Schriftarten
~~~~~~~~~~~~

Mit dem X11 Paket werden gleichzeitig auch die Xorg Bitmap Fonts
installiert, welche manchmal als Fallback für nicht vorhandene Schriften
dienen und dazu führen das Text ziemlich pixelig dargestellt wird. Die
Bitmap Fonts deaktivieren wir wie folgt:

::

   # cd /usr/local/etc/fonts/conf.d
   # ln -s ../conf.avail/70-no-bitmaps.conf .

Grafisches Login
----------------

Für ein grafisches Login installieren wir noch
`x11/xdm <https://www.google.com/search?q=x11/xdm&btnI=lucky>`__ und
aktivieren es in der Datei /etc/ttys durch ändern des Eintrags:

::

   ttyv8 "/usr/local/bin/xdm -nodaemon -config /usr/local/lib/X11/xdm/xdm-config-cb" xterm   on  secure

oder alternativ
`x11/slim <https://www.google.com/search?q=x11/slim&btnI=lucky>`__ und
aktivieren es in der Datei /etc/ttys durch ändern des Eintrags:

::

   ttyv8 "/usr/local/bin/slim"       xterm   on  secure

**Achtung:** bei Verwendung von Slim muss die letzte Zeile im .xinitrc
folgendermassen lauten:

::

   exec openbox-session </dev/null >$HOME/.xsession-out 2>&1

Grund ist, dass Slim beim Start über /dev/ttys die Dateidescriptoren für
stdin/stdout/stderr nicht erzeugt. Und auch beim Start als Service
zeigen diese Descriptoren nur auf /dev/null, d.h. alle Fehlermeldungen
gehen verloren. Außerdem wertet Slim die /etc/login.conf nicht aus.
Unter \*BSD empfehle ich die Verwendung von xdm, da Slim für Linux
gebaut wurde, zusätzlich die Linux-Daemons Polkit und Consolekit
startet, und zudem schlecht auf \*BSD portiert ist.

Optionale Pakete
----------------

Kalender
~~~~~~~~

Für die Uhr im Panel gibt es auch einen Kalender:
`gsimplecal <http://dmedvinsky.github.io/gsimplecal/>`__. Dafür werden
die Pakete
`devel/autoconf <https://www.google.com/search?q=devel/autoconf&btnI=lucky>`__
und
`devel/automake <https://www.google.com/search?q=devel/automake&btnI=lucky>`__
benötigt. Anschliessend kann wie folgt installiert werden:

::

   git clone git://github.com/dmedvinsky/gsimplecal.git
   cd gsimplecal
   git pull
   ./autogen.sh
   ./configure --enable-gtk2
   make
   sudo make install

Systemmonitor
~~~~~~~~~~~~~

Systemmonitor für den Desktop-Hintergrund:
`sysutils/conky <https://www.google.com/search?q=sysutils/conky&btnI=lucky>`__

Applet zur Lautstärkeregelung
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Als Lautstärkeregler im Panel eignet sich
`VolWheel <http://oliwer.net/b/volwheel.html>`__ sehr gut. Für die
Installation muss vorher das Perl Shell Modul und Gtk2 installiert
werden mit:

::

   sudo cpan Shell
   sudo cpan Gtk2

(Hier gibts noch ein Problem: beim Start findet er kein Modul Alsa.pm.
Auf meinem Test-PC klappte das, muss ich suchen wie ich das damals
hinbekommen habe)

Launcher
~~~~~~~~

Launcher für das Panel:
`deskutils/launchy <https://www.google.com/search?q=deskutils/launchy&btnI=lucky>`__

* :ref:`genindex`

Zuletzt geändert: |date|

