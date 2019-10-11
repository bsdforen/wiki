csup
====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Mit csup kann der Inhalt von CVS Repositories heruntergeladen werden. Csup geht
dabei inkrementell vor, das heißt, es lädt nur die Dateien, die sich vom Inhalt
des Repositories unterscheiden. Der Haupteinsatzzweck von csup ist die FreeBSD
Quelltexte herunterzuladen. Hierbei dient es als Ersatz für *cvsup*. Der
Vorteil von csup ist, dass es in C geschrieben und Teil des Basissystems ist.
Für CVSup wurde ein eigener Compiler benötigt.

Supfiles
--------

Die Informationen, welches Repository, was und von wann aus diesem
Repository csup herunterladen soll, erhält es aus einem Supfile. FreeBSD
kommt mit einigen fertigen Supfiles, die in
**/usr/share/examples/cvsup** liegen. Es empfiehlt sich, die gewünschten
Dateien in ein eigenes Verzeichnis unter **/etc** zu kopieren.

::

   # mkdir -p /etc/csup
   # cd /usr/share/examples/cvsup
   # cp stable-supfile ports-supfile doc-supfile /etc/csup
   # cd /etc/csup

In diesem Beispiel wurde als Verzeichnis **/etc/csup** gewählt. Der
Autor verwendet dieses Verzeichnis und nicht etwa
**/usr/local/etc/csup**, da csup Teil des Basissystems ist.

Server auswählen
----------------

In die Supfiles muss nun noch der FreeBSD Mirror, von dem das Repository
geladen werden soll, eingetragen werden. Alternativ kann der Server csup
auch später über die Kommandozeile übergeben werden. Auf jeden Fall
sollte vorher ein geeigneter Server ermittelt werden. Die wahrscheinlich
beste Variante das zu erledigen ist mit
`fastest_cvsup <fastest_cvsup>`__. Dieses findet sich in den Ports unter
**sysutils/fastest_cvsup**.

Jetzt kann mit:

::

   # fastest_cvsup -c de

der Server mit der schnellsten Antwortzeit in Deutschland ermittelt
werden. Wer in der Nähe einer Grenze wohnt, kann auch mehrere Länder
angeben:

::

   # fastest_cvsup -c de,fr

Am Ende der Ausgabe werden die besten drei Server aufgelistet. Das sieht
von Karlsruhe etwa so aus:

::

   >>  Speed Daemons:
       - 1st: cvsup8.fr.freebsd.org
       - 2nd: cvsup.de.freebsd.org
       - 3rd: cvsup4.de.freebsd.org

Für gewöhnlich lohnt es sich, den Befehl mehrmals aufzurufen und dann
einen Server zu nehmen, der oft in der Liste der besten Drei erscheint.

Der gewählte Server kann nun in die Supfiles in **/etc/csup**
eingetragen werden. Alternativ kann das auch später in der
**/etc/make.conf** getan werden.

Supfiles Bearbeiten
-------------------

Am Interessantesten ist für die Meisten wohl der **stable-supfile**.
Angepasst werden sollte zuerst die Zeile:

::

   *default host=CHANGE_THIS.FreeBSD.org

CHANGE_THIS.FreeBSD.org wird dann mit dem zuvor ermittelten Server, beim
Autor cvsup8.fr.freebsd.org, ersetzt.

Wer nicht FreeBSD Stable, sondern einen Security Branch (Release mit
kritischen Updates) verwenden will, kann die Zeile:

::

   *default release=cvs tag=RELENG_6

ändern. Mit dem Ersetzen von RELENG_6 mit RELENG_6_2 können nun
zukünftig die Quellen für FreeBSD 6.2 mit sicherheitskritischen Updates,
statt denen von FreeBSD 6-Stable geladen werden. Ansonsten gibt es in
der Datei nichts was geändert werden sollte.

Supfile Testen
--------------

Um zu prüfen, ob der Supfile funktioniert, reicht ein einfacher Aufruf:

::

   # csup /etc/csup/stable-supfile

Der Aufruf sollte das Verzeichnis **/usr/src** anlegen oder
aktualisieren.

make update
-----------

Wenn **/usr/src** bereits vorhanden ist, ist es bequemer, die Quellen
einfach mit

::

   # cd /usr/src
   # make update

auf den neusten Stand zu bringen. Dazu müssen ein paar Einträge in die
**/etc/make.conf** gemacht werden:

::

   SUP_UPDATE=     yes
   SUP=            /usr/bin/csup
   SUPFLAGS=       -L 2 -z
   SUPHOST=        cvsup8.fr.freebsd.org
   SUPFILE=        /etc/csup/stable-supfile
   #DOCSUPFILE=        /etc/csup/doc-supfile

Die **SUPFLAGS** sind nicht wirklich notwendig. Der Parameter **-L 2**
sorgt für eine detailiertere Ausgabe, der Parameter **-z** erzwingt eine
komprimierte Datenübertragung, selbst wenn die Kompression im Supfile
deaktiviert ist.

Wer **DOCSUPFILE** (im Beispiel auskommentiert) einträgt kann mit make
udpate auch die Quellen der neuesten FreeBSD Dokumentation
herunterladen.

Um auch den Ports Tree auf den neusten Stand zu bringen sind noch die
folgenden zwei Zeilen notwendig:

::

   PORTSSUPFILE=       /etc/csup/ports-supfile

In den meisten Anwendungsfällen ist aber für diesen Zweck
`portsnap <portsnap>`__ das bessere Mittel.

Verweise
--------

-  Die **csup(1)** Manpage
-  Die **make.conf(5)** Manpage
-  Der `portsnap <portsnap>`__ Artikel

* :ref:`genindex`

Zuletzt geändert: |date|

