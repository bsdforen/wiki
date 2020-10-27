Computerspiele unter FreeBSD
============================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Bevor man weiterliest sollte man sichergehen, dass die 3D-Beschleunigung
funktioniert und installiert ist. Nutzer von NVIDIA-Chipsätzen
installieren dazu den Port **x11/nvidia-driver** und konfigurieren ihre
xorg.conf entsprechend, alle anderen folgen den Anweisungen aus dem
Wiki-Beitrag
`3D-Grafik-Beschleunigung </howto/3D-Grafik-Beschleunigung>`__. Ob alles
richtig eingerichtet ist kann man mit dem Befehl "glxgears" prüfen.

Bevor wir uns daran machen sie zu installieren, ist es wichtig zu
unterscheiden für welche Platform die Spiele gedacht sind.

Es gibt vier Gruppen: Spiele, die...

#. nativ unter FreeBSD,
#. unter Linux-Emulation,
#. unter Windows-Emulation oder
#. unter anderen Emulatoren laufen.

Da uns speziell die Installationsweise der Spiele interessiert, fasse
ich im folgenden unter Gruppe 1 alle Spiele zusammen die sich durch die
FreeBSD-Ports installieren lassen, also auch solche die durch
Linux-Emulation laufen (unter 2. sind dann solche Linux-Spiele, die man
manuell installieren muss). Falls es möglich ist ein Spiel auf mehrere
der folgenden Arten zu installieren sollte man immer oben anfangen ;-)

Spiele aus den Ports
--------------------

In den FreeBSD Ports gibt es einen - inzwischen relativ großen - games
Ordner, der alles mögliche enthält. Auf die tausend Kniffel-, Knobel-
und ASCII-Spiele, sowie auf Minigames gehe ich nicht weiter ein, das
würde den Rahmen sprengen. Ich werde hauptsächlich beschreiben was "man
schon kennt". Ich möchte außerdem darauf hinweisen, dass ich nicht alle
hier aufgelisteten Spiele getestet habe, aber davon ausgehe, dass sie
funktionieren.

Bekannte Spiele, die früher proprietär waren, aber jetzt Open-Source sind
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  DOOM 1 + 2 + Final Doom ( es existieren mehrere Ports, z.t. mit
   Weiterentwicklungen)
-  `Duke Nukem 3D <http://www.eduke32.com>`__, (games/eduke32, wahlweise
   mit dem `High Resolution Pack <http://hrp.duke4.net/>`__)
-  `Quake I <http://www.quakeforge.net>`__, ( games/quake-\* ,
   weiterentwickelt-> games/tenebrae )
-  `Quake II <http://www.quakeforge.net>`__, ( games/quake2\* )
-  `Quake III <http://www.quake3arena.com>`__\ ¹, ( games/ioquake3,
   games/quake3 ; games/linux-quake3 gibts auch, ist aber überflüssig)
-  `Warzone 2100 <http://warzone2100.sf.net>`__, ( games/warzone2100,
   weiterentwickelt `Warzone
   Resurrection <https://gna.org/projects/warzone>`__ )

Bekannte Spiele, die proprietär sind
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  America's Army Online ( games/linux-americasarmy )
-  `Neverwinter Nights <http://nwn.bioware.com>`__\ ¹ (
   games/linux-nwnclient, games/linux-nwnserver, games/nwndata,
   games/nwnusers )
-  `Quake IV <http://www.quake.com>`__\ ¹ ( games/linux-quake4 )
-  `Savage <http://www.s2games.com/savage/>`__ ( games/linux-savage )
-  `Wolfenstein: Enemy Territory <http://www.enemy-territory.com>`__
   (games/linux-enemyterritory\* )

Demos zu neueren Spielen
~~~~~~~~~~~~~~~~~~~~~~~~

-  `Doom3 <http://www.doom3.com>`__ ( games/linux-doom3-demo )
-  `Quake III <http://www.quake3arena.com>`__ ( games/linux-quake3-demo
   )
-  `Unreal Tournament 2003 <http://www.unrealtournament.com/>`__ (
   games/linux-ut2003-demo )
-  `Unreal Tournament 2004 <http://www.unrealtournament.com/>`__ (
   games/linux-ut2004-demo )

Empfehlenswerte OpenSource-Spiele
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `Balazar <http://home.gna.org/oomadness/en/balazar/index.html>`__ (
   `games/balazar <https://www.google.com/search?q=games/balazar&btnI=lucky>`__
   ) : ein 3D Adventure (in Python)
