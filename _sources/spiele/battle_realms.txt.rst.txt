Battle Realms
=============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Hier wird eine Empfehlung dafür gegeben wie man Battle Realms mit Hilfe von
wine betreibt.

Installation
------------

Die Installation muss leider unter Windows erfolgen, da sie mit wine
nicht durchführbar ist. Meine Spiele liegen auf einer Fat32 Partition
die in **/mnt/shared** gemountet ist. Der Einfachheit halber werde ich
bei Beispielen meine Verzeichnisstruktur nutzen.

Alles was man jetzt noch tun muss ist die gewünschte Auflösung
einzustellen. Das kann man in der Datei **/mnt/shared/games/rts/Battle
Realms/Battle_Realms.ini** tun.

Konfiguration
-------------

Ab hier geht man wie in dem Artikel `Wine </howto/Wine>`__ beschrieben
vor. Ich liste hier lediglich die Einstellungen die für mich persönlich
am besten funktionieren.

Graphics
~~~~~~~~

== =========================================================
X  Allow DirectX apps to stop the mouse leaving their window
== =========================================================
X  Enable desktop double buffering
\ 
\_ Allow the window manager to control the windows
\ 
\_ Emulate a virtual desktop
== =========================================================

Audio
~~~~~

===================== ================
Hardware Acceleration
Full                 
\                    
\_                    Driver Emulation
===================== ================

Probleme
~~~~~~~~

-  Das Spiel erkennt seine CDs nicht, egal ob es eine CD im Laufwerk
   oder ein ISO image ist. Man muss sich also einen NOCD-Crack besorgen.
-  Das Spiel erkennt kein Netz.

Wrapper
-------

Um das Spiel komfortabler zu starten habe ich mir in **~/bin** ein
Skript angelegt. Das Skript setzt das Skript im `Wine </howto/Wine>`__
Artikel voraus.

::

   #!/bin/sh

   wine_log="/dev/null"
   mount_images="/mnt/msdos/vault/images/battle realms.iso
   /mnt/msdos/vault/images/winter of the wolf.iso"
   run_binary="Battle_Realms_F.exe"
   run_folder="/mnt/msdos/software/games/rts/Battle Realms"

   . winexec

* :ref:`genindex`

Zuletzt geändert: |date|

