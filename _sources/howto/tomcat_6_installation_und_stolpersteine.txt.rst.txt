Tomcat 6 Installation und Stolpersteine
=======================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Angeblich passt JavaEE und damit Tomcat nicht wirklich zu FreeBSD. Ich
seh keinen Grund was dagegen spricht. Die Installation ist relativ
einfach, wenn man ein paar Stolpersteine gleich zu umschiffen weiß. Gut,
die Installation von Java dauert ein paar Minuten, da das komplette Java
SDK übersetzt werden muss, aber danach hat man dann elegante
Webanwendungen auf einem eleganten Betriebssystem. Bei mir läuft FreeBSD
7 RC1, aber das HowTo sollte sich auf andere \*BSDs übertragen lassen,
insbesondere FreeBSD 6.3. Denn Tomcat ist komplett in Java geschrieben
und für Java gilt ja bekanntlich - Build once, run everywhere. Dieser
Artikel beschreibt nun die Installation von
`www/tomcat6 <https://www.google.com/search?q=www/tomcat6&btnI=lucky>`__,
dann wie eine Webanwendung am besten installiert wird (deployet im
Tomact/JavaEE Slang), und außerdem wie man noch
`java/jdk16 <https://www.google.com/search?q=java/jdk16&btnI=lucky>`__
(Java 6 'Mustang') für Tomcat verwendet. Da ohne das Java SDK beim
Tomcat garnichts geht, sollte bei Problemen zuerst Java korrekt
installiert werden (siehe:
`Java Installation </HowTo/Java Installation>`__).

Installation
------------

Der Port
`www/tomcat6 <https://www.google.com/search?q=www/tomcat6&btnI=lucky>`__
installiert alles nötige mit, so dass man von hier aus make install
starten kann. Das Java SDK wird - falls es noch nicht installiert ist -
gleich mitinstalliert. Da die Installation des Java SDK recht lange
dauert, empfehle ich jedem, der Java noch nicht installiert hat folgende
Vorgehensweise:

::

   [root@kyle ~]# cd /usr/ports/www/tomcat6
   [root@kyle /usr/ports/www/tomcat6]# make fetch-recursive

Dieser Befehl bewirkt, dass das Portssystem erstmal alles zum
Installieren der benötigten Pakete runterläd. Das ist deshalb sinnvoll,
weil die Java-Pakete per Hand (über den Webbrowser, nach abnicken
verschiedener Lizenzen) heruntergeladen werden müssen. Das sind aktuell
vier Pakete die nach /usr/ports/distfiles kopiert werden müssen. Die
Anweisungen dazu, die make fetch-recursive dazu ausspuckt sind aber
deutlich und leicht verständlich. Genau beschrieben wird das ganze auch
nochmal in `Java Installation </HowTo/Java Installation>`__.

Danach geht es damit weiter alle Ports, die installiert werden müssen,
zu konfigurieren:

::

   [root@kyle /usr/ports/www/tomcat6]# make config-recursive

<note important>Für Java nicht IPv6 aktivieren. Andere Ports haben keine
Probleme mit IPv6 Support eingebaut, da läuft IPv6 eher unter
"zusätzlich unterstützt", bei Java gibt das Probleme falls man ein IPv4
Netz hat. Das ist auch schon der erste Stolperstein, Java gleich
schonmal "zukunfstsicher" mit IPv6 auszustatten hindert den Kater
(Tomcat) am Starten.</note>

Die meisten Einstellungen kann man so übernehmen, lediglich "Enable the
browser plugin and Java Web Start" kann man vielleicht abwählen wenn man
eh kein Desktopsystem hat. Sind dann alle Einstellungen vorgenommen
geht's an das Kompilieren. Wer (Java-)Kaffe mag kann sich nun eine Tasse
selbigens holen (ich mag keinen), denn es dauert wie gesagt.

::

   [root@kyle /usr/ports/www/tomcat6]# make && make install

Start und Test
--------------

|Standardseite einer frischen Tomcat 6 Installation|

Ist Tomcat 6 nun inklusive Abhängigkeiten installiert, kann er per

::

   [root@kyle ~]# echo 'tomcat60_enable="YES"' >> /etc/rc.conf

aktiviert, und gestartet werden mit:

::

   [root@kyle ~]# /usr/local/etc/rc.d/tomcat6 start
   Starting tomcat60.

Tomcat lauscht nun standardmäßig auf Port 8180. Das sollte getestet
werden, in einem Browser einfach http://IP.des.ser.vers:8180 aufrufen,
oder alternativ sollte folgender Test auch genügen.

::

   [elmar@kyle ~]$ fetch http://localhost:8180
   localhost:8180 100% of 7347  B 1349 kBps

