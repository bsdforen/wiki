DVD-Rip mit mencoder
====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Dieser Artikel erklärt wie mit mencoder und einem Skript komfortabel DVDs in
XviD kodierte avi Filme umgewandelt werden.

Programme
---------

Benötigt werden `mplayer </anwendungen/mplayer>`__ und mencoder. Unter
FreeBSD werden die Ports
`multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
und
`multimedia/mencoder <https://www.google.com/search?q=multimedia/mencoder&btnI=lucky>`__
benötigt. In Pkgsrc gibt es die Pakete
`multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
und
`multimedia/mencoder <https://www.google.com/search?q=multimedia/mencoder&btnI=lucky>`__.
Die Arbeit im Vorfeld des Rip geht mit einer graphischen Oberfläche
(GUI) leichter von Hand. In Beispielen wird fortan die GTK oberfläche
vorausgesetzt die mit dem Befehl gmplayer aufgerufen wird. Alternative
Oberflächen sollten äquivalent funktionieren.

Installation des Skripts
------------------------

Der empfohlene Installationsort des Skripts ist in **$HOME/bin**, dort
ist es nur im **PATH** des entsprechenden Nutzers vorhanden. Das Skript
kann direkt aus dem Wiki exportiert werden.

::

   # mkdir -p ~/bin
   # fetch -o ~/bin/ripXviD 'http://wiki.bsdforen.de/_export/code/howto/dvd-rip_mit_mencoder?codeblock=3'
   # chmod +x ~/bin/ripXviD

Nach der Installation muss je nach Shell möglicherweise das Kommando
``rehash`` ausgeführt werden.

Vorbereitung
------------

Mencoder kann beliebige Quellen statt sie am Bildschirm auszugeben in
Dateien umleiten oder umwandeln, dieser Artikel behandelt jedoch
ausschließlich DVD Rips. Bevor der Rip einer DVD gestartet wird, sollte
mplayer mit entsprechenden Parametern aufgerufen werden. Es fängt damit
an die gewünschte DVD einzulegen (unter FreeBSD ist /dev/acd0 das
Standardlaufwerk).

Der Richtige Film
~~~~~~~~~~~~~~~~~

Der erste Aufruf ist

::

   # gmplayer dvd://

Der Aufruf spielt den längsten Titel auf der DVD ab. Sollte der längste
Titel nicht der gewünschte sein müssen die Titel bis zum gewünschten
durchprobiert werden. Zur Probe kann mplayer noch einmal gestartet
werden. In diesem Beispiel mit dem 5. Titel.

::

   # mplayer dvd://5

Sollte der mit dem ersten Aufruf gespielte Titel korrekt sein, ist es
nicht notwendig eine Quelle bei Aufruf des Skript anzugeben, da das
Skript **dvd://** als Standardquelle behandelt.

Die Richtige Tonspur
~~~~~~~~~~~~~~~~~~~~

Wenn die Standardtonspur nicht die gewünschte ist schafft der Parameter
**-alang** abhilfe. Der Aufruf

::

   # mplayer dvd://5 -alang de

spielt das vorherige Beispiel mit einer deutschen Tonspur, sofern
vorhanden, ab. Falls es mehrere Tonspuren einer Sprache gibt und mplayer
die falsche auswählt kann auch explizit eine Tonspur mit dem Parameter
**-aid** angesprochen werden. Die IDs vorhandener Spuren gibt mplayer
bei der Wiedergabe aus. Zum Beispiel:

::

   audio stream: 0 format: ac3 (5.1) language: de aid: 128.
   audio stream: 1 format: dts (5.1) language: de aid: 137.
   audio stream: 2 format: ac3 (5.1) language: ja aid: 130.

Untertitel
~~~~~~~~~~

Unter Umständen sind auch Untertitel erwünscht. Analog zu den Tonspuren
gibt es die Paramenter **-slang** und **-sid**. Der Aufruf

::

   # mplayer dvd:// -alang ja -slang de

spielt einen Film auf Japanisch mit deutschen Untertiteln. Auch die
vorhanden Untertitel gibt mplayer bei der Wiedergabe aus:

::

   subtitle ( sid ): 0 language: de
   subtitle ( sid ): 1 language: de

DVDs kennen nur die Seitenverhältnisse 16:9 und 4:3, viele Filme sind
jedoch ein einem breiteren Format, was zur Folge hat, dass der Film oben
und unten einen schwarzen Rand hat auf dem die Untertitel häufig
komplett untergebracht werden können. Das Skript bietet den Parameter
**-dvd-sub-bottom** um die Untertitel von DVDs so weit unten wie möglich
am Rand des Films zu platzieren. Die Option entspricht den mplayer
Parametern **-spuaa 16 -spualign 2**.

