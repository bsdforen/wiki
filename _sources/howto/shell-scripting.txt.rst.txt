Shell-Scripting
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Dieser Artikel soll einen Einstieg in das Schreiben von Shell-Skripten
geben. Die Beispiele sind oft FreeBSD-spezifisch, die beschriebenen
Techniken sind aber universell verwendbar und sollten durch die
Erklärungen nachvollziehbar sein. Wer den ganzen Artikel liest sollte am
Schluss in der Lage sein mächtige Skripte zu schreiben.

Bei vielen Beispielen beginnen die Zeilen mit einem ``$`` oder ``>``.
Das bedeutet dann, dass das Beispiel dafür gedacht ist, direkt in einer
Shell ausprobiert zu werden.

Skripte anlegen
---------------

Als erstes wird mit dem Editor der Wahl eine neue Datei angelegt. Die
erste Zeile enthält den Aufruf des Interpreters. Im Rahmen dieses
Artikels ist das immer ``/bin/sh``.

.. code:: bash

   #!/bin/sh

Hier können allerdings in der tat beliebige Interpreter verwendet
werden. Für ``awk`` sähe die erste Zeile folgendermaßen aus:

.. code:: bash

   #!/usr/bin/awk -f

Nach dem Anlegen der Datei muss sie noch als ausführbar markiert werden.

.. code:: bash

   $ chmod +x MYSCRIPT

Hier ist **MYSCRIPT** mit dem Dateinamen des gerade angelegten Skripts
zu ersetzen.

Variablen
---------

Variablen werden benötigt um Werte zwischenzuspeichern. Auch die
Übergabeparameter werden als Variablen übergeben. Es gibt verschiedene
Schreibweisen um auf Variablen zuzugreifen. Beim setzen einer Variable
geht aus der Syntax hervor, dass es sich um eine Variable handelt,
deshalb wird nur der Variablenname verwendet. Bei Shell-Skripten sind
Variablen immer Strings.

.. code:: bash

   i=0

.. code:: bash

   for i in 0 1 2 3 4 5 6 7 8 9; do ... done

Bei beiden Beispielen handelt es sich um eine Variablenzuweisung für die
Variable mit dem Namen ``i``.

Um diese Variable irgendwo einzufügen wird die Syntax ``${i}``
verwendet. Nur zwischen einfachen Anführungszeichen oder wenn vor dem
``$`` ein ``\`` ist, wird die Variable nicht ausgewertet. Häufig kann
die vereinfachte Syntax ``$i`` verwendet werden. Das funktioniert
natürlich nur, wenn die Syntax eindeutig ist. ``$itest`` würde natürlich
zu einer Variablen mit dem Namen ``itest`` aufgelöst, ``${i}test``
allerdings zu ``0test`` (vorrausgesetzt ``i=0``).

Zur Erzeugung formatierter Ausgaben wird oft die Länge einer Variable
benötigt, diese Länge lässt sich mit Hilfe des ``#``-Zeichens auslesen.

