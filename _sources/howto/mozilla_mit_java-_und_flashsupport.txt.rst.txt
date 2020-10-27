Mozilla mit Java- und Flashsupport
==================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel führt Schritt-für-Schritt durch die Installation der Flash-,
Java-, Mplayer-, Adobe Reader- und RealPlayer-Plugins für den nativen
Firefox/Seamonkey. Zudem wird aufgezeigt, wie die Flash-Unterstützung im
Konqueror aktiviert werden kann.

nspluginwrapper
---------------

Praktisch alle Mozilla-Plugins benötigen den
`www/nspluginwrapper <https://www.google.com/search?q=www/nspluginwrapper&btnI=lucky>`__,
sofern es sich bei den Plugins um Linux-Versionen handelt. Der
nspluginwrapper wird mit::

  # portinstall nspluginwrapper

installiert.

Flash-Plugin
------------

Der Linux-Flashplugin wird wie folgt installiert::

  # portinstall www/linux-flashplugin9

Damit Firefox3 das Flashplugin findet, wird noch ein symbolischer Link
benötigt::

  # cd /usr/local/lib/firefox3/plugins && \
  ln -s /usr/local/lib/browser_plugins/npwrapper.libflashplayer.so npwrapper.libflashplayer.so

Sofern der nspluginwrapper noch nicht installiert ist, sollte dies nun
erfolgen:

::

  # portinstall www/nspluginwrapper

Die Initialisierung erfolgt dann unter dem root-Account systemweit und
jeweils bei den entsprechenden Benutzern mit::

  # nspluginwrapper -v -a -i

Schließlich können Sie die `Funktionstüchtigkeit vom
Flash-Plugin <http://www.adobe.com/shockwave/welcome/>`__ kontrollieren.
Mit der Erweiterung
`www/xpi-flashblock <https://www.google.com/search?q=www/xpi-flashblock&btnI=lucky>`__
kann man die Anzeige von Flash auf Webseiten kontrollieren.

Problembeseitigung
------------------

Leider hängt sich flash9 gerne mal auf. Dieser Zustand kann durch
folgende Eingabe leicht beseitigt werden::

  # killall -9 npviewer.bin 

Sollte bei Flashspielen bzw. -Anwendungen die Schrift fehlen muss noch
x11-fonts/webfonts installiert werden::

  # portinstall x11-fonts/webfonts

Java
----

Das Java-Plugin ist im Java-Paket enthalten.

Es gibt zwei verschiedene Arten von Java in den Ports:

-  Die Java Runtime Environment (JRE) welche zum Ausführen von
   Programmen ausreichend ist
-  Das Java Development Kit (JDK) welches für Java Entwickler
   interessant ist

Im Grunde reicht die JRE, es sei denn, Sie wollen selber Java Programme
oder Applets entwickeln. Im Zweifelsfall installieren Sie die JRE,
sollten Sie das JDK irgendwann einmal benötigen, deinstallieren Sie die
JRE einfach und installieren danach das JDK.

`Laden <http://www.freebsdfoundation.org/downloads/java.shtml>`__ Sie
die für Ihr System passende Datei runter. (Entweder FreeBSD 5 oder 6 und
JRE oder JDK.) Verschieben Sie die heruntergeladene Datei nach
/usr/ports/distfiles/.

Und schließlich installieren Sie Java über den entsprechenden Port::

  # portinstall diablo-jre15

oder

::

  # portinstall diablo-jdk15 

Testen Sie nun, ob alles korrekt installiert wurde::

  # java -version 

Folgende oder ähnliche Ausgabe sollte angezeigt werden::

  java version "1.5.0"
  Java(TM) 2 Runtime Environment, Standard Edition (build diablo-1.5.0-b01)
  Java HotSpot(TM) Client VM (build diablo-1.5.0_07-b01, mixed mode)

Schließlich können Sie die `Funktionstüchtigkeit vom
Java-Plugin <http://www.bodo.com/javame.htm>`__ kontrollieren.

Mplayer-Plugin
--------------

Fügen Sie in der Datei /etc/make.conf die Zeile

::

   WITH_GECKO= firefox mozilla seamonkey

ein. Natürlich können Sie die nicht benötigten Programme entfernen und
z.B. nur

::

   WITH_GECKO= firefox

definieren.

Nun installalieren Sie das Plugin mit::

  # portinstall www/mplayer-plugin

Adobe Reader
------------

Das Adobe Reader-Plugin benötigt den Linuxpluginwrapper.

Adobe Reader installieren::

  # portinstall acroread8

Schließlich können Sie die `Funktionstüchtigkeit vom Adobe Reader-Plugin
kontrollieren <http://www.sph.unc.edu/courses/skills_test/pdf/test.pdf>`__.
Mit der Erweiterung ``www/xpi-pdf_download`` kann man die Anzeige von PDF
Dokumenten kontrollieren.

