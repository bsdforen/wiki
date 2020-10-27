Java Installation
=================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Java bezeichnet sowohl eine Technologie als auch eine Programmiersprache
von Sun Microsystems. Das besondere an Java dabei ist, dass beim
Kompilieren kein Maschinencode erzeugt wird, sondern Bytecode, der dann
von einer Virtuellen Maschine (der JVM - Java Virtual Machine)
interpretiert wird. Somit läuft Java auf unterschiedlichen Hard- und
Softwareplattformen ohne Neukompilieren der Sourcen, es muss lediglich
eine Java Runtime Environment vorhanden sein, grob gesagt die virtuelle
Maschine. Da Java (also die Technologie) lange Zeit (bis 8. Mai 2007)
nicht frei war, gestaltet sich die Installation etwas aufwändiger als
bei anderen Ports - Verfügbarkeit von Binärpaketen, kein direkter
Download von Sourcen als Beispiel. Aktuell (Januar 2008) gibt es
immernoch kein komplett freies Java, an Teilen des Java-Codes hält Sun
nicht alle Rechte, so dass diese nicht freigegeben werden können, und
echten Ersatz für diese Bestandteile gibt es noch nicht. Früher war Java
vor allem wegen der Applets (Applikationen innerhalb eines Browsers)
beliebt, die heute allerdings stark von Ajax-Anwendungen verdrängt
werden. Trotzdem gibt es viele Gründe sich Java zu installieren, sei es
Eclipse, eine IDE, die etliche Programmiersprachen unterstützt,
Java-Programme die genutzt werden sollen, oder das Interesse am Erlernen
von Java. Java wird immerhin nicht nur Informatikern, sondern auch in
vielen anderen Studiengängen (VWL, Maschinenbau etc.) als
Programmiersprache gelehrt, meiner Meinung nach zu Recht, da es quasi
die einfachste Objektorientierte Programmiersprache ist, mit der man
auch praktisch "was anfangen kann". Die Installation von Java unter
FreeBSD, sowie die Unterschiede der Java-Versionen in den Ports sollen
hier erklärt werden.

Quickstart
----------

Wer ohne viel Hintergrundinfos lesen zu wollen einfach schnell "Java
installieren" will, springt zu "Installation des Diablo JDK".

JDK vs JRE
----------

JDK steht für Java Development Kit, JRE für Java Runtime Environment.
Der Name sagt es, die Laufzeitumgebung (JRE) braucht man, wenn man nur
Javaprogramme ausführen will. Es enthält die oben beschriebene virtuelle
Maschine, die den Bytecode interpretiert, sowie "was sonst noch dazu
benötigt wird". Das JDK enthält dabei sowohl die gesamte JRE, sowie
zusätzlich z.B. den Javacompiler und noch andere Werkzeuge die zum
Entwickeln von Javaprogrammen benötigt werden. D.h. also, das JDK
alleine reicht aus, die JRE wird nicht zusätzlich benötigt. Wer
Festplattenplatz und Bandbreite schonen will (es handelt sich um ca.
20-30 MB Unterschied zwischen beiden Versionen), nimmt das JRE, das JDK
installiert zu haben schadet aber auch nicht (wer weiß wann man es
braucht). Einige Ports hängen außerdem von JDK und nicht vom JRE ab, so
dass man eventuell später das JDK nochmal installieren muss.

Java 1.5 vs Java 5 bzw. Java 1.6 vs Java 6
------------------------------------------

Der Unterschied ist fix erklärt. Marketing. Java 5 hört sich
fortgeschrittener an als Java 1.5, deshalb wird seit Version 1.5 das
"1." weggelassen. Und so heißt Java 1.6 jetzt auch Java 6. Die "1."
Versionierung begegnet einem aber noch sehr oft.

Diablo Java vs Java
-------------------

