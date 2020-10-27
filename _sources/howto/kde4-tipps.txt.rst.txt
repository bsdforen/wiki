KDE4 Tipps
==========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Performance-Probleme
--------------------

Bei KDE4 (besonders unter FreeBSD) hakt es noch an der einen oder
anderen Stellen. Hier werden einige Tipps gesammelt

KWin (Fenstermanager)
~~~~~~~~~~~~~~~~~~~~~

Das setzen von QT_NO_GLIB kann Abhilfe schaffen. Dies kann man bequem im
KDE-startskript /usr/local/kde4/bin/startkde passieren. Füge einfach zu
Beginn des Files irgendwo folgendes ein

::

   export QT_NO_GLIB=1

Konqueror
~~~~~~~~~

Dateibrowsen zu langsam
^^^^^^^^^^^^^^^^^^^^^^^

Da unter FreeBSD im Moment kein FamD läuft, wird stat zum überprüfen von
Dateien benutzt was relativ langsam ist und außerdem die CPU belastet
(auffälig an permanent hoher CPU-Belastung durch kded4).

Abhilfe schafft die Rate der Verzeichnis-überwachung von kded4
herunterzusetzen. Dazu trägt man in .kde4/share/config/kdedrc folgendes
ein

::

    [DirWatch]
    PollInterval=60000

Das reduziert die Rate auf 1/Minute von dem Standard > 1/Sekunde. [1]

Verweise
--------

`kde4 </freebsd/kde4>`__

[1] https://bugs.kde.org/show_bug.cgi?id=155904

* :ref:`genindex`

Zuletzt geändert: |date|