.. code:: bash

   $ test='This is some useless text.'
   $ echo ${#test}
   26

vordefinierte Variablen
~~~~~~~~~~~~~~~~~~~~~~~

Es gibt einige vordefinierte Variablen, die auch häufiger benötigt
werden.

-  ``$0`` enthält den Aufruf des Programms, im interaktiven Terminal ist
   das Normalerweise der Aufruf der Shell, in Skripten der Name des
   Skripts.
-  ``$1`` enthält den ersten Parameter, der dem Skript übergeben wurde.
   Die folgenden Parameter sind fortlaufend nummeriert.
-  ``$$`` enthält die ``PID`` (Process ID) des laufenden Skripts.
-  ``$?`` enthält das Fehlerbyte des letzten Kommandos. Die ``0``
   bedeutet ``true`` oder kein Fehler. Alles andere ist ein Fehlercode.
   An diese Konvention sollte sich jeder halten.
-  ``$@`` wird zu allen Parametern des Skripts expandiert. In doppelten
   Anführungszeichen werden die Parameter so übergeben, als seien sie
   alle in Anführungszeichen.
-  ``$!`` enthält die ``PID`` des zuletzt im Hintergrund (mit ``&``)
   gestarteten Prozesses.

Diese Liste ist nicht vollständig und enthält nur die *häufiger*
gebrauchten vordefinierten Variablen.

Variablen mit Standardwerten belegen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Um einer Variable einen Standardwert zuzuweisen, der nur dann Gültigkeit
hat, wenn kein anderer Wert vorhanden ist wird folgende Schreibweise
verwendet:

.. code:: bash

   ${variable=value}

Der Wert **value** der Variable kann dabei in die üblichen einfachen
oder doppelten Anführungszeichen gesetzt werden.

In dieser Form gibt es jedoch einen meist unerwünschten Nebeneffekt. Die
Variable wird an Ort und Stelle auch gleich ausgewertet und der
Interpreter ``/bin/sh`` interpretiert sie als Kommando. Das kann mit
folgender Syntax umgangen werden:

.. code:: bash

   : ${variable=value}

``:`` ist ein Kommando, das nichts tut. Die Variable wird dem Kommando
als Parameter übergeben und hat so keinen Effekt. Der Standardwert wird
aber wie erwünscht zugewiesen.

Ausgabe
-------

Shell-Kommandos haben oft eine Ausgabe, dieser Abschnitt beschäftigt
sich damit, wie eigene Ausgaben erzeugt oder Ausgaben von Kommandos in
Pipes weitergeleitet werden können.

echo
~~~~

Das ``echo`` Kommando gibt die übergebenen Parameter wieder aus.

.. code:: bash

   $ echo Hello $USER!
   Hello kamikaze!

Unter Umständen ist es nützlich Parameter in Anführungsstriche zu
setzen. Zum Beispiel um mehrere aufeinander folgende Leerzeichen
darzustellen.

.. code:: bash

   $ echo "Hello    $USER!"
   Hello   kamikaze!

Variablenersetzung findet mit einfachen Anführungszeichen nicht statt.

.. code:: bash

   $ echo 'Hello $USER!'
   Hello $USER!

printf
~~~~~~

Mit dem Kommando ``printf`` können formatierte Ausgaben erzeugt werden.
Das Kommando nimmt mindestens einen Parameter. Dieser ist ein einfacher
String, an den kein Zeilensprung angehängt wird. Dieser kann mit ``\n``
einkodiert werden. Auch ein Carriage Return ist mit ``\r`` möglich (der
Begriff stammt noch von den Schreibmaschinen). Damit wird der Cursor an
den Anfang der Zeile gesetzt.

Zusätzlich können formatierte Variablen in den Text eingefügt werden.
Dazu werden Platzhalter in den Text eingesetzt, die später mit den
folgenden Parametern des ``printf``-Aufrufs substituiert werden. Für
Zeichenfolgen wird der Platzhalter ``%s`` verwendet. Weitere
Möglichkeiten, vor allem zur Zahlendarstellung, sind in der Manpage
dokumentiert, wichtig sind vor allem ``%d`` für ganzzahlige Werte und
``%f`` für Gleitkommawerte. Der Clou an den Platzhaltern ist, dass man
sie mit Formatierungsinformationen ausstatten kann.

.. code:: bash

   $ printf 'name: %15s\nvorname: %12s\n' Hans Meise
   name:            Hans
   vorname:        Meise

Die Zahl zwischen den Zeichen ``%`` und ``s`` gibt die Mindestbreite des
Ausdrucks an. In diesem Fall werden die Fehlenden Zeichen von Links mit
Leerzeichen aufgefüllt. Bei der Wahl einer negativen Zahl werden die
fehlenden Zeichen von Rechts aufgefüllt.

.. code:: bash

   $ for file in $(ls); do printf '%-45sX\n' $file; done
   DragonForce-My_Spirit_Will_Go_On.mp3         X
   DragonForce-Through_the_Fire_and_Flames.mp3  X
   DragonForce-Valley_of_the_Damned.mp3         X
   machinae_supremacy-fury.ogg                  X
   machinae_supremacy-loot_burn_rape_kill_repeat.oggX
   machinae_supremacy-march_of_the_undead_2.ogg X
   machinae_supremacy-sidology_2-trinity.ogg    X

Dieses Beispiel verdeutlicht nicht nur das Auffüllen von Rechts, sondern
auch, dass überschüssige Zeichen nicht abgeschnitten werden. Dinge wie
die ``for``-Schleife werden später erklärt.

Befehlsersetzung
----------------

Befehle können an Ort und Stelle mit ihrer Ausgabe ersetzt werden. Zum
Ersetzen wird der Befehl mit dem Linksapostroph :literal:`\`` umgeben.
Alternativ kann der Befehl auch mit ``$(`` und ``)`` umgeben werden. Das
ist oft besser lesbar. Beides funktioniert auch in doppelten
Anführungszeichen ``"``.

.. code:: bash

   $ echo date
   date
   $ echo `date`
   Wed 27 Feb 2008 09:17:26 CET
   $ echo $(date)
   Wed 27 Feb 2008 09:17:26 CET
   $ echo "$(date)"
   Wed 27 Feb 2008 09:17:26 CET
   $ echo '$(date)'
   $(date)

Befehlsersetzung kann zwischen einfachen Anführungszeichen und in
Kommentaren nicht verwendet werden. Abgesehen davon kann sie beliebig
eingesetzt werden.

Im Gegensatz zur Ersetzung mit Linksapostroph :literal:`\`` kann die
Ersetzung mit Klammern ``$()`` auch verschachtelt werden.

Shell-Arithmetik
~~~~~~~~~~~~~~~~

Die Shell kann einfache Integer-Arithmetik durchführen. Diese ist
jedoch, auch unter 64 Bit Systemen auf 32 Bit beschränkt. Für die
meisten Shell aufgaben reicht das aber. Mathematische Ausdrücke werden
einfach zwischen ``$(``\ ``(`` und ``)``\ ``)`` geklammert. Das
funktioniert auch innerhalb von doppelten Anführungszeichen.

.. code:: bash

   $ a=1
   $ a=$((a + 1))
   $ echo $a
   2

Suffix- und Präfix-Entfernung
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Die Shell kann relativ simples Entfernen von Präfixen und Suffixen. Das
Ersetzungsausdruck funktioniert wie bei Dateinamen, es funktionieren
keine regulären Ausdrücke. Für die Ersetzung von Präfixen wird der
Operator ``#`` beziehungsweise ``##`` verwendet. Für Suffixe ``%`` oder
``%%``. Die einfache Schreibweise entfernt immer ein möglichst kleines
Prä-/Suffix, die doppelte Schreibweise ein möglichst großes. Das
folgende Beispiel verdeutlicht das.

.. code:: bash

   $ file='/usr/local/bin/firefox'
   $ echo "${file#*/}"
   usr/local/bin/firefox
   $ echo "${file##*/}"
   firefox

Pipes
-----

Viele Programme verwenden Daten die sie über die Standardeingabe
``/dev/stdin`` erhalten und machen ihre Ausgabe auf die Standardausgabe
``/dev/stdout`` und die Standardfehlerausgabe ``/dev/stderr``. Von
diesen hat jedes Programm seine eigenen, die über Pipes miteinander
Verbunden oder in Dateien umgeleitet werden können.

.. code:: bash

   $ dmesg | grep -i pci

Das Zeichen ``|`` ist eine gewöhnliche Pipe. Die Standardausgabe von
``dmesg`` wird in die Standardeingabe von grep umgeleitet.

.. code:: bash

   $ grep -E '(WW|EE)' < /var/log/Xorg.0.log

Mit dem Zeichen ``<`` wird die Standardeingabe auf eine Datei verwiesen.
Dieses Beispiel zeigt alle Fehler und Warnungen von Xorg. Natürlich wird
``grep`` nicht so verwendet, da es sowieso Dateien als Parameter nimmt.
Das Kommando:

.. code:: bash

   $ grep -E '(WW|EE)' /var/log/Xorg.0.log

erzeugt genau die gleiche Ausgabe.

.. code:: bash

   $ echo $$ > pid

Mit ``>`` wird die Standardausgabe umgeleitet. In diesem Beispiel in die
Datei ``pid``.

Nun kommt es vor, dass Programme Fehlerausgaben erzeugen.

.. code:: bash

   $ rm idonotexists 2> /dev/null

Mit ``2>`` leitet man die Standardfehlerausgabe in eine Datei um. In
diesem Fall wird die Ausgabe durch das Umleiten in ``/dev/null``
verworfen.

.. code:: bash

   $ ifconfig ipw0 | grep associated 2> /dev/null

Wie in diesem Beispiel zu sehen ist, können die verschiedenen
Umleitungen kombiniert werden.

.. code:: bash

   $ echo Fehler! 1>&2

Mit ``>&`` kann eine Ausgabe in die Andere umgeleitet werden. Mit
``2>&1`` wird die Fehlerausgabe in die Standardausgabe geleitet und mit
``1>&2`` die Standardausgabe in die Fehlerausgabe. Das kann dazu
verwendet werden um in eigenen Skripten, wie im Beispiel, die
Fehlerausgabe zu verwenden oder um Fehler in einer Pipe an andere
Programme weiterzugeben.

.. code:: bash

   $ pkg_info -agq > /dev/null 2>&1

Dieses Beispiel leitet sowohl Standardfehler, als auch Standardausgabe
nach ``/dev/null``.

.. code:: bash

   $ (pkg_info -agq > /dev/null) 2>&1

Dieses Beispiel ist schon deutlich sinnvoller. Es leitet die
Standardausgabe nach ``/dev/null``. Danach wird der Standardfehler in
die Standardausgabe umgeleitet. Ohne die Klammerung würde der
Standardfehler nicht in die Standardausgabe sondern in die Umleitung der
Standardausgabe (``/dev/null``) umgeleitet werden.

.. code:: bash

   $ (pkg_info -agq > /dev/null) 2>&1 | sed -E 's/^pkg_info: //1' | sed -E 's/ doesn.t exist$//1'

Wie dieses Beispiel zeigt kann das genutzt werden um die Fehlerausgabe
separat in einer Pipe weiterzuverarbeiten. Dieser Befehl gibt unter
FreeBSD alle von Paketen registrierten aber nicht existierenden Dateien
aus.

Die Umleitungsanweisungen können übrigens überall im Befehl stehen. Die
folgenden Befehle tun also alle das Gleiche.

.. code:: bash

   $ echo $$ > pid
   $ > pid echo $$
   $ echo > pid $$

Aus Gründen der Lesbarkeit gehören Umleitungen aber an das Ende der
Anweisung.

dynamische Parameter
--------------------

In einem Skript werden Daten verarbeitet. Dazu müssen Befehle mit
dynamisch erzeugten Parametern aufgerufen werden. Bei dynamisch erzeugte
Parametern kann es sich um Variablen oder die Ausgabe von Befehlen
handeln. Die hier erläuterten Möglichkeiten sind teilweise bereits in
vorherigen Beispielen aufgetaucht.

Es gibt Befehle, die ihre Parameter als solche entgegennehmen, Andere
hingegen Werten die Standardeingabe ``/dev/stdin`` aus und müssen
deshalb über eine Pipe mit Daten *gefüttert* werden. Viele kombinieren
beide Verfahren.

Befehle mit Parametern
~~~~~~~~~~~~~~~~~~~~~~

Befehle die Parameter verwenden können sehr einfach mit Variablen
dynamische Parameter zugewiesen werden.

.. code:: bash

   $ cpus=`sysctl -n hw.ncpu`

.. code:: bash

   $ echo There are $cpus CPU cores in this system.

Das ist äquivalent zu folgendem Befehl in dem auf das Zwischenspeichern
in einer Variable verzichtet wird.

.. code:: bash

   $ echo There are `sysctl -n hw.ncpu` CPU cores in this system.

Wenn Daten aus einer Pipe kommen können sie auch statt mit
Befehlsersetzung mit dem Kommando ``xargs`` übergeben werden.

.. code:: bash

   $ find /usr/src -name Makefile | xargs echo Makefile found:

Befehle die aus der Standardeingabe lesen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Viele Befehle wie ``grep`` oder ``sed`` lesen aus der Standardeingabe,
wenn ihnen keine Dateien genannt werden aus denen sie lesen sollen. Das
ist üblich bei Befehlen, die Daten Zeilenweise verarbeiten. Diese
Befehle werden dann über eine Pipe mit ihren Daten versorgt.

.. code:: bash

   $ dmesg | grep -i usb

Daten in Variablen müssen mit Hilfe von ``echo`` *injiziert* werden.

.. code:: bash

   $ cpus=`sysctl -n hw.ncpu`
   $ echo $cpus | sed -E 's/^(.*)$/There exist \1 cpu cores on this system/1'

Eingabe
-------

Die meisten Shell-Skripte funktionieren nur mit übergebenen Parametern,
aber auch interaktive Benutzereingaben sind möglich.

read
~~~~

Mit dem ``read``-Kommando wird eine Zeile eingelesen. Die einzelnen
Worte werden in eine Liste von Variablen gespeichert, überschüssige
Werte kommen in die letzte Variable.

.. code:: bash

   #!/bin/sh

   read -p 'Please enter: title name surname> ' title name surname
   echo "Title:        $title"
   echo "Name:     $name"
   echo "Surname:  $surname"

Das ganze sieht dann in der Ausführung so aus:

.. code:: bash

   Please enter: title name surname> Herr Dominic Fandrey
   Title:          Herr
   Name:           Dominic
   Surname:        Fandrey

Die Möglichkeiten sind jedoch begrenzt:

.. code:: bash

   Please enter: title name surname> Herr Dominic Christian Fandrey
   Title:          Herr
   Name:           Dominic
   Surname:        Christian Fandrey

Hier landet der zweite Vorname in der Variablen für den Nachnamen. Da
hilft nur alle Werte doch einzeln abzufragen.

head
~~~~

Beim Einlesen einer Zeile werden mehrere aufeinander folgende
Leerzeichen geschluckt. Will man eine unveränderte Zeile oder sogar
mehrere auf einmal einlesen, bietet sich das Kommando ``head`` an, das
ohne die Angabe einer Datei von ``/dev/stdin`` liest.

.. code:: bash

   #!/bin/sh

   echo "Please enter your delivery address (4 lines):"
   delivery=$(head -l4)

   echo "
   You entered the following delivery address:
   $delivery"

Beim Ausführen sieht es dann so aus:

.. code:: bash

   Please enter your delivery address (4 lines):
   Dominic Fandrey
   Mustergasse 13
   23666 Ödnis
   Germany

   You entered the following delivery address:
   Dominic Fandrey
   Mustergasse 13
   23666 Ödnis
   Germany

Leider endet mit dem Kommando ``head -c1`` nicht die Ausgabe nach dem
ersten Tastendruck. Jedoch wird in einer Variable tatsächlich nur das
erste Zeichen gespeichert.

Bedingte Anweisungen
--------------------

Ob und wie oft eine Anweisung ausgeführt wird steht nicht immer schon
beim Programmieren fest. Dafür gibt es bedingte Anweisungen wie ``if``
und Schleifen, die es erlauben Code nur unter bestimmten Bedingungen
auszuführen.

Befehlsketten
~~~~~~~~~~~~~

Anders als bei Pipes werden Kommandos bei Befehlsketten nicht
gleichzeitig (parallel) sondern nacheinander (seriell) ausgeführt. Der
Interpreter wartet also den Rückgabewert des Befehls aus und macht dann
abhängig von diesem und dem Verkettungsoperator weiter.

Es gibt drei verschiedene Verkettungsoperatoren:

-  ``;`` -- führt den nächsten Befehl unabhängig vom Rückgabewert des
   vorherigen Befehls aus.
-  ``&&`` -- führt den nächsten Befehl aus, wenn der Rückgabewert des
   vorherigen Befehls ``0`` (true) ist.
-  ``||`` -- führt den nächsten Befehl aus, wenn der Rückgabewert des
   vorherigen Befehls ungleich ``0`` (false) ist.

Hier ist ein kleines Beispiel, das immer *true* ausgeben wird:

.. code:: bash

   $ : && echo true || echo false

Wer sich an das `Kapitel über Variablen mit
Standardwerten <shell-scripting#Variablen mit Standardwerten belegen>`__
erinnert, weiß dass ``:`` ein Befehl ist, der nichts tut. Der Befehl
gibt immer ``0`` (true) zurück. ``/bin/sh`` bietet noch die Möglichkeit
mit dem Vorausstellen eines ``!`` den Rückgabewert zu invertieren. Die
``0`` wird dadurch zu ``1`` und alle anderen Rückgabewerte zu ``0``.

Der folgende Befehl wird also immer *false* ausgeben:

.. code:: bash

   $ ! : && echo true || echo false

Hier mal ein etwas sinnvolleres Beispiel, das prüft ob eine Datei
existiert und eine entsprechende Ausgabe erzeugt.

.. code:: bash

   $ test -e FILE && echo "Yes, the file exists." || echo "No, the file does not exist"

Da solche Zeilen oft lang und unleserlich werden sollten sie
gegebenenfalls mit einem ``\`` am Zeilenende über mehrere Zeilen
verteilt werden:

.. code:: bash

   $ test -e FILE \
   >   && echo "Yes, the file exists." \
   >   || echo "No, the file does not exist."

Je nachdem ob **FILE** existiert erscheint dann die entsprechende
Ausgabe.

Wie bei den Pipes kann auch hier mit Klammern geschachtelt werden:

.. code:: bash

   $ test -e FILE \
   >   && echo "Yes, the file exists." \
   >   || ( echo "No, the file does not exist."; touch FILE)

Im Fehlerfall werden in diesem Beispiel also gleich 2 Befehle
ausgeführt.

if, then, else
~~~~~~~~~~~~~~

Obwohl Befehlsketten in Verbindung mit Klammerung die gleiche
Mächtigkeit wie ``if``-Anweisungen haben, sind sie doch schnell ziemlich
unübersichtlich. Deshalb werden in Skripten ``if``-Anweisungen deutlich
häufiger verwendet.

Die häufigste Verwendung von ``if`` ist in Verbindung mit ``[``:

.. code:: bash

   if [ "$1" = "$2" ]; then
       echo "The parameters 1 and 2 are identical."
   else
       echo "The parameters 1 and 2 differ."   
   fi

Die ``else``-Anweisung ist natürlich optional. Bei ``[`` handelt es sich
um ein Synonym für das Kommando ``test``. Die Syntax kann also der
Manual-Page **test(1)** entnommen werden.

Das folgende Beispiel ist also äquivalent zum letzten Beispiel des
vorherigen Abschnitts:

.. code:: bash

   if [ -e "$1" ]; then
       echo "Yes, the file exists."
   else
       echo "No, the file does not exist."
       touch "$1"
   fi

Der Dateiname **FILE** wurde hier durch ``$1`` ersetzt. Er wird also als
erster Parameter zum Skript erwartet.

Auch ``if`` prüft nur den Rückgabewert eines Befehls. Statt ``[`` oder
``test`` kann jeder beliebige Befehl verwendet werden, auch eigene
Funktionen.

case
~~~~

Mit ``case`` können Variablen mit sogenannten Shell-Patterns gefiltert
werden. Shell-Pattern heißt, es gelten die gleichen Regeln wie bei
Dateinamen. Shell-Patterns können im ``case``-Statement mit ``|``
verodert werden.

.. code:: bash

   case "$1" in
       '')
           echo "No parameter given."
       ;;
       [0-9])
           echo "'$1' is a digit."
       ;;
       --* | -?)
           echo "'$1' is an unknown option."
           return 1
       ;;
       *)
           echo "'$1' is unknown."
       ;;
   esac

