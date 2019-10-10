Drucken mit Apsfilter
=====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Dieser Artikel erklärt wie unter FreeBSD mit Hilfe von Apsfilter einfach mit
dem Lpd ohne CUPS gedruckt werden kann. Außerdem können Drucker über das
Netzwerk anderen Rechnern zugänglich gemacht werden.

Einleitung
----------

Im Gegensatz zum `CUPS </howto/CUPS>`__ Artikel beschreibt dieser
Artikel eher ein vorgehen wie es im `FreeBSD
Handbuch <http://www.freebsd.org/doc/handbook/printing.html>`__
vorgeschlagen wird. Vorteile vom Verzicht auf CUPS sind vor allem der
deutlich geringere Aufwand bei der Installation, eine sehr einfach zu
realisierende Samba Freigabe und, dass keine Dateien in **/usr/bin** und
**/usr/sbin** ersetzt werden müssen.

Installation und Einrichtung
----------------------------

Als erstes sollte der Spooler Daemon aktiviert werden, dazu muss
folgende Zeile in die Datei **/etc/rc.conf** eingefügt werden:

::

   lpd_enable="YES"

Mit dem Kommando:

::

   # /etc/rc.d/lpd start

kann der Daemon ohne einen Systemneustart aktiviert werden.

Bis auf den Port
`print/apsfilter <https://www.google.com/search?q=print/apsfilter&btnI=lucky>`__
werden keine zusätzlichen Programme benötigt. Nach erfolgter
installation muss folgender Befehl ausgeführt werden:

::

   # /usr/local/share/apsfilter/SETUP

Es startet ein Skript, das durch die vollständige Druckerkonfiguration
führt. Die Schritte sind wie folgt.

-  Lizenz bestätigen.
-  Der Autor bittet per Mail über die Verwendung von apsfilter
   informiert zu werden. Dieser Punkt kann übersprungen und später
   nachgeholt werden.
-  Als nächstes erscheint ein Begrüßungsbildschirm, mit der Enter Taste
   beginnt endlich die eigentliche Einrichtung.

Ab hier sollte alles genau gelesen und befolgt werden. Die
Konfigurationsschritte sollten in der gegebenen Reihenfolge abgearbeitet
werden. Apsfilter hat keinerlei Standardeinstellungen, also darf kein
Schritt übersprungen werden. Nach erfolgter Konfiguration sollte auf
jeden Fall von der Option eine Testseite zu drucken gebrauch gemacht
werden. Wird die Testseite falsch gedruckt, passt der gewählte Treiber
nicht zum Drucker. Der Autor dieses Artikels verwendet einen LaserJet 4
Treiber für einen HP LaserJet 6. Die LaserJet 5 und 6 Treiber führen
beim Autor zu keinem erkennbaren Druckbild.

Sollte der Drucker beim Versuch eines Testdrucks keinerlei Reaktion
zeigen, sollte noch einmal die gesamte Konfiguration geprüft werden. Das
gilt insbesondere für die Anschlusseinstellungen.

Wer weitere Konfigurationen vornehmen will, kann dies in der Datei
**/etc/printcap** erledigen. Dies ist jedoch in der Regel nicht nötig.

Erste Druckversuche
-------------------

Einfache Textdateien oder Postscript Dokumente sollten jetzt über einen
einfachen Aufruf funktionieren. Zum Testen kann zum Beispiel eine kurze
Konfigurationsdatei ausgedruckt werden.

::

   # lpr /etc/make.conf

Mit vorherigem Kommando sollte zum Beispiel die make.conf ausgedruckt
werden. Es empfiehlt sich auch das Drucken mit häufig verwendeten
Anwendungen zu testen. Da die meisten Anwendungen ihre Ausgabe in
Postscript an den Drucker Daemon weitergeben bereitet dies für
gewöhnlich keinerlei Probleme.

Die Warteschleife des Druckers kann mit dem Befehl:

::

   # lpq

eingesehen werden. Mit dem Parameter **-P** kann ein Drucker explizit
angegeben werden, das ist dann Sinnvoll, wenn mehrere Drucker verwendet
werden.

Um einen Druckauftrag zu Löschen kann der Befehl **lprm** verwendet
werden. Um die Druckaufträge anderer Benutzer zu manipulieren werden
Root Zugriffsrechte benötigt.

Drucken über das Netz
---------------------

Netzdrucke sind recht einfach zu realisieren. Clients benötigen im
übrigen weder Apsfilter noch Druckertreiber um über das Netz zu drucken.
Es sei denn es handelt sich um Windows Clients.

Drucker Freigeben (Server)
~~~~~~~~~~~~~~~~~~~~~~~~~~

