CUPS - Drucken in heterogener Umgebung
======================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Oft wird FreeBSD in einem Windows-Netzwerk eingesetzt.  Dabei möchte man
natürlich auch Drucken können. Diese Kurzanleitung soll erläutern, wie CUPS in
einem Windows-Netzwerk eingesetzt werden kann.

Windows als Druckserver
-----------------------

Am einfachsten lässt sich CUPS mit der KDE-Druckereinrichtung
einrichten, wenn Windows als Druckserver dient:

-  KDE-Druckereinrichtung öffnen
-  Next
-  SMB shared printer (Windows) => Falls nicht anwählbar:

::

   # pkg_add -r samba
   # ln -s /usr/local/bin/smbspool /usr/local/libexec/cups/backend/smb
   # /usr/local/etc/rc.d/cupsd restart

-  Drucker einrichten mit KDE-Druckereinrichtung (selbsterklärend)
-  Drucken!

Ohne KDE
~~~~~~~~

Anleitung zum einrichten des SMB Protokolls
`hier <http://www.cups.org/doc-1.1/sam.html#8_9>`__. Danach einfach als
Drucker Device eine ``smb://`` URI angeben.

FreeBSD als Druckserver
-----------------------

Wenn FreeBSD als Druckserver eingesetzt werden soll, ist es auf jeden
Fall der Einsatz von CUPS als Druckdienst empfehlenswert. Eine
ausführliche Anleitung zur Installation und Konfiguration von CUPS
finden Sie unter:

`cups_-_installation_und_konfiguration <cups_-_installation_und_konfiguration>`__

Auf der Clientseite druckt man entweder direkt über den CUPS-Druckdienst
(IPP) oder man hängt den Windows-Netzwerkfreigabedienst Samba
dazwischen. Samba ist die einfachere Lösung bei Windows
9x/ME/NT-Clients.

IPP
~~~

Windows 2000/XP unterstützen das Drucken über den CUPS-Druckserver. Dazu
wird das Protokoll IPP verwendet. Eine Anleitung finden Sie unter:

`Drucken mit Windows <Drucken mit Windows>`__

`Druckjob abbrechen <http://www.bsdforen.de/showthread.php?p=149947>`__

Samba
~~~~~

Samba selber muss nicht gross konfiguriert werden, da Samba CUPS-Drucker
automatisch erkennt! Als Beispiel meine /usr/local/etc/smb.conf:

::

   [global]
   workgroup = ARBEITSGRUPPE
   server string = Samba Server
   security = user
   load printers = yes
   log file = /var/log/samba/log.%m
   max log size = 50
   passdb backend = tdbsam
   socket options = TCP_NODELAY
   local master = yes
   dns proxy = no

   [homes]
   comment = Home Directories
   browseable = no
   writable = yes
   path = /usr/home/%S/Desktop/Eigenes/Windows 

   [printers]
   comment = All Printers
   path = /var/spool/samba
   browseable = no
   guest ok = no
   writable = no
   printable = yes

CUPS-eigene Druckertreiber
^^^^^^^^^^^^^^^^^^^^^^^^^^

Falls die CUPS-eigenen Druckertreibern verwendet werden (nicht externe
linuxprinting.org-PPD's, nicht GIMP-Print-Treiber etc..) muss man unter
Windows den Drucker gemäss folgender Anleitung installieren:
http://www.tecchannel.de/betriebssysteme/1335/4.html

Eventuell könnte auch der Port
`print/cups-samba <https://www.google.com/search?q=print/cups-samba&btnI=lucky>`__
interessant sein, welchen ich persönlich noch nie testete, da er neu
ist.

CUPS-fremde Druckertreiber
^^^^^^^^^^^^^^^^^^^^^^^^^^

Falls man linuxprinting.org-PPD's verwenden möchte, muss man den Drucker
auf dem CUPS-Server mit dem raw-Treiber installieren!! Bevor man die
Rohdruckdaten durch den CUPS-Server durchschleusen kann, muss man
folgenden Dateien gewisse Zeilen entkommentieren:

``/usr/local/etc/cups/mime.types``:

::

   ################################################### #####################

     1.
     2. Raw print file support...
     3.
     4. Uncomment the following type and the application/octet-stream
     5. filter line in mime.convs to allow raw file printing without the
     6. -oraw option.
     7.

   application/octet-stream

``/usr/local/etc/cups/mime.convs``:

::

   ################################################## ######################

     1.
     2. Raw filter...
     3.
     4. Uncomment the following filter and the application/octet-stream type
     5. in mime.types to allow printing of arbitrary files without the -oraw
     6. option.
     7.

   application/octet-stream application/vnd.cups-raw 0 -

Wenn man den Raw-Druckertreiber einsetzt, muss man auf den
Windows-Rechner die entsprechenden Windows-Druckertreibern installieren
(wie als wäre der Drucker am Windows-Rechner angeschlossen). Dabei
hilft:

-  http://www.tecchannel.de/server/linux/402263/index11.html
-  http://www.tecchannel.de/server/linux/402263/index12.html

Weiterführende Literatur
------------------------

-  `Samba und CUPS
   (html-Version) <http://www.linuxprinting.org/kpfeifle/SambaPrintHOWTO/Samba-HOWTO-Collection-3.0-PrintingChapter-11th-draft.html>`__
-  `Samba und CUPS
   (pdf-Version) <http://www.linuxprinting.org/kpfeifle/SambaPrintHOWTO/Samba-HOWTO-Collection-3.0-PrintingChapter-11th-draft.pdf>`__
-  `Turboprint und Drucken von Windows-Rechnern
   aus <http://bjou.homeunix.net/blog/?p=36>`__

* :ref:`genindex`

Zuletzt geändert: |date| 