Da Java wie beschrieben nicht frei ist, sind ein paar Restriktionen an
die Benutzung geknüpft. So darf z.B. nicht jeder hingehen und
Javabinärpakete einfach so weiterverteilen, d.h. für FreeBSD, dass Java
JDK (ebenso JRE) Pakete nicht wie andere Pakete per FTP frei verteilt
werden können. Damit aber nicht jeder der Java benutzen möchte
ersteinmal lange kompilieren muss, hat die FreeBSD Foundation im April
2006 mit Sun eine spezielle Lizenz ausgehandelt, damit offizielle Java
Binärpakete zum Download bereit gestellt werden dürfen (nach abnicken
einer Lizenz). Um den Status einer offiziellen Java Distribution zu
erhalten sind die Pakete mit Hilfe des "Sun compatibility test"
getestet. Diablo Java ist also einfach nur eine Javadistribution der
FreeBSD Foundation mit offiziellem Segen von Sun.

Java Vielfalt in den Ports
--------------------------

In den Ports befinden sich unter
`java/ <https://www.google.com/search?q=java/&btnI=lucky>`__ etliche
Ports, die alle mehr oder weniger passend erscheinen um Javaprogramme
auszuführen oder zu entwickeln.

-  `java/diablo-jre15 <https://www.google.com/search?q=java/diablo-jre15&btnI=lucky>`__:
   Javadistribution der FreeBSD Foundation (nur JRE), der Port um selber
   das Paket zu bauen
-  `java/diablo-jdk15 <https://www.google.com/search?q=java/diablo-jdk15&btnI=lucky>`__:
   wie
   `java/diablo-jre15 <https://www.google.com/search?q=java/diablo-jre15&btnI=lucky>`__,
   allerdings das gesamte JDK
-  `java/jdk16 <https://www.google.com/search?q=java/jdk16&btnI=lucky>`__:
   Port um Suns JDK 1.6 selber zu kompilieren
-  `java/jdk16-doc <https://www.google.com/search?q=java/jdk16-doc&btnI=lucky>`__:
   JDK Dokumentation, interessant nur für Java-Entwickler, enthält die
   Api-Referenz, Tutorials etc.

Die jdk\* Ports gibt es dabei noch in verschiedenen Versionen, von 13
bis 16. Reine JRE Pakete sind in den Ports - bis auf das diablo-jre15 -
aktuell nicht vorhanden.

Java Sourcen obwohl Java unfrei ist?
------------------------------------

Vielleicht wundert der ein oder andere sich nun, wie man Java
kompilieren kann, wenn Java doch unfrei ist. Nun, unter einer
`Forschungslizenz <http://www.java.net/jrl.csp>`__ kann man die Java
Sourcen und alles weitere was man zum Bauen von Java braucht bei Sun
herunterladen.

Installation
------------

Nun stellt sich natürlich die Frage, wer braucht welche Version. Wer
Javaprogramme und Applets ausführen will, oder vielleicht auch Java
programmieren will nimmt das Diablo JDK. Wer auf die aktuelle Version
Java 6 angewiesen ist installiert zusätzlich den Port
`java/jdk16 <https://www.google.com/search?q=java/jdk16&btnI=lucky>`__.

Installation des Diablo JDK
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Von der `FreeBSD
Foundation <http://www.freebsdfoundation.org/downloads/java.shtml>`__
muss das passende diablo-jdk heruntergeladen werden, im Abschnitt
"End-User Downloads" unter "Packages" gibt es aktuell

-  diablo-jdk-freebsd5.i386.1.5.0.07.01.tbz: Für das Diablo JDK unter
   FreeBSD 5.5 auf i386
-  diablo-jdk-freebsd6.amd64.1.5.0.07.01.tbz: Für das Diablo JDK unter
   FreeBSD 6.1 auf amd64
-  diablo-jdk-freebsd6.i386.1.5.0.07.01.tbz: Für das Diablo JDK unter
   FreeBSD 6.1 auf i386

