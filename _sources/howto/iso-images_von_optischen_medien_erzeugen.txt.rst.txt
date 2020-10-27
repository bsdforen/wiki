ISO-Images von optischen Medien erzeugen
========================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Um ein ISO-Image einer CD, DVD oder ähnlichem zu erzeugen werden lediglich
Leserechte für die Device Node des optischen Laufwerks benötigt. Das Image wird
mit dem Befehl:

::

   # dd if=/dev/cd0 of=image.iso bs=2048

erzeugt. Natürlich muss /dev/cd0 durch den für sein System passenden
Eintrag ersetzt werden.

Alternativ zu **dd** kann **cp** verwendet werden um ein Image
anzulegen. Das hat den Vorteil, dass die Blockgröße nicht beachtet
werden muss.

::

   # cp /dev/cd0 image.iso

Nach dem Anlegen eines Images sollte es auf Fehlerfreiheit geprüft
werden. Dazu kann man einen Hash-Algorithmus wie md5 verwenden. Dem
vorherigen Beispiel folgend sollte die Ausgabe der folgenden Kommandos
identisch sein:

::

   # md5 < image.iso
   # md5 < /dev/cd0

Sind sie es nicht, so ist beim Kopieren ein Fehler aufgetreten.

Siehe auch
----------

-  `ISO-Images mounten </howto/ISO-Images mounten>`__

* :ref:`genindex`

Zuletzt geändert: |date|

