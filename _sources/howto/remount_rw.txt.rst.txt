Remount RW Read/Write
=====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Bei verschiedenen Boot-Problemen kommt es vor, dass die Dateisysteme aus
Sicherheitsgründen nur im Readonly-Modus gemountet werden. Um Fehler zu
beheben, muss jedoch oft das Dateisystem beschreibbar sein.

Wenn das Root-Dateisystem z.B. nur readonly gemountet ist, dann kann es
wie folgt in den RW-Modus gebracht werden:

::

   mount -uw /

Wenn die Variable **PATH** noch nicht gesetzt ist, (z.B. im
Single-User-Mode) lautet der Befehl:

::

   /sbin/mount -uw /

* :ref:`genindex`

Zuletzt geändert: |date|