-  `Cube <http://www.cubeengine.com/>`__ (
   `games/cube <https://www.google.com/search?q=games/cube&btnI=lucky>`__
   ) : ein Ego-Shooter mit Live-Level-Editing
-  `Eternal Lands <http://www.eternal-lands.com>`__ (
   `games/el <https://www.google.com/search?q=games/el&btnI=lucky>`__ )
   : ein 3D Multiplayer Online Rollenspiel (nur Engine is FOSS)
-  `Nexuiz <http://www.nexuiz.com>`__ (
   `games/nexuiz <https://www.google.com/search?q=games/nexuiz&btnI=lucky>`__
   ) : ein hektischer Ego-Shooter
-  `Spring <http://spring.clan-sy.com/>`__ (
   `games/spring <https://www.google.com/search?q=games/spring&btnI=lucky>`__
   ) : ein Total Annihilation - Clone

<note> Mit ¹ markierte Ports benötigen entweder Dateien von den Original
CDs/DVDs oder eine Seriennummer zum spielen. </note>

Spiele aus den Ports installieren
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Spiele aus den FreeBSD-Ports lassen sich wie alle anderen Ports einfach
durch ein **make install clean** in **/usr/ports/games/spielename**,
durch **portinstall spielename** oder durch ein Frontend wie KPorts
installieren. Mehr dazu in `Paketsysteme </howto/Paketsysteme>`__. Damit
man nicht mit Unmengen an Spieledaten sein **/usr/local** vollmüllt
empfehle ich jedoch einen Ordner extra für Spiele anzulegen. Dies wird
spätestens nötig wenn man auch Spiele aus 2) oder 3) installieren
möchte. Dazu müssen wir die **/etc/make.conf** anpassen:

-  Nehmen wir an, man hat den Ordner **/mnt/games** für diesen Zweck
   ausgewählt (andernfalls den jeweiligen Pfad immer anpassen).
-  Damit jedes aus **/usr/ports/games** installierte Spiel sein eigenes
   Verzeichnis in dem games Ordner bekommt (dazu rate ich, damit man
   nicht mit den anderen, manuell in das Verzeichnis installierten
   Spielen durcheinander kommt), öffnen wir die **/etc/make.conf** (als
   root); dort fügen wir einfach am Ende folgendes ein:

::

   .if${.CURDIR:M/usr/ports/games/*}
   PREFIX=/mnt/games/${PORTNAME}
   DATADIR=${PREFIX}/share
   .endif

Das wars! Ab jetzt werden alle eure Spiele nach
**/mnt/games/spielename** installiert. <note> Dies gilt nur, wenn die
Spiele aus den Ports installiert werden, nicht für Packages! Mehr Infos
darüber gibts hier im Wiki, aber da die meisten Spiele eh kein Package
haben dürfte das egal sein. </note>

<note> da die Ports nicht nach **/usr/local** installiert werden, sind
ihre ausführbaren Dateien nicht im PATH. Man sollte also nach der
Installation die ausführbaren Dateien eines Spieles (die im bin
Unterordner) mit **ln -s** nach **/usr/games** linken, damit man sie
auch ohne Pfadangabe aufrufen kann. </note>

Spiele ohne Ports unter Linux-Emulation
---------------------------------------

Zuerst sollte man sichergehen, dass die Linux-Emulation installiert ist.
Dazu sollten folgende Ports installiert sein:

-  **emulators/linux_base-fc6**
-  **textproc/linux-f10-expat**
-  **x11-fonts/linux-f10-fontconfig**
-  **x11/linux-f10-xorg-libs**
-  **devel/linux-f10-sdl12**

außerdem sollte in **/etc/rc.conf** der Eintrag **linux_enable="YES"**
nicht fehlen.

Wenn man ein Spiel aus den Ports installiert hat, welches den Prefix
"linux-" hat und es funktioniert, dann weiß man, dass man alles richtig
gemacht hat. :)

Jetzt kann man die Linux-Spiele auf "ihre natürliche Weise"
installieren, d.h. ausführen des jeweiligen Scripts oder der jeweiligen
Binary von der CD. Einige Programmierer haben den Bankrott von Loki
überlebt und bieten kostenlos "Installationhelfer" für Linux-Spiele an.
Diese einfachen Install-Wizards findet man hier:
http://www.liflg.org/?catid=6

Theoretisch sollten alle Linux-Spiele unter FreeBSD laufen! Beachtet
bitte, dass alle 3D Grafiktreiber mit Außnahme proprietären nVidia
Treiber im Moment kein "Mixed Mode" unterstützten. Das bedeutet, dass
auf FreeBSD/amd64 ein Linux-Spiel keine 3D-Beschleunigung nutzen kann.

