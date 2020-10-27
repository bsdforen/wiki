ZSH
===

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Die Z-Shell ist eine der mächtigsten UNIX Shells überhaupt. Es mag deshalb auf
den ersten Blick schwierig erscheinen, sie zu verstehen bzw. ihr Potential
auszuschöpfen, da sie einen mit Features regelrecht erschlägt.

Minimal Konfiguration
---------------------

Nachdem die Zsh installiert wurde, kann man in die **~/.zshrc** seine
Konfiguration eintragen. Ein einfaches und minimalistisches Beispiel
sieht zum Beispiel so aus:

::

   export PATH="/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:/usr/X11R6/bin"
   export EDITOR=vim
   export PAGER=less
   export LC_CTYPE="de_DE.ISO8859-1"

   alias ls='ls -A'

Bis jetzt ist noch kein großen Unterschied zu anderen Shells erkennbar,
deshalb kommen wir nun zu den Dingen, welche die Zsh so besonders
machen!

Besonderheiten
--------------

Prompt
~~~~~~

Ein Standardprompt:

::

   export PROMPT="%n@%m:[%~]%# "
   foo@bar:[/etc]%

Wenn das Arbeitsverzeichnis länger wird, empfiehlt sich solch ein
Prompt:

::

   export PROMPT="%n@%m:[%(4c.%1c.%~)]%# "

Dadurch weiß die Zsh, dass, wenn das aktuelle Verzeichnes mehr als 3
Elemente hat, nur noch der "Basisname" des Verzeichnisses im Prompt
angezeigt werden soll.

::

   foo@bar:[~]% cd /etc
   foo@bar:[/etc]% cd /usr/local/share
   foo@bar:[/usr/local/share] cd doc
   foo@bar:[doc]

Eine Ergänzung zu $PROMPT ist $RPROMPT. $RPROMPT wird ganz rechts am
Rand des Bildschirms angezeigt:

::

   export $RPROMPT=""%T"
   foo@bar:[~] echo foo                                         13:37
   foo
   foo@bar:[~]                                                  13:38

Folgender $RPROMPT zeigt einen traurigen Smiley, falls der letzte Befehl
fehlschlug:

::

   export RPROMPT="%(0?..:()"
   foo@bar:[~] ls /root
   ls: root: Permission denied
   foo@bar:[~]                                                  :(

Vervollständigung
~~~~~~~~~~~~~~~~~

Um bei einem Aufruf von xpdf nur .pdf Dateien angezeit zu bekommen, wird
**compctl** verwendet:

::

   $ xpdf <TAB>
   foo.txt bar.gif bla.pdf
   $ compctl -g '*.pdf' xpdf
   $ xpdf <TAB>
   $ xpdf bla.pdf

Ungemein praktisch ist *zstyle*, damit wird z.B. bei einem <TAB> nach
dem Befehl *kill* die zu eine Liste der Prozesse angezeigt. Dazu muss
nur folgendes in die .zshrc eingetragen werden (oder auf der
Befehlszeile eingegeben werden):

::

   autoload compinit
   compinit

Und so sieht das dann aus:

::

   $ zstyle ':completion:*:*:kill:*:jobs' verbose yes
   $ kill<TAB>
   [...]
   18555 p1  Is+     0:01.18 /usr/local/bin/zsh                                                                   
   21590 C0  I+      0:00.02 xinit /home/foobar/.xinitrc --              
   24025 p3  I+      0:00.01 /usr/local/bin/zsh
   32055 p3  Is+     0:00.92 /usr/local/bin/zsh
   [...]

Globbing
~~~~~~~~

Globbing generiert Dateinamen (filename generation) aus bestimmten
Operatoren.

Das bekannteste Beispiel für Globbing ist wohl der \*-Operator:

::

   $ cat *<TAB>
   $ cat foo.txt bar.txt bla.pdf bar.jpeg

bzw.

::

   $ vi *.txt<TAB>
   $ vi foo.txt bar.txt

Mit der Zsh ist aber noch viel umfangreicheres und genauers Globbing mit
Hilfe von sogenannten "Glob Qualifiers" möglich. Für Globbing muss in
der .zshrc die Option *glob* und evtl. auch *extendedglob* mittels
*setopt* aktiviert werden.

Hier zwei Beispiele:

\* Liste nur Verzeichnisse auf (wobei / für den Verzeichnis Glob
Qualifier steht):

::

   $ ls *(-/)

\* Liste alle Dateien, die keine spezielle Dateiendung haben, auf
(benötigt *extendedglob*):

::

   $ printf '%s\n' ^?*.*

strcat hat in der Zsh Kategorie seines Blogs noch viel mehr solcher
Beispiele.

Links
-----

-  `Zsh Homepage <http://www.zsh.org/>`__
-  `Zsh-Wiki <http://zshwiki.org/>`__ (Howtos, FAQs und .zshrc's)
-  `Die Zsh als interaktive
   Shell <http://cssun.rrze.uni-erlangen.de/~sipakale/zshreferat.html>`__
   [de]
-  `Die
   Zsh-Liebhaber-Seite <http://www.michael-prokop.at/computer/tools_zsh_liebhaber.html>`__
-  `strcat's Zsh
   Blog <http://strcat.de/blog/index.php?/categories/9-Zsh>`__ (`RSS
   Feed <http://strcat.de/blog/index.php?/feeds/categories/9-Zsh.rss>`__)
   und `strcat's Zsh Konfiguration <http://strcat.de/dotfiles/#zsh>`__

* :ref:`genindex`

Zuletzt geändert: |date|

