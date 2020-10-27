make.conf optimieren
====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel ist für experimentelle Naturen gedacht, die gerne versuchen Code
mit verschiedenen Compilern und Flags zu kompilieren. Dazu wird erklärt, was
mit der Datei **/etc/make.conf** alles beeinflusst werden kann und wie
for-Schleifen und einfache if-Abfragen gestaltet werden, um wiederkehrendes zu
automatisieren.  Einiges lässt sich hier wahrscheinlich auch auf andere BSDs
übertragen.

Die Datei **/etc/make.conf** wird immer dann ausgelesen, wenn der Befehl
**make** aufgerufen wird, also beim Bauen des
`Kernels </kategorie/Kernel>`__, der `Welt </howto/Make World>`__ und
auch beim Bauen von `Ports </howto/paketsysteme#freebsd-ports>`__. In
FreeBSD gibt es feste Vorgaben zur Gestaltung von Makefiles für Ports,
so dass das Verhalten recht einheitlich ist. Manchmal kommt es jedoch zu
Ausnahmen. Auch dazu gibt es Beispiele, die klären sollen wie Probleme
erkannt und gelöst werden.

Verwandte Artikel
-----------------

Es gibt einige verwandte Artikel die sich auch teilweise mit der
**/etc/make.conf** beschäftigen und für Leser dieses Artikels
wahrscheinlich ebenfalls interessant sind.

-  `csup </anwendungen/csup>`__
-  `bsdadminscripts </anwendungen/bsdadminscripts>`__
-  `kernel erstellen </freebsd/kernel erstellen>`__

make Syntax
-----------

Üblicherweise werden in der Datei **/etc/make.conf** nur Variablen
gesetzt, jedoch wird sie wie jedes andere Makefile interpretiert. Das
hat zur Folge, dass Schleifen und if-Abfragen verwendet oder Nachrichten
auf die Konsole ausgeben werden können.

Kommentare
~~~~~~~~~~

Kommentare sind ein sinnvolles Mittel, um große Dateien lesbarer zu
gestalten und zu strukturieren.