Spieleliste
~~~~~~~~~~~

-  `Aquaria <http://www.bit-blot.com/aquaria/>`__ (Opensource, baut man
   den Code funktioniert das sich ergebende Binary unter FreeBSD jedoch
   fehlerhaft.)
-  `Heavy Metal FAKK2 <http://www.ritualistic.com/games.php/fakk2/>`__
   (Benötigt alte libz, stürzt sonst ab!)
-  `Heretic II <http://www.hereticii.com/>`__ (läuft in der von Loki
   Games entwickelten Linuxfassung)
-  `Hyperspace Delivery
   Boy <http://www.linuxgamepublishing.com/info.php?id=17&>`__
-  `Prey <http://icculus.org/prey/index-de.html>`__ (Achtung, benötigt
   libSDL aus den Ports, nicht die beiliegende!)
-  `Return to Castle
   Wolfenstein <http://www.idsoftware.com/games/wolfenstein/rtcw/>`__
   (Opensource, baut jedoch nur unter FreeBSD/i386 und man verliert
   Punkbuster)
-  `Rune <http://www.lokigames.com/products/rune/>`__, (läuft in der von
   Loki Games entwickelten Linuxversion)
-  `Serious Sam <http://www.serioussam.com/>`__
-  `Serious Sam - The Second Encounter <http://www.serioussam.com/>`__
-  `Serious Sam II <http://www.serioussam.com/>`__
-  `Soldier of
   Fortune <http://de.wikipedia.org/wiki/Soldier_of_Fortune>`__, (läuft
   in der von Loki Games entwickelten Linuxversion)
-  `Trine <http://www.trine-thegame.com>`__
-  `Unreal Tournament 2004 <Unreal Tournament 2004>`__
-  `Crossfire </Unreal Tournament 2004, TO/Crossfire>`__

Spiele mittels Wine
-------------------

Statusübersicht Wine Application Database
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In der Wine Application DB wird der Status der Spiele laufend erfasst.
Für die wichtigsten FreeBSD-Versionen findet man unter den folgenden
Links Informationen, welche Spiele (und Anwendungen) laufen oder auch
nicht:

-  `FreeBSD
   5.4 <http://appdb.winehq.org/objectManager.php?bIsQueue=false&bIsRejected=false&sClass=distribution&iId=55&sAction=view&sTitle=View+Distribution>`__
-  `FreeBSD
   6.2 <http://appdb.winehq.org/objectManager.php?bIsQueue=false&bIsRejected=false&sClass=distribution&iId=301&sAction=view&sTitle=View+Distribution>`__
-  `FreeBSD 7.0
   BETA1 <http://appdb.winehq.org/objectManager.php?bIsQueue=false&bIsRejected=false&sClass=distribution&iId=506&sAction=view&sTitle=View+Distribution>`__

Vorbereitung
~~~~~~~~~~~~

Zunächst sollte man eine aktuelle Version von Wine **emulators/wine**
installieren und mit dem Befehl **winecfg** konfigurieren, man sollte
nicht vergessen den Einhängepunkt des CD-Laufwerks als Laufwerk
hinzuzufügen und manuell als CD-Laufwerk zu definieren. Spiele die
keinen Kopierschutzmechanismus besitzen oder einen der von Wine
unterstützt wird, können auch .iso-Images der CDs benutzen, siehe dazu
`Wine </howto/Wine>`__. Man kann sehr einfach mittels **cp /dev/acd0
~/image.iso** ein CD-Image erzeugen.

Wine funktioniert unter FreeBSD leider etwas schlechter als unter
GNU/Linux, aber ein paar Sachen gehen schon.

Hier werden nur Spiele aufgelistet, die getestet wurden. Soweit nicht
anders angegeben installiert man das Spiel mit **wine setup.exe** und
startet das Spiel mit **wine spiel.exe** (man sollte schon den richtigen
Pfad und Namen angeben).

Bei OpenGl-Spielen sollte man noch ein "-opengl" als Parameter
dranhängen. Zur Zeit ist wine's threading im Umbau. Die meisten Spiele
brauchen im Moment noch die kthreads-binary, also einfach
**wine-kthread** statt **wine** als Befehl nehmen.

Ist bei *Installation* "nein" angegeben, muss man das Spiel unter
Windows installieren und den Installationsordner "rüberkopieren". Wer
ein GNU/Linux auf der Festplatte hat, kann auch versuchen es dort
mittels Cedega zu installieren. Manchmal (z.B. bei Starcraft und
Warcraft) ist die Installation auch nur unter Wine auf FreeBSD kaputt,
dann kann man auch unter Wine auf GNU/Linux installieren.

