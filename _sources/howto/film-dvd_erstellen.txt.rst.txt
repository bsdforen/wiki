Film-DVD erstellen
==================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Dieser Artikel beschäftigt sich mit dem Erstellen von Film-DVDs mit Menüs.

Einleitung
----------

Dieses Tutorial beschreibt wie man aus einem Film
(*AVI/MPEG/MP4/MKV/OGG*) oder einem DVD Backup wieder eine DVD macht.
Des weiteren wird behandelt, wie man DVD-Menüs erstellt. Soweit nichts
anderes geschrieben steht, gilt für alle vier BSDs das Gleiche. Die
Mencoder-Einstellungen sind zum Teil von der Mplayer-Dokumentation
übernommen.

Aus den Ports installieren
--------------------------

Wir müssen nun einige Programme aus den Ports installieren.

OpenBSD
~~~~~~~

#. x11/mplayer
#. sysutils/dvd+rw-tool
#. multimedia/mjpegtools
#. multimedia/dvdauthor

Optional:

#. audio/normalize
#. multimedia/gimp

NetBSD
~~~~~~

#. `multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
#. `sysutils/dvd+rw-tools <https://www.google.com/search?q=sysutils/dvd+rw-tools&btnI=lucky>`__
#. `multimedia/mjpegtools <https://www.google.com/search?q=multimedia/mjpegtools&btnI=lucky>`__
#. `multimedia/dvdauthor <https://www.google.com/search?q=multimedia/dvdauthor&btnI=lucky>`__

Optional:

#. `audio/normalize <https://www.google.com/search?q=audio/normalize&btnI=lucky>`__
#. `graphics/gimp <https://www.google.com/search?q=graphics/gimp&btnI=lucky>`__

FreeBSD
~~~~~~~

#. `multimedia/mplayer <https://www.google.com/search?q=multimedia/mplayer&btnI=lucky>`__
#. `sysutils/dvd+rw-tool <https://www.google.com/search?q=sysutils/dvd+rw-tool&btnI=lucky>`__
#. `multimedia/mjpegtools <https://www.google.com/search?q=multimedia/mjpegtools&btnI=lucky>`__
#. `multimedia/dvdauthor <https://www.google.com/search?q=multimedia/dvdauthor&btnI=lucky>`__

Optional:

#. `audio/normalize <https://www.google.com/search?q=audio/normalize&btnI=lucky>`__
#. `graphics/gimp <https://www.google.com/search?q=graphics/gimp&btnI=lucky>`__

Film ins "DVD"-Format
---------------------

Um einen Film ins MPEG2-Format zu konvertieren, verwenden wir hier den
`Mencoder </anwendungen/mplayer#mencoder>`__. Man könnte auch den
Transcode nehmen. Der Transcode hat einfach nicht die gleichen
Möglichkeiten wie der Mencoder. Mit Mencoder kann man z.B. Filter
einsetzen, welche die Qualität steigern können. Des weiteren ist es
einfacher, die Audio/Video-Synchronisation aufrecht zu erhalten.

Führen wir uns als erstes mal diese Tabelle zu Gemüte:

======== ================================== ======== =========== ================ ================ ========== ============ =============================
Format   Auflösung                          V. Codec A. Codec    V. Bitrate (Max) A. Bitrate (Max) Samplerate FPS          Seitenverhältnis
======== ================================== ======== =========== ================ ================ ========== ============ =============================
PAL DVD  720x576, 704x576, 352x576, 352x288 MPEG-2   MP2,AC3,PCM 9800 kbps        1536 kbps        48000 Hz   25           4:3, *16:9 (nur für 720x576)*
NTSC DVD 720x480, 704x480, 352x480, 352x240 MPEG-2   AC3,PCM     9800 kbps        1536 kbps        48000 Hz   29.97, 23.97 4:3, *16:9 (nur für 720x480)*
======== ================================== ======== =========== ================ ================ ========== ============ =============================