Schleifen
~~~~~~~~~

Oft müssen die gleichen Befehle wiederholt auf verschiedenen Datensätzen
angewendet werden. Dafür werden Schleifen verwendet.

for-Schleifen
^^^^^^^^^^^^^

Die ``for``-Schleife bei Shell-Skripten entspricht den aus anderen
Sprachen bekannten ``foreach``-Schleifen mit denen man durch Listen
iterieren kann. ``for`` betrachtet die übergebenen Parameter als
Elemente einer Liste. Normalerweise funktionieren Tabulatoren oder
Leerzeichen als Trennzeichen. Das kann mit der Variable ``IFS`` (Input
Field Separator) verändert werden. Es kann häufiger Vorkommen, dass man
Informationen Zeilenweise verarbeiten will. Das ist zum Beispiel
nützlich um auch mit Leerzeichen enthaltenden Dateinamen klarzukommen.

.. code:: bash

   #!/bin/sh

   # Use line breaks as field separators.
   IFS='
   '

   lastName=
   lastChksum=
   printed=
   for file in $(sha256 "$@" | sort -k5); do
       name="${file#*(}"
       name="${name%)*}"

       chksum="${file##*= }"

       if [ "$chksum" = "$lastChksum" ]; then
           if [ -z "$printed" ]; then
               echo "The following files have the same checksum ($chksum):"
               echo "$lastName"
               printed=1
           fi
           echo "$name"
       else
           printed=
       fi
   done