.. _spieleliste-1:

Spieleliste
~~~~~~~~~~~

=========================================== ==================================================================================================================================================================================================================================
Baldur's Gate II & Throne of Bhaal         
=========================================== ==================================================================================================================================================================================================================================
Hersteller                                  Bioware Corp
Installation                                Mit manuellem "unshielden" der .cabs oder unter GNU/Linux mit dem Loki-Installer.
Spielbar                                    Nein, dies liegt an FreeBSD. Unter GNU/Linux mit gleichen Einstellungen läufts 100%.
Patches                                     afaik nein
WineDB                                      http://appdb.winehq.org/appview.php?iAppId=272
WineDB-Rating                               Bronze
Webseite                                    http://www.bioware.com/games/shadows_amn/, http://www.bioware.com/games/throne_bhaal/
Sonstiges                                   Auf http://gemrb.sf.net gibt es einen freien Nachbau der Infinity-Engine, der Planescape:Torment, BGI, BGII+ToB, IWDI und IWDII unterstützen soll. Leider gibt es (noch) keinen Port, ich schaue mal was sich da machen lässt.
\                                          
Battle Realms                              
Hersteller                                  UBI
Wiki                                        `Battle Realms <Battle Realms>`__
Genre                                       Echtzeit Strategie
Patches                                     NoCD
Webseite                                    http://battlerealms.ubi.com
WineDB                                      http://appdb.winehq.org/appview.php?iAppId=3013
WineDB-Rating                               Gold
\                                          
Counter Strike 1.6 **ohne Steam**          
Installation                                nicht getestet
Spielbar                                    Nein, stürtzt beim Starten ab
WineDB                                      http://appdb.winehq.org/appview.php?iAppId=8
Rating                                      Bronze
Sonstiges                                   mit Steam und viel Gefrickel gehts angeblich
\                                          
Diablo II & Lord of Destruction            
Installation                                nein
Spielbar                                    Singleplayer und Multiplayer(LAN-UDP), Multiplayer(Battle.net)
Patches                                     NoCD
Wiki                                        `Diablo II <Diablo II>`__
WineDB                                      http://appdb.winehq.org/appview.php?iAppId=74
WineDB-Rating                               Gold
Webseite                                    http://www.blizard.com/diablo2, http://www.blizzard.com/diablo2exp
Sonstiges                                   wine-kthread benutzen
\                                          
Dungeon Keeper II                          
Installation                                mit manuallem Entpacken und viel Gefrickel
Spielbar                                    momentan nicht
Patches                                     NoCD
WineDB                                      http://appdb.winehq.org/appview.php?iAppId=631
WineDB-Rating                               Bronze
\                                          
Guild Wars                                 
Version                                     24.155
Installation                                Ja
Spielbar                                    Ja
Wine-Version                                0.9.48
WineDB                                      http://appdb.winehq.org/appview.php?iAppId=2243
WindDB-Rating                               Gold
Sonstiges                                   Der Guild Wars-Ordner kann von Windows komplett kopiert werden und ist unter wine lauffähig
\                                          
Perfect World                              
Version                                     1.26
Installation                                Ja
Spielbar                                    wird zur Zeit getestet --- `Elwood <elwood@bsdforen.de>`__\ *04.11.2007 - 22:41*
Wine-Version                                0.9.48
WineDB                                     
WindDB-Rating                              
Webseite                                    http://www.perfectworld.com.my
Sonstiges                                   Kostenloses MMORPG in englisch oder chinesisch
\                                          
Shogo: Mobile Armor Devision               
Installation                                ja
Spielbar                                    momentan nicht
Rating                                      Bronze
Webseite                                    http://www.shogo-mad.com
\                                          
Soldier of Fortune 2                       
Installation                                manuell, siehe http://www.bsdforen.de/showthread.php?p=124653
Spielbar                                    Singleplayer und Multiplayer.
Patches                                     NoCD
WineDB                                      0
WineDB-Rating                               Gold
Webseite                                    http://www.ravensoft.com/soldier2.html
Sonstiges                                   Texturen im Spiel nicht auf "very high" setzen, Texturkompressionen aktivieren
\                                          
Starcraft & Starcraft BroodWar             
Installation                                nein, aber unter Wine auf GNU/Linux.
Spielbar                                    Singleplayer und Multiplayer(LAN-UDP) funktionieren, Multiplayer(Battle.net) sehr unkomfortabel
Wiki                                        `Starcraft <Starcraft>`__
WineDB                                      http://appdb.winehq.org/appview.php?iAppId=72
WineDB-Rating                               Silber
Webseite                                    http://www.blizzard.com/starcraft, http://www.blizzard.com/broodwar
Sonstiges                                   wine-kthread benutzen
\                                          
Warcraft III & Frozen Throne               
Installation                                nein, aber unter Wine auf GNU/Linux, jedoch nur mit Wine-Versionen >= 0.9.6 nach dem Rüberkopieren, per ``'regedit``' unter HKEY_CURRENT_USER\\Software\\Blizzard Entertainment\\Warcraft III folgende String-Einträge hinzufügen:
\                                           "InstallPath"="Z:\\mein\\unix\\pfad\\nach\\WarcraftIII"
\                                           "Program"="Z:\\mein\\unix\\pfad\\nach\\WarcraftIII\\war3.exe"
\                                           "War3CD"="E:\\"
Spielbar                                    Singleplayer und Multiplayer(LAN), Multiplayer(Battle.net)
Patches                                     Es werden nur Patches unterstützt, die nicht "Wrapper" sind (siehe AppDb)
WineDB                                      http://appdb.winehq.org/appview.php?iAppId=897
Wine Version                                0.9.14
WineDB-Rating                               Silber
Webseite                                    http://www.blizzard.com/war3, http://www.blizzard.com/war3x
Sonstiges                                   wine-kthread benutzen.
\                                          
Warhammer 40k: Dawn of War & Winter Assault
Installation                                nein
Spielbar                                    Nein, beide stürzen beim Starten ab.
Patches                                     NoCD
WineDB                                      http://appdb.winehq.org/appview.php?iAppId=1918
WineDB-Rating                               Müll
Sonstiges                                   unter GNU/Linux geht es angeblich.
\                                          
World of Warcraft                          
Wiki                                        WoW
WineDB                                      http://appdb.winehq.org/appview.php?iAppId=1922
WineDB-Rating                               Gold
Webseite                                    http://www.worldofwarcraft.com
=========================================== ==================================================================================================================================================================================================================================