Eine Webanwendung per Web Archive ``(*.war)`` deployen
------------------------------------------------------

Normalerweise liegt eine Webanwendung als war-Datei vor. In Eclipse z.B.
kann man mit einem einfachen Befehl im Kontextmenü des Projektes (Export
-> War-Archive) diese War-Datei erzeugen, die alle Klassen und
Einstellungen, sowie - falls gewünscht - die Sourcen enthält.

Ich beschreibe hier das statische Deployen eines solchen Web Archivs.

Dafür hält man zuerst den Tomcat an, und kopiert dann einfach die
war-Datei in das Webapps Verzeichnis. Ein Unterverzeichnis zu erstellen
ist **nicht** nötige, das wird bei dem Entpacken automatisch erstellt.
Anschließend wird der Server neugestartet und entpackt nun die neue
Anwendung.

::

   [root@kyle ~]# /usr/local/etc/rc.d/tomcat6 stop
   Stopping tomcat60.
   Waiting (max 10 secs) for PIDS: 95111, 95111, 95111, 95111.
   [root@kyle ~]# cp /tmp/wgkasse.war /usr/local/apache-tomcat6.0/webapps/
   [root@kyle ~]# /usr/local/etc/rc.d/tomcat6 start
   Starting tomcat60.

Für das erste Deployen beim FreeBSD Port ist zu beachten, dass die
Berechtigungen für das Verzeichnis webapps für ein automatisches
Deployen nicht passt. Der Tomcat läuft nämlich standardmäßig als
Benutzer www, Gruppe www, der darf aber nicht in das Verzeichnis webapps
schreiben, und kann somit auch schwer das Web Archive entpacken. Also
setzt man am Besten www als Besitzer des Ordners webapps.

::

   [root@kyle ~]# ll /usr/local/apache-tomcat6.0/ | grep webapps
   drwxr-xr-x  8 root   wheel      512 Jan  7 15:40 webapps/
   [root@kyle ~]# chown www:www /usr/local/apache-tomcat6.0/webapps/
   [root@kyle ~]# ll /usr/local/apache-tomcat6.0/ | grep webapps
   drwxr-xr-x  8 www   www      512 Jan  7 15:40 webapps/

Nun ist die neu eingerichtete Webanwendung unter
http://ip.des.ser.vers:8180/anwendung erreichbar. Bei mir wäre das also
http://kyle:8180/wgkasse/

Fertig deployet (schönes denglisch ;-))

Tomcat auf Port 80 lauschen lassen
----------------------------------

Wer neben dem Tomcat keinen zusätzlichen Webserver betreibt, möchte
vielleicht, dass der Tomcat auf den HTTP Standardport 80 lauscht. Der
Port auf den der Tomcat lauscht wird in der Datei server.xml
eingestellt.

::

   <Connector port="8180" protocol="HTTP/1.1" 
                   connectionTimeout="20000" 
                   redirectPort="8443" />

Hier einfach den Port auf 80 umstellen geht nicht, da Tomcat nach dem
Start sofort als User www läuft und somit (weil er nicht root ist) sich
an keinen Port < 1024 binden darf. Mechanismen wie als root starten,
Port 80 binden und dann die eigenen Rechte beschneiden kennt Tomcat, bzw
Java nicht.

Man kann aber mit Hilfe einer Firewall einfach alle Anfragen auf Port 80
auf Port 8180 umleiten. Mit pf reicht schon folgende Regel voll und ganz
aus:

::

   # pf.conf
   # forward port 80 to port 8180 for tomcat
   rdr pass on fxp0 proto tcp to $tomcat_if port 80 -> $tomcat_if port 8180

Für die weitere Konfiguration von pf sei hier auf das FreeBSD Handbuch,
bzw die OpenBSD FAQ verwiesen. Natürlich lässt sich dieser Portredirect
auch mit IPFW oder sonstigem einstellen.

Java 6 für Tomcat
-----------------

`www/tomcat6 <https://www.google.com/search?q=www/tomcat6&btnI=lucky>`__
hängt ab von
`java/diablo-jdk15 <https://www.google.com/search?q=java/diablo-jdk15&btnI=lucky>`__,
also Java 5. Soll der Tomcat Java 6 Anwendungen verwenden, muss
natürlich erstmal
`java/jdk16 <https://www.google.com/search?q=java/jdk16&btnI=lucky>`__
installiert werden. Wie das geht steht genau beschrieben im `Java
Installations Howto </HowTo/Java Installation>`__, deshalb gehe ich an
dieser Stelle darauf nicht weiter ein. Nur an dieser Stelle nochmal:
Java 6 und Java 5 können friedlich koexistieren. Wenn nämlich die
Umgebungsvariable JAVA_HOME gesetzt ist automatisch die in $JAVA_HOME
installierte Java-Version verwendet. Möchte man JAVA_HOME nicht direkt
in die /usr/local/etc/javavm_opts.conf schreiben, kann man auch Tomcat
die JAVA_HOME über eine Anweisung in der rc.conf mitteilen. Dann
natürlich tomcat neustarten und dann verwendet er Java 6.