Die JRE Pakete würde ich nur installieren wenn Bandbreite und
Festplattenplatz extrem knapp sind.

::

  pkg_add diablo-jdk-freebsd\*

Mit neueren FreeBSD Versionen (hier 7 RC1) scheitet das unter Umständen
an anderen Paketen die fehlen, wie z.B. xorg 6.9. Dann muss das Diablo
JDK über die Ports installiert werden. Dafür müssen auch wieder händisch
Dateien runtergeladen werden (Lizenzen müssen akzeptiert werden) und
nach /usr/ports/distfiles kopiert werden. Konkret sind das:

-  `diablo-caffe-freebsd6-i386-1.5.0_07-b01.tar.bz2 <http://www.FreeBSDFoundation.org/cgi-bin/download?download=diablo-caffe-freebsd6-i386-1.5.0_07-b01.tar.bz2>`__
   von der FreeBSD Foundation
-  `tzupdater-1.3.3-2007k.zip <http://java.sun.com/javase/downloads/index.jsp>`__
   von Sun. Hierfür muss man sich (falls noch nicht geschehen) bei Sun
   registrieren und dann einloggen.

Diese Links hier sind wahrscheinlich recht schnell veraltet (bitte dann
aktualisieren, das ist ein Wiki ;)), aber bei der Installation des Ports
werden die aktuellen Links mit den passenden Instruktionen verständlich
angezeigt.

::

  [root@kyle /usr/ports/java/diablo-jdk15]# make install
  ===>  diablo-jdk-1.5.0.07.01_9 :
   Because of licensing restrictions, you must fetch the distribution
   manually.
  
   Please access
  
   http:www.FreeBSDFoundation.org/cgi-bin/download?download=diablo-caffe-freebsd6-i386-1.5.0_07-b01.tar.bz2
  
   with a web browser and „Accept“ the End User License Agreement for
   „Caffe Diablo 1.5.0“.  Please place the downloaded
   diablo-caffe-freebsd6-i386-1.5.0_07-b01.tar.bz2 in /usr/ports/distfiles.
  
   Please open http://java.sun.com/javase/downloads/index.jsp
   in a web browser and follow the „Download“ link for
   „JDK US DST Timezone Update Tool - 1.3.3“ to obtain the
   time zone update file, tzupdater-1.3.3-2007k.zip.
  
  .*** Error code 1
  
  Stop in /usr/ports/java/diablo-jdk15.

OK, der Port beschwert sich, dass die Dateien die er braucht nicht da
sind, herunterladen und nach /usr/ports/distfiles kopieren:

::

  [root@kyle /usr/ports/java/diablo-jdk15]# scp elmar@tweek:/tmp/tzupdater\* /usr/ports/distfiles/
  elmar@tweek's password:
  tzupdater-1.3.3-2007k.zip                                         100%  258KB 257.5KB/s   00:00
  [root@kyle /usr/ports/java/diablo-jdk15]# scp elmar@tweek:/tmp/diablo-caffe\* /usr/ports/distfiles/
  elmar@tweek's password:
  diablo-caffe-freebsd6-i386-1.5.0_07-b01.tar.bz2                   100%   52MB   7.4MB/s   00:07

Wobei ich die Dateien auf einem anderen Rechner heruntergeladen habe.
Danach geht's dann weiter mit einem make und make install.

::

  [root@kyle /usr/ports/java/diablo-jdk15]# make

Das "make" an sich dauert nicht lange (keine Minute auf einem Pentium
III, 1 Ghz hier), da die Datei diablo-caffe bereits die Java Binaries
enthält. Allerdings kann es sein, dass noch abhängige Ports vorher
kompiliert werden müssen. Das dauert dann länger (23 Minuten auf dem P3
hier), da da noch einiges an X11 Kram mitkommt. Danach ein make install
und dann sollte ein "java -version" eine ähnliche Ausgabe liefern:

