Unreal
======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieses HowTo beschreibt die Installation und Einrichtung von Unreal. Hier geht
es nicht um "Unreal Gold" oder die Erweiterung "Return to Na Pali".

Vorraussetzungen
----------------

-  Eine Unreal Installations CD.
-  Den UnrealClassicPatch227i von
   `hier <http://www.oldunreal.com/oldunrealpatches.html>`__.
-  Den UnrealPatch226Final von
   `hier <http://www.oldunreal.com/officialpatches.html>`__

Benötigte Pakete
----------------

Folgende Pakete werden zur Installation bzw. zum Spielen benötigt:

-  `emulators/linux_base-c6 <https://www.google.com/search?q=emulators/linux_base-c6&btnI=lucky>`__

::

   auf amd64 Systemen:
   * [[freshports>emulators/i386-wine]] Nur zur installation des Spiels und der Patches.
   auf i386 Systemen:
   * [[freshports>emulators/wine]] Nur zur Installation des Spiels und der Patches.
   Ein Programm zum dearchivieren der Patches:
   * [[freshports>archivers/p7zip]] oder
   * [[freshports>archivers/unzip]] oder
   * [[freshports>archivers/unrar]] 

Die Linux hosts Datei bearbeiten
--------------------------------

Damit im Spiel das Speichern/Laden funktioniert muss der Hostname des
Rechners in der Datei /compat/linux/etc/hosts eingetragen werden.

::

   127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4 hostname
                                                                                ^^^^^^^^

Installation
------------

Mit winecfg ein CD Laufwerk anlegen, das auf den Mountpunkt der Unreal
CD zeigt, da es für die Installation der Patches benötigt wird. |image1|

Unreal mit wine installieren.

::

   # wine /PATH/TO/CD/SETUP.EXE

Im Installer dann den Zielpfad eingeben und im Fenster "Select
Components" DirectX 5 und den UnrealEd abwählen. |image2|

Das Spiel Patchen
-----------------

Zuerst den offiziellen Patch 226Final installieren da der Patch 227i
mindestens den Patch 224 vorraussetzt.

::

   # wine /PATH/TO/PATCHES/UnrealPatch226Final.exe

Dann den Patch 227i

::

   # wine /PATH/TO/PATCHES/UnrealClassicPatch227i.exe

Und nicht vergessen den Haken bei "Linux native Dateien" zu setzen.
|image3|

Benötigte Libs kopieren
-----------------------

Im Installationsverzeichnis gibt es im Ordner "Help" die Datei
lin_convenience_libs.tar.bz2. Diese muss entpackt und die Dateien in das
Verzeichnis "System" verschoben werden.

::

   # cd /PATH/TO/UNREAL/Help
   # tar -xjf lin_convenience_libs.tar.bz2
   # cd lin_convenience_libs
   # mv * ../../System

Binary ausführbar machen
------------------------

Die Datei UnrealLinux.bin hat bei mir zu Problemen mit der Maus geführt
weshalb ich das Spiel über UnrealXLinux.bin starte. Dazu muss diese noch
ausführbar gemacht werden.

::

   # cd /PATH/TO/UNREAL/System
   # chmod u+x UnrealXLinux.bin

Spiel starten
-------------

Nun kann das Spiel endlich gestartet werden:

::

   # cd /PATH/TO/UNREAL/System
   # ./UnrealXLinux.bin

verwandte Artikel
-----------------

Es empfiehlt sich zu diesem Thema auch folgenden Artikel gelesen zu
haben.

-  `FreeBSD - Spiele <hauptseite>`__

Links
-----

-  `Homepage von Old Unreal <http://www.oldunreal.com/index.html>`__

.. |image1| image:: images/unreal-winecfg.png
   :class: align-center
   :width: 200px
.. |image2| image:: images/unreal-select_components.png
   :class: align-center
   :width: 200px
.. |image3| image:: images/unreal-patch227i.png
   :class: align-center
   :width: 200px

* :ref:`genindex`

Zuletzt geändert: |date|