Die einfachste Möglichkeit Drucker freizugeben ist in der Datei
**/etc/hosts.lpd** die IP Adressen oder Hostnamen der Rechner von denen
aus gedruckt werden darf einzufügen. Dazu wird schlicht in jede Zeile
eine IP Adresse oder ein Hostname eingetragen.

Auch die Samba Konfiguration ist realtiv trivial. Im folgenden ist die
Konfigurationsdatei **/usr/local/etc/smb.conf** des Autors angegeben.
Daraus sollte eigentlich klar werden wie die eigene Konfiguration zu
erweitern ist.

::

   [global]
   # workgroup = NT-Domain-Name or Workgroup-Name, eg: MIDEARTH
   workgroup = NORAD

   # server string is the equivalent of the NT Description field
   server string = homeKamikaze Samba Server

   # Security mode. Defines in which mode Samba will operate. Possible 
   # values are share, user, server, domain and ads. Most people will want 
   # user level security. See the Samba-HOWTO-Collection for details.
   security = share

   # This option is important for security. It allows you to restrict
   # connections to machines which are on your local network. The
   # following example restricts access to two C class networks and
   # the "loopback" interface. For more examples of the syntax see
   # the smb.conf man page
   hosts allow = 192.168.0. 192.168.1. 127.

   # If you want to automatically load your printer list rather
   # than setting them up individually then you'll need this
   load printers = yes

   # It should not be necessary to specify the print system type unless
   # it is non-standard. Currently supported print systems include:
   # bsd, cups, sysv, plp, lprng, aix, hpux, qnx
   printing = bsd

   # this tells Samba to use a separate log file for each machine
   # that connects
   log file = /var/log/samba/log.%m

   # Put a capping on the size of the log files (in Kb).
   max log size = 50

   # NOTE: If you have a BSD-style print system there is no need to 
   # specifically define each individual printer
   [printers]
   comment = All Printers
   path = /var/spool/samba
   browseable = no
   # Set public = yes to allow user 'guest account' to print
   guest ok = yes
   writable = no
   printable = yes

Wichtig ist hierbei, dass die Sektion für die Drucker **printers**
heißt, da das Drucken sonst nicht funktionieren wird.

Drucker verwenden (Client)
--------------------------

Ein Drucker Client kommt vollständig mit Mitteln aus dem Basissystem
aus. Leider bedeutet das auch, dass einige Arbeiten, die auf dem Server
vom Apsfilter Setup erledigt wurden, auf dem Client manuell erfolgen
müssen.

Zuerst müssen die Spooler Verzeichnisse (Für die lokale
Druckwarteschleife) angelegt werden:

::

   # mkdir -p /var/spool/lpd
   # chown root:daemon /var/spool/lpd
   # chmod 0755 /var/spool/lpd
   # cd /var/spool/lpd
   # mkdir DRUCKER

**DRUCKER** ist hierbei mit dem Namen zu ersetzen, mit dem der Drucker
dann im folgenden Schritt in der **/etc/printcap** konfiguriert wird. Im
folgenden ist die Konfiguration für einen HP LaserJet 6, der als ljet6
über das Netz freigegeben ist angegeben:

::

   lp|DRUCKER|HP LaserJet 6L|ljet4;r=600x600;q=photo;c=gray;p=a4;m=auto:\
       :lp=:\
       :rm=HOST:\
       :rp=REMOTE_DRUCKER:\
       :sd=/var/spool/lpd/DRUCKER:

Die erste Zeile kann einfach von der Serverkonfiguration kopiert werden.
**HOST** muss natürlich mit der IP Adresse des Servers oder dessen
Hostnamen ersetzt werden. **DRUCKER** wäre im Beispiel als ljet6 zu
setzen. Theoretisch kann der Drucker auf dem Client und auf dem Server
unterschiedlich benannt werden. **REMOTE_DRUCKER** sollte also mit dem
Namen des Druckers auf dem Server ersetzt werden. Üblicherweise ist die
Benennung jedoch gleich. Der Name **lp** steht immer für den
Standarddrucker.

Nun muss lediglich noch der Lpd, wie bereits im Kapitel Installation
beschrieben auch auf dem Client eingerichtet werden. Damit ist die
Konfiguration abgeschlossen. Drucken sollte jetzt ohne weiteres möglich
sein.

Verweise
--------

-  Die Apsfilter `Homepage <http://www.apsfilter.org>`__.
-  Das Apsfilter
   `Handbuch <http://apsfilter.org/docs/apsfilter-handbook-stable.html>`__.
-  Das Kapitel
   `Drucken <http://www.freebsd.org/doc/handbook/printing.html>`__ im
   FreeBSD Handbuch.
-  `print/apsfilter <https://www.google.com/search?q=print/apsfilter&btnI=lucky>`__
   in den FreeBSD Ports.
-  **printcap(5)**
-  **hosts.lpd(5)**

* :ref:`genindex`

Zuletzt geändert: |date|

