UNICODE
=======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Viele FreeBSD-Benutzer haben Probleme mit der Zeichendarstellung von
chinesischen, japanischen, koreanischen, russischen Schriften. Insbesondere im
Zusammenspiel mit Samba, mount -t ntfs bereitet unwillkommener Buchstabensalat
oft Kopfzerbrechen. Andere Betriebssysteme wie Windows NT, Mac OS X und Linux
entschärfen dieses Problem mit dem Einsatz von UNICODE-Zeichensatzkodierung.
Für FreeBSD ist die UNICODE-Unterstützung in Entwicklung. Wie man die bereits
realisierte UNICODE-Unterstützung aktivieren kann, soll diese Anleitung
erläutern.

.. note::

  Diese Anleitung ist auch für FreeBSD-Benutzer interessant, welche nicht
  gleich UNICODE einsetzen möchten, aber trotzdem Ihre Sprach- und
  Umlauteprobleme lösen möchten. Diese Personen sollen bitte die Anleitung mit
  de_DE.ISO8859-15 statt de_DE.UTF-8 durcharbeiten. ISO8859-15 ist ein vom
  ISO8859-1 abstammender 1-Byte-Zeichensatz mit dem €-Zeichen.

Konsole
-------

Im Normalfall verwendet die Konsole einen 1-Byte-Zeichensatz.
Standardmäßig ist dies ISO-8859-1. Um nicht-westeuropäische Zeichen in
der Konsole darstellen zu können, muß man entweder die Konsole mit einem
anderen Zeichensatz füttern oder eine sprachspezifische Konsole nehmen!

Tastatureinstellung
-------------------

Es ist darauf zu Achten, daß die Konsolen-Tastatureinstellungen stimmen.
Die Konsolen-Tastatureinstellungen erfolgt in der Konfigurationsdatei
``/etc/rc.conf``:

::

   # Zu verwendende Konsoleschrift (iso8859-15)
   font8x14="iso15-8x14"
   font8x16="iso15-8x16"
   font8x8="iso15-8x8"
     
   # Deutsche Tastaturbelegung
   keymap="german.iso"
    
   # Schweizerdeutsche Tastaturbelegung
   # keymap="swissgerman.iso" 

Falls man im SingleUser-Mode nicht mit der amerikanischen
Tastaturbelegung arbeiten möchte, kann man den Kernel mit folgenden
Optionen neukompilieren:

::

   options ATKBD_DFLT_KEYMAP                         # Andere Tastaturbelegung im SingleUser-Mode
   makeoptions ATKBD_DFLT_KEYMAP="german.iso"        # Deutsche Tastaturbelegung
   # makeoptions ATKBD_DFLT_KEYMAP="swissgerman.iso" # Schweizerdeutsche Tastaturbelegung

Eine Anleitung zur Kernelkompilierung findet man unter
`hier </freebsd/Kernel erstellen>`__.

X11-Server
----------

Der X11-Server ist UNICODE-kompatibel. Man muß nur die Tastatur korrekt
konfigurieren. Eine Anleitung zur Tastaturkonfiguration findet man
unter:
`X11-Server-Tastaturkonfiguration <FreeBSD_-_X11-Serverkonfiguration#Tastatur>`__

.. _unicode-1:

UNICODE
-------