Hallte dich soweit an die Infos in der Tabelle. Mit der Bitrate kannst
du natürlich spielen. Zu erst müssen wir unseren Film in ein
DVD-konforme Datei umwandeln. Dazu verwenden wir den Mencoder. Wenn du
dich nicht an die Tabelle hältst, wird der Film nicht mehr konform und
er wird nicht abgespielt!

Kontainer
~~~~~~~~~

Wir brauchen noch einen Kontainer für unsere Audio- und Video-Spur. Es
gibt nur einen Kontainer, der auch DVD-konform ist, und das ist der
MPEG-Kontainer. Um dem Mencoder mitzuteilen, welchen Kontainer er
gebrauchen soll, geben wir ihm folgende Syntax mit:

::

   -of mpeg -mpegopts format=dvd

Video und Audio
~~~~~~~~~~~~~~~

Nun kommt die Video- und Audio-Spur an die Reihe. Um es etwas zu
vereinfachen, wird hier die komplette Syntax angegeben und erst danach
werden wir darauf eingehen. Die Syntax ist so gewählt, dass sie das
beste Resultat erzielt. Wenn die Breite und Höhe nur mit *B_V* und *H_V*
angegeben sind, gibt es mehrere Möglichkeiten. In diesem Fall kannst du
selbst aus der Tabelle die Werte einsetzen. Die Bitrate ist immer mit
*BR* angegeben. Auch hier heißt es selbst einsetzen. Tipps gibt es
dennoch.

PAL DVD 16/9
^^^^^^^^^^^^

**-lavcopts**

::

   vcodec=mpeg2video:vrc_buf_size=1835:vrc_maxrate=9800:vbitrate=''BR'':\
   keyint=15:trell:mbd=2:precmp=2:subcmp=2:cmp=2:dia=-10:predia=-10:cbp:mv0:\
   vqmin=1:lmin=1:dc=10:acodec=ac3:abitrate=''BR'':aspect=16/9

**-vf**

::

   scale=720:576,harddup 

**-af**

::

   lavcresample=48000

**sonstiges**

::

   -srate 48000 -ofps 25

PAL DVD 4/3
^^^^^^^^^^^

**-lavcopts**

::

   vcodec=mpeg2video:vrc_buf_size=1835:vrc_maxrate=9800:vbitrate=''BR'':\
   keyint=15:trell:mbd=2:precmp=2:subcmp=2:cmp=2:dia=-10:predia=-10:cbp:mv0:\
   vqmin=1:lmin=1:dc=10:acodec=ac3:abitrate=''BR'':aspect=4/3

**-vf**

::

   scale=''B_V'':''H_V'',harddup

**-af**

::

   lavcresample=48000

**sonstiges**

::

   -srate 48000 -ofps 25

Erklärung der Syntax
^^^^^^^^^^^^^^^^^^^^

Leider kann man hier nicht genau auf die einzelnen Punkte eingehen. Das
würde den Rahmen für dieses HowTo sprengen. Wer mehr wissen will, kann
`hier <http://www.mplayerhq.hu/DOCS/man/de/mplayer.1.html#CODEC-SPEZIFISCHE%20ENCODING-OPTIONEN%20(NUR%20BEI%20MENCODER)>`__
mehr dazu lesen. Die wichtigsten Punkte werden wir aber anschauen
können. Aber wie schon geschrieben, die "Default"-Syntax hier sollte die
beste Qualität liefern.

======================================= ===============================================================================
Syntax                                  Erklärung
======================================= ===============================================================================
*-vf* **harddup**                       Ist dafür verantwortlich, dass die A/V-Synchronisation aufrecht erhalten bleibt
*-vf* **scale**\ =\ *W:H*               Ändert die Bildgröße auf W und H
**-ofps** *25*                          Zur Sicherheit, dass immer 25 Bilder in der Sekunde kommen.
*-lavcopts* **vbitrate**\ =\ *BR*       Gibt die Video-Bitrate an
*-lavcopts* **abitrate**\ =\ *BR*       Gibt die Audio-Bitrate an
*-lavcopts* **vcodec**\ =\ *mpeg2video* Gibt den Videocodec an
*-lavcopts* **acodec**\ =\ *ac3*        Gibt den Audiocodec an
*-lavcopts* **aspect**\ =\ *x/y*        Ändert den Aspect auf x/y
======================================= ===============================================================================