Dieses Skript listet Duplikate aus einer Liste von Dateien.

Der Ausdruck ``$(sha256 "$@" | sort -k5)`` gibt eine Liste von Dateien
mit Prüfsummen zurück. Der ``sort -k5`` Befehl sorgt dafür, dass die
Liste nach den Prüfsummen sortiert wird. Aus diesem Grund folgen
Duplikate immer aufeinander. Statt den Ausdruck in der Schleife in
geschweifte Klammern zu stellen, können auch die Schlüsselwörter ``do``
und ``done`` verwendet werden.

Eine ``for``-Schleife ohne Liste geht alle dem Skript übergebenen
Parameter durch:

.. code:: bash

   #!/bin/sh

   for parameter do
       echo "Parameter '$parameter' was given."
   done

Diese Syntax ist jedoch nicht portabel. Deshalb und für die Lesbarkeit
empfiehlt es sich explizit die Liste der Parameter zu übergeben:

.. code:: bash

   #!/bin/sh

   for parameter in "$@"; do
       echo "Parameter '$parameter' was given."
   done

while-Schleifen
^^^^^^^^^^^^^^^

Eine ``while``-Schleife läuft bis der folgende Programmaufruf einen
Fehler zurückgibt. Mit dem Zeichen ``!`` kann die Logik umgekehrt
werden.

