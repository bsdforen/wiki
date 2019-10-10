PC-Speaker deaktivieren
=======================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Kommt ein schriller Warnton aus den Lautsprechern, ist vermutlich der
PC-Speaker aktiviert. Oft kann man aber auf diese Art der Mitteilung
verzichten. Der folgende Artikel beschreibt die Möglichkeiten, wie man
diesen verstummen lässt.

In der Konsole
--------------

Temporär ausschalten
~~~~~~~~~~~~~~~~~~~~

In NetBSD kann über wsconsctl der Speaker leiser beziehungsweise
ausgeschaltet werden. Folgende Befehle deaktivieren diesen in der
Konsole:

::

   # wsconsctl -w bell.pitch=0 

beziehungsweise

::

   # wsconsctl -w bell.volume=0

Die Einstellungen gehen nach dem nächsten Bootvorgang verloren.

Persistent
~~~~~~~~~~

Nun können die beiden Befehle auch **rc.local** hinzugefügt werden,
welches den Vorteil bietet robust auf Aktualisierungen zu reagieren. Die
richtige Stelle dies dem System bekannt zugeben ist aber
**wscons.conf**. Dort definiert man das Überschreiben der Variable, hier
am Beispiel bell.pitch:

::

   # setvar wskbd bell.pitch=0

Ein OS-Upgrade übersteht diese Änderung nicht.

In XWindow
----------

In X können Einstellungen über **xset** manipuliert werden. Der folgende
Befehl, welcher im lokalen Startskript **.xinitrc** geschrieben werden
kann, schaltet auch dort den Speaker aus:

::

   #  xset b off

Im Kernel
---------

Zu guter Letzt besteht auch die Möglichkeit im Kernel das entsprechende
Device auszuschalten. Hierzu kommentiert man das Gerät **pcppi** und
alle Verweise darauf aus. Dadurch wird das Gerät nicht mehr erkannt und
kann dementsprechend auch nicht mehr verwendet werden. Zwar haben die
Auswirkungen global, d.h. auch in X ist der Ton nicht mehr zu hören,
andererseits muss bei jeden Kernel Update diese Änderung erneut
vorgenommen werden.

* :ref:`genindex`

Zuletzt geändert: |date|