Alle ISO8859-Zeichensätze haben einen großen Nachteil, sie verwenden pro
Zeichen 1 Byte und können so pro Zeichensatz nur 256 Zeichen darstellen.
Deshalb wurde UNICODE ins Leben gerufen. UNICODE hat sich als Ziel
gesetzt, alle menschlichen Zeichen in einem Zeichensatz zu fassen. Mehr
Informationen zu UNICODE findet man unter
[http://eyegene.ophthy.med.umich.edu/unicode/ UNICODE-Informationen].

Aus Kompatibilitätsgründen eignet sich für FreeBSD nur der
UNICODE-Zeichensatz UTF-8 (Variabler Multibyte-Zeichensatz). Übrigens,
Windows NT und höher verwenden intern UTF-16 (2 Byte-Zeichensatz).

UNICODE-Installation
--------------------

Nachfolgend wird die Umstellung auf UTF-8 im Detail erläutert:

.. note::

  Viele Programme sind noch nicht 100%-UTF-8 kompatibel!  Prominente Programme
  sind: [STRIKEOUT:mc], [STRIKEOUT:xterm], mount -t smbfs, tar. Das
  FreeBSD-Dateisystem UFS verträgt Zeichensatzwechsel nicht! Alle Sonderzeichen
  in Dateinamen gehen bei der UNICODE-Umstellung verloren! Die FreeBSD-Konsole
  ist zur Zeit noch nicht 100%-UNICODE fähig, darum bitte ausschließlich den
  x11-Terminalemulator: uxterm verwenden! 

Lokalisierung
-------------

Damit die Programme wissen, welchen Zeichensatz und welche Sprache man
verwenden möchte, muß man dies mit der so genannten Lokalisierung
(`http://www.gnu.org/software/gettext/manual/html_node/gettext_3.html#SEC3 I18N/L10N <http://www.gnu.org/software/gettext/manual/html_node/gettext_3.html#SEC3 I18N/L10N>`__)
mitteilen. Mit:

::

  # locale -a

wird die Liste aller verfügbaren Lokalisierungen (engl. locale)
angezeigt. Die Lokalisierungen sind immer nach dem gleichen Prinzip
aufgebaut:

::

   <Sprache>_<Land>.<Zeichensatz> -> de_DE.UTF-8

Lokalisierung einstellen
~~~~~~~~~~~~~~~~~~~~~~~~

Für jeden Benutzer muß die zu verwendende Lokalisierung eingestellt
werden. Folgende Schritte sind dazu notwendig:

/etc/login.conf modifizieren
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

   german:German Users Accounts:\
   :charset=UTF-8:\
   :lang=de_DE.UTF-8:\
   :tc=default:

/etc/login.conf.db anlegen
^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  # cap_mkdb /etc/login.conf

Jedem einzelnen Benutzer seine Benutzerklasse zuweisen
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  # vipw
  user:password:1111:11:german:0:0:USER_NAME:/home/USER_NAME:/bin/csh

.. note::

  Die Datei /etc/csh.cshrc mit:

  **setenv EDITOR ee**

  und die Datei /etc/profile mit:

  **EDITOR=ee; export EDITOR**

  ergänzen. Danach kann vipw mit dem benutzerfreundlicheren Easy-Editor
  bedient werden. Wenn man mit dem Easy-Editor nicht weiter weiß, einfach
  die Esc-Taste drücken..

Lokalisierung kontrollieren
---------------------------

Mit:

::

  # locale

kannst Du abfragen, welche Lokalisierung du gerade verwendest.

Notlösung
~~~~~~~~~

In seltenen Fällen funktionieren die obengenannten Einstellungen nicht,
in diesem Fall gibt es eine zweite Lösung. Es müssen folgende Zeilen in
die entsprechenden Konfigurationsdateien geschrieben werden:

``/etc/profile`` -> Globale Konfigurationsdatei für sh und bash

::

   LANG=de_DE.UTF-8; export LANG
   MM_CHARSET=UTF-8; export MM_CHARSET

``/etc/csh.cshrc`` -> Globale Konfigurationsdatei für csh

::

   setenv LANG de_DE.UTF-8
   setenv MM_CHARSET UTF-8

UNICODE-Test
------------

Testen wir die UNICODE-Unterstützung:

-  Mozilla starten und die Testseite von Yudit aufrufen:
   `Yudit-Testseite <http://yudit.org/cgi-bin/test.cgi>`__ -> kein
   Problem mit UTF-8!
-  Die chinesischen Zeichen in einen Texteditor (z.B. kedit) kopieren.
-  Die Textdatei speichern.
-  Den für die Textdatei verwendeten Zeichensatz feststellen: file
   ./Textdatei => ./Textdatei: UTF-8 Unicode text
-  Die Textdatei in einem x11-Terminalemulator (z.B. KDE-Konsole)
   öffnen.
-  # more ./Textdatei -> Wenn die Darstellung fehlerhaft ist, die
   Schrift wechseln. Bitstream- und Lucida-Schriften sind
   empfehlenswert!
-  Die Textdatei in der Konsole anzeigen lassen. -> Welch ein
   Buchstabensalat!

Konvertierung
-------------

Textdatei
~~~~~~~~~

Um in Textdateien den Zeichensatz zu wechseln, sollte der KDE-Editor
KWrite genügen:

Zuerst die richtige Kodierung unter Menü->Ansicht->Kodierung auswählen.
Dann den Text einfügen und schließlich den Text speichern. Im "Speichern
unter"-Dialogfenster kann der gewünschte Zeichensatz ausgewählt werden.

Wie kann man an einem Plaintext eigentlich erkennen, wie er kodiert
wurde?

Gar nicht!! Viele Programme (Mozilla, file) besitzen
Zeichensatz-Erkennungsalgorithmen, welche aber nicht immer
funktionieren. So erkennt zum Beispiel **file** BIG5-kodierten Text als
ISO8859-kodierten Text. Aus diesem Grund enthalten die meisten
HTML-Seiten und E-Mails eine Zeichenkodierungsinformation. Die
Zeichenkodierungsinformation der Startseite von http://www.BSDForen.de
findest du in im HTML-Source-Code auf der 17. Zeilen -> ISO8859-15!

Dateinamen
~~~~~~~~~~

Sie haben Ihr System auf UNICODE umgestellt und können jetzt Datei- und
Verzeichnisnamen, die Umlaute enthalten, nicht mehr lesen? Dagegen hilft
das Werkzeug **convmv**. Convmv konvertiert Datei- und Verzeichnisnamen
vom verwendeten Zeichensatz in den gewünschten Zeichensatz. Wenn Sie zum
Beispiel ein Verzeichnis erstellt haben, als der Zeichensatz ISO8859-1
aktiv war und jetzt UTF-8 verwenden, so können Sie mit dem Befehl:

::

  # convmv --notest -r -f ISO8859-1 -t UTF-8 <Verzeichnisname>

alle Datei- und Verzeichnis-Namen unter <Verzeichnisname> wieder lesbar
machen. Für mehr Informationen siehe bitte
http://freshmeat.net/projects/convmv/ und
http://j3e.de/linux/convmv/man/.

Fremde Datei-/Netzwerksystem einbinden
--------------------------------------

Praktisch allen mount-Befehle muss der aktuell verwendete Zeichensatz
mitgeteilt werden. Dies geschieht meistens über eine "-L" oder "-C"
-Option.

mount -t ntfs
~~~~~~~~~~~~~

NTFS ist im Gegensatz zu FAT16/32 UNICODE-basierend. Für mehr
Informationen zu NTFS siehe http://www.ntfs.com.

Bei Problemen mit der Darstellung von Dateinamen mit Sonderzeichen auf
NTFS-Partitionen sind zwei Gründe möglich:

1. Man muß mount -t ntfs mitteilen, welchen Zeichensatz man unter
FreeBSD braucht. ``mount -t ntfs`` konvertiert alle Dateinamen von
UNICODE in den lokalen/FreeBSD-Zeichensatz. Standardmäßig verwendet
``mount -t ntfs`` ISO8859-1. Mit:

::

  # mount -t ntfs -C <lokalen Zeichensatz> /dev/<Festplatte>

teilt man mount mit, welchen lokalen Zeichensatz man verwendet. Man verwendet nicht

::

  # mount -t ntfs -W

da dieser Schalter veraltet ist! 

2. Der lokale Zeichensatz enthält gar nicht die zu darstellenden
Sonderzeichen. Im westeuropäischen Raum ist das selten ein Problem, da
ISO8859-1 und ISO8859-15 (ISO8859-1 mit Euro-Zeichen) alle benötigten
Zeichen enthalten.

Wenn man aber zum Beispiel mit japanischen oder chinesischen Zeichen
arbeitet, so empfiehlt sich dringend eine Umstellung des lokalen
Zeichensatzes auf UTF-8.

``'mount -t msdosfs``' Benötigt eine "-L"-Option, welche den lokalen
Zeichensatz enthält, damit die Dateinamen-Zeichensatz-Konvertierung
sauber funktioniert. Zum Beispiel:

::

  # mount -t msdosfs -L de_CH.UTF-8 /dev/da0s4 /mnt/zip

``'mount -t udf``' Um auf eine DVD zugreifen zu können, benötigt man
FreeBSD 5.x und mount -t udf. Mit der "-C"-Option, welche den lokalen
Zeichensatz enthält, koordiniert man die
Dateinamen-Zeichensatz-Konvertierung. Zum Beispiel:

::

  # mount -t udf -C UTF-8 /dev/acd0 /mnt/cdrom

Midnight Commander
------------------

Einzige Abhilfe gegen die hässliche Rahmendarstellung des Midnight
Commander in der Konsole bei ISO8859-1(5)- oder Unicode-Zeichensätzen
ist die Verwendung der Startoption::

  # mc -a

Problemlösung
-------------

Probleme bei der Zeichensatzumstellung und bei der Darstellung von
Umlauten? Vielleicht hilft:

-  http://www.bsdforen.de/showthread.php?p=153272
-  http://www.bsdforen.de/showthread.php?t=14858
-  http://www.bsdforen.de/showthread.php?t=10535

weiter.

Lesenswerte Literatur
---------------------

-  `FreeBSD-Handbuch, Kapitel
   Lokalisierung <http://www.de.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/l10n.html>`__
-  `Linux-Anwenderinformationen zur
   Zeichenkodierung <http://www.tu-chemnitz.de/urz/linux/faq/unicode.html>`__
-  `FreeBSD und
   Umlaute <http://www.bsdforen.de/showthread.php?t=5786>`__
-  `mount -t ntfs und
   Umlaute <http://www.bsdforen.de/showthread.php?t=4833>`__
-  `FreeBSD und
   Chinesisch <http://www.bsdforen.de/showthread.php?t=6369>`__
-  `Übersicht über die wichtigsten
   Zeichensätze <http://bibliofile.mc.duke.edu/gww/fonts/Charsets.html>`__
-  `Farbige Konsole <http://www.bsdforen.de/showthread.php?t=7472>`__

* :ref:`genindex`

Zuletzt geändert: |date|