Kommentare werden erzeugt indem eine Zeile mit einer Raute (#) begonnen
wird:

::

   # Diese Zeile wird von make ignoriert. ;-)

Variablen
~~~~~~~~~

Zum Bearbeiten der Datei **/etc/make.conf** ist es wichtig zu wissen,
wie Variablen gesetzt oder verändert werden.

Variablen werden nach dem Schema:

::

   VARIABLE= value

gesetzt. Der Name der Variablen muss dabei mit dem ersten Zeichen der
Zeile beginnen. Leerzeichen und Tabulatoren vor und hinter dem Wert
(value) werden von make entfernt. Wenn Leerzeichen im Wert einer
Variable erwünscht sind, muss der Wert in Anführungszeichen (") gesetzt
werden.

Variablen können auch Listen von Werten enthalten, die beispielsweise in
einer for-Schleife ausgewertet werden kann. Zulässige Trennzeichen sind
Tabulatoren und Leerzeichen. Da lange Wertelisten leicht unübersichtlich
werden, kann durch Einfügen von Zeilensprüngen mit einem vorhergehenden
Backslash (\) die Lesbarkeit deutlich erhöht werden. Es ist wichtig,
dass sich hinter dem Backslash außer dem Zeilensprung keine weiteren
Zeichen befinden. Hier ein schematisches Beispiel:

::

   VARIABLE=  foo \
              bar \
              value3

Um Variablen zu verwenden werden sie mit runden oder spitzen Klammern
umschlossen und ein Dollar-Symbol vorangestellt. Hier ein Beispiel das
den Wert der Variablen **CPUTYPE** und **CFLAGS** ausgibt:

::

   .warning CPUTYPE=${CPUTYPE} CFLAGS=$(CFLAGS)

Wer Variablen nur dann einen Wert zuweisen will, falls sie noch nicht
gesetzt sind kann das mit:

::

   VARIABLE?=  value

tun. Um an eine Liste weitere Werte anzuhängen wird der <tt>+=</tt>
Operator verwendet:

::

   VARIABLE+=  value4 \
               value5

Um eine Variable zu löschen, wird folgender Befehl verwendet:

::

   .undef VARIABLE

Wer einer Variablen das Ergebnis eines Shell Aufrufs zuweisen will macht
das folgendermaßen:

::

   VARIABLE!=  COMMAND

Ein solcher Aufruf sollte allerdings wohl überlegt sein. Beispielsweise
wird beim Autor auf dem Notebook **/usr/src** vom Automounter gemountet,
was nicht immer verfügbar ist. Der '!=' Operator wird in diesem Fall
verwendet um das physische Verzeichnis **/usr/src** zu finden.

::

   SOURCEDIR!=  (cd /usr/src 2> /dev/null && pwd -P; exit 0)

Da **/usr/src** nicht immer vorhanden ist, werden Fehlermeldungen des
'cd'-Befehls unterdrückt und 'pwd' nur aufgerufen, falls 'cd'
erfolgreich war. 'exit 0' stellt sicher das 'make' nicht abbricht, wenn
hier ein Fehler auftritt.

Ausgabe
~~~~~~~

In der Datei **/etc/make.conf** machen Ausgaben normalerweise nicht viel
Sinn, jedoch gibt es Fälle in denen sie helfen herauszufinden warum die
gesetzten Variablen keinen oder nicht den gewünschten Effekt haben. Es
gibt zwei Befehle, die Ausgaben erzeugen. Der Befehl:

::

   .error message

gibt eine Nachicht aus und terminiert dann den make-Prozess.
Entsprechend ist er für unsere Zwecke nicht besonders geeignet. Besser
ist der Befehl:

::

   .warning message

der ohne Nebeneffekt eine Nachicht ausgibt. Die Nützlichkeit zur
Diagnose liegt darin, dass mit dem Befehl natürlich auch Variablen
ausgeben werden können.

for-Schleifen und if-Konstrukte
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hier ist ein Schema für eine for-Schleife:

::

   .for ITERATOR in ${LIST}
   ...
   .endfor

Diese Art von Schleife ist in vielen Sprachen als foreach-Schleife
bekannt. Der Schleifeninhalt wird für jeden Wert in **$LIST** einmal
aufgerufen. Der aktuelle Wert ist dabei in **${ITERATOR}** abrufbar.

Für unsere Zwecke benötigen wir lediglich zwei verschiedene
if-Konstrukte. Als erstes wäre da ein normaler Vergleich der Form

::

   .if value == ${VARIABLE}

Falls **VARIABLE** der Iterator ist, ist es wichtig, dass dieser auf der
rechten Seite steht. Hier ein Beispiel aus einer **/etc/make.conf**, in
der verschiedene Compiler für verschiedene Ports verwendet werden:

::

   .for BUILDPATH in ${BASE_BUILD}
   .if ${.CURDIR:M*/ports/$(BUILDPATH)}
   CC=         cc
   CXX=            c++
   CPP=            cpp
   .endif
   .endfor

**BASE_BUILD** ist eine Liste von Ports, die immer mit dem
Standardcompiler gebaut werden. **.CURDIR** wird von 'make' gesetzt und
enthält das Verzeichnis in dem 'make' läuft.

Das zweite if-Konstrukt das benötigt wird dient dazu festzustellen, ob
der Wert einer Variablen einem bestimmten Verzeichnis (\* erlaubt)
entspricht.

::

   .if${VARIABLE:Mpath}

Dieses Konstrukt lässt sich besonders gut für Parameter verwenden, die
nur für einen bestimmten Port gesetzt werden sollen. Hier wieder ein
Beispiel aus der **/etc/make.conf** des Autors:

::

   .if${.CURDIR:M*/ports/www/firefox*-i18n}
   WITHOUT_SWITCHER=   yes
   FIREFOX_I18N=   en-GB
   .endif

Compiler Flags
--------------

'make' wird für gewöhnlich zum Kompilieren von Programmen verwendet. Der
Standardcompiler unter FreeBSD ist GCC. Es ist jedoch üblich, den Aufruf
für den Compiler aus Variablen die in der **/etc/make.conf** gesetzt
werden können zu entnehmen, so dass zentral festgelegt werden kann womit
und wie kompiliert wird.

Detailierte Informationen zu den behandelten und weiteren Variablen
finden sich in der Manpage **make.conf(5)**.

CC, CXX und CPP
~~~~~~~~~~~~~~~

Die Standardeinstellungen lauten:

::

   CC=    cc
   CXX=   c++
   CPP=   cpp

Wer einen anderen Compiler aus den Ports gebaut hat und verwenden möchte
kann das so tun:

::

   CC=    gcc41
   CXX=   c++41
   CPP=   cpp41

In diesem Fall wird der GCC 4.1 verwendet.

Einige Ports verwerfen diese Angaben und schreiben fest einen bestimmten
Compiler vor. Dafür gibt es meist einen guten Grund. Deshalb ist es
meist eine schlechte Idee, sich an den jeweiligen Makefiles zu schaffen
machen, um dies zu *korrigieren*.

CPUTYPE
~~~~~~~

Die nächste wichtige Variable ist CPUTYPE. Sie wird üblicherweise mit:

::

   CPUTYPE?=   cpu type

gesetzt. **CPUTYPE** spielt beim Bauen des Basissystems und des Kernels
eine Rolle und hängt Compiler-Parameter an die Variablen **CFLAGS** und
**COPTFLAGS** an.

CFLAGS und COPTFLAGS
~~~~~~~~~~~~~~~~~~~~

Die Variable **CFLAGS** enthält Parameter für den Compiler, der in
**CC** spezifiziert ist. **COPTFLAGS** enthält die Parameter für
**CXX**. Wenn **COPTFLAGS** nicht gesetzt wird, erbt es den Wert von
**CFLAGS**, was in den meisten Fällen genügt.

Für diesen Artikel sind vor allem die **CFLAGS** **-O**, **-O2**,
**-fno-strict-aliasing** und **-pipe** wichtig.

-  **-pipe**

   -  sorgt lediglich dafür das der Compiler Pipes verwendet statt in
      Dateien zwischenzuspeichern. Es beschleunigt also leicht das
      kompilieren, falls genügend Arbeitsspeicher vorhanden ist.

-  **-fno-strict-aliasing**

   -  weist den Compiler an auf bestimmte Optimierungen zu verzichten.
      Diese Option ist Standard.

-  **-O**

   -  weist den Compiler an den Code zu optimieren um schnellere
      Binaries zu erzeugen. Bei einigen (sehr) wenigen Ports kann das zu
      Problemen führen.

-  **-O2**

   -  ist eine stärkere Optimierung als <tt>-O</tt> und ist für Welt und
      Kernel üblich. Die Wahrscheinlichkeit von Problemen beim
      kompilieren von Ports ist etwas größer als bei <tt>-O</tt> aber
      immer noch äußerst gering. Diese Option ist Standard.

Es gibt zwar noch höhere Optimierungsstufen, diese werden hier jedoch
nicht behandelt.

Desweiteren gibt es diverse Parameter mit denen der Compiler angewiesen
wird, verschiedene Befehlssatzerweiterungen zu verwenden.

-  **-mmmx**

   -  für den MMX Befehlssatz.

-  **-msse -msse2 -msse3**

   -  für die verschiedenen Versionen des SSE Befehlssatzes.

-  **-m3dnow**

   -  für den 3DNow Befehlssatz von AMD.

Nach der gcc-Dokumentation werden die erfahrungsgemäß besten CPU
Optimierungen bereits vom **-march** Parameter impliziert der wiederum
wird automatisch von **CPUTYPE** ermittelt. Es ist also in der Regel
nicht nötig diese Parameter einzusetzen.

Optimierungen für Kernel und Welt
---------------------------------

Die meisten Einstellungen werden beim Kernel ja bekanntlich in einer
entsprechenden Datei vorgenommen (siehe
`Handbuch <http://www.de.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/kernelconfig.html>`__).
Mehr Möglichkeiten gibt es in der **/etc/make.conf** beim Manipulieren
der Welt. Da das jedoch selten erwünscht ist, fällt dieser Abschnitt
sehr kurz aus.

Buildworld und buildkernel sind beim Autor übrigens stets gescheitert,
wenn stärkere Optimierungen als **-O2 -fno-strict-aliasing -pipe**
versucht wurden. Deswegen ist es sinnvoll, CFLAGS für buildkernel und
buildworld seperat zu setzen, wenn bei den Ports mehr erwünscht ist. Wie
das geht wird mit dem Lesen des nächsten Kapitels klar.

Kernel
~~~~~~

Die sinnvollste Variable für den Kernelbau ist sicherlich **KERNCONF**.
In **KERNCONF** wird der Dateiname (ohne Pfad) der gewünschten
Kernelkonfiguration angegeben. Wird mehr als eine Kernelkonfiguration
angegeben, wird nur die erstgenannte installiert. Der Standardwert ist
**GENERIC**. Es gibt auch Variablen, um den Bau bestimmter Module zu
verhindern oder das Verhalten des Bootloaders zu verändern.

Welt
~~~~

Für die Welt können Variablen gesetzt werden, um bestimmte Bestandteile
nicht zu bauen. Für Einige ist vielleicht auch die Möglichkeit, die Welt
mit Binärkompatibilität für ältere FreeBSD Versionen zu bauen,
interessant. Dazu dienen:

::

   COMPAT_FREEBSD4
   COMPAT_FREEBSD5

Optimierungen für Ports
-----------------------

Viele Einstellungen für die Ports können mit **make config** vorgenommen
werden. Jedoch gibt es immer wieder einige Dinge, die man nur als
Parameter übergeben oder in der **/etc/make.conf** eingestellt werden
können. Einige Variablen haben überall den gleichen oder gar keinen
Effekt. Einige sollten nur für einen bestimmten Port gesetzt werden und
natürlich gibt es solche, die je nach Port andere Werte haben müssen.
Wie das am besten realisiert wird, wird in diesem Abschnitt behandelt.

Vorbereitungen
~~~~~~~~~~~~~~

Am besten wird damit begonnen, sich einen Bereich in der
**/etc/make.conf** anzulegen, der ausschließlich für Einstellungen die
Ports betreffen reserviert ist. Dazu wird folgendes if-Konstrukt
verwendet:

::

   .if ${.CURDIR:M*/ports/*} && !${.CURDIR:M*/work/*}

   .endif

Auf diese Weise wird alles was hier geschieht von Welt und Kernel
abgekapselt. Das ist nicht wirklich notwendig, aber sicherlich eine gute
Angewohnheit, um Fehlerquellen zu minimieren. Die zweite Bedingung
verhindert ein erneutes Einlesen aus dem Verzeichnis in dem das
eigentliche Bauen des Ports stattfindet, da das zu Problemen führen
kann.

Als nächstes können in diesem Bereich ein paar Variablen gesetzt werden,
die nicht schaden können und manchmal nützlich sind. Zum Beispiel:

::

   WITHOUT_ARTS=       yes
   WITH_IPV6=      yes

Jetzt können noch versuchsweise Variablen für einen bestimmten Port
gesetzt werden. Zum Ausprobieren am besten einen Port verwenden, der
nicht allzu lange zum Bauen braucht:

::

   # x11-wm/fluxbox
   .if${.CURDIR:M*/ports/x11-wm/fluxbox-devel}
   WITHOUT_SLIT=       yes
   WITH_IMLIB2=        yes
   .endif

Jemand der nicht Fluxbox verwendet sollte natürlich ein anderes Beispiel
suchen.

Wenn ein Probeversuch funktioniert hat können noch Variablen gesetzt
werden in den Ports einen anderen Compiler zu verwenden. Das kann gut
mit einem kleinen Port wie **archivers/zip** getestet werden. Nicht
jeder Port funktioniert mit jedem Compiler. Ports die Probleme bereiten
sollten also besser den Standardcompiler verwenden.

Nach diesen Versuchen sollte der Bereich in der **/etc/make.conf** jetzt
ungefähr so aussehen:

::

   .if ${.CURDIR:M*/ports/*} && !${.CURDIR:M*/work/*}

   CC=         gcc41
   CXX=            g++41
   CPP=            cpp41

   WITHOUT_ARTS=       yes
   WITH_IPV6=      yes

   # x11-wm/fluxbox
   .if${.CURDIR:M*/ports/x11-wm/fluxbox}
   CC=         cc
   CXX=            c++
   CPP=            cpp
   WITHOUT_SLIT=       yes
   WITH_IMLIB2=        yes
   .endif

   .endif

Einen Port optimieren
~~~~~~~~~~~~~~~~~~~~~

Um die Anpassungsmöglichkeiten eines Ports zu erfahren wird in das
Ports-Verzeichnis gewechselt.

Mit dem Befehl:

::

   # grep defined Makefile

finden sich meist alle Variablen. Wenn die Namen nicht selbsterklärend
sind, ist es sinnvoll das **Makefile** zu lesen. Das geht beispielsweise
mit:

::

   # less Makefile

Die gewünschten Parameter können nun wie für Fluxbox beschrieben in der
**/etc/make.conf** gesetzt werden.

Einsatz von Listen und Schleifen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Im Laufe der Zeit wächst die **/etc/make.conf** oft aufgrund immer
weiterer Einstellung für immer mehr Ports. Die Datei **/etc/make.conf**
platzt also schon bald aus allen Nähten, dann ist es an der Zeit häufige
Ausnahmen zusammenzufassen.

Das wohl häufigste Ereignis ist wohl, dass ein Programm mit dem Compiler
der Wahl nicht gebaut werden kann oder nicht funktioniert. Die Lösung
ist simpel, die Skriptfähigkeiten von **make** erlauben es solche
Programme in einer Liste zu sammeln. Wenn nun der aktuellen Pfad
(**.CURDIR**) mit dieser Liste verglichen wird, kann automatisch
festgestellt werden, ob nicht doch lieber der Standardcompiler verwendet
werden sollte.

Wie Listen und Schleifen verwendet werden, wurde bereits geklärt, also
gibt es hier nur ein Beispiel:

::

   BASE_BUILD=     \
               devel/libtool15 \
               devel/sdl12 \
               net/linc \
               net/liveMedia \
               net/samba-libsmbclient \
               print/ghostscript-gnu-nox11 \
               x11/libgnome \
               x11/xorg-libraries \
               x11-toolkits/gtkmm24 \
               x11-toolkits/libbonoboui \
               x11-toolkits/libgnomeui

   .for BUILDPATH in ${BASE_BUILD}
   .if ${.CURDIR:M*/ports/$(BUILDPATH)}
   CC=         cc
   CXX=            c++
   CPP=            cpp
   .endif
   .endfor

Beispiele, Anwendungsfälle und Problemlösungen
----------------------------------------------

Als letzten Punkt dieses Absatzes gibt es ein Beispiel für eine
**/etc/make.conf**, die wahrscheinlich vor allem Anfängern einen guten
Überblick über die Möglichkeiten gibt. Ansonsten gibt es hier Beispiele
von Problemen mit denen der Autor irgendwann mal konfrontiert wurde.
Diese Probleme tauchen wahrscheinlich nicht auf jedem System so auf und
sollen eher als Beispiele dafür verstanden werden, wie mit Problemen
beim Kompilieren von Ports jenseits der Standardvorgaben umgegangen
wird.

archivers/unzip
~~~~~~~~~~~~~~~

Bei einem **ports-mgmt/portupgrade** wurde plötzlich der Port
**java/jdk15** nicht mehr gebaut. Die Prüfsumme der Dateien die aus dem
Archiv **jdk-1_5_0-src-scsl.zip** entpackt wurden stimmte nicht, also
lud ich die Datei erneut von Suns Homepage. Das brachte keine Besserung,
obwol die Prüfsumme des Archivs korrekt war.

Also versuchte ich den Port **archiver/unzip** zu bauen, was ohne
Probleme funktionierte. Bei einem manuellen Test mit einem Zip-Archiv
stellte sich jedoch heraus das unzip ausschließlich Dateien der Länge 0
produziert.

Der nahelegendste Ansatz war es unzip mit dem Standardcompiler zu bauen.
Das funktionierte ebenso gut, aber unzip produzierte immer noch die
Dateien der Länge 0. Der nächste Ansatz war also nicht die
Compilerversion sondern die Optimierung.

Ohne Optimierung funktioniert unzip einwandfrei, unabhängig davon
welchen der beiden Compiler ich verwendete. Damit war die Quelle des
Problems erkannt. Weitere Tests ergaben, das -O noch zuverlässig
funktioniert, -O2 aber nicht mehr.

Ungewöhnlich an diesem Beispiel war nicht, dass es ein Problem gab,
sondern dass es nicht bereits beim Kompilieren aufgetreten ist. Solche
Probleme können sehr unangenehm werden, da sie lange Zeit unentdeckt ein
ansonsten reibungslos funktionierendes System stören können. Der
Schlüssel zur Lösung war in diesem Fall der manuelle Test von unzip.
Leider wird das bei komplexeren Programmen ungleich schwieriger.

devel/sdl12 und x11/xorg-libraries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nach einem portupgrade -a konnten plötzlich ports die von
**devel/sdl12** abhängig waren nicht mehr gebaut werden. Die Ports
scheiterten mit dem Fehler, dass keine Version von sdl11-config höher
oder gleich (>=) der benötigten Version vorhanden war. Der Befehl:

::

   # pkg_info -W /usr/local/bin/sdl11-config

brachte Aufschluss darüber zu welchem Paket die Datei gehörte, also fing
ich an den Port **devel/sdl12** neu zu bauen. Das jedoch brachte keine
Besserung.

Glücklicherweise hatten die Ports mit dem Scheitern der
Versionserkennung auch ein Logfile angelegt, aus dem hervorging, dass
ein Fehler beim Aufruf einer Library aufgetreten war. Ein weiteres
**pkg_info -W** brachte Aufschluss darüber, dass die Library zum Port
**x11/xorg-libraries** gehörte.

Alles weitere war mehr oder weniger Routine. Es stellte sich heraus,
dass x11/xorg-libraries zwar ohne sichtliche Fehler mit dem gcc41 gebaut
werden kann, jedoch funktionierten auf diese Weise nicht alle erzeugten
Libraries so wie sie sollten.

Dieser Fehler hat mich sehr lange beschäftigt, da ich von den
Fehlermeldungen zum Glauben verleitet war, der Fehler läge beim Port
devel/sdl12. Ich hätte mir viele Nerven und einige Zeit sparen können,
wenn ich die Logfiles von Anfang an gründlicher gelesen hätte.

/boot/loader
~~~~~~~~~~~~

Wenn ich buildworld und installworld mit **CPUTYPE?=pentium-m**
ausführe, friert bei mir das System beim nächsten boot ein bevor der
Kernel geladen wird. Verantwortlich dafür ist die Datei
**/boot/loader**, die ich deswegen immer vor dem Reboot mit einem alten
Backup ersetzt habe.

Nach meinen Experimenten mit den Ports habe ich beschlossen auch dieses
Problem zu lösen. Mit:

::

   # find /usr/src -name loader

fand ich das Verzeichnis **/usr/src/sys/boot/i386/loader**. Jetzt
brauchte ich nur noch:

::

   CPUTYPE?=    pentium3

für das Verzeichnis einzutragen und ich kann mir die Backup-schieberei
sparen.

/etc/make.conf
~~~~~~~~~~~~~~

Diese **/etc/make.conf** war auf gigantische Ausmaße angewachsen, weil
es für alle Ports, die jetzt unter **BASE_BUILD** zusammengefasst sind,
eigene Einträge gab. Die unübersichtliche Größe hat letztlich dazu
geführt, dass ich mich ein wenig in die Grundlagen von make
eingearbeitet und diesen Artikel geschrieben habe.

::

   # ---< updating >--------------------------------------------------------------
   SUP_UPDATE=     yes
   DOC_LANG=       en_US.ISO8859-1
   SUPFLAGS=       -L 2 -z
   SUPHOST=        cvsup4.de.freebsd.org
   SUPFILE=        /usr/local/etc/cvsup/sources
   PORTSSUPFILE=       /usr/local/etc/cvsup/ports
   #DOCSUPFILE=        /usr/local/etc/cvsup/doc

   # Fetching from de sites preferred
   MASTER_SORT_REGEX?=://[^/]*\.de[/.]
   # -----------------------------------------------------------------------------

   # wget statt fetch zum downloaden der ports benutzen. falls existiert
   .if exists(/usr/local/bin/wget)
   DISABLE_SIZE=   yes
   FETCH_CMD=      /usr/local/bin/wget --continue --passive-ftp -t 2 -T 15
   .endif

   # ---< common >----------------------------------------------------------------
   # Default build flags.
   CPUTYPE?=       pentium-m
   CFLAGS=         -O2 -pipe

   # This causes the java vm wrapper to choose my preferred version of Java.
   #JAVA_PREFERRED_PORTS?= JAVA_PORT_NATIVE_BSDJAVA_JDK_1_5
   # -----------------------------------------------------------------------------

   # ---< configure buildworld/buildkernel >--------------------------------------
   KERNCONF=       TPR40-6

   # /boot/loader crashs with pentium-m
   .if ${.CURDIR:M*/src/sys/boot/i386/loader*}
   .undef CPUTYPE
   CPUTYPE?=       pentium3
   CFLAGS=         -O2 -pipe
   .endif
   # -----------------------------------------------------------------------------

   # ---< configure ports >-------------------------------------------------------
   .if ${.CURDIR:M*/ports/*} && !${.CURDIR:M*/work/*}

   # Set gcc version for ports.
   CC=         gcc41
   CXX=            g++41
   CPP=            cpp41

   # Build these ports with the compiler provided by the base system.
   BASE_BUILD=     \
               deskutils/adesklets \
               devel/pkgconfig \
               devel/sdl12 \
               games/scummvm \
               graphics/gimp \
               graphics/inkscape \
               graphics/OpenEXR \
               graphics/png \
               lang/gcc41 \
               lang/gcc-ooo \
               mail/thunderbird \
               math/octave \
               multimedia/bmpx \
               multimedia/ffmpeg \
               multimedia/mpeg4ip-libmp4v2 \
               multimedia/vlc \
               multimedia/xmms \
               net/linc \
               net/samba-libsmbclient \
               print/ghostscript-gnu-nox11 \
               security/gnupg \
               sysutils/cdrtools \
               sysutils/k3b \
               textproc/aspell \
               www/firefox \
               www/linuxpluginwrapper \
               x11-fm/krusader \
               x11-toolkits/wxgtk26 \

   # Don't build these ports.
   IGNORE_BUILD=       \
               editors/openoffice.org-* \

   # Build these ports with -O instead of -O2.
   WEAK_OPT_BUILD=     \
               archivers/unzip \

   # Build these ports without optimizations.
   NO_OPT_BUILD=       \

   # Common settings that are applied to all ports in hope to do some good.
   WITHOUT_ARTS=       yes
   WITH_IPV6=      yes
   WITH_MOZILLA=       firefox
   WITHOUT_DEBUG=      yes
   WITH_GTK2=      yes
   WITH_FAM_SYSTEM=    fam

   # editors/openoffice.org*
   .if ${.CURDIR:M*/ports/editors/openoffice.org*}
   #WITH_GNUGCJ=       yes
   LOCALIZED_LANG=     en-GB
   WITH_TTF_BYTECODE_ENABLED=yes
   WITHOUT_MOZILLA=    yes
   .endif

   # java/jdk1?
   .if ${.CURDIR:M*/ports/java/jdk1?}
   .undef WITH_IPV6
   .endif

   # multimedia/bmpx
   .if ${.CURDIR:M*/ports/multimedia/bmpx}
   WITHOUT_PYTHON=     yes
   WITHOUT_PERL=       yes
   .endif

   # devel/subversion
   .if ${.CURDIR:M*/ports/devel/subversion}
   WITHOUT_BDB=        yes
   .endif

   # multimedia/vlc
   .if ${.CURDIR:M*/ports/multimedia/vlc}
   WITH_FAAD=      yes
   WITH_REALAUDIO=     yes
   WITH_SDL=       yes
   WITH_X264=      yes
   WITH_THEORA=        yes
   WITH_WIN32_CODECS=  yes
   WITH_DVDREAD=       yes
   # WITH_MOZILLA_PLUGIN=  yes
   WITH_SSL=       yes
   WITH_OPTIMIZED_CFLAGS=  yes
   .endif

   # Check weather building with the base compiler is required.
   .for BUILDPATH in ${BASE_BUILD}
   .if ${.CURDIR:M*/ports/$(BUILDPATH)}
   CC=         cc
   CXX=            c++
   CPP=            cpp
   .endif
   .endfor

   # Check weather this port should be ignored.
   .for BUILDPATH in ${IGNORE_BUILD}
   .if ${.CURDIR:M*/ports/$(BUILDPATH)}
   IGNORE=         Build me later.
   .endif
   .endfor

   # Check weather building with weak optimization is required.
   .for BUILDPATH in ${WEAK_OPT_BUILD}
   .if ${.CURDIR:M*/ports/$(BUILDPATH)}
   CFLAGS=         -O -pipe
   .endif
   .endfor

   # Check weather building without optimization is required.
   .for BUILDPATH in ${NO_OPT_BUILD}
   .if ${.CURDIR:M*/ports/$(BUILDPATH)}
   CFLAGS=         -pipe
   .endif
   .endfor

   .endif
   # -----------------------------------------------------------------------------

   # added by use.perl 2006-01-05 23:47:54
   PERL_VER=5.8.7
   PERL_VERSION=5.8.7

Quellen
-------

-  Der Artikel `Make.conf Optimal
   nutzen <http://wiki.bsd-crew.de/index.php/Make.conf_Optimal_nutzen>`__
   der `BSD-Crew Dresden <http://www.bsd-crew.de/>`__ hat den Anstoß für
   die Versuche des Autors gegeben.
-  Die Manpage **make.conf(5)**.
-  Die Manpage **make(1)**.

* :ref:`genindex`

Zuletzt geändert: |date|