Tipps
^^^^^

Nun gibt es einige Tipps und etwas Fachwissen. Das Encoder-Wissen muss
man sich aneignen, aber mit Hilfe dieses HowTos sollte man im Stande
sein, eine qualitativ hochwertige DVD zu erstellen. Kommt halt auch
etwas auf das Ausgangsmaterial an.

Ton (AC3)
'''''''''

AC3 ist nicht gerade der beste Audio-Codec, da er verlustbehaftet ist.
Mit etwas Wissen kann man es aber etwas erträglicher machen. AC3 ist ein
Mehrkanal-Ton-Codec und kann 2, 4 oder 6 (5.1) Kanäle beinhalten. Tipps
für die Bitrate

::

   Stereo (2-Kanal)        192kbps oder 224kbps
   4-Kanal                 320kbps
   5.1 (6-Kanal)           384kbps oder 448kbps

Wenn das Ausgangsmaterial 6 oder 4 Kanäle hat, und man will, dass die 6
(5.1) oder 4 Kanäle auch auf der DVD sind, muss man das dem Mencoder
auch mitteilen. Man fügt einfach folgendes an:

::

   -channels 6
   bzw.
   -channels 4

Hat das Material Stereo-Ton und man will auf der DVD einen 5.1-Ton, so
fügt man einfach folgendes an:

::

   -channels 6 -af pan=6:100:0:0:0:100:100:0:100:0:0:100:100:0:0:100:0:0:0:0:0:0:100:0:0,delay,volnorm

Bild (Filter)
'''''''''''''

Jetzt kommt ein wichtiges Thema. Mplayer kennt Filter. Diese Filter kann
man dazu gebrauchen, die Qualität des Ausgangsmaterials etwas zu
verbessern. Die Filter funktionieren sehr gut. Ist aber das Bild zu
schlecht, so wird sich ein Filter auch nicht gerade positiv auswirken,
da er das Bild zu weich macht. Die Filter werden wie folgt gebraucht:
*-vf* filter1,filter2=opt1:opt2,filter3

Um mehr darüber zu erfahren, bitte in der Manpage nachschauen.

Dieser Filter versucht, Bildrauschen zu unterdrücken. Er sollte vor
*hharddup* stehen.

::

   denoise3d

Donald Grafts adaptiver Kernel-Deinterlacer, für Deinterlacing. Er
sollte nach *harddup* stehen.

::

   kerndeint

Anamorph
''''''''

Ein analoges Bild, welches im 16/9-Format ist, aber durch schwarze
Balken ein 4/3-Seitenverhältnis hat, ist nicht anamorph. Ein anamorphes
Bild ist im 16/9-Format bzw. hat 720*576 Pixel. Um ein anamorphes Bild
in ein 4/3-Seitenverhältnis zu verwandeln, kann man den Filter *expand*
verwenden. Er fügt dem Bild schwarze Balken an.

Um ein analoges 16/9-Bild in ein anamorphes Bild zu verwandeln, oder
einfach auf gut Deutsch die schwarzen Balken zu entfernen, kann man den
Filter *crop* nehmen. Das ist sinnvoll, weil man so Bitrate für die
schwarzen Balken spart.

Wie man die zwei Filter verwendet, steht in der Manpage von Mplayer.

Bild (Bitrate)
''''''''''''''

Eine Frage, die sich jeder hier stellen wird, ist, welche Bitrate ich
für mein Bild brauche. Das ist auch etwas Erfahrungssache. Auch eine
Bitrate von 2000kbps ergibt ein Bild. Also Vergleich:

::

   100min Film bei 5000kbps an 720*576 (16/9)  =  2.9GB
   mit 1.4 GHz an 62min.

Spiele am besten etwas bei deinem ersten Versuch. Du kannst ja während
des Encodens das Ergebnis betrachten (mit Mplayer). So kannst du noch
Anpassungen machen.

Zusammenfassung
^^^^^^^^^^^^^^^

So, nun fügen wir alles zusammen. Wer den Mencoder nicht kennt, kann
jetzt alles mit Copy-&-Paste übernehmen und die Werte eintragen.
Vielleicht auch noch einen Filter mehr einbauen.

.. _pal-dvd-169-1:

PAL DVD 16/9
''''''''''''

::

   mencoder eingabe_film.avi -oac lavc -ovc lavc -vf scale=720:576,harddup \
   -srate 48000 -af lavcresample=48000 -lavcopts vcodec=mpeg2video:vrc_buf_size=1835:\
   vrc_maxrate=9800:vbitrate=BR:keyint=15:trell:mbd=2:precmp=2:subcmp=2:cmp=2:\
   dia=-10:predia=-10:cbp:mv0:vqmin=1:lmin=1:dc=10:acodec=ac3:abitrate=BR:aspect=16/9 \
   -ofps 25 -of mpeg -mpegopts format=dvd -o ausgabe_fim.mpg

.. _pal-dvd-43-1:

PAL DVD 4/3
'''''''''''

