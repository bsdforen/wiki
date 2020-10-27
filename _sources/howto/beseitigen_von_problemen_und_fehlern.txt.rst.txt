Beseitigen von Problemen und Fehlern
====================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel beschreibt die Ermittlung und Beseitigung von Problemen.

Wo liegt das Problem?
---------------------

Rechner sind komplexe Geräte, somit kann man nie mit Sicherheit aufgrund
weniger Symptome auf die Ursache schließen.

Folgende Punkte sind daher zu beachten:

Treten die Probleme seit einer Änderung der Hardware auf?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In diesem Fall sollte, sofern dies möglich ist, die Hardware-Änderung
rückgängig gemacht werden und geprüft werden, ob der Fehler weiterhin
auftritt. Mögliche Ursachen wären:

-  Defekt an der neuen Hardware
-  Die neue Hardware funktioniert nicht mit der alten Hardware zusammen
-  Der Treiber der neuen Hardware ist fehlerhaft

Treten die Probleme seit einer Änderung der Software auf?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In diesem Fall sollte nach Möglichkeit der alte Stand der Software
mittels Backup wieder hergestellt werden.

Kann das Problem gelöst werden, in dem aktuelle Firmware bei Hardware-Komponenten eingespielt wird?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Immer wieder gibt es Hardware-Komponenten z.B. das Mainboard oder auch
Festplatten-Controller, die mit fehlerbehafteter Firmware ausgeliefert
werden. Um solche Fehler der Firmware auszuschließen, empfiehlt es sich,
auf den Websites der Hersteller nach der akutellen Firmware der
jeweiligen Komponenten umzusehen. Unter Umständen haben Hersteller auch
Foren auf den Sites, in denen man auch schon weitere Hilfe zu Problemen
finden kann.

Häufig auftretende Defekte bzw. Ursachen
----------------------------------------

-  `#Arbeitsspeicher <#Arbeitsspeicher>`__
-  Lose `#Kabelverbindungen <#Kabelverbindungen>`__
-  `#Lüfter <#Lüfter>`__ von CPU, Grafikkarte oder Chipssatz sind
   verschmutzt oder haben Lagerschaden
-  Zu klein dimensionierte `#Netzteile <#Netzteile>`__ (bei zusätzlicher
   Hardware)
-  Unzureichende `#Belüftung <#Belüftung>`__ des Rechners
-  Rechner steht in einem Bereich, indem keine
   `#Luftzirkulation <#Luftzirkulation>`__ stattfinden kann
-  `#Feuchtigkeit <#Feuchtigkeit>`__ in Geräten, z.B. Tastatur

Kabelverbindungen
~~~~~~~~~~~~~~~~~

Kabel wie Netzkabel, Netzwerkkabel, Tastatur- und Mausanschluß sollten
immer als Erstes überprüft werden. In vielen Fällen liegt das Problem an
Wackelkontakten. Im Büro sind Reinigungskräfte ab und an auch nicht
gnädig mit der doch relativ empfindlichen Rechnertechnik.

Die Stromanschlüsse der Festplatten gehen in der Regel sehr schwergängig
oder sehr leichgängig. Bei Letzteren und bei Y-Kabelverlängerungen
sollte mal ggf. mit Kabelbinder die Buchse und den Stecker sichern.

Nach Arbeiten am Rechner wird auch gerne die zusätzliche Stromversorgung
für die CPU oder Grafikkarte vergessen anzuschließen. Wenn man Glück
hat, wird dies vom BIOS der Grafikkarte oder des Motherboards gemeldet.

Arbeitspeicher
~~~~~~~~~~~~~~

Ein defekter Arbeitsspeicher kann zu Abstürzen führen, da aufgrund nicht
mehr konsistenter Daten der Prozessor z.B. unsinnige Befehle oder
Adressen zur Verarbeitung bekommt. Es gibt einige Speichertestprogramme,
aber diese können nicht mit Sicherheit Defekte am Speicher erkennen. In
der Praxis hat sich der Austausch von RAM-Riegeln bewährt, zumal die
Speicherpreise einen Austausch in der Regel erlauben. Sollte die
Probleme mit einem Austausch des Speichers nicht behoben sein, kann ein
Defekt am Speicher zunächst ausgeschlossen werden.

Lüfter
~~~~~~

Lüfter transportieren leider nicht nur Luft, sondern auch Fusseln und
Staub. Einige regelmäßige Kontrolle kann den Hitzetod vermeiden. Lüfter
und Kühlkörper lassen sich gut mit Druckluft reinigen.

Netzteile
~~~~~~~~~

Netzteile müssen ausreichend dimensioniert sein. Es gibt leider auch
Fälle, in denen das Netzteil zwar augenscheinlich funktioniert, aber
tatsächlich nicht ausreichend Strom liefert. In diesem Fall sollte man
ein Ersatznetzteil ausprobieren, um den Fehler zu lokalisieren. Hier
findet man auch Informationen, wie man ein Netzteil
`überprüfen <http://www.pctipp.ch/praxishilfe/kummerkasten/hardware/20737/stromspannung_eines_netzteils_ueberpruefen.html>`__
kann.

