Unicode auf der Konsole
=======================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel beschreibt wie `Unicode </kompendium/Unicode>`__-Unterstützung
mittels VESA-Treiber und jfbterm unter `FreeBSD </FreeBSD>`__ umgesetzt werden
kann.

Verwandte Artikel
-----------------

Der folgende Artikel wird vorausgesetzt.

-  `Auflösung der Konsole ändern <Auflösung der Konsole ändern>`__
-  `UNICODE <UNICODE>`__

Insbesondere ist ist auf eine korrekt gesetzte ``locale`` zu achten!

Benötigte Programme
-------------------

Aus den FreeBSD Ports ist lediglich
`sysutils/jfbterm <https://www.google.com/search?q=sysutils/jfbterm&btnI=lucky>`__
zu installieren. Jfbterm nutzt den VESA-Modus um verschiedene
internationale Kodierungen darzustellen. Insbesondere **UTF-8**.

Jfbterm Einrichten
------------------

Benutzer, die ``jfbterm`` verwenden wollen, sollten ihre ``~/.termcap``
Datei erweitern:

::

   > cat /usr/local/share/jfbterm/termcap.jfbterm >> ~/.termcap

Später kann dann in ``/usr/local/etc`` noch eine ``jfbterm.conf``
angelegt werden um die Konfiguration von ``jfbterm`` zu verändern. Als
Ausgangspunkt bietet sich dafür die Datei
``/usr/local/etc/jfbterm.conf.sample`` an.

Um ``jfbterm`` nun zu testen reicht es in der Regel auf eine Konsole zu
gehen, sich dort einzuloggen und ``jfbterm`` aufzurufen. Ab dem Moment
sind internationale Schriftzeichen auf der Konsole darstellbar. Um das
zu Testen kann man sich einfach mal die Namen einiger Länder von
`wikipedia.org <http://wikipedia.org>`__ in eine Datei kopieren und
diese mit ``cat <Dateiname>`` dann auf der ``jfbterm``-Konsole ausgeben.

Um ``jfbterm`` mit jedem Benutzer verwenden zu können, muss die Datei
``/usr/share/misc/termcap`` erweitert werden:

::

   # cat /usr/local/share/jfbterm/termcap.jfbterm >> /usr/share/misc/termcap
   # cap_mkdb /usr/share/misc/termcap

Nach dem Vornehmen dieser Änderung, kann der entsprechende Abschnitt in
der Datei ``~/.termcap`` entfernt werden.

Probleme
--------

-  Leider erkennen viele Programme die neuen Möglichkeiten nicht und
   filtern eigentlich darstellbare Zeichen, zum Beispiel der Textbrowser
   ``links``
-  Der *schöne* Mauszeiger der FreeBSD Konsole funktioniert nicht, es
   gibt lediglich ein langweiliges Rechteck
-  Die Eingabe von Umlauten und ähnlichen Zeichen ist extrem
   problematisch, sie geht entweder gar nicht oder nur in bestimmten
   Programmen
-  Copy'n'Paste (mit der Maus) funktioniert nur innerhalb einer
   ``jfbterm`` Instanz

Jfbterm und TCSH
----------------

Wer das will, kann jfbterm mit seinem Terminal automatisch starten
lassen. Wer zum Beispiel die ``csh`` oder ``tcsh`` verwendet, kann
einfach folgenden Code in seine ``~/.cshrc`` einfügen:

::

   # Start jfbterm on the console.
   switch ($TERM)
   case cons25:
       exec jfbterm
       breaksw
   endsw

<note warning>Dergleichen sollte nicht mit dem ``root``-Konto
eingerichtet werden, da es dann bei Konfigurations- oder VESA-Problemen
nicht mehr möglich ist, sich auf der Konsole einzuloggen.</note>

Hinweise zu bestimmten Programmen
---------------------------------

In diesem Abschnitt werden Hinweise zu bestimmten Konsolen-Programmen
aufgezeigt um die Unicode-Darstellung zu optimieren.

links
~~~~~

Der Textbrowser links *glaubt* leider nicht an Unicode auf der Konsole,
zumindest lokalisierte Zeichensätze sind aber möglich. Dazu müssen
direkt in ``links`` einige Einstellungen vorgenommen werden. Diese
Einstellungen werden übrigens für jeden Terminal-Typ separat
abgespeichert, so dass diese Einstellungen dann keinen Einfluss auf
links in ``xterm`` oder ``rxvt`` hat. Das folgende Beispiel ist für die
Darstellung westeuropäischer Zeichen.

#. Setup -> Character set -> ISO 8859-15 (Western European)
#. Setup -> Terminal options -> Color
#. Setup -> Terminal options -> UTF-8 I/O
#. Setup -> Save options

Verweise
--------

-  `sysutils/jfbterm <https://www.google.com/search?q=sysutils/jfbterm&btnI=lucky>`__
   in den FreeBSD Ports
-  http://www.ac.auone-net.jp/~baba/jfbterm/ - jfbterm for Freebsd
-  **jfbterm(1)**, **jfbterm.conf(5)**

* :ref:`genindex`

Zuletzt geändert: |date|