::

   mencoder eingabe_film.avi -oac lavc -ovc lavc -vf scale=B_W:H_W,harddup \
   -srate 48000 -af lavcresample=48000 -lavcopts vcodec=mpeg2video:vrc_buf_size=1835:\
   vrc_maxrate=9800:vbitrate=BR:keyint=15:trell:mbd=2:precmp=2:subcmp=2:cmp=2:\
   dia=-10:predia=-10:cbp:mv0:vqmin=1:lmin=1:dc=10:acodec=ac3:abitrate=BR:aspect=4/3 \
   -ofps 25 -of mpeg -mpegopts format=dvd -o ausgabe_fim.mpg

DVD-Struktur mit DVDAuthor
--------------------------

Um eine DVD-Struktur zu erzeugen brauchen wir den DVDauthor. Wir
schreiben zuerst eine XML-Datei. Für die ganz faulen gibt es auch GUIs,
die machen aber bekanntlich selten was man will. Es wird nun nicht mehr
eine 16/9- und 4/3-Variante geben, da ich nun davon ausgehe, dass jeder
selbst die Werte ersetzen kann.

Nur abspielen
~~~~~~~~~~~~~

Die einfachste DVD-Struktur ist die "Einlegen und Abspielen"-Variante.
Dazu schreiben wir folgende XML-Datei.

**dvdauthor.xml**

::

   <dvdauthor>
    <vmgm />
    <titleset>
      <titles>
      <video format="pal" aspect="16:9"/>
      <audio format="ac3" lang="de"/>
        <pgc>
          <vob file="mein_film.mpg"/>
        </pgc>
      </titles>
    </titleset>
   </dvdauthor>

Kapitel kann man wie folgt machen:

::

   [...]
      <vob file="mein_film.mpg" chapters="00:00:00.000,00:03:05.200, ... "/>
   [...]

Will man eine ganze Serie auf DVD bannen, so kann man auch so "Kapitel"
machen:

::

   [...]
      <vob file="mein_film1.mpg">
      <vob file="mein_film2.mpg">
      <vob file="mein_film3.mpg">
      <vob file="mein_film4.mpg">
      ...
   [...]

Mit Menüs und Knöpfen
~~~~~~~~~~~~~~~~~~~~~

Mit Menüs und Knöpfen ist es etwas komplizierter. Wer es lieber einfach
haben will, sollte die "einlegen und abspielen"-Methode nehmen. Ich
empfehle hier die Verwendung von Gimp.

-  Zuerst erzeugen wir ein Bild mit dem Hintergrund und allen Knöpfen in
   der Größe 768*576.
-  Wir skalieren nun das Bild auf 720*576 und speichern es als
   ``'dvd_menu.ppm``'.
-  Es wird nun ein mpeg2-Stream aus dem Bild erstellt.

