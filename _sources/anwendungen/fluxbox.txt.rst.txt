Fluxbox
=======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Der Fluxbox Windowmanager ist ein schlanker Windowmanager, der auch noch leicht
über Konfigurationsdateien konfigurierbar ist. Er setzt sich eigentlich nur aus
der "toolbar" zusammen. Mit den Styledateien kann man Dinge wie Farben, die
Uhr, Buttons, die Titelleiste usw. leicht ändern. Fluxbox ist "nur" ein
Windowmanager und bringt kein Softwarepaket oder Konfigurationsprogramm mit
sich. Fluxbox basiert auf Blackbox 0.61.1 (100% Theme-Kompatibilität).

Features
--------

|fluxbox.jpg|

::

   *Window Tabs
   *Iconbar
   *WheelScroll zum Wechsel des virtuellen Desktops
   *Titlebar
   *Hohe Konfigurationsmöglichkeiten
   *Leicht konfigurierbare Keybindings

Konfiguration
~~~~~~~~~~~~~

Fluxbox benötigt natuerlich einen Xserver, den man vorher installiert
und konfiguriert haben sollte. Die Installation von Fluxbox selber
erfolgt wie normal entweder über die Ports oder die Packages. Wenn
Fluxbox installiert ist, dann kann man fluxbox so über das startx Skript
starten:

::

    touch ~/.xinitrc
    echo 'exec fluxbox &' >> ~/.xinitrc
    echo 'fbsetbg -f $wallpaper' >> ~/.xinitrc
    startx &

Nun kann man also Fluxbox von einem Terminal aus starten. Wenn man aber
will, dass Fluxbox beim Systemstart gleich mitgestartet wird, muss man
einen Displaymanager (z.B. `xdm </anwendungen/xdm>`__) installieren.
(eine Beschreibung für Fluxbox mit gdm gibt es hier:
`Fluxbox_mit_gdm </howto/Fluxbox_mit_gdm>`__)

Keybindings
-----------

Fluxbox besitzt keine Icons, aber alles aus der Konsole oder dem Menü
herauszustarten ist doch teilweise umständlich. Abhilfe schaffen hier
Keybindings. So kann man dann durch eine Tastenkombination eine
Anwendung, beispielsweise xterm starten. Konfiguriert werden diese
Keybindings über die datei '~/.fluxbox/keys'. Die Syntax ist relativ
einfach:

::

   <Mod-Taste> <Taste> :<Fluxbox-Befehl> <Parameter>

Mod-Tasten sind Tasten, wie Strg, Alt (Mod1) und die "Windows-Taste"
(Mod4). Fluxbox-Befehle sind beispielsweise Workspace (schaltet die
Arbeitsflächen um) oder ExecCommand (führt Befehle aus). Aber nun genug
der Theorie, hier ein Praktisches Beispiel zum start von 'xterm' über
'Alt+x':

::

   Mod1 x :ExecCommand xterm

An xterm kann man sogar noch Parameter dazusetzen:

::

   Mod1 x :ExecCommand xterm -ls

Hinweise
~~~~~~~~

Wenn man sich das Paket `eterm </anwendungen/eterm>`__ installiert, kann
man den Hintergrund, also das root-Window, ändern. Außerdem kann man
sicht mit `iDesk </anwendungen/iDesk>`__-Icons auf das root-Window legen
lassen. Mit einem Systemmonitor wie `GKrellM </anwendungen/GKrellM>`__
oder `Torsmo </anwendungen/Torsmo>`__ hat man seine Systemauslastung
ständig im Blick.

Weblinks
--------

-  `Projekthomepage <http://fluxbox.org/>`__
-  `Themes <http://themes.freshmeat.net/browse/962>`__

.. |fluxbox.jpg| image:: images/fluxbox.jpg
   :class: align-right
   :width: 200px

* :ref:`genindex`

Zuletzt geändert: |date|

