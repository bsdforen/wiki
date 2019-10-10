Deutsche Sprachausgabe mit Festival
===================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Dieses Tutorial beschreibt, wie man ein deutschsprachiges Festival als
Sprachausgabe unter FreeBSD einrichtet. Zusätzlich wird gezeigt, wie man dies
mit einem GUI - inklusive Wortvorschlagsliste - verbindet, sodass es als Ersatz
für einen Talker (Sprachcomputer) genutzt werden kann.

Festival
--------

Festival ist ein Open Source Text to Speech System (TTS), welches an der
Universität von Edinburgh entwickelt wird. Es ermöglicht einen
beliebigen Text einzugeben, welcher vom Computer gesprochen wird. Leider
gibt es Festival offiziell nur in 9 Sprachen, Deutsch ist - vor allem
aufgrund seiner komplizierten Phonetik - nicht darunter.

Daher hat man an der Universität Stuttgart einen Weg entwickelt Mbrola
mit Festival zu verbinden und so ein brauchbares deutsches TTS zu
ermöglichen. Diese Patches stehen allerdings nur teilweise unter einer
Open Source Lizenz; sie können allerdings frei genutzt werden.

Der Vollständigkeit halber soll hier noch erwähnt werden, dass es
Festival nicht einmal im Ansatz mit einer guten Hardwarelösung aufnehmen
kann.

Anforderungen
-------------

Festival errechnet die Sprache erst und gibt sie dann aus. Daher kann es
auch auf alter Hardware ohne Probleme genutzt werden. Nur dauert es dann
eben sehr lange bis die Verarbeitung beendet ist. Ich empfehle daher
einen PII 400MHz mit 128MB RAM als absolutes Minimum. Alles darunter
verzögert zu sehr. Annähernd verzögerungsfrei wird es aber erst ab ca. 1
GHz.

Die Software
------------

Leider kann Festival nicht über die Ports installiert werden, da
tiefgreifende Änderungen am Code nötig sind. Hält man sich an dieses
Tutorial, liegt es in einem Ordner und kann durch löschen von diesem
vollständig entfernt werden.

Unabhängig hiervon müssen folgende Anforderungen erfüllt sein:

-  Ein FreeBSD ab 4.7 (das Tutorial wurde unter 6.0 geschrieben)
-  Eine konfigurierte Soundkarte
-  Eine konfigurierter Linuxulator mit installierter Base

Folgende Ports müssen installiert sein:

-  `devel/gmake <https://www.google.com/search?q=devel/gmake&btnI=lucky>`__
-  `/audio/sox <https://www.google.com/search?q=/audio/sox&btnI=lucky>`__

Die Dateien
-----------

Folgende Dateien müssen herunter geladen werden:

-  http://www.cstr.ed.ac.uk/downloads/festival/1.95/festival-1.95-beta.tar.gz
-  http://www.cstr.ed.ac.uk/downloads/festival/1.95/speech_tools-1.2.95-beta.tar.gz
-  http://tcts.fpms.ac.be/synthesis/mbrola/bin/pclinux/mbr301h.zip
-  http://tcts.fpms.ac.be/synthesis/mbrola/dba/de1/de1-980227.zip
-  http://tcts.fpms.ac.be/synthesis/mbrola/dba/de2/de2-990106.zip

Das deutsche Patchset ist leider ein wenig komplizierter. Hierzu geht
man auf
http://www.ims.uni-stuttgart.de/phonetik/synthesis/festival_opensource.html,
liest die Bedingungen und lädt diese Dateien herunter:

-  festival_lexfix.tgz
-  ims_german_1.2c-os.tgz
-  ims_german_1.2c-doc.tgz
-  bomp_full.tgz

Schließlich fehlen noch einige Patches, damit Mbrola unterstützt wird
und sich Festival unter FreeBSD kompilieren lässt:

-  http://www.ims.uni-stuttgart.de/phonetik/synthesis/festival/festival-1.95-fixes.tgz
-  http://www.freebsd.org/cgi/cvsweb.cgi/ports/audio/festival/files/patch-speech_tools_install.mak
-  http://www.freebsd.org/cgi/cvsweb.cgi/ports/audio/festival/files/patch-speech_tools_voxware.cc

Die Installation
----------------

Festival muss in dem Verzeichnis kompiliert werden, in dem es später
ausgeführt werden soll. Dabei stellt sich das Problem, dass jeder Nutzer
von Festival auf fast alle zugehörigen Dateien schreib-lese-ausführen
Rechte haben muss. Dies kann unter Umständen ein großes
Sicherheitsproblem darstellen. Soll es nur von einem Nutzer genutzt
werden empfiehlt sich, es unter diesem zu bauen. Ich habe es ins
Verzeichnis */usr/local/festival* installiert aber auch jedes andere ist
möglich. Im Folgenden werden, wenn nicht anders gesagt, alle Kommandos
von diesem Verzeichnis aus ausgeführt.