::

  [root@kyle /usr/ports/java/diablo-jdk15]# java -version
  java version "1.5.0"
  Java(TM) 2 Runtime Environment, Standard Edition (build diablo-1.5.0-b01)
  Java HotSpot(TM) Client VM (build diablo-1.5.0_07-b01, mixed mode)

Java 5 ist nun installiert und Einsatzbereit.

Installation des JDK 1.6
~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

  Das Kompilieren vom JDK 1.6 dauert lange und benötigt außerdem mindestens 2.5
  GB (Angabe stammt vom Port selbst) freien Speicher auf der Partition auf der
  es gebaut wird.

Wer Java 6 braucht kann nicht auf das Diablo JDK zurückgreifen, da es
aktuell nur für Version 5 verfügbar ist. Hierfür gibt es den Port
`java/jdk16 <https://www.google.com/search?q=java/jdk16&btnI=lucky>`__.
Für die Installation müssen 3 Dateien heruntergeladen werden und in
/usr/ports/distfiles abgelegt werden.

-  Update 3 Source, jdk-6u3-fcs-src-b05-jrl-24_sep_2007.jar
-  Source Binaries, jdk-6u3-fcs-bin-b05-jrl-24_sep_2007.jar
-  Mozilla Headers, jdk-6u3-fcs-mozilla_headers-b05-unix-24_sep_2007.jar

jeweils von
`Sun <http://download.java.net/jdk6/6u3/promoted/b05/index.html>`__.
Hierfür ist inzwischen keine Registrierung mehr notwendig.

-  `bsd-jdk16-patches-3.tar.bz2 <http://www.eyesbeyond.com/freebsddom/java/jdk16.html>`__
   Patches für Java unter FreeBSD

Wie bei den Links für das Diablo JDK gilt hier: Die Links werden sich
mit der nächsten Java Version sicher ändern, also schnell nichtmehr
aktuell sein. Bei der Installation wird aber wiederum deutlich
beschrieben welche Dateien wo heruntergeladen werden müssen.

Das Bauen des Ports dauert nun erheblich länger als beim
`java/diablo-jdk15 <https://www.google.com/search?q=java/diablo-jdk15&btnI=lucky>`__,
da nun das JDK komplett kompiliert werden muss (3 Stunden auf einem
Pentium 3, 1 Ghz hier). Aber der Ablauf ist gleich, bzw wie bei allen
Ports.

::

  [root@kyle ~]# cd /usr/ports/java/jdk16 [root@kyle
  /usr/ports/java/jdk16]# make ===> jdk-1.6.0.3p3_1 : Due to licensing
  restrictions, certain files must be fetched manually.
  
  Please open http://download.java.net/jdk6/6u3/promoted/b05/index.html in
  a web browser. Download the Update 3 Source,
  jdk-6u3-fcs-src-b05-jrl-24_sep_2007.jar and the Source Binaries,
  jdk-6u3-fcs-bin-b05-jrl-24_sep_2007.jar and the Mozilla Headers,
  jdk-6u3-fcs-mozilla_headers-b05-unix-24_sep_2007.jar .
  
  Please download the patchset, bsd-jdk16-patches-3.tar.bz2, from
  http://www.eyesbeyond.com/freebsddom/java/jdk16.html.
  
  Please place the downloaded file(s) in /usr/ports/distfiles and restart
  the build.
  
  .**\* Error code 1
  
  Stop in /usr/ports/java/jdk16.

Die Pakete runterladen und nach /usr/ports/distfiles schieben, und make
starten.