Bitrate
~~~~~~~

Die Bitrate bestimmt den Platz den das fertige Video auf der Festplatte
belegen wird. Die Standardbitrate des Skripts für das XviD Bild ist 500
kbit/s und für die MP3 Tonspur 96 kbit/s. Diese Werte sind recht
niedrig, das hängt damit zusammen, dass das Skript ursprünglich dazu
geschrieben wurde DVDs für den PDA des Autors zu konvertieren.

Das Skript bietet für die Änderung der Tonqualität den Parameter
**-mp3rate**. Werte höher als der Standard sind nur bei großen Rips (1.4
GB) oder Musikvideos sinnvoll.

Für die Bitrate des XviD Bildes gibt es die Parameter **-xvidrate** und
**-size**. Der Parameter **-size** hat immer Priorität und gibt die
gewünschte Dateigröße in Megabyte an. Daraus wird dann nach dem ersten
Durchlauf die Bitrate für das Bild errechnet. Das Endergebnis kann
durchaus auch mal 2% größer als gedacht werden, deshalb sollte die Größe
sicherheitshalber 3-4% kleiner angeben werden.

Der Parameter **-xvidrate** gibt einfach die gewünschte Bitrate an.

Bildbreite
~~~~~~~~~~

Um das Bild in eine andere Auflösung zu skalieren kann der Parameter
**-width** verwendet werde, der einfach die Bildbreite in Pixeln
erwartet. Ursprünglich hat der Autor den Parameter eingeführt um 480
Pixel breite Rips für seinen PDA zu machen, aber auch um den optischen
eindruck von Rips mit einer niedrigen Bitrate (zum Beispiel Rips auf
eine CD) zu verbessern, ist der Parameter nützlich. Normalerweise ist
ein DVD Bild 720 Pixel breit, wer zum Beispiel 5 Pixel zu vier Pixeln
zusammenfassen will wählt also eine Breite von 576 Pixeln. Eine
niedrigere Auflösung sieht bei niedrigen Bitraten oft besser aus, weil
dann weniger Artefakte erscheinen und das Bild stattdessen stärker
weichgezeichnet wird.

Dateiname
~~~~~~~~~

Standardmäßig landet das Ergebnis eines Rip in der Datei movie.avi, der
Parameter **-o** erlaubt die angaben eines anderen Dateinamens.

Lautstärke
~~~~~~~~~~

Normalerweise ist der Ton auf DVD Filmen in einem Sourround Format mit
mindestens 6 Spuren vorhanden. Beim Abmischen auf 2 Kanäle (Stereo) geht
einiges an Lautstärke verloren. Deshalb hebt das Skript die Lautstärke
normalerweise gehörig an. Dieses Verhalten kann mit dem Parameter
**-mp3vol** der Werte von 0 bis 10 erwartet verändert werden. Der
Standardwert ist 4.

Rippen
------

Wenn die Vorbereitungen abgeschlossen sind, kann der eigentliche Rip
beginnen. Abgesehen von den in der Vorbereitung beschriebenen Parametern
bietet das Skript noch weitere Parameter, die normalerweise nicht
benötigt werden. Parameter die das Skript nicht kennt werden unverändert
an mencoder weitergegeben.

Der Vorgang des Rippens kann je nach Quellmaterial, Bitrate und
gewählter Auflösung sehr lange dauern. Das Rippen findet in 2
Durchgägnen statt. Der erste Durchgang geht deutlich schneller. Wenn der
zweite Durchgang erreicht ist kann nach ein paar weiteren Minuten der
Vorgang mit CTRL-C abgebrochen werden. Das erlaubt es anhand des ersten
Schnipsels zu prüfen, ob das Ergebnis den Erwartungen entsprechen wird.
Wenn das der Fall ist, kann das Skript mit den gleichen Parametern
aufgerufen werden.

Nach dem der Rip fertig ist, existiert eine Arbeitsverzeichnis, das
gelöscht werden kann. Wenn kein Dateiname angegeben wurde heißt das
Verzeichnis dann 'movie.avi.ripXviD'. Wenn mit dem Parameter ``-o`` ein
Dateiname festgelegt wurde, können mehrere Rips gleichzeitig laufen. Zum
Beispiel von 2 verschiedenen DVD-Laufwerken um mehrere CPU-Kernel
auszulasten.

Wenn nur ein Laufwerk vorhanden ist, kann das Skript auch mit der Option
``-dump`` aufgerufen werden. Dann wird die DVD nur für den ersten
Durchgang benötigt. Sobald der abgeschlossen ist, kann die DVD
herausgenommen werden und ein anderer Rip kann begonnen werden.