::

   [root@eric ~]# echo 'tomcat60_java_home="/usr/local/jdk1.6.0/"' >> /etc/rc.conf

Troubleshooting
---------------

In diesem Abschnitt möchte ich nocheinmal kurz alle "Stolpersteine"
auflisten, ich hab es anscheinend geschafft alle möglichen Fehler zu
machen. Vielleicht hilft dies hier ja jemandem der über Google mit der
Fehlermeldung hier landet.

Als erstes bei Problemen hilft ein Blick in die Dateien
/usr/local/apache-tomcat6.0/logs/stderr.log und stdout.log. stdout.log
enthält dabei auch alle System.out.prinln() Ausgaben aus
Webapplikationen.

Tomcat startet nicht - compat6x (libz.so.3)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

  # stderr.log /libexec/ld-elf.so.1: Shared object "libz.so.3" not found, required by "java"

Das Problem ist kein Tomcatproblem, sondern eines von Java unter FreeBSD
7 allgemein. Die Eingabe von "java" auf der Kommandozeile wirft den
gleichen Fehler. Es fehlen Kompatibilitätslibraries für FreeBSD 6, diese
finden sich in
`misc/compat6x <https://www.google.com/search?q=misc/compat6x&btnI=lucky>`__,
diesen Port einfach installieren.

IPv6 in Java
~~~~~~~~~~~~

::

  # stdout.log SEVERE: StandardServer.await: create[8005]: java.net.BindException: Can't assign requested address

Wird Java mit IPv6 Unterstützung kompiliert, kann Tomcat nicht die
richtige IP Adresse finden auf der er einen Port öffnen und lauschen
soll. Also Java nochmal installieren mit IPv6 deaktiviert (make
deinstall, make clean, make config, make, make reinstall).

Fehler 503
~~~~~~~~~~

Versucht man per Browser auf eine Webanwendung zuzugreifen, und erhält
einen Fehler 503, kann das verschiedene Ursachen habe. Aufschluss gibt
da meistens die stdout.log in der die ungefangenen Exceptions sowie ein
Stacktrace geloggt werden.

::

  # Browserausgabe HTTP Status 503 - This application is not currently
  available description The requested service (This application is not
  currently available) is not currently available

Falsche Berechtigungen für das webapps Verzeichnis
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  # stdout.log INFO: Deploying web application archive wgkasse.war
  Jan 10, 2008 6:16:41 PM org.apache.catalina.startup.ContextConfig init
  SEVERE: Exception fixing docBase: {0}

Wahrscheinlich wurden die Berechtigungen für das webapps Verzeichnis
nicht richtig gesetzt. Überprüfen:

::

   [root@kyle ~]# ll /usr/local/apache-tomcat6.0/ | grep webapps
   drwxr-xr-x  8 root   wheel      512 Jan  7 15:40 webapps/

Und gegebenenfalls korrigieren:

::

   [root@kyle ~]# chown www:www /usr/local/apache-tomcat6.0/webapps/
   [root@kyle ~]# ll /usr/local/apache-tomcat6.0/ | grep webapps
   drwxr-xr-x  8 www   www      512 Jan  7 15:40 webapps/

Falsche Java Version
^^^^^^^^^^^^^^^^^^^^

Eine weitere Fehlerquelle, weshalb der Tomcat eine Webanwendung nicht
auslierft kann eine falsche Javaversion sein.

::

  # stdout.log SEVERE: Error deploying web application archive wgkasse.war
  java.lang.UnsupportedClassVersionError: Bad version number in .class file

Das passiert, wenn man die Webanwendung mit einer höheren Java-Version
kompiliert als auf dem Server vorhanden ist. Man kann z.B. in Eclipse
die Zielplattform von Tomcat 6 mit Java 6 auf Tomcat 6 mit Java 5
umstellen. Alternativ wird auf dem Server die neure Javaversion
installiert und JAVA_HOME für Tomcat 6 gesetzt (siehe oben).

Links
-----

`Tomcat 6
Dokumentation <http://tomcat.apache.org/tomcat-6.0-doc/index.html>`__

`Suns Seite zu JavaEE <http://java.sun.com/javaee/>`__, inklusive
Api-Doc, Tutorials etc., sehr umfangreich

.. |Standardseite einer frischen Tomcat 6 Installation| image:: images/tomcat_standard.jpg
   :class: align-right
   :width: 200px

* :ref:`genindex`

Zuletzt geändert: |date|

