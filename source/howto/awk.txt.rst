AWK
===

.. |date| date::

Dies ist eine Ergänzung zu `shell-scripting <shell-scripting>`__ in der
es um die Sprache AWK geht.

Es wird eine allgemeine Schnelleinführung geben und ein Sammlung von
nützlichen awk-Schnipseln, die awk erläutern und auch so für den
täglichen BSD-Einsatz nützlich sind.

AWK-Basics
----------

AWK ist eine Skriptsprache die als einfacher Vorgänger von Perl gilt.
AWK zerlegt die Eingabedaten, ``/dev/stdin`` oder eine Datei, in so
genannte *records* (normalerweise eine Zeile) und diese *records* in
*fields* (eine Spalte, normalerweise getrennt von Leerzeichen oder
Tabulatoren). Das Skript selbst wird awk entweder als Parameter
übergeben oder kann in einer Datei liegen, die mit dem Parameter ``-f``
aufgerufen wird.

Diese Eingabedaten können dann mit mehreren Blöcken Code verarbeitet
werden. Jedem Block kann optional ein Regulärer Ausdruck vorangestellt
werden, mit dem gesteuert wird ob der aktuelle *record* von dem Block
bearbeitet wird oder nicht.

Das folgende Beispiel übergibt das Skript als Parameter. Dem Block ist
in ``/`` eingeschlossen ein regulärer Ausdruck vorangestellt, der
Groß-/Kleinschreibung ignoriert (das ``i`` hinter dem abschließenden
``/``).

::

   > dmesg | awk '/intel/i {if ($1 ~ ":$" ) {print $1;}}'

Zusätzlich gibt es die Möglichkeit einen **BEGIN**-Block und einen
**END**-Block anzulegen. Die Blöcke werden dann unabhängig von den
Eingabedaten jeweils vor und nach der Verarbeitung der Eingabedaten
aufgerufen.

Alle Variablen in AWK sind global. Jedoch können Funktionen angelegt
werden, die mit Werte-Übergabe funktionieren. Diesen Umstand kann man
ausnutzen um Wrapper-Funktionen um die Grundfunktionen von AWK zu
schreiben, die nicht destruktiv für die übergebenen Parameter sind.

AWK stört sich nicht daran, wenn einer Funktion weniger Parameter als
definiert übergeben werden. Deshalb kann man über den Umweg der
Funktionsparameter lokale Variable erzeugen.

nützliche Minibeispiele
-----------------------

Alle Pakete auflisten, die von keinem anderen Paket benötigt werden
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

   pkg_info -aQR | awk '$0 ~ ":" { split($1,arr,":"); if (arr[2]=="") print arr[1];}'

Alle installierten Pakete mit Origin auflisten
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

   find /var/db/pkg/ -name "+CONTENTS" | xargs awk '$1=="@comment" && substr($2,1,6)=="ORIGIN" { split(FILENAME,a,"/"); print a[5] ":" substr($2,8) }'

das macht dasselbe wie

.. code:: bash

   pkg_info -aQo

(und ist auch nicht wirklich schneller) TODO ergänzen

Duplikate ohne Sortieren entfernen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

   awk '!occured[$0]++'

Es kann auch nach bestimmten Spalten gefiltert und ein
Spaltentrennzeichen angeben werden (im folgenden Beispiel die erste
Spalte und das Trennzeichen ":").

.. code:: bash

   awk -F: '!occured[$1]++'

Nur mehrfach vorkommende Zeilen ausgeben
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Der folgende Aufruf gibt nur Zeilen aus, die schon einmal aufgetaucht
sind:

.. code:: bash

   awk 'occured[$0]++'

Der nächste Aufruf gibt mehrfach vorkommende Zeilen genau ein mal aus,
egal wie oft sie vorkommen:

.. code:: bash

   awk 'occured[$0]++ == 1'

Pakete die Dateien in einen bestimmten Pfad installieren
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Listet alle Pakete die Dateien in denen im Pfad ``PFAD`` vorkommt
installiert haben.

.. code:: bash

   pkg_info -La | awk '/^Information for/ {pkg=$3;sub(":","",pkg)} pkg && /PFAD/ {print pkg;pkg=0}'

Das ganze geht auch wenn man den Pfad als Parameter übergeben will.

.. code:: bash

   pkg_info -La | awk 'BEGIN{path=ARGV[1];delete ARGV[1]} /^Information for/ {pkg=$3;sub(":","",pkg)} pkg && $0 ~ path {print pkg;pkg=0}' PFAD

nützliche AWK-Skripte
---------------------

Dieses Skript rückt XML-Dateien automatisch ein.

.. code:: perl

   #!/usr/bin/awk -f
   #
   # Copyright (c) 2010
   # Dominic Fandrey <kamikaze@bsdforen.de>
   #
   # Do as you please, don't blame me for the results.
   #
    
   #
   # This script indents XML files.
   #
    
   {
       # Comments are not indented.
       if (!comment) {
           # Trim line
           sub("^[[:space:]]*", "");
    
           # Indent depth depending on whether this is a closing line or not
           i = 0;
           if ($0 ~ "^</")
           i++;
    
           # Do indent
           for (; i < depth; i++)
               $0 = "  " $0;
       }
       # Print
       print($0);

       # Deal with closing comments.
       if (comment and $0 ~ "->") {
           comment = 0;

           # Throw everything up to the first closing comment away.
           sub("->", "89ureioqfhkafhka382rafsdk");
           sub(".*89ureioqfhkafhka382rafsdk", "");
       }
    
       # Don't change indention within comments.
       if (!comment) {
           # Throw away tags that do not count, i.e. tags that start with <?,
           # self-closing tags, e.g. <tag/>, and comments.
           gsub("<\\?[^>]*>|<[^>]*/>|<!-.*->", "");
           if ($0 ~ "<!-") {
               sub("<!-.*", "");
               comment = 1;
           }
    
           # Indent depth for the following lines
           depth -= gsub("</[^>]*>|/>", "");
           depth += gsub("<[^>]*>|<(![^-]|[^!])", "");
       }
   }

Verweise
--------

-  Die Manpage **awk(1)**.
-  http://www-e.uni-magdeburg.de/urzs/awk/
-  `AngrySwarm (Stichwort
   AWK) <http://angryswarm.blogspot.de/search/label/awk>`__ - Blog mit
   hoher AWK-Dichte

* :ref:`genindex`

Zuletzt geändert: |date| 

