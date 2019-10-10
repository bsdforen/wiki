Xmodmap
=======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

``xmodmap`` ist praktisch zum Anpassen des X11-Keyboard-Layouts, falls z.B. bei
der Tastatur ein paar Tasten fehlen oder falsch gemappt werden. Aber auch wenn
das gewünschte Layout nicht vorhanden ist, wie z.B. angepasste Dvorak-Layouts
oder ähnliches.

Installation
------------

``xmodmap`` gehört zu Xorg bzw. XFree. Siehe daher X11-Installation des
jeweiligen Betriebssystems.

Ändern von Tastenbelegungen
---------------------------

Zum Ändern von einzelnen Tastenbelegungen kann man auf drei Arten
vorgehen.

1. mit dem Parameter ``-e``:

::

   $ xmodmap -e "keycode 78 = l"

2. über die Standardeingabe:

::

   $ xmodmap -
   keycode 78 = l
   ^D

3. über eine Datei:

::

   $ cat > mymap << EOF
   keycode 78 = l
   EOF
   $ xmodmap mymap

Die Syntax von ``keycode`` sieht folgendermaßen aus:

::

   keycode <nummer> = <keysym>

Es können bis zu Acht Keysyms angegeben werden, jedoch sind die letzten
vier in normalen X11-Umgebungen nicht nutzbar.

Die erste Keysym ist die Taste bei normalem Drücken, die zweite bei
gedrücktem Shift, die dritte bei Mode_switch (normalerweise AltGr), die
vierte wenn sowohl Mode_switch als auch Shift gedrückt sind.

Um die aktuelle Belegung der Tasten zu bekommen, kann man den Befehl
``xmodmap -pke`` benutzen.

Wenn dies in eine Datei umgelenkt wird, ist es einfach, mehrere Tasten
auf einmal umzuändern und diese dann über die Datei zu laden.

Zum automatischen Laden beim Start von X11, füge man dann einfach den
Aufruf in seine ``.xinitrc`` oder ``.xsession`` ein, je nachdem welche
man verwendet.

Beispiel:

::

   $ xmodmap -pke > ~/.xmodmaprc
   $ vi ~/.xmodmaprc
   // Bearbeiten
   $ vi ~/.xinitrc
   // Einfügen: xmodmap ~/.xmodmaprc

Herausfinden von Keycodes
-------------------------

Zum herausfinden von Keycodes einer bestimmten Taste kann man entweder
die aktuelle Tastenbelegung ansehen und die gewünschte Taste
heraussuchen, oder man verwendet ``xev`` und drückt die Taste.

Modifier
--------

Einige Tasten sind als Modifier festgelegt. diese bekommt man mit
``xmodmap -pm`` angezeigt:

::

   $ xmodmap -pm
   xmodmap:  up to 4 keys per modifier, (keycodes in parentheses):
    
   shift       Shift_L (0x32),  Shift_R (0x3e)
   lock        Caps_Lock (0x42)
   control     Control_L (0x25),  Control_R (0x6d)
   mod1        Alt_L (0x40),  Alt_L (0x7d),  Meta_L (0x9c)
   mod2        Num_Lock (0x4d)
   mod3      
   mod4        Super_L (0x73),  Super_R (0x74),  Super_L (0x7f),  Hyper_L (0x80)
   mod5        Mode_switch (0x5d),  ISO_Level3_Shift (0x71),  ISO_Level3_Shift (0x7c) 

Will man die Taste für etwas anderes benutzen, sollte man den Modifier
wegnehmen. dies geschieht mittels

::

   clear <modifier>

Hat man sich vertan oder will man den Modifier auf eine andere Taste
legen, benutze man

::

   add <modifier> = <keysym>

Dies ist auch mit mehreren Tasten möglich.

Wie oben genannt geschieht dies entweder via ``-e``, Standardeingabe
oder in einer Datei.

Beispiel
--------

Hier eine Beispiel-.xmodmaprc-Datei (legt < > \| auf Caps-Lock):

::

   clear lock
   keycode 66 = less greater bar brokenbar bar brokenbar

US-Tastatur mit Umlauten und einigen Sonderzeichen
--------------------------------------------------

Manchmal kommt es vor, daß man keine deutsche Tastatur zur Verfügung
hat, beipielsweise im Ausland, und an allen möglichen fremden Rechnern
mit meist US-amerikanischer Tastatur arbeiten muß, dann ist es
hilfreich, eine eigene ``.Xmodmap`` zu erstellen. Ich beziehe mich hier
auf folgenden Artikel: `LinuxUser
11/2002:xmodmap <http://www.linux-user.de/ausgabe/2002/11/072-keyboard/index.html>`__,
der das sehr schön darstellt. Die ``.Xmodmap`` habe ich aber bei **é**
und **è** modifiziert und das gebräuchlichere auf <CAPS LOCK>+e gelegt,
da man doch weit öfter ein **é** benötigt.

::

   ! CAPS LOCK disable
   clear lock
   !the next line is normally not needed
   !add Mod3 = Mode_switch
   !keysym Caps_Lock = Mode_switch
   !!or
   keycode 66 = Mode_switch Multi_key
   keycode 117 = Mode_switch Multi_key
   ! now the key definitions, use xev to look up the keycode
   ! number if needed. The first 2 columns after the equal sign
   ! are the normal functions of the keys. The last two columns are
   ! used when Mode_switch is pressed or Mode_switch + Shift is
   ! pressed.
   keycode 39 = s S ssharp
   keycode 38 = a A adiaeresis Adiaeresis
   keycode 30 = u U udiaeresis Udiaeresis
   keycode 32 = o O odiaeresis Odiaeresis
   keycode 14 = 5 percent ssharp  degree
   keycode 26 = e E eacute Eacute
   keycode 28 = t T EuroSign EuroSign
   keycode 27 = r R ecircumflex Ecircumflex
   keycode 25 = w W egrave Egrave
   keycode 31 = i I idiaeresis Idiaeresis
   keycode 21 = equal plus plusminus notsign
   keycode 57 = n N ntilde Ntilde
   keycode 58 = m M Multi_key
   keycode 15 = 6 asciicircum dead_acute  dead_circumflex
   keycode 19 = 0 parenright degree masculine
   keycode 10 = 1 exclam exclamdown onehalf
   keycode 54 = c C ccedilla Ccedilla
   keycode 24 = q Q copyright registered 

Nach dem Abspeichern im Home-Verzeichnis des jeweiligen Users kann man
mit ``xmodmap .Xmodmap``, sofort testen, ob es funktioniert. Will man,
daß die neue Tastaturbelegung beim Einloggen automatisch gestartet wird,
kann man so vorgehen:

Zunächst muß man sicherstellen, daß das Kommando beim Einloggen auch
ausgeführt wird. Je nach Login-Prozedur sollte dies entweder in
``/etc/X11/xinit/xinitrc`` oder in ``/etc/X11/xdm/Xsession`` geschehen.
In diesen Dateien sollte sich etwa folgendes relativ weit am Anfang
finden:

::

   userresources=$HOME/.Xresources
   usermodmap=$HOME/.Xmodmap
   sysresources=/usr/X11R6/lib/X11/xinit/.Xresources
   sysmodmap=/usr/X11R6/lib/X11/xinit/.Xmodmap
   # merge in defaults and keymaps
   if [[x11|-f $sysresources ]]; then
      xrdb -merge $sysresources
   fi
   if [[x11|-f $sysmodmap ]]; then
      xmodmap $sysmodmap
   fi
   if [[x11|-f $userresources ]]; then
      xrdb -merge $userresources

* :ref:`genindex`

Zuletzt geändert: |date|

