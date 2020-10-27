CUPS - Installation und Konfiguration
=====================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel behandelt die grundlegende Installation des Common UNIX
Printing System (http://www.cups.org/) und anschließende Einrichtung
eines Druckers. Das HowTo beschreibt hierbei Vorgehensweise zur
Installation eines Brother HL-2030. Der Artikel entstand in Anlehnung an
den nicht mehr verfügbaren Artikel "CUPS unter FreeBSD einrichten" von
http://www.newbie-net.de/

Verwandte Artikel
-----------------

-  `CUPS - Drucken in heterogener
   Umgebung <cups_-_drucken_in_heterogener_umgebung>`__
-  `Wikipedia.de Common Unix Printing
   System <http://de.wikipedia.org/wiki/Common_Unix_Printing_System>`__

Installation der benötigten Pakete
----------------------------------

| Zur korrekten Installation von CUPS ist es ratsam, alle anderen evtl.
  installierten Druckersysteme (z.B. LPD) zu deinstallieren.
| **Folgende Pakete werden benötigt:**

-  cups-base

::

   # cd /usr/ports/print/cups-base/ && make install clean

-  ghostscript8 oder ghostscript7

::

   # cd /usr/ports/print/ghostscript8/ && make install clean

   ODER

   # cd /usr/ports/print/ghostscript7/ && make install clean

**Für nicht postscript-fähige Drucker (und das sind viele):**

-  cups-pstoraster

::

   # cd /usr/ports/print/cups-pstoraster/ && make install clean

-  gimp-gutenprint (über 700 versch. Druckermodelle)

::

   # cd /usr/ports/print/gimp-gutenprint/ && make install clean

-  hpijs (für HP-Drucker)

::

   # cd /usr/ports/print/hpijs/ && make install clean

-  evtl. die foomatic Pakete siehe
   http://www.linux-foundation.org/en/OpenPrinting/Database/Foomatic

::

   # cd /usr/ports/print/foomatic-filters/ && make install clean

Konfiguration
-------------

Nachdem alle Pakete korrekt installiert wurden, müssen ein paar kleine
Änderungen im System vorgenommen werden.

/etc/devfs.rules anpassen
~~~~~~~~~~~~~~~~~~~~~~~~~

<note>Diese Einstellung ist für Netzwerkdrucker nicht notwendig. Weiter
mit `/etc/rc.conf
anpassen <cups_-_installation_und_konfiguration#etcrc.conf_anpassen>`__\ </note>

| Der Brother HL-2030 ist ein USB-Drucker und wird vom System auf
  **/dev/ulpt0** bzw. **/dev/unlpt0** gemappt. Da aber die
  default-Rechte es nicht erlauben, dass auf ihn zugegriffen werden
  darf, müssen die Rechte angepasst werden. Dazu gehört in die Datei
  **/etc/devfs.rules** folgender Eintrag: (Wenn die Datei nicht
  existiert, muss sie angelegt werden.)

::

   [system=10]
   add path 'lpt*' mode 0660 group cups
   add path 'ulpt*' mode 0660 group cups
   add path 'unlpt*' mode 0660 group cups

/etc/rc.conf anpassen
~~~~~~~~~~~~~~~~~~~~~

Der nächste Eintrag hat folgende Bewandtnis:

-  LPD beim Systemstart deaktivieren
-  CUPS-Daemon beim Systemstart aktivieren
-  das ruleset vom vorherigen Kapitel ausführen

::

   lpd_enable="NO"
   cupsd_enable="YES"
   devfs_system_ruleset="system"

/etc/make.conf anpassen
~~~~~~~~~~~~~~~~~~~~~~~

Die folgenden Einträge erfolgen, um einen Konflikt mit dem systemeigenen
*lpd* zu vermeiden:

::

   CUPS_OVERWRITE_BASE=yes
   NO_LPR=yes
   WITH_CUPS=yes

symbolische Links für das CUPS-Backend
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CUPS installiert in **/usr/local/bin** eigene lpr, lpq und lprm. Damit
z.B. lpr korrekt auf das CUPS-Backend zugreifen kann, sollten folgende
symbolische Links gesetzt werden:

::

   # ln -sf /usr/local/bin/lp* /usr/bin

foomatic Skripte downloaden und installieren
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Die meisten von uns werden die foomatic Skripte einsetzen, da sie
benötigt werden, wenn kein postscript-fähiger Drucker zur Verfügung
steht.

::

   # cd /usr/local/bin
   # fetch http://www.linuxprinting.org/foomatic-rip
   # fetch http://www.linuxprinting.org/foomatic-gswrapper
   # chmod 755 foomatic-rip foomatic-gswrapper
   # ln -s /usr/local/bin/foomatic-rip /usr/local/libexec/cups/filter/foomatic-rip

<note important>An dieser Stelle sollte man nun einmal neustarten, damit
alle Einstellungen wirksam werden!</note>

Drucker einrichten unter CUPS
=============================

| Nun kommen wir zur eigentlichen Installation des Druckers. Dafür
  benötigen wir einen Webbrowser (Konqueror, FireFox, o.ä.). Mit
  folgender Adresse gelangt man ins CUPS-Web-Frontend:
  http://localhost:631/
| Sollte hier nichts erscheinen, sollte man sicherstellen, dass cupsd
  korrekt läuft. Ein "ps -aux \| grep cupsd" sollte folgendes
  (ähnliches) Ergebnis liefern:

::

   [root@daemonland ~]# ps -aux | grep cupsd
   root      76202  0.0  0.2  7452  3400  ??  Is    1:50AM   0:00.37 /usr/local/sbin/cupsd

Den CUPS-Daemon kann man auch manuel mit folgendem Befehl
Starten/Stoppen/Neustarten:

::

   /usr/local/etc/rc.d/cupsd [fast|force|one](start|stop|restart|rcvar|reload|status|poll)

   --> # /usr/local/etc/rc.d/cupsd start
   --> # /usr/local/etc/rc.d/cupsd stop

Im Frontend wählt man nun "Add Printer" aus, um einen Drucker
hinzuzufügen.

USB-Drucker
-----------

Für meinen Brother HL-2030 habe ich folgende Einstellungen vorgenommen:

::

   Name: BrotherHL-2030
   Location: /dev/ulpt0
   Description: Brother HL-2030 Laserdrucker

Im nächsten Schritt sollte man, da der HL-2030 ein USB-Drucker ist, als
Device "USB Printer #1 (no reset)" auswählen.

| Im darauffolgenden Fenster kann man nun die PPD ausssuchen, die man
  für seinen Drucker verwenden möchte. Wenn man die foomatic Pakete
  (speziell foomatic-db) installiert hat, kann man sich nun seinen
  Drucker aus der Liste auswählen. Falls nicht, sollte man sich seine
  PPD-Datei aus dem Internet unter
  http://openprinting.org/printer_list.cgi downloaden.
| Das sollte es gewesen sein.

Virtuelle Drucker
-----------------

Ausgabe in eine PDF-Datei
~~~~~~~~~~~~~~~~~~~~~~~~~

Soll ein Ausdruck nicht auf einem Drucker erfolgen, sondern in eine
PDF-Datei, wird der Port /usr/ports/print/cups-pdf installiert:

::

   # cd /usr/ports/print/cups-pdf
   # make PDF_VERSION=1.5 HOME_SUBDIR=Desktop install clean

Bei der Angabe HOME_SUBDIR=Desktop werden die PDF-Dokumente direkt auf
dem Desktop abgelegt.

Folgende Optionen stehen zur Verfügung:

::

   PDF_VERSION=1.2|1.3|1.4|1.5   PDF-version of PDF-files produced
   HOME_SUBDIR=<subdir>          Place produced PDF-files in the
                                 directory ~/<subdir>/
   OUTPUT_DIRECTORY=<dir>        Place produced PDF-files in the
                                 directory <dir>/
   LOG_DIRECTORY=<dir>           Place logfile into <dir>/cups-pdf_log

Nach dem Installieren des Ports muß cups neu gestartet werden.

::

   # /usr/local/etc/rc.d/cupsd restart

Nun kann über das Webinterface der Virtuelle Drucker hinzugefügt werden:

-  URL: http://localhost:631
-  **Drucker hinzufügen** anwählen
-  Authorisieren (als root)
-  Die Felder **Name**, **Ort** und **Beschreibung** können frei gewählt
   werden zB: Name: PDF, Ort: Büro, Beschreibung: Ausdruck in PDF-Datei
-  Im Feld **Gerät** wird: CUPS-PDF (Virtuell PDF Printer) ausgewählt.
-  Im Feld **Oder stellen Sie eine PPD Datei bereit:** wird die Datei
   /usr/local/share/cups/model/CUPS-PDF.ppd ausgewählt und der Drucker
   hinzugefügt.

Nun ist der Virtuelle Drucker installiert.

Abschließende Einstellungen
---------------------------

| Abschließend sollte man unter **Set Printer Options** bzw.
  **Druckeroptionen festlegen** die korrekte Papiergröße (A4 usw) und
  die Auflösung einstellen.
| Über **Print Test Page** bzw **Testseite drucken** kann man nun
  überprüfen, ob alles geklappt hat.

* :ref:`genindex`

Zuletzt geändert: |date| 