Zu Beginn wird das Verzeichnis erstellt und dieses hineingewechselt:

::

   # mkdir /usr/local/festival
   # cd /usr/local/festival

Nun wird Festival entpackt und gleich das deutsche Patchset eingespielt:

::

   # tar xfz festival-1.95-beta.tar.gz
   # tar xfz speech_tools-1.2.95-beta.tar.gz
   # tar xfz festival_lexfix.tgz
   # tar xfz ims_german_1.2c-os.tgz
   # tar xfz ims_german_1.2c-doc.tgz
   # tar xfz bomp_full.tgz

Schließlich müssen noch die Patches für FreeBSD eingebaut werden:

::

   # patch -i patch-speech_tools_install.mak
   # patch -i patch-speech_tools_voxware.cc

Damit ist Festival fürs erste abgeschlossen. Nun ist ist Mbrola an der
Reihe. Dazu wird ein Verzeichnis erstellt und es entpackt:

::

   # mkdir mbola
   # cd mbrola
   # unzip mbr301h.zip

Nun werden alle Dateien in diesem Verzeichnis außer *mbrola-linux-i386
gelöscht*. Jetzt bennent man diese um und deklariert sie als
Linux-Binary:

::

   # mv mbrola-linux.i386 mbrola
   # brandelf -t Linux mbrola

Es folgen die Stimmen, wir sind immer noch im Verzeichnis mbrola:

::

   # unzip de1-980227.zip
   # unzip de2-990106.zip

In den beiden neuen Unterordnern *de1* und *de2* können alle Dateien
außer *de1* und *de2* gelöscht werden. Damit ist die Installation von
mbrola beendet und es geht wieder ins Hauptverzeichnis.

Nun kann Festival konfiguriert und kompiliert werden. Dies geschieht in
mehreren Schritten:

::

   # cd speech-tools
   # ./configure
   # gmake
   # cd ..

Nun Festival selbst. Dies ist leider ein wenig komplizierter:

::

   # cd festival

Nun wird die Datei *libs/siteinit.scm* in einem Editor geöffnet und
hinter die letzte Zeile diese

::

   require 'ims_german_opensource)

eingefügt. Außerdem wird die Zeile

