Auflösung der Konsole ändern
============================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

VESA steht für Video Electronics Standards Association und bezeichnet eine
nicht profitorientierte Vereinigung, der viele internationale Unternehmen
angehören - darunter befinden sich z.B. Dell, Fujitsu, LG, Microsoft und
Philips. Die VESA bemüht sich um die Schaffung von Industriestandards bezüglich
Grafikadapter und Darstellungsgeräte. VESA definierte auch eine Schnittstelle
bzw.  Grafikmodi, die es u.a. ermöglichen höhere Auflösungen ohne
kartenspezifische Treiber zu verwenden. Die folgenden Abschnitte widmen sich
der Aufgabe, diese VESA-Modi auf der FreeBSD Konsole zu verwenden.

Kernel kompilieren
------------------

Den Kernel mit folgenden Optionen
`kompilieren </freebsd/Kernel erstellen>`__:

::

   options VESA                   # VGA VESA Video Modi
   options VGA_WIDTH90            # optional für 90-Spalten-Modus
   options SC_PIXEL_MODE          # Raster Textmodus

Alternativ reicht auch ein Eintrag in die ``/boot/loader.conf``.

::

   vesa_load="YES"

Einen Neustart kann durch Laden des Moduls zur Laufzeit vermieden
werden:

::

   # kldload vesa

Auflösung ändern
''''''''''''''''

Nach erfolgter Installation des neuen Kernels oder Laden des Moduls kann
man sich mit

::

   $ vidcontrol -i mode

die unterstützten Videomodi anzeigen lassen.

Die gewünschte Auflösung einschalten mit:

::

   $ vidcontrol MODE_<Nummer>

Damit das bei jedem Start automatisch passiert und auf jede virtuelle
Konsole angewendet wird, braucht man einfach nur

::

   allscreens_flags="MODE_<Nummer>"

in die ``/etc/rc.conf`` eintragen.

Mehr dazu unter: **vidcontrol(1)**

Bildschirmschoner
-----------------

Mit dem Eintrag

::

   saver="logo"

in ``/etc/rc.conf`` hat man einen netten Bildschirmschoner, der nach
fünf Minuten Inaktivität angeht.

Mehr Infos unter: **splash(4)**

* :ref:`genindex`

Zuletzt geändert: |date| 