Single CD Rip
~~~~~~~~~~~~~

Ein Single CD Rip ist ein Rip bei dem eine Dateigröße gewählt wird die
auf eine einzige CD passt. Hier empfiehlt es sich das Bild ein wenig zu
verkleinern. Das folgende Beispiel ript eine DVD für einen 700 MB
Rohling mit englischer Tonspur.

::

   # ripXviD -size 675 -width 576 -alang en -o singlCD.avi

3 Filme auf eine DVD
~~~~~~~~~~~~~~~~~~~~

Wer Filme mitnehmen will ohne seine wertvollen Original DVDs mit sich
herumzutragen kann bei (subjektiv) sehr guter Qualität 3 Filme auf einen
Single Layer DVD Rohling unterbringen. Dazu werden Pro Film 1500 MB
angepeilt. Das gibt auch Raum für eine bessere Soundqualität, wie 128
oder sogar 192 kbit/s.

::

   # ripXviD -size 1450 -mp3rate 192

Skript
------

.. code:: bash

   #!/bin/sh
   command=mencoder
   source=dvd://
   mp3rate=128
   mp3vol=4
   xvidthreads="$(sysctl -n hw.ncpu)"
   xvidrate=500
   xvidqpel="noqpel"
   xvidgmc="nogmc"
   #xvidchopt="nochroma_opt"
   xvidchopt="chroma_opt"
   quant_type="mpeg"
   #cache="16384"
   fps=25
   parameters=""
   filename=movie.avi
   dump=""
   type=""
   filesize=""
   seek=""
   crop=""
   width=""
   for parameter; {
       case "$parameter" in
           -dump) {
               # Dump the first pass output into a file.
               # This file will be used for the second pass,
               # and sound conversion will already be performed
               # during the first pass. So this should give a slight
               # speedup and allow much earlier access to the optical
               # drive.
               dump=1
           };;
           -dvd-sub-bottom) {
               # Show DVD subtitles at the bottom.
               parameters="$parameters -spuaa 16 -spualign 2"
           };;
           -xvidgmc) {
               # Activate global motion compensation.
               # Problematic for some hardware and portable players.
               xvidgmc="gmc"
           };;
           -xvidqpel) {
               # Activate quarter pixel precission.
               xvidqpel="qpel"
           };;
           -xvidchopt) {
               # Activate chroma optimization.
               xvidchopt="chroma_opt"
           };;
           *://*) {
               # Set the video source.
               source="$parameter"
           };;
           -*) {
               # Set parameter type.
               type=`echo "$parameter"|sed -E 's|^-||1'`
               # Go to proceed parameter.
               continue
           };;
           *) {
               # Proceed parameters.
               case "$type" in
                   size) {
                       # Set wanted filesize in mb.
                       # This will override '-xvidrate'.
                       filesize=$parameter
                   };;
                   crop) {
                       # Crops the image.
                       crop="$parameter"
                   };;
                   fps) {
                       # Assume a diffrent frame rate when
                       # calculating the length of the video.
                       fps=$parameter
                   };;
                   mp3rate) {
                       # Set mp3 rate in kbps.
                       mp3rate=$parameter
                   };;
                   mp3gain) {
                       # Increase volume (0-9).
                       mp3vol=$parameter
                   };;
                   xvidrate) {
                       # Set the video bitrate.
                       xvidrate=$parameter
                   };;
                   xvidthreads) {
                       # Set the number of threads.
                       xvidthreads=$parameter
                   };;
                   o) {
                       # Set the video filename.
                       filename="$parameter"
                   };;
                   ss) {
                       # Seeking a certain position in the
                       # source video.
                       seek="-ss $parameter"
                   };;
                   width) {
                       # Scale the video to the given width.
                       width="$parameter"
                   };;
                   cache) {
                       # Choose the media read cache size.
                       cache="$parameter"
                   };;
                   quant_type)
                       # Chose between h263 and mpeg quantizer.
                       quant_type="$parameter"
                   ;;
                   *) {
                       # Add unknown parameters.
                       if [ "$type" ]; then
                           parameters="$parameters -$type"
                       fi
                       parameters="$parameters $parameter"
                   };;
               esac
           };;
       esac
       type=""
   }

   # Assemble scaling filters.
   if [ -n "$width" -a -n "$crop" ]; then
       scale="-vf scale,crop=$crop -zoom -xy $width"
   elif [ -n "$width" ]; then
       scale="-vf scale -zoom -xy $width"
   elif [ -n "$crop" ]; then
       scale="-vf crop=$crop"
   fi

   # Create working directory.
   wrkdir="$filename.$(basename $0)"
   if ! mkdir -p "$wrkdir"; then
       echo "Creating working directory '$wrkdir' failed"
       return 129
   fi

   # Prepare for clean operation.
   rm "$wrkdir/xvid2pass.log" 2> /dev/null

   if test -n "$dump"; then
       dumpfile="$wrkdir/pass1.avi"
       audio_first="-oac mp3lame -lameopts aq=0:abr:br=$mp3rate:vol=$mp3vol"
       audio_second="-oac copy"
       cache_first="${cache:+-cache $cache}"
       cache_second=""
       seek_first="$seek"
       seek_second=""
   else
       dumpfile="/dev/null"
       audio_first="-nosound"
       audio_second="-oac mp3lame -lameopts aq=0:abr:br=$mp3rate:vol=$mp3vol"
       cache_first="${cache:+-cache $cache}"
       cache_second="${cache:+-cache $cache}"
       seek_first="$seek"
       seek_second="$seek"
   fi

   if ! [ -f "$wrkdir/pass1.log" -a -e "$dumpfile" ]; then
       if ! ($command "$source" $scale $cache_first -ovc xvid $audio_first \
           -xvidencopts pass=1:autoaspect:$xvidqpel:$xvidgmc:$xvidchopt \
           -xvidencopts threads=$xvidthreads \
           -xvidencopts quant_type=$quant_type \
           -passlogfile "$wrkdir/xvid2pass.log" \
           -o "$dumpfile" $seek_first $parameters \
           && cp "$wrkdir/xvid2pass.log" "$wrkdir/pass1.log"); then
           error=$?
           echo "First pass failed."
           return $error
       fi
   else
       echo "Skipping first pass, apparently already done."
   fi

   if test -n "$filesize"; then
       echo "****** determine video bitrate *******************************************"
       printf "%-16s %-55s %s\n" "* wanted size: " "${filesize}m" "*"
       # Calculate the length of the film in seconds.
       seconds=`export LANG=C ; wc -l "$wrkdir/pass1.log" | awk "{print((\\$1 - 3) / $fps);}"`
       printf "%-16s %-55s %s\n" "* length: " "${seconds}s" "*"
       printf "%-16s %-55s %s\n" "* mp3rate: " "${mp3rate}kb/s" "*"
       # Calculate bitrate to match size (mb).
       xvidrate=`awk "BEGIN {printf(\"%d\", ($filesize * 2^23) / (1000*$seconds) - $mp3rate);}"`
       printf "%-16s %-55s %s\n" "* xvidrate: " "${xvidrate}kb/s" "*"
       echo "**************************************************************************"
   fi

   test -n "$dump" && source="$dumpfile"

   if cp "$wrkdir/pass1.log" "$wrkdir/xvid2pass.log"; then
       if ! $command "$source" $scale $cache_second \
           $audio_second -ovc xvid \
           -xvidencopts pass=2:bitrate=$xvidrate:autoaspect:$xvidqpel \
           -xvidencopts $xvidgmc:$xvidchopt:threads=$xvidthreads \
           -xvidencopts quant_type=$quant_type \
           -passlogfile "$wrkdir/xvid2pass.log" \
           -o "$filename" $seek_second $parameters; then
           error=$?
           echo "Second pass failed."
           return $error
       fi
   else
       echo "Cannot perform second pass. Apparently the first pass failed."
       return 130
   fi

Verweise
--------

-  Die mplayer `Homepage <http://www.mplayerhq.hu>`__.
-  Die
   `Dokumentation <http://www.mplayerhq.hu/DOCS/HTML/de/index.html>`__
   und die
   `Manpage <http://www.mplayerhq.hu/DOCS/man/de/mplayer.1.html>`__ auf
   Deutsch (nicht immer vollständig).
-  Die
   `Dokumentation <http://www.mplayerhq.hu/DOCS/HTML/en/index.html>`__
   und die
   `Manpage <http://www.mplayerhq.hu/DOCS/man/en/mplayer.1.html>`__ auf
   Englisch.
-  `multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
   und
   `multimedia/mencoder <https://www.google.com/search?q=multimedia/mencoder&btnI=lucky>`__
   in den FreeBSD Ports.
-  `multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
   und
   `multimedia/mencoder <https://www.google.com/search?q=multimedia/mencoder&btnI=lucky>`__
   in Pkgsrc.
-  `x11/mplayer <https://www.google.com/search?q=x11/mplayer&btnI=lucky>`__
   in den OpenBSD Ports.

* :ref:`genindex`

Zuletzt geändert: |date|