::

   ppmtoy4m -v 0 -n 720 -r -F 25:1 -A 59:54 -I p dvd_menu.ppm | \
   mpeg2enc -c -q 6 -4 2 -2 1 -R 2 -b 8000 -v 0 -a 2 -M 2 -f 8 -o dvd_menu.m2v
   #(ppmtoy4m) -n gibt die Länge des "Films" an und muss gleichlang wie der Sound sein. 25 = 1 sec

-  Wir erzeugen nun eine mp2-Datei aus einem Lied. In unserem Fall ist
   der Sound 30 Sekunden lang.

::

   mplayer mein_audio.ogg -ao pcm:file=audio.wav
   toolame -s 48000 -b 224 audio.wav audio_menu.mp2
   rm audio.wav

-  Nun fügen wir Bild und Ton zusammen. Man braucht immer einen Ton.

::

   mplex -v 0 -f 8 -o dvd_menu.vob dvd_menu.m2v audio_menu.mp2
   * Nun lade ich im Gimp das Anfangsbild "dvd_menu.ppm" und mache eine neue schwarze Ebene.
   * Zum Bearbeiten lassen wir den Hintergrund etwas durchschimmern. Wir ziehen einen Strich durch die Schrift der Knöpfe. Man kann es auch anders animieren, darf aber nicht mehr als vier Farben gebrauchen.
   * Wir speichern nun die obere Schicht (Hintergrund ist schwarz!) als knpf_dvd_pre.png ab.
   * Nun wird das PNG-Bild wie folgt transparent gemacht:

::

   pngtopnm knpf_dvd_pre.png | pnmdepth 3 | \
   pnmtopng -transparent "#000000" > knpf_dvd.png

-  Nun erstellen wir eine XML-Datei für spumux, **knpf_dvd.xml**:

::

   <subpictures>
    <stream>
     <spu start="00:00:00.00" end="00:00:00.00" highlight="knpf_dvd.png" force="yes" >
      <button x0="50"  y0="420" x1="230" y1="430" />
      # <button x0="23"  y0="324" x1="453" y1="232" />
      # ...
     </spu>
    </stream>
   </subpictures>
   # Die Positionen:
   # x0 und y0 sind die Koordinaten von der linken oberen Ecke des Buttons
   # x1 und y1 sind die Koordinaten von der rechten unteren Ecke des Buttons
   # Diese Koordinaten kann man mit Gimp ganz einfach herausfinden, wenn man knpf_dvd.png über dvd_menu.ppm legt.
   # Die Reihenfolge sollte man sich fürs dvdauthor.xml merken!

-  Jetzt ist spumux an der Reihe.

::

   spumux -v 0 -P knpf_dvd.xml < dvd_menu.vob > dvd_menu_knpf.vob
   * Nun haben wir unser animiertes DVD-Menü mit Sound. Wenn man keinen Sound will, muss man dennoch einen Leersound nehmen.
   * Nun erzeugen wir unser //dvdauthor.xml//. Einfachheitshalber habe ich nur einen Button. Wenn man darauf "klickt" startet der Film. Nach 60 Sekunden wird automatisch der Film gestartet.

::

   <dvdauthor>
    <vmgm>
      <menus>
        <pgc entry="title">
          <vob file="dvd_menu_knpf.vob" pause="60"/>
          <button> jump title 1; </button>
          <post> jump title 1; </post>
        </pgc>
      </menus>
    </vmgm>
    <titleset>
      <titles>
      <video format="pal" aspect="16:9"/>
      <audio format="ac3" lang="de"/>
        <pgc>
          <vob file="mein_film.mpg"/>
        </pgc>
      </titles>
    </titleset>
   </dvdauthor>

Beispiel-Code für erweiterte Menüs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Das sind Beispiele aus der Doku. Wenn man sie ganau betrachtet, versteht
man sie sehr gut. Darum gibt es auch keine Erklärung.

DVD-Menü mit Kapitel
^^^^^^^^^^^^^^^^^^^^

