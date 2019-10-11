Diablo II
=========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Hier wird eine Empfehlung dafür gegeben wie man Diablo II mit Hilfe von Wine
betreibt.

.. note::

  Seit der Umstellung von kthread auf pthread unter FreeBSD funktioniert Diablo
  II leider nicht ohne weiteres unter wine. Die Verwendung der Binary
  `wine#wine-kthread </howto/wine#wine-kthread>`__ schafft hier Abhilfe.
  
Installation
------------

Die Installation muss leider unter Windows erfolgen, da sie mit Wine
nicht durchführbar ist. Bevor man das Spiel dann unter Wine startet
sollte man sich noch ins Battle.net einloggen um es auf den neusten
Stand zu bringen. Den Rest kann man dann vom BSD seiner Wahl aus tun.

Je nachdem ob man Diablo II oder Lord of Destruction verwendet, sollte
man ein `Image </howto/ISO-Images von optischen Medien erzeugen>`__ der
entsprechenden CD anlegen.

Jetzt versucht man einmal Diablo II zu starten, was höchstwahrscheinlich
scheitern wird, aber einige Einträge in der Registry hinterlässt. Danach
sollte man den Registry Editor, entweder mit

::

   $ regedit

oder mit

::

   $ wine regedit

starten.

Jetzt navigiert man zu **HKEY_CURRENT_USER/Software/Blizzard
Entertainment/Diablo II** und legt dort den String **InstallPath** an,
der das Diablo II Verzeichnis in Windows Schreibweise enthalten sollte
(z.B. **Z:\mnt\msdos\software\games\rpg\Diablo II**).

Konfiguration
-------------

Ab hier geht man wie in dem Artikel `Wine </howto/Wine>`__ beschrieben
vor. Die genannten Einstellung werden vom Autor verwendet.

Graphics
~~~~~~~~

=============================== =========================================================
\_                              Allow DirectX apps to stop the mouse leaving their window
=============================== =========================================================
\                              
X                               Enable desktop double buffering
\                              
X                               Allow the window manager to control the windows
\                              
Vertex Shader Support: hardware
\                              
X                               Allow Pixel Shader
=============================== =========================================================

Das Spiel starten
-----------------

Ab diesem Punkt sollte sich die 'Game.exe' starten lassen. Es empfiehlt
sich 'D2VidTest.exe' zu starten und auszuprobieren ob der Direct3D Modus
funktioniert - beim Autor ist das der Fall. Wenn es irgendwelche
Probleme gibt sollte man aber erst einmal versuchen das Spiel mit
DirectDraw zu starten.

Updates
-------

Updates im Battle.net funktionieren ohne Einschränkungen.

Wrapper
-------

Um das Spiel komfortabler zu starten kann man sich in '~/bin' ein Skript
angelegen. Das Skript setzt das Skript im `Wine
Artikel </howto/Wine#Programme die Images benötigen>`__ voraus.

::

   #!/bin/sh

   wine_suffix="-kthread"
   #wine_log="/dev/null"
   mount_images="/mnt/msdos/vault/images/lord of destruction.iso"
   mount_nodes="yes"
   run_binary="Game.exe"
   run_folder="/mnt/msdos/software/games/rpg/Diablo II"
   run_parameters="-skiptobnet"

   run_after_cmd="xrefresh"

   . winexec

* :ref:`genindex`

Zuletzt geändert: |date|

