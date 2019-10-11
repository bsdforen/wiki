Screen
======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Mittels Screen kann man mehrere virtuelle Konsolen nebeneinander laufen
haben (sog. *Multiplexing*) und diese Session auch *detachen*, d.h.
Screen läuft auch nach dem Ausloggen des Benutzers im Hintergrund
weiter, so muss man z.B. seinen IRC Clienten beim Ausloggen nicht
beenden.

Anwendung
---------

Die ersten Schritte mit Screen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nachdem man Screen installiert hat (in den Ports meist unter misc/screen
zu finden), tippt man

::

   $ screen

Nun sollte eine Nachricht auftauchen, welche die Version von Screen und
GPL enthält. Durch drücken der Enter- oder Leertaste verschwindet die
Nachricht und man hat eine ganz normale Shell vor sich.

Die Bedienung von Screen ist vielleicht am Anfang etwas seltsam, man
gewöhnt sich aber schnell daran. Um eine weiter Shell zu öffnen, drückt
man *Strg+a* und dann *c*. Um nun zurück zur vorherigen Shell zu kommen,
drückt man *Strg+a* und *p* (für previous). Wie man sehen kann, sind
Befehle für Screen so aufgebaut, dass man erst *Strg+a* drückt und dann
die Taste für die jeweilige Aktion.

Hier ist nun eine kleine Liste mit Befehlen, die für den Anfang sehr
nützlich sind:

::

   Strg+a c  Startet eine neue Shell
   Strg+a p  Zurück zum vorherigen Fenster
   Strg+a n  Zum nächsten Fenster
   Strg+a 1  Zeigt das 1. Fenster an, Strg+a 2 das 2. usw.
   Strg+a "  Zeigt eine Liste aller Fenster an
   Strg+a A  Editiert den Titel des aktuellen Fensters

Die Manpage enthält die komplette Liste aller Befehle.

Detachen und Reattachen
~~~~~~~~~~~~~~~~~~~~~~~

Mittels dieses Features läuft die Screensession im Hintergrund weiter,
selbst wenn man sich ausloggt. *Strg+a* und *d* drücken und Screen
detached sich und man ist wieder dort, von wo aus man Screen gestartet
hat. Nun kann man sich ausloggen und später einfach mittels

::

   screen -r

die vorherige Session wieder in den Vordergrund bringen, d.h.
reattachen.

Konfiguration von Screen
------------------------

Im Folgenden werden ein paar Tipps zu Optionen der **~/.screenrc**
aufgezeig, die man evtl. ändern bzw. hinzufügen möchte.

::

   startup_message off

Hiermit verschwindet der Startbildschirm, wenn Screen gestartet wird.

::

   vbell on
   vbell_msg "AAARGH!!!"

Statt eines Pieptons wird vbell_msg angezeigt.

::

   bindkey -k k8 prev
   bindkey -k k9 next

Mit dieser Option kann man mit der F8 bzw. F9 Taste zum vorherigen bzw.
zum nächsten Fenster wechseln.

::

   bind K kill

Mit ``Strg+a`` und ``K`` wird der Befehl für Screen gegeben, das
aktuelle Fenster zu zerstören.

::

   bind i screen -t irssi irssi
   bind l screen -t lynx lynx http://www.bsdforen.de

Wenn man ``Strg+a`` und ``i`` drückt, wird ein Fenster mit dem Titel
``irssi`` geöffnet und irssi in diesem gestartet. Gleiches Schema für
``Strg+a`` und ``l``, nur mit lynx.

::

   hardstatus alwayslastline "%{= dd}%t %W%=%c"

Hiermit wird in der untersten Zeile der Konsole eine Statuszeile
angezeit.

Eine kurze Erklärung der hier verwendeten Variablen:

::

   %{= dd} Setzt die Hintergrundfarbe und die Farbe der Schrift auf Default (weiß auf schwarz)
   %t      Titel des aktuell angezeigten Fensters
   %W      Titel aller übrigen Fenster
   %=      Alles, was nach diesem Zeichen kommt, wird an der rechten Seite der Konsole angezeigt
   %c      Uhrzeit

Weiteres dazu kann man in der Screen Manpage unter "STRING ESCAPES"
nachlesen. In strcat's .screenrc (siehe Anhang) gibt es viele
Statusline-Beispiele, die man ausprobieren und den eigenen Bedürfnissen
anpassen kann.

Sonstiges
---------

Wer nur das *detach*-Feature von Screen benötigt, sollte sich mal das
`dtach <http://dtach.sourceforge.net/>`__ Projekt ansehen.

Verweise
--------

-  `Homepage von GNU/Screen <http://www.gnu.org/software/screen/>`__
-  `Offizielle Screen
   FAQ <http://www4.informatik.uni-erlangen.de/~jnweiger/screen-faq.html>`__
-  `strcat's .screenrc <http://strcat.de/dotfiles/misc/screenrc>`__ -
   Hilf- und lehrreiche .screenrc

* :ref:`genindex`

Zuletzt geändert: |date|

