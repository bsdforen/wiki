Starcraft
=========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-netbsd.png

Hier wird eine Empfehlung dafür gegeben wie man Starcraft mit Hilfe von wine
betreibt.

.. note::

  Seit der Umstellung von kthread auf pthread unter FreeBSD funktioniert
  Starcraft leider nicht ohne weiteres unter wine. Die Verwendung der Binary
  `wine#wine-kthread </howto/wine#wine-kthread>`__ schafft hier Abhilfe.
  
Installation
------------

Die Installation muss leider unter Windows erfolgen, da sie mit wine
nicht durchführbar ist. Meine Spiele liegen auf einer Fat32 Partition
die in /mnt/msdos/software gemountet ist. Der Einfachheit halber werde
ich bei Beispielen meine Verzeichnisstruktur nutzen.

Nachdem das Spiel für mein OS erreichbar ist muss ich es einmal starten
damit die Registry Einträge gemacht werden.

::

   $ wine /mnt/msdos/software/games/rts/Starcraft/starcraft.exe

Selbst wenn das Spiel gleich funktioniert sollte es trotzdem gleich
beendet werden. Denn jetzt muss noch der korrete Pfad in die Registry
eingetragen werden. Dazu wird

::

   $ regedit

aufgerufen. Sollte das nicht funktionieren geht es wahrscheinlich mit

::

   $ wine regedit

Es öffnet sich der Registrierungseditor wie unter Windows. Dort
navigiert man zu **My
Computer->HKEY_CURRENT_USER->Software->Blizzard->Starcraft**. Dann wählt
man **Edit->new->String Value**. Den neuen String benennt man nun in
**InstallPath** um und weist ihm den Wert
**z:\mnt\msdos\software\games\rts\Starcraft** zu.

Konfiguration
-------------

Ab hier geht man wie in dem Artikel `Wine </howto/Wine>`__ beschrieben
vor. Ich liste hier lediglich die Einstellungen die für mich persönlich
am besten funktionieren.

Graphics
~~~~~~~~

========= =========================================================
\_        Allow DirectX apps to stop the mouse leaving their window
========= =========================================================
\        
X         Enable desktop double buffering
\        
X         Allow the window manager to control the windows
\        
X         Emulate a virtual desktop
\        
640 x 480
========= =========================================================

Audio
~~~~~

===================== ================
Hardware Acceleration
Emulation            
\                    
X                     Driver Emulation
===================== ================

Updates und Battle.net
----------------------

Um Starcraft auf den neusten Stand zu bringen kann man eine Verbindung
zum Battle.net herstellen. Von dort aus kann man auch an Spielen auf
Blizzards Servern teilnehmen. Leider verwendet Blizzard für das
Battle.net Interface eine eigene Engine, die nicht so gut funktioniert
wie das eigentliche Spiel.

Die Engine stellt das eigentliche Interface im Hintergrund da, darüber
kommt in einem eigenen Layer ein Text. Leider ist der Teil dieses Layers
der unsichtbar sein sollte schwarz und wird zudem nicht aktuallisiert.
Das heißt man sieht einen haufen teilweise übereinander liegenden
Textes. Der Hintergrund blitzt lediglich gelegentlich auf.

Trotzdem kann man mit etwas Übung im Battle.net navigieren und sogar
Spiele starten oder ihnen beitreten. Wenn das Spiel in die eigentliche
Starcraft Engine zurückkehrt muss man eine Weile warten. Geduld, das
Spiel ist nicht abgestürzt.

Damit heruntergeladene Updates funktionieren muss man wie zu Beginn
beschrieben InstallPath in der Registry richtig gesetzt haben.

Wrapper
-------

Um das Spiel komfortabler zu starten habe ich mir in **~/bin** ein
Skript angelegt. Das Skript setzt das Skript im `Wine
Artikel </howto/Wine#Programme die Images benötigen>`__ voraus.

::

   #!/bin/sh

   wine_suffix="-kthread"
   mount_images="/mnt/msdos/vault/images/starcraft.iso
   /mnt/msdos/vault/images/broodwar.iso"
   run_binary="starcraft.exe"
   run_folder="/mnt/msdos/software/games/rts/Starcraft"

   . winexec

.. note::

  Seit der Version 1.15.2 wird keine CD / keine ISOs mehr zum Spielen benötigt.
  Wie man das einrichtet steht hier:
  http://us.blizzard.com/support/article.xml?articleId=21150&rhtml=true .  Auf
  die mount_images-Option kann im Script also auch verzichtet werden!  

* :ref:`genindex`

Zuletzt geändert: |date|

