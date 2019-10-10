Backup auf CD-R
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Wer keinen Streamer hat, nur eine Platte, kein RAID am Laufen oder wer keine
200 Disketten kaufen möchte, der kann ein Backup auch auf CD-ROM machen. Dazu
sollte `sysutils/mkisofs
<https://www.google.com/search?q=sysutils/mkisofs&btnI=lucky>`__ installiert
sein.

Folgende Verzeichnisse könnten beispielsweise gesichert werden:

::

   /home/$USER
   /usr/local/etc
   /etc
   /var
   /boot

Grösse der Verzeichnisse bestimmen
----------------------------------

::
  
  # du -sh /home/$USER

Dies würde die Grösse des gesammten Verzeichnisse ``/home/$USER``
ausgeben, manche Dateien wollen wir aber überhaupt nicht sichern, daher
ist folgender Befehl zu nutzen, und nach Bedarf an die eigenen
Bedürfnisse anzupassen: 

::

  # du -sh -I \\'*iso\' -I \\'*mpg\' -I '*mp3\' /home/$USER

Hierbei wird die Größe des Verzeichnisses bestimmt, wobei Dateien mit
der Endung \*iso, \*mpg oder \*mp3 nicht mit eingerechnet werden. Diese
braucht man nicht wirklich zu sichern bzw. sollte man seine mp3-Dateien
in einem gesondertem Verzeichnis ablegen. Auch \*.core könnte man noch
herausnehmen, da ein Coredump eines Programmes auch nicht wirklich
gesichert werden muß.

Dementsprechend sind auch die anderen Verzeichnisse zu betrachten:

::

  # du -sh /usr/local/etc
  # du -sh /etc
  # du -sh /var
  # du -sh /boot

Angenommen die Grösse der gesamten Dateien würde, wenn diese mittels
``tar`` zusammengeführt würden, auf eine CD passen, können wir wie folgt
weitermachen:

Verzeichnisse archivieren und zippen
------------------------------------

Hierbei ist anzumerken, daß ein Backup nicht unbedingt komprimiert
werden sollte, da nur ein falsches Bit die komplette Sicherung wertlos
machen könnte. Ich werde hier trotz des Mankos die Dateien zippen.

Dazu muss ein Verzeichnis angelegt werden, in das die archivierten und
gezippten Verzeichnisse angelegt werden. Wir nehmen hier:
/storage/backup. 

::

  # tar --gzip --create --verbose --absolute-paths --preserve --exclude \\'*.mpg\' --exclude \\'*iso\' --exclude \\'*.mp3\' --exclude \\'*cache*\' <br>--file /storage/backup/$USER-home-datum.tgz /home/$USER

Hierbei wird das Verzeichnis (OHNE \*.mpg, \*iso und \*.mp3) gepackt und
archiviert.

Die anderen Verzeichnisse wie folgt:

::

  # tar --gzip --create --verbose --absolute-paths --preserve --file /storage/backup/usr_local_etc-datum.tgz /usr/local/etc 
  # tar --gzip --create --verbose --absolute-paths --preserve --file /storage/backup/etc-datum.tgz /etc 
  # tar --gzip --create --verbose --absolute-paths --preserve --file /storage/backup/var-datum.tgz /var 
  #tar --gzip --create --verbose --absolute-paths --preserve --file /storage/backup/etc-datum.tgz /etc

für openbsd: 

::

  # tar -czvpf /ziel/pfad/etc-datum.tgz /etc

Falls ein Verzeichnis im gepacktem Zustand größer sein sollte, als das
Fassungsvermögen einer CD, so kann man diese mit dem Kommando ``split``
aufteilen: 

::

  # split -b 649m mp3dateien.tgz

Daraus entstehen dann mehrere Dateien mit der Grösse von 649 MB die
folgende Bezeichnung haben: xaa, xab xac, ...

Jede dieser Dateien wird dann auch in ein ISO-Image umgewandelt und
gebrannt. Man kann den Befehl ``cat`` nutzen, um die Dateien später
wieder zusammenzufügen und die Datei mp3dateien.tgz zu erhalten. Diese
kann man dann wieder entpacken und schon hat man seine mp3 wieder.

ISO files erstellen
-------------------

Um die Dateien auf CD zu brennen, müssen wir mit dem Programm
``mkisofs`` noch ISO-Dateien erstellen.

Folgende Syntax hat das ganze: 

::

  # mkisofs -o OUTPUTFILE.ISO -P \"Veröffentlicher\" -sysid \"Backup\" -V \"Volume\" -l /DIR/IN/WELCHEM/DIE/GEPACKTEN/DATEIEN/LIEGEN

Für uns sieht das dann also wie folgt aus:

::

  # mkisofs -o /storage/backup_datum.iso -P \"$USERNAME\" -sysid \"backup\" -V \"backup\" -l /storage/backup

Alle Dateien die sich unter ``/storage/backup`` befinden werden unter
``/storage`` zu dem ISO file ``backup_datum.iso`` zusammengefaßt.

Brennen
-------

Dazu gibt es bei IDE-Brennern das Programm ``burncd``, welches zur
Standardinstallation gehört.

Folgende Syntax gibt es hier: 

::

  # burncd -f /dev/acd0c -s 4 data dateiname.iso fixate

-  ``/dev/acd0c`` ist der Brenner
-  ``-s 4`` ist die Geschwindigkeit (4 fach)
-  ``data`` da es sich um Daten und nicht um audio handelt
-  ``fixate`` bedeutet das die CD geschlossen wird

Für uns sieht das wie folgt aus: 

::

  # burncd -f /dev/acd0c -s 4 data /storage/backup_datum.iso fixate

Fertig. Aus. Ende. Das wars.

Sollte das ISO größer sein als das Fassungsvermögen einer CD, so müssen
natürlich mehrere ISO's erstellt werden. Bei den nun gesicherten Daten
ist der Ausfall der Platte zu vekraften. Die Daten des/der Users sind
gesichert, sowie die angepassten Konfigurationsdateien des Systems
(``'/etc``') und die der installierten Programme (``/usr/local/etc``).

* :ref:`genindex`

Zuletzt geändert: |date|