RealPlayer-Plugin
-----------------

Das RealPlayer-Plugin benötigt den Linuxpluginwrapper.

RealPlayer installieren::

  # portinstall linux-realplayer 

Mozilla-Plugins mit Konqueror benutzen
--------------------------------------

Eine ausführliche Anleitung bietet das `FreeBSD KDE
Team <http://freebsd.kde.org/howtos/konqueror-flash.php>`__.

Testen
------

-  Welche Plugins funktionieren, kann man anhand `einiger
   Beispieldateien <http://www.linspire.com/file_types/filetypes.php>`__
   testen. Alle dort aufgeführten Dateien werden sich allerdings nicht
   per Plugin öffnen lassen.
-  Nach der Installation des jeweiligen Plugins starten Sie Firefox von
   einem Terminal aus. Halten Sie Ausschau nach eventuellen
   Fehlermeldungen im Terminalfenster.
-  Kontrollieren Sie im Firefox unter der Adresse ``about:plugins`` ob
   die Plugins erkannt wurden.

Fehlerbehebung
--------------

Symbolische Links
-----------------

Bei Problemen sehen Sie am besten in der /etc/libmap.conf nach, wo sich
ein Mozilla-Plugins genau befinden muss. Erstellen Sie einen
symbolischen Link des fehlenden Mozilla-Plugins ins Verzeichnis
/usr/local/lib/browser_plugins/

Zum Beispiel:

-  Flash-Plugin

::

  # ln -s /usr/local/lib/npapi/linux-flashplugin/flashplayer.xpt /usr/local/lib/browser_plugins/flashplayer.xpt 
  # ln -s /usr/local/lib/npapi/linux-flashplugin/libflashplayer.so /usr/local/lib/browser_plugins/libflashplayer.so

Für JDK

::

  # ln -s /usr/local/diablo-jdk1.5.0/jre/plugin/i386/ns7/libjavaplugin_oji.so /usr/local/lib/browser_plugins/libjavaplugin_oji.so

oder für JRE

::

  # ln -s /usr/local/diablo-jre1.5.0/plugin/i386/ns7/libjavaplugin_oji.so /usr/local/lib/browser_plugins/libjavaplugin_oji.so

- Adobe Reader-Plugin

  ::

    # ln -s /usr/X11R6/Adobe/Acrobat7.0/ENU/Browser/intellinux/nppdf.so /usr/local/lib/browser_plugins/nppdf.so

- RealPlayer-Plugin

  ::

    # ln -s /usr/X11R6/lib/linux-mozilla/plugins/nphelix.so /usr/local/lib/browser_plugins/nphelix.so
    # ln -s /usr/X11R6/lib/linux-mozilla/plugins/nphelix.xpt /usr/local/lib/browser_plugins/nphelix.xpt

Rechner neustarten
------------------

In der Vergangenheit zeigte sich, dass in einigen Fällen ein
Rechner-Neustart notwendig ist, damit alles funktioniert.

Vorher sollte aber getestet werden ob der Aufruf::

  # /etc/rc.d/ldconfig start

nicht ausreicht.

Alternativen
------------

zu Flash
--------

.. note::

  Die folgenden offenen Varianten des Flashplayers unterstützen zur Zeit nicht
  den vollen Funktionsumfang des Flash-Protokolls. Die Verwendung vom
  linuxpluginwrapper ist nicht notwendig, da es sich um native Implementationen
  handelt.

-  `graphics/swfdec <https://www.google.com/search?q=graphics/swfdec&btnI=lucky>`__
   und
   `www/swfdec-plugin <https://www.google.com/search?q=www/swfdec-plugin&btnI=lucky>`__
   - Ab der Version 0.5.1 werden auch Flashfilme wie von z.B. YouTube
   angezeigt.
-  `graphics/gnash <https://www.google.com/search?q=graphics/gnash&btnI=lucky>`__
   - Ab der Version 0.8.0 werden auch Flashfilme wie von z.B. YouTube
   angezeigt.
-  `www/plugger <https://www.google.com/search?q=www/plugger&btnI=lucky>`__
   - Das Flash-Plugin lässt von Zeit zu Zeit Firefox abstürzen. Wem
   Firefox durch das Flash-Plugin zu instabil wird, der kann diesen Port
   `ausprobieren <http://www.bsdforen.de/showthread.php?t=4746>`__.

Firefox und externe Anwendungen
-------------------------------

Für die Darstellung spezieller Inhalte können `externe Anwendungen
benutzt werden <http://www.bsdforen.de/showthread.php?t=12215>`__.

Links
-----

-  `Firefox </anwendungen/Firefox>`__ enthält noch weitere Infos.
-  `make.conf </howto/make.conf_optimieren#beispiele_anwendungsfaelle_und_problemloesungen>`__
   optimieren.

* :ref:`genindex`

Zuletzt geändert: |date|