.. code:: bash

   #!/bin/sh

   while ! ifconfig fxp0 | grep -q UP; do
       sleep 5
   done

   ifconfig fxp0 down
   $0 &

Dieses Beispielskript macht eine recht sinnlose Tätigkeit. Es überwacht
das Netzwerkinterface fxp0 und deaktiviert es, falls es aktiv ist. Wer
tatsächlich etwas dergleichen erreichen will, sollte das natürlich
lieber über Zugriffsrechte, devd oder einen Paketfilter regeln.

Schleifen mit Zähler
^^^^^^^^^^^^^^^^^^^^

Eine Schleife mit Zähler, die die Funktion wie eine ``for``-Schleife in
Programmiersprachen wie C erfüllt, kann mit einer ``while``-Schleife und
Shell-Arithmetik realisiert werden.

Das folgende Skript gibt den ersten übergebenen Parameter rückwärts aus.

.. code:: bash

   #!/bin/sh

   out=""
   i=$((${#1} - 1))
   while [ $i -ge 0 ]; do
       tail=$((${#1} - $i - 1))
       out="$out$(echo $1 | sed -E -e "s/^.{$i}(.).{$tail}\$/\1/1")"
       i=$(($i - 1))
   done
   echo "$out"

Die Schleifenlogik kann auf folgende Zeilen reduziert werden:

.. code:: bash

   i=$((${#1} - 1))
   while [ $i -ge 0 ]; do
       i=$(($i - 1))
   done

Die Zuweisung von $i und auch $i selbst muss nie in Anführungszeichen
gesetzt werden, weil wir wissen, dass wir auf jeden Fall mit gültigen
Zahlen arbeiten. Im Kapitel `Variablen <#Variablen>`__ wurde erklärt,
dass ``${#1}`` das gleiche ist wie die Länge von ``$1``. Selbst wenn
also kein Parameter übergeben wird, ist ``$i`` immer noch eine gültige
Zahl ``-1``.

Diese Schleife zählt rückwärts, Schleifen die vorwärts zählen sind
natürlich auch möglich:

.. code:: bash

   i=0
   while [ $i -lt 10 ]; do
       i=$(($i + 1))
   done

Diese Schleife zählt von 0 bis 9.

continue und break
^^^^^^^^^^^^^^^^^^

Mit den Befehlen ``continue`` und ``break`` kann der Schleifendurchlauf
beeinflusst werden. Der Befehl ``continue`` bricht den aktuellen
Durchlauf ab und beginnt mit dem nächsten Durchlauf. Der Befehl
``break`` bricht die Schleife komplett ab. Optional nehmen die Befehle
die Schleifentiefe auf der sie Operieren sollen als Parameter. Wenn also
zum Beispiel zwei ineinander verschachtelte Schleifen vorhanden sind und
sowohl die innere, wie auch die äußere Schleife beendet werden soll,
lautet der Befehl ``break 2``.

Funktionen
----------

Umfangreiche Shell-Skripte können in Funktionen unterteilt werden. In
der Verwendung sind Funktionen fast wie eigenständige Befehle, mit
eigenen Parametern.

Funktionen deklarieren
~~~~~~~~~~~~~~~~~~~~~~

Eine Funktionsdeklaration beginnt mit dem Namen der Funktion, gefolgt
von einem Klammerpaar. Die eigentlichen Befehle folgen innerhalb
geschweifter Klammern.

.. code:: bash

   function_name() {
       ...
   }

lokale Variablen
~~~~~~~~~~~~~~~~

Variablen in Shell-Skripten sind für gewöhnlich global. Mit dem Befehl
``local`` können sie jedoch lokal deklariert werden. Die Variable wird
innerhalb der Funktion erzeugt und nachdem die Funktion terminiert auch
wieder vernichtet. Eventuell bereits vorhandene Variablen, werden wenn
sie als lokal deklariert werden, kopiert. Innerhalb der Funktion wird
dann nur noch auf der Kopie gearbeitet. Änderungen sind Nach außen nicht
wirksam.

.. code:: bash

   #!/bin/sh

   output='live'

   live() {
       local output
       output='die'
   }

   ordie() {
       output=' or die'
   }

   live
   printf "$output"
   ordie
   echo "$output"

Das Kommando ``local`` kann mehrere Variablen verarbeiten.

Parameter
~~~~~~~~~

Funktionen haben ihre eigenen Parameter. Nur wenn sie ohne Parameter
aufgerufen werden, erben sie die Parameter des Skripts oder der
aufrufenden Funktion.

Rückgabewerte
~~~~~~~~~~~~~

Funktionen können mit den Befehlen ``return`` und ``exit`` beendet
werden. Beide Befehle erwarten einen Parameter, der den numerischen
Rückgabewert bestimmt. Dieser Wert kann im Bereich von ``0`` bis ``255``
liegen. Es ist eine Konvention für ``true`` oder *Alles in Ordnung* den
Wert ``0`` zu verwenden. Viele Befehle wie ``if`` oder ``while``
verlassen sich auf die Einhaltung dieser Konvention. Alle anderen Werte
werden als Fehlercode und somit als ``false`` aufgefasst.

Skripte die Dateien verarbeiten
-------------------------------

Beim erzeugen von Skripten gibt es einige Einschränkungen und Dinge die
zu beachten sind. Zuerst ist die Zahl der Parameter die ein Skript
annehmen kann begrenzt, damit auch die Zahl der Dateiname die dem Skript
übergeben werden können. Das ist dann kritisch wenn ein Skript sehr
viele Dateien verarbeiten soll, zum Beispiel um viele Dateien nach einem
bestimmten Schema umzubenennen.

Das zweite Problem ist, dass Dateinamen Leerzeichen enthalten können.

Der Input Field Seperator
~~~~~~~~~~~~~~~~~~~~~~~~~

Die einzige uneingeschränkt funktionierende Methode mit Leerzeichen in
Dateinamen umzugehen ist die Änderung des ``IFS`` (Input Field
Seperator). Der ``IFS`` ist eine Variable, die eine Liste aller Zeichen
enthält, die Parameter trennen. Das sind normalerweise Leerzeichen,
Tabulatoren und Zeilenumbrüche. Beim Verarbeiten von Dateien dürfen nur
noch Zeilenumbrüche zur Parametertrennung verwendet werden.

.. code:: bash

   #!/bin/sh

   IFS='
   '

Der einzige Nachteil ist, dass das Anhängen an eine Liste (zum Beispiel
um sie später mit ``for`` zu verarbeiten) etwas hässlicher wird.

.. code:: bash

   list="$list
   $newEntry"

Das ist besonders bei eingerücktem Code hässlich.

.. code:: bash

   while ...; do
       if [ ... ]; then
           list="$list
   $newEntry"
       fi
   done

Das lässt sich jedoch nicht vermeiden, da keine Leerzeichen hinzugefügt
werden dürfen, da sie nun ein Teil der Daten wären und nicht mehr beim
Verarbeiten verworfen werden.

Dateinamen aus der Standardeingabe
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Eine gute Methode die Beschränkung der Parameterzahl zu umgehen ist die
Dateinamen aus der Standardeingabe zu lesen. So kann das Skript dann
später mit dem ``find``-Kommando über eine Pipe mit den Dateinamen
versorgt werden. Dazu wird die Standardausgabe Zeilenweise eingelesen.

.. code:: bash

   #!/bin/sh

   IFS='
   '
   while read file; do
       ...
   done

Im ``while``-Block steht nun jeweils in der Variable ``file`` der
aktuelle Dateiname zur Verfügung. Das kann natürlich nicht bloß für
Dateinamen verwendet werden, sondern für alle zeilenweise arbeitenden
Skripte.

Im folgenden ein praktisches Beispiel aus der Skriptsammlung des Autors:

.. code:: bash

   #!/bin/sh

   IFS='
   '

   command="$(basename "$0" | sed -E 's/^sed//1')"

   while read file; do
       wait
       $command -v "$file" "$(echo "$file" | sed "$@")" &
   done

   wait

Die Zeile ``command="$(basename "$0" | sed -E 's/^sed//1')"`` erzeugt
ein Kommando aus dem Dateinamen des Skripts. Beim Autor heißt das Skript
``sedcp`` und es existiert ein Hardlink mit Namen ``sedmv``. Das
resultierende Kommando ist also jeweils ``cp`` und ``mv``. Das folgende
Beispiel verdeutlicht den Verwendungszweck.

::

   $ find downloads/ -type f | sedmv -E 's/$/.txt/1'
   downloads/cauliflower -> downloads/cauliflower.txt
   downloads/pirates -> downloads/pirates.txt
   downloads/marks -> downloads/marks.txt
   downloads/iCTF -> downloads/iCTF.txt

Das Skript setzt etwas Wissen aus dem Kaptiel `Prozesse
forken <shell-scripting#Prozesse forken>`__ voraus. Trotzdem folgt jetzt
schon mal eine kurze Erklärung. Damit das Skript schon mal den nächsten
Dateinamen einlesen kann, wird das eigentlich Kommando geforkt, das
heißt es wird parallel zum weiterlaufenden Skript ausgeführt.

Durch das ``wait`` am Schluss wird sichergestellt, dass das Skript erst
dann terminiert, wenn alle geforkten Prozesse terminiert sind. Das
stellt eine sauber Ausgabe und die sequentielle Verwendbarkeit sicher.
Das bedeutet so viel wie, wenn das Skript terminiert, ist es auch
tatsächlich fertig.

Das ``wait`` innerhalb der ``while``-Schleife kann weggelassen werden.
Das bürgt aber Risiken, denn dann kann es passieren, vor allem wenn
lange Dateien kopiert werden, dass das Skript sehr viele Prozesse auf
einmal forkt, die alle um die Ressource Dateisystem konkurrieren.
Außerdem kann es passieren, dass die maximale Anzahl von Prozessen
erreicht wird, oder noch schlimmer, der Speicher ausgeht. In letzterem
Fall werden Prozesse vom System abgeschossen. Da das System dafür die
größten Prozesse nimmt erwischt es also wichtige Dinge wie den X-Server
oder eine Datenbank. Die eigentlichen Übeltäter, bleiben verschont.
Deshalb sorgt das ``wait`` in der Schleife dafür, dass erst der nächste
Prozess geforkt wird, wenn der vorherige terminiert.

Skripte zusammenführen
----------------------

Mit dem Befehl ``.`` können andere Skripte innerhalb der aktuellen
Umgebung ausgeführt werden. Das ist nützlich um Funktionen und Variablen
aus diesen Skripten zu erhalten. Ein gutes Beispiel sind dafür die
``rc``-Skripte in NetBSD (die NetBSD ``rc``-Implementierung wird auch
von FreeBSD verwendet). Jedes Skript im Verzeichnis ``/etc/rc.d``
importiert mit dem Befehl das Skript ``/etc/rc.subr``. So stehen dessen
Funktionen zur Verfügung, die es erlauben sehr kurze und mächtige
Startskripte für Dienste zu schreiben.

.. code:: bash

   . /etc/rc.subr

Prozesse forken
---------------

UNIX und seine Derivate sind seit jeher Multitasking-Systeme. Das heißt
es können mehrere Prozesse parallel laufen. Beim Forken wird ein Prozess
komplett dupliziert und unterscheidet sich erst einmal lediglich durch
die **PID**, das ist die Process ID. Außerdem hat der neue Prozess keine
Standardeingabe.

In einem Shell-Skript kann ein Prozess mit Hilfe des ``&``-Zeichens
geforkt werden. Dazu wird ein Befehl mit ``&`` abgeschlossen. Die
interpretierende Shell wird kopiert und führt den Befehl aus. Sie
terminiert nachdem der Befehl ausgeführt ist. Der Originalprozess kann
die **PID** aus der Variable ``$!`` auslesen.

Der Effekt lässt sich gut an einem kleinen Beispiel auf der Shell
nachvollziehen:

::

   $ sleep 10; printf "time out" &

Die Befehlskette wird durch das '&' in den Hintergrund geforkt und die
Shell nimmt sofort wieder weitere Eingaben an. Nach 10 Sekunden wird die
Nachricht "time out" ausgegeben, wo immer der Cursor sich gerade
befindet.

auf geforkte Prozesse warten
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Um auf Prozesse zu warten steht der Befehl ``wait`` zur Verfügung. Ohne
Parameter wartet der Befehl auf alle geforkten Prozesse. Alls Parameter
nimmt der Befehl die **PIDs** der Prozesse auf die er warten soll.

Folgendes Skript ist wieder ein Beispiel aus der Sammlung des Autors.
Der Name des Skripts ist pingC. Das Skript pingt alle Adressen eines
Klasse C Netzes an.

.. code:: bash

   #!/bin/sh

   ip=0
   if ! echo "$1" | grep -E '^(([0-9]{1,3})\.){2}[0-9]{1,3}$' > /dev/null; then
       echo "Not a valid class C net. ###.###.### format expected."
       return 1
   fi

   do_ping() {
       if ping -c 1 -t 1 "$1.$2" > /dev/null 2>&1; then
           name=`route get "$1.$2" | awk '/route to:/ {print($3);}'`
           if [ "$name" != "$1.$2" ]; then
               echo "$1.$2 $name"
           else
               echo "$1.$2"
           fi
       fi
   }

   (
       while [ $ip -le 255 ]; do
           ip=$(($ip + 1))
           do_ping "$1" $ip &
       done

       wait
   ) | sort -gk 4 -t \.

Der ``if``-Ausdruck am Anfang prüft ob der Parameter im korrekten Format
übergeben wurde. Die Funktion ``do_ping()`` führt den eigentlichen Ping
und die Ausgabe durch. Der hier wirklich interessante Teil ist aber der
untere geklammerte Block.

Im unteren geklammerten Block wird die Funktion ``do_ping()`` 256 mal
geforkt. Dann wird darauf gewartet, dass alle Kommandos terminieren. Die
Klammerung ist notwendig um die Ausgabe aller Befehle zusammenzufassen,
damit der Befehl ``sort`` sie sortieren kann. Durch den Fork benötigt
das Skript beim Autor in einem kleinen Test ca. 2 statt 252 Sekunden.

Kommunikation über Signale
~~~~~~~~~~~~~~~~~~~~~~~~~~

Die simpelste Art der Prozesskommunikation ist über Signale. Signale
werden mit dem Kommando ``kill`` verschickt. Mit dem Kommando ``trap``
können Signale abgefangen werden. Beispielsweise können so noch letzte
Aufräumaktionen gestartet werden.

.. code:: bash

   trap "rm $pidfile" EXIT

**EXIT** ist ein Pseudosignal, dass auftritt wenn das Skript terminiert.
Eine Liste der verfügbaren Signale mit Erklärung gibt es in der
Manual-Page **signal(3)**. Erwähnenswert sind die Signale **USR1** und
**USR2**, die für eigene Verwendung vorgesehen sind.

Im folgenden eine Beispielzeile aus einem Skript des Autors:

.. code:: bash

   trap "echo 'wi: signal hup trapped'; (wi_start $* &) ; exit 0" hup

Hier wird ``SIGHUP`` abgefangen und zum Anlass genommen einen Daemon neu
zu starten.

nützliche Befehle
-----------------

Dies ist eine grobe Übersicht zum Nachschlagen. Nährere Beschreibungen
sind in den Manual-Pages zu finden.

awk
~~~

Awk ist eigentlich eine komplette (kleine) Skriptsprache, die besonders
dafür geeignet ist Daten Zeilen- und Spaltenweise zu verarbeiten. Statt
einem kompletten Skript reicht aber meist schon ein Einzeiler um die
gewünschte ausgabe zu erzeugen.

cat
~~~

Cat konkateniert Dateien. Natürlich kann es auch einzelne Dateien
ausgeben.

cut
~~~

Mit ``cut`` können bestimmte Spalten aus einer Ausgabe *geschnitten*
werden. Optional kann auch das Trennzeichen angegeben werden.

expr
~~~~

.. code:: bash

   $ expr 5 - 9 \* 7
   -58

Das Kommando ``expr`` kann einfache Integeroperationen ausführen. Es
versteht Zeichen wie "``+-*/()``". Zeichen die dabei von der Shell
interpretiert werden müssen natürlich ein ``\`` vorangestellt werden.
Dies ist eine häufige Fehlerquelle.

grep
~~~~

Das Kommando ``grep`` erlaubt es Daten zeilenweise nach regulären
Ausdrücken zu filtern.

.. _head-1:

head
~~~~

Mit dem Kommando ``head`` kann der Anfang einer Datei betrachtet werden.
Optional können auch feste Zeilen- oder Bytezahlen vorgegeben werden.

sed
~~~

Mit dem ``sed``-Kommando können Ersetzungen mit Hilfe regulärer
Ausdrücke durchgeführt werden.

sort
~~~~

Das Kommando ``sort`` erlaubt es Ausgaben zu sortieren. Optional kann
auch eine Spalte nach der sortiert wird angegeben werden. Auch ist es
möglich doppelt vorkommende Zeilen zu entfernen.

tail
~~~~

Das Kommando ``tail`` ist das Gegenstück zu ``head``. Es gibt das Ende
einer Datei aus.

test
~~~~

Das Kommando ``test`` kann verwendet werden um logische Ausdrücke
auszuwerten. Bekannter ist die Sonderform ``[`` in der es oft mit dem
Kommando ``if`` verwendet wird.

touch
~~~~~

Das Kommando ``touch`` stellt sicher, dass eine Datei existiert und
aktualisiert das Datum der letzten Änderung.

uniq
~~~~

Mit diesem Kommando können doppelte Zeilen aus einer bereits
vorsortierten Ausgabe gefiltert werden.

Verweise
--------

-  Die Manpage **sh(1)**.
-  Die Manpage **test(1)**.
-  Die Manpage **xargs(1)**.

* :ref:`genindex`

Zuletzt geändert: |date|