dvdauthor.xml:

::

   <dvdauthor>
    <vmgm>
      <menus>
        <pgc entry="title">
          <vob file="menu_title.vob" pause="60"/>
          <button> jump title 1; </button>
          <button> jump titleset 1 menu entry root; </button>
          <post> jump title 1; </post>
        </pgc>
      </menus>
    </vmgm>
    <titleset>
      <menus>
        <pgc entry="root">
          <vob file="menu_root.vob" pause="60"/>
          <button> jump title 1 chapter 01; </button>
          <button> jump title 1 chapter 02; </button>
          <button> jump title 1 chapter 03; </button>
          <post> jump title 1; </post>
        </pgc>
      </menus>
      <titles>
      <video format="pal" aspect="16:9"/>
      <audio format="mp2" lang="de"/>
        <pgc>
          <vob file="stream.mpg" chapters="0,0:0:10,0:0:20,0:0:30"/>
          <post> call vmgm menu 1; </post>
        </pgc>
      </titles>
    </titleset>
   </dvdauthor>

DVD mit Sprachauswahl
^^^^^^^^^^^^^^^^^^^^^

dvdauthor.xml

::

   <dvdauthor>
    <vmgm>
      <menus>
      <video format="pal" aspect="4:3"/>
        <pgc entry="title">
          <pre> { g1=0; button=3072; } </pre>        
          <vob file="menu_title.vob" pause="60"/>
          <button> jump titleset 1 menu entry audio; </button>
          <button> jump titleset 1 menu entry root; </button>
          <button> jump title 1; </button>
          <post> jump title 1; </post>
        </pgc>
      </menus>
    </vmgm>
    <titleset>
      <menus>
      <video format="pal" aspect="4:3"/>
        <pgc entry="root">
          <vob file="menu_root.vob" pause="60"/>
          <button> jump title 1 chapter 01; </button>
          <button> jump title 1 chapter 02; </button>
          <button> jump title 1 chapter 03; </button>
          <button> jump title 1 chapter 04; </button>
          <button> jump vmgm menu entry title; </button>
          <post> jump title 1; </post>
        </pgc>
        <pgc entry="audio">
          <pre> button=2048; </pre>        
          <vob file="menu_audio.vob" pause="inf"/>
       <button name="1"> { audio=0;
                       if (g1 eq 0) jump vmgm menu entry title;
                           if (g1 eq 1) resume;
                         } </button>
       <button name="2"> { audio=1;
                       if (g1 eq 0) jump vmgm menu entry title;
                           if (g1 eq 1) resume;
                         } </button>
        </pgc>
      </menus>
      <titles>
      <video format="pal" aspect="4:3"/>
      <audio format="ac3" lang="en"/>
      <audio format="ac3" lang="de"/>
       <pgc>
          <pre> g1=1; </pre>        
       <vob file="stream_1.mpg"/>
       <vob file="stream_2.mpg"/>
       <vob file="stream_3.mpg"/>
       <vob file="stream_4.mpg"/>
       <post> call vmgm menu 1; </post>
        </pgc>
      </titles>
    </titleset>
   </dvdauthor>

DVD-Struktur erstellen
~~~~~~~~~~~~~~~~~~~~~~

So, nun sind wir fast am Ende. Aus dem XML-File und den Video-Files
müssen wir jetzt nur noch einen Ordner erstellen, der die DVD-Struktur
wiederspiegelt. Das macht der DVDauthor zum Glück von selbst.

::

   dvdauthor -o DVD_DIR -x dvdauthor.xml

Brennen
-------

Noch wenige Minuten, bis wir uns einen gemütlichen DVD-Abend machen
können.

::

   growisofs -Z /dev/CD ROM_DEVICE -dvd-video DVD_DIR

Verweise
--------

-  `Mplayer-DVD-Anleitung <http://www.mplayerhq.hu/DOCS/HTML/de/menc-feat-vcd-dvd.html>`__
   (de)

* :ref:`genindex`

Zuletzt geändert: |date|