::

  [root@kyle /usr/ports/java/jdk16]# scp
  elmar@tweek:/tmp/jdk16/\\* /usr/ports/distfiles/
  elmar@tweek.oxy.homelinux.org's password: bsd-jdk16-patches-3.tar.bz2
  100% 926KB 926.2KB/s 00:00 jdk-6u3-fcs-bin-b05-jrl-24_sep_2007.jar 100%
  2067KB 2.0MB/s 00:00
  jdk-6u3-fcs-mozilla_headers-b05-unix-24_sep_2007.jar 100% 8406KB 8.2MB/s
  00:01 jdk-6u3-fcs-src-b05-jrl-24_sep_2007.jar 100% 111MB 7.4MB/s 00:15
  [root@kyle /usr/ports/java/jdk16]# make

Moment, noch nicht vom Rechner weglaufen, es muss erst noch die Lizenz
akzeptiert werden! Also die Lizenz lesen, und dann antworten.

::

  Do you agree to the above license terms? [yes or no] y

Jetzt wird der Port gebaut.

Das ganze installieren und testen.

::

   [root@kyle /usr/ports/java/jdk16]# make install
   [root@kyle /usr/ports/java/jdk16]# java -version
   java version "1.6.0_03-p3"
   Java(TM) SE Runtime Environment (build 1.6.0_03-p3-elmar_20_jan_2008_15_34-b00)
   Java HotSpot(TM) Client VM (build 1.6.0_03-p3-elmar_20_jan_2008_15_34-b00, mixed mode)

Falls hier die "falsche" Version angezeigt wird, keine Panik, dann ist
noch eine andere JavaVM installiert.

Verschiedene Java Versionen parallel
------------------------------------

Verschiedene Javaversionen lassen sich unter FreeBSD sehr einfach
parallel nutzen. Ein Blick auf die Programme wie java oder javac zeigt,
dass diese nur Links auf das Shellscript javavm sind.

::

  [elmar@kyle ~]$ ll \`which java\` && ll \`which javac\`
  lrwxr-xr-x 1 root wheel 21 Jan 5 13:38 /usr/local/bin/java@ ->
  /usr/local/bin/javavm lrwxr-xr-x 1 root wheel 21 Jan 5 13:38
  /usr/local/bin/javac@ -> /usr/local/bin/javavm

JavaVM liefert auch gleich die Erklärung warum.

::

  [elmar@eric ~]$ head -n 7 /usr/local/bin/javavm \| tail -n 5 #
  javawrapper.sh # # Allow the installation of several Java Virtual
  Machines on the system. # They can then be selected from based on
  environment variables and the # configuration file.

Ja super, das macht doch alles sehr einfach. Die Umgebungsvariable heißt
nämlich JAVA_HOME und muss auf das jeweilige Basisverzeichnis der Java
Installation zeigen.

::

  [elmar@kyle ~]$ echo $JAVA_HOME
  [elmar@kyle ~]$ java -version java version "1.5.0" Java(TM) 2 Runtime
  Environment, Standard Edition (build diablo-1.5.0-b01) Java HotSpot(TM)
  Client VM (build diablo-1.5.0_07-b01, mixed mode) [elmar@kyle ~]$ export
  JAVA_HOME=/usr/local/jdk1.6.0/ [elmar@kyle ~]$ java -version java
  version "1.6.0_03-p3" Java(TM) SE Runtime Environment (build
  1.6.0_03-p3-elmar_19_jan_2008_23_15-b00) Java HotSpot(TM) Client VM
  (build 1.6.0_03-p3-elmar_19_jan_2008_23_15-b00, mixed mode)

Verfügbare Java Versionen stehen dabei in der Datei
/usr/local/etc/javavms

::

   /usr/local/jdk1.6.0/bin/java # FREEBSD-JDK1.6.0
   /usr/local/diablo-jdk1.5.0/bin/java # DiabloCaffe1.5.0

Systemweit kann JAVA_HOME auch bequem in der Datei
/usr/local/etc/javavm_opts.conf gesetzt werden.

::

  [root@kyle ~]# echo "JAVA_HOME=/usr/local/jdk1.6.0/" >> /usr/local/etc/javavm_opts.conf

* :ref:`genindex`

Zuletzt geändert: |date|

