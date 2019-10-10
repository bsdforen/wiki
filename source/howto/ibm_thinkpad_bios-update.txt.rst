IBM Thinkpad BIOS-Update
========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Der Artikel zeigt wie bei einem IBM Thinkpad (R51) das BIOS (ohne
Windows oder DOS) aktualisiert. Das verfahren sollte auf alle IBM
Thinkpads übertragbar sein.

BIOS-Updates beschaffen
-----------------------

Zunächst schaut man auf http://www.ibm.com/support/de/ nach, welche
BIOS-/Embedded Controler-Updates für das eigene Thinkpad Modell
verfügbar sind. Die benötigten Updates lädt man herunter (unbedingt die
"Non-Diskette"-Version!), wobei die zusätzlichen Infos zum jeweiligen
Update zu beachten sind (man kann z.B. das BIOS nur zusammen mit einem
passenden Embedded Controler Update auf den neusten Stand bringen;
unbedingt Abhängigkeiten beachten!).

Benötigte Pakete
----------------

Als nächstes muss das Programm ``cabextract`` installiert werden.

-  `archivers/cabextract <https://www.google.com/search?q=archivers/cabextract&btnI=lucky>`__
-  `archivers/cabextract <https://www.google.com/search?q=archivers/cabextract&btnI=lucky>`__
-  `archivers/cabextract <https://www.google.com/search?q=archivers/cabextract&btnI=lucky>`__

ISO-Image erstellen
-------------------

Man geht nun auf der Konsole in das Verzeichnis mit den .exe-Dateien und
führt den Befehl 'cabextract -F \*.IMG 1ruj29ud.exe' für jede
heruntergeladene .exe-Datei aus, wobei der Dateiname 1ruj29ud.exe
natürlich anzupassen ist. Hat das Extrahieren der .IMG-Dateien ohne eine
Fehlermeldung geklappt, können wir mit dem Erstellen der ISO-Images
fortfahren.

Dazu müssen wir den Befehl

::

   mkisofs -b 1RUJ29UD.IMG -o 1RUJ29UD.iso 1RUJ29UD.IMG

für jede der ``.IMG``-Dateien eingeben.

.. note::

  Den Dateinamen ``1RUJ29UD.IMG`` selbstverständlich jeweils anpassen, selbiges
  gilt für den Namen ``1RUJ29UD.iso`` der erstellten Image-Datei!

Die erstellten ISO-Images werden nun der Reihe nach auf CD gebrannt
(z.B. mit cdrecord: 'cdrecord dev=0,0,0 1RUJ29UD.iso', dev= und
1RUJ29UD.iso bitte anpassen!). Selbstverständlich ist es auch möglich,
einen RW-Rohling zu benutzen, den man dann jedoch nach jedem Update
löschen muss.

Update durchführen
------------------

Das Thinkpad muss man nun in der richtigen Reihenfolge von den
erstellten CDs starten lassen, wobei richtige Reihenfolge bedeutet, dass
die Abhängigkeiten der BIOS-/Embedded Controler-Updates berücksichtigt
werden müssen.

Hat der Laptop von der CD richtig gestartet, startet das Update-Programm
von IBM, dessen Benutzerführung sehr einfach zu verstehen ist. Nach
jedem Update schaltet sich der Laptop automatisch aus, sodass man ihn
wieder anschalten muss (nicht vergessen die CD wieder rauszunehmen!).
Zur Sicherheit sollte man im BIOS kontrollieren, ob dort nun die
entsprechende Versionnummer des gerade durchgeführten Updates erscheint.

Bei mir persönlich hat das Aktualisieren des BIOS/Embedded Controler
problemlos mit den selbsterstellten CDs geklappt, jedoch kann ich nicht
versprechen, dass dies bei jedem anderen IBM Thinkpad der Fall ist,
weshalb ich keine Haftung für fehlgeschlagene Updates übernehme. Wenn
Ihr ein Update von CD durchführt, dann macht Ihr dies auf eigene
Verantwortung!

Verweise
--------

-  Inspiriert durch: http://www.thinkwiki.org/wiki/BIOS_Upgrade

* :ref:`genindex`

Zuletzt geändert: |date|