Belüftung
~~~~~~~~~

Unter Umständen reicht die Belüftung ausschließlich durch das Netzteil
nicht aus. Es sollte dann ein Zusatzlüfter am Gehäuse angebracht werden.
Hier gilt: Umso größer der Lüfter, desto leiser der Rechner.

Luftzirkulation
~~~~~~~~~~~~~~~

Wenn der Rechner zwischen Schreibtisch und Schrank geklemmt und ggf.
noch mit Büchern oder Gegenständen zugebaut wird, gibt zwangsläufig
Probleme mit der Wärmeabfuhr. Die vom Rechner abgegebene Wärme wird ohne
großen Umweg wieder ins Gehäuse gesogen. Der Kühleffekt liegt naturgemäß
bei Null.

Feuchtigkeit
~~~~~~~~~~~~

Feuchtigkeit ist der Feind jeder Tastatur und der anderen
Rechnerkomponenten. So kann es sein, daß eine Tastatur nach einer
Kaffee- oder Bierdusche schon beim Hochfahren des Rechners, wirre
Zeichen an den Controller liefert und das System beim Hochfahren schon
in Schwierigkeiten bringt.

Rechner stürzt ab und bootet neu
--------------------------------

In diesem Fall sollte folgende Schritte unternommen werden:

-  Mit ``dmesg`` prüfen, ob beim Hochfahren des Rechners Fehler
   angezeigt werden.
-  In **/var/log/messages** nach Ursachen suchen, die zum Absturz
   geführt haben können.
-  Wurden USB-Geräte oder andere Geräte angeschlossen oder abgezogen
   bzw. entfernt?
-  `#Kerneldump <#Kerneldump>`__ überprüfen.

Kerneldump
----------

Beim Absturz versucht das System ein Abbild in der swap-Partition
anzulegen. Dieses kann nach erneutem Hochfahren des Rechners ausgelesen
und ausgewertet werden.

Hierzu für FreeBSD folgende
`Anleitung <http://www.freebsd.org/doc/en_US.ISO8859-1/books/developers-handbook/kerneldebug.html>`__
(zur Zeit leider nur in englisch) lesen.

Dieser Artikel hat leider nicht geholfen, ich will im Forum fragen
------------------------------------------------------------------

Hier können natürlich nicht alle Möglichkeiten der Fehlerbeseitigung
aufgeführt sein. Im Forum sollte man daher zunächst die
**Such-Funktion** benutzen und sehen, ob das Problem unter Unterständen
schon einmal behandelt wurde. Wenn dies nicht der Fall ist, sind
folgende Angaben zum Problem dringend erforderlich, um eine Antwort zu
erhalten:

#. Welches Betriebssystem wird genutzt und welchen Versionsstand hat es
   (``uname -a``)?
#. Was wurde gemacht und wie wurde es probiert?
#. Welche man-Pages, Anleitungen, HowTos habe ich schon gelesen?
#. Welche Fehlermeldungen kommen (cut and paste)?
#. Was steht in den Logdateien (Anschnitt aus dem Logfile (z.B.
   **/var/log/messages** oder ``dmesg``)?
#. Konfigurationsdateien (sofern vorhanden) anhängen, die mit dem
   Problem zusammenhängen!

Testprogramme und -Tools
------------------------

Unter FreeBSD gibt es folgende Programme und Tools zum Testen von
Hardware:

-  `graphics/lcdtest <https://www.google.com/search?q=graphics/lcdtest&btnI=lucky>`__
   - Testmuster für LCD-Monitore anzeigen
-  `sysutils/memtest <https://www.google.com/search?q=sysutils/memtest&btnI=lucky>`__
   - RAM-Testprogramm
-  `sysutils/memtest86 <https://www.google.com/search?q=sysutils/memtest86&btnI=lucky>`__
   - RAM-Testprogramm
-  `sysutils/testdisk <https://www.google.com/search?q=sysutils/testdisk&btnI=lucky>`__
   - Partitionsreparaturprogramm für diverse Partitionstypen
   (http://www.cgsecurity.org/wiki/TestDisk)
-  `sysutils/mbmon <https://www.google.com/search?q=sysutils/mbmon&btnI=lucky>`__
   - Monitor für Temperatur von CPU und Hardware, Geschwindigkeit der
   Ventilatoren im System (CLI-Version)
-  `sysutils/xmbmon <https://www.google.com/search?q=sysutils/xmbmon&btnI=lucky>`__
   - Monitor für Temperatur von CPU und Hardware, Geschwindigkeit der
   Ventilatoren im System (GUI-Version)

<note warning>1. RAM-Testprogramme können einen Fehler aufspüren. Es ist
aber auch möglich, dass ein Speicherfehler nicht erkannt wird!

2. Gerade bei neueren Systemen sind u.U. Chips integriert, die von o.g.
Programmen nicht ausgelesen werden können (mbmon, xmbmon)</note>

Siehe auch
----------

-  `Datenrettung <Datenrettung>`__

* :ref:`genindex`

Zuletzt geändert: |date| 

