mkisofs und cdrecord
====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

mkisofs und cdrecord sind Kommandozeilen-Programme. Zusammengenutzt kann man
mit ihnen schnell und einfach Daten-CDs brennen.

Einleitung
----------

`sysutils/mkisofs <https://www.google.com/search?q=sysutils/mkisofs&btnI=lucky>`__
erstellt ein ISO-Image. Dieses ISO-Image kann man dann mit ``cdrecord``
aus
`sysutils/cdrtools <https://www.google.com/search?q=sysutils/cdrtools&btnI=lucky>`__
auf eine CD-R brennen.

ISO-Images erstellen
--------------------

Als erstes muss ein ISO-Image $Image.iso von den gewünschten Daten
$Files mit dem Namen $Name erstellt werden:

::

    # cd /wherer/to/find/the/files
    # mkisofs -R -J -V ${Name} -o ../${Image}.iso ${Files}

Wichtig ist, das sich das Image nicht dort befindet, wo man die Dateien
her kopieren will ('../'). Mit -R wird ein Image vom Type RockRidge
erstellt, da dieser lange Dateinamen und Dateiattribute erlaubt. Als
Ergänzung zum ISO-9660-Format wird das Joliet-Format verwendet, um lange
Dateinamen auf der CD zu speichern. Dies ist das Standardformat von CDs
unter Windows. Die Option -V ${Name} ist nur nötig, wenn man unter
Windows die CD bezeichnet vorfinden will.

CD brennen
----------

Nun wird das Image $Image auf eine CD im Brenner $Device mit der
Geschwindigkeit $Speed gebrannt:

::

    # cd ..
    # cdrecord -dev /dev/${Devive} -speed=${Speed} ${Image}.iso
    # rm ${Image}.iso

Mit der Option dev gibt man das Device seines Brenners in /dev an.
Weiter wird mit der Option speed die Brenngeschwindigkeit auf $Speed
gesetzt. Lässt man die Option weg, dann wird die Geschwindigkeit auf 1
gesetzt. Zum Schluss kann man dann das nicht mehr benötigte Image wieder
löschen.

CD brennen durch die Pipe
~~~~~~~~~~~~~~~~~~~~~~~~~

Ist die Datenquelle auf der Festplatte, dann ist diese so schnell, dass
man das Image direkt an das Brennprogramm schicken kann. Der Vorteil
liegt darin, dass nur 4MB als Puffer vom Plattenspeicher benötigt
werden, statt bis zu 700MB für das Image.

::

    # mkisofs -R /where/to/find/the/files/${Files}  | cdrecord speed=${Speed} dev=${Device} 

CD kopieren
-----------

Schnelle CD-Laufwerke brauchen manchmal länger, um auf Ihre
Zielgeschwindigkeit zu erreichen. In diesem Moment kann der Datenstrom
länger ausbleiben als der Brenner es verkraften kann. Man kann dieses
Problem vermeiden, indem man auf der Festplatte eine Zwischenkopie
anfertigt. Um das Image einer CD zu erstellen, kann man das Programm dd
verwenden:

::

    # dd if=/dev/${cdrom} of=./${Image}.iso
    # cdrecord -v dev=/dev/${Device} speed=${Speed} ./${Image}.iso

Man kann auch direkt auf das Image des Devices als Datenquelle für
cdrecord zu und braucht entsprechend mkisofs nicht. Allerdings ist das
nur empfohlen, wenn das CDROM-Laufwerk wesentlich schneller liest als
der Brenner schreibt ;):

::

    # cdrecord -v dev=${Device} speed=${Speed} -isosize /dev/${cdrom}

Beidesmal ist $cdrom das Quelllaufwerk, welches die CD enthält, die
kopiert werden soll.

Siehe auch
----------

-  `k3b </anwendungen/k3b>`__
-  `xcdroast </anwendungen/xcdroast>`__


* :ref:`genindex`

Zuletzt geändert: |date|