::

   ;(set! voice_default 'voice_cmu_us_awb_arctic_hts)

für eine männliche Stimme in

::

   (set! voice_default 'voice_german_de2_os)

und für eine weibliche in

::

   (set! voice_default 'voice_german_de1)

geändert. Es folgt die - noch nicht vorhandene - Datei
*libs/sitevars.scm*. In diese trägt man folgende Zeilen ein (eventuell
muss der Pfad in der oberen Zeile geändert werden!):

::

   (set! mbrola-path "/usr/local/festival/mbrola/")
   (set! mbrola_progname (string-append mbrola-path "mbrola -e"))

Jetzt wird Festival konfiguriert:

::

   # ./configure

Die entstandene *config/config* wird wiederum in einem Editor geöffnet
und hinter

::

   ALSO_INCLUDE +=

dies

::

   ALSO_INCLUDE += ims_german_text

eingefügt. Jetzt kann Festival selbst kompiliert werden.

::

   # gmake

Test
----

Nachdem der Bau beendet ist, kann Festival getestet werden. Dazu
wechselt man in das Verzeichnis bin und startet Festival.

::

   # cd festival/bin
   # ./festival

Es sollte keine Fehlermeldung geben. Wenn doch bietet es sich an, noch
einmal alles durchzugehen. Sollte dies keine Lösung bringen kann man
Google nach der Fehlermeldung suchen. Durch Eingabe von

::

   (SayText #Hallo!")

kann man #Hallo!" aussprechen lassen. Der Text sollte auf deutsch
gesprochen werden. Nun kann man Festival mittels

::

   (quit)

verlassen.

Festival ist nun eingerichtet und spricht Deutsch. Es kann mittels
Script z.B. mit irssi oder Gaim verbunden werden. Hierzu finden sich
diverse Anleitungen im Netz. Ich werde im zweiten Teil beschreiben wie
man es mit Kmouth verbindet und so einen richtigen Sprachcomputer baut.

KMouth mit Festival
===================

KMouth ist ein Frontend für diverse Sprachausgabesysteme und in
bedingtem Maße auch echten Talkern (Sprachcomputern). Es erlaubt das
Anlegen von Lexika zum schnellen Aufruf von Phrasen. Es enthält eine
Wortvorschlagsliste und ein minimales Journal.

Installation
------------

KMouth kann einfach über die Ports installiert werden. Der Fairness
halber sei hier gesagt, dass es sich um ein KDE-Programm handelt und
damit Dependencies auf qt und kdebase hat. Auf langsamen Systemen kann
das bauen dieser Ports unter Umständen Stunden dauern, weshalb man über
das Nutzen von Paketen nachdenken sollte. Der benötigte Port ist
*/accessibility/kdeaccessibility*. Dieser installiert den Daemon kttsd,
KMouth und Ksayit. Letzteres wird hier allerdings nicht weiter
behandelt.

Konfiguration
-------------

Alle Programme sollten bis zum Ende dieses Abschnittest aus einem
Terminal heraus gestartet werden, damit man eventuelle Fehlermeldungen
lesen kann.

Die Konfiguration beginnt mit dem Start von kttsd durch Eingabe von

::

   # kttsd

Dieser Daemon muss auch in Zukunft vor Kmouth gestartet werden, sonst
ist die Sprachausgabe nicht möglich.

Da dies der erste Start ist erscheint ein Hinweis, dass kttsd noch nicht
konfiguriert sei. Eine Einrichtung soll an dieser Stelle nicht
vorgenommen werden, daher klickt man auf #Nein".

Nun wird KMouth durch

::

   # kmouth

gestartet. Es meldet sich wieder ein Einrichtungsassistent, der mit
Klick auf *Cancel* beendet wird. Nach dem Erscheinen des Hauptfensters
öffnet man über Settings -> Configure KMouth die Einstellungen.

Ich zeige hier nur die nötigen Anpassungen, alle anderen Optionen können
nach jeweiligen Wünschen angepasst werden. Die Darstellung erfolgt hier
schematisch:

::

   General Options -> Text to Speech:
   Command for seaking text: leer
   Character Encoding: Local oder ISO 8859-1
   [[]] Send the Data as Standard Input
   [[x]] Use KTTSD speech service if possible

   KKTSD Speech Service -> General:
   [[x]] Enable Text-to-Speech System (KTTSD)

   KKTSD Speech Service -> Talkers -> Add:
   Language: Other
   Synthesizer: Command (eventuell muss ein unbeschrifteter Button unter #Show All" zuvor geklickt werden.
   -> OK:
   Language: German
   -> OK:
   Command for speaking Texts: /usr/local/festival/festival/bin/text2wave %f -o %w (eventuell muss der Pfad angepasst werden!)
   Character Encoding: Local oder ISO 8859-1
    [[]] Send the data as standard input
   -> Test

Es sollte etwas aus den Boxen kommen. Dies kann auch ein Text in fremder
Sprache, aber deutsch ausgesprochen, sein!

::

   -> OK

KMouth ist nun eingerichtet und kann verwendet werden.

Wortvervollständigung
---------------------

Hier wird zum Abschluss noch gezeigt wie man die Wortvorschlagsliste von
KMouth einrichtet. Dies sollte nur bei Bedarf geschehen, da dieses
Feature das Programm sehr stark ausbremst!

Laden Sie folgende Datei herunter:

-  ftp://ftp.ox.ac.uk/pub/wordlists/german/words.german.Z

Nun wird ein neues Verzeichnis erstellt und die Liste entpackt:

::

   # cd /usr/local/festival
   # mkdir wortliste
   # cd wortliste
   # uncompress word.german.Z

In KMouth werden wieder die Einstellungen aufgerufen und man geht auf
Word

::

   Completion -> Add Dictionary:
   [[x]] Create new Dictionary
   [[x]] From File
   -> Next:
   Filename: /usr/local/festival/wortliste/words.german
   Character Encoding: ISO 8859-1
   Language: US English (ja wirklich!)
   -> Finish

Die Vorschlagsliste sollte nun funktionieren und während der Eingabe in
Frage kommende Wörter anbieten. Zusätzlich kann man natürlich auch eine
andere oder weitere Dateien als Dictionary mitgeben.

--------------

Dieser Artikel ist das Ergebnis stundenlanger Arbeit, logischen Denkens,
Lesens und vor allem Ausprobierens. Es würde ihn nicht geben, wenn es
mir nicht diverse Leute - egal ob direkt oder durch andere Tutorials -
geholfen hätten. Stellvertretend für all diese Menschen möchte ich mich
bei 2 besonders bedanken:

-  Axel Schmidtmann, der mir Funktion und Aufbau künstlicher
   Sprachsysthese erklärt hat
-  Jörg Günther, er hat mich zu diesem Projekt motiviert

* :ref:`genindex`

Zuletzt geändert: |date|