Außer Wine gibt es noch zwei gängige Windowsemulatoren: WineX/Cedega und
Crossover Wine, beide sind proprietär und funktionieren nur bedingt
unter FreeBSD. Spiele unter Cedega werden hier deshalb nicht besprochen,
wer aber frickeln will findet hier einen Link.

Links
~~~~~

-  Cedega unter FreeBSD (mit Port): http://cedega.firepipe.net/
-  Spiele unter Wine (nur auf GNU/Linux getestet):
   http://www.frankscorner.org
-  Wine Hauptseite mit Tipps, AppDb und Bugcenter: http://www.winehq.org

Konsolen- und DOS-Spiele
------------------------

Da es unzählige Spiele für unzählige Konsolen gibt, werden hier keine
Spiele getestet, sondern nur die Emulatoren aufgelistet, nach Platform
sortiert:

-  Amiga : UAE (emulators/uae)
-  Atari : Atari800 (emulators/atari800), Hatari (emulators/hatari)
-  Commodere 64 : Frodo (emulators/frodo), YAPE (emulators/yape)
-  Lucasarts Adventures : scummvm (games/scummvm)
-  PC-engine : VPCE (emulators/vpce)
-  NeoGeo : GnGeo (emulators/gngeo)
-  Nintendo 64 : Mupen64 (emulators/mupen64*)
-  Nintendo Gameboy : GBE (emulators/gbe), GNGB (emulators/gngb), VGB
   (emulators/vgb)
-  Nintendo Gameboy Advance : VBA (emulators/vba*)
-  Nintendo Gameboy Color : GnuBoy (emulators/gnuboy)
-  Nintendo Gamecube : GCube (emulators/gcube)
-  Nintendo NES : ines (emulators/ines), TuxNES (emulators/tuxnes)
-  Nintendo SNES : Snes9x (emulators/snes9x), Zsnes (emulators/zsnes)
-  Sega Genesis : DGen (emulators/dgen-sdl), Generator
   (emulators/generator*), GenS (emulators/gens)
-  Sega Mastersystem/GameGear : MasterGear (emulators/mastergear)
-  Sony Playstation : ePSXe (emulators/linux-ePSXe)

-  MSDOS : DosBox (emulators/dosbox)

und dann gibt es noch M.A.M.E, der fast alles emuliert, unter
(emulators/xmame). Für xMAME existieren auch diverse grafische
Front-Ends in den Ports. Einfach mal schauen!


* :ref:`genindex`

Zuletzt geändert: |date|

