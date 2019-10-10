nVidia Twinview
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Aritkel beschreibt, wie man beim proprietären Treiber von Nvidia den
Mehrbildschirmbetrieb - Twinview einstellt.

In der Konfigurationsdatei von Xorg z.B. /etc/X11/xorg.org wird zunächst
die Zeile Section “Screen” gesucht. Folgende Zeilen einfach hinter
DefaultDepth “24” einfügen:

::

   Option "TwinView"
   Option "MetaModes" "1280x1024,1280x1024;1024x768,1024x768;800x600,800x600;640x480,640x480"
   Option "TwinViewOrientation" "Clone"

In *MetaModes* müssen die entsprechenden Einstellung gemäß der
Spezifikationen des Monitors eingestellt werden. Bitte bei den Werten
aufpassen, die Hardware könnte unter Umständen beschädigt werden. Die
Auflösungen werden im Pärchenweise angegeben z.B. 1024x768,1024x768 und
durch ein Kommata getrennt. Die Pärchen werden durch ein Semikolon
getrennt. Folgende Optionen gibt es für *TwinViewOrientation*:

-  "LeftOf"
-  "RightOf" - Standardeinstellung
-  "Above"
-  "Below"
-  "Clone"

Bei Clone wird auf dem 2. Monitor der gleiche Bildschirminhalt
angezeigt.

Siehe auch
----------

-  `Mehrmonitor-Betrieb <Mehrmonitor-Betrieb>`__ - Hier eine allgemeine
   Beschreibung zur Einrichtung von Mehrmonitorbetrieb
-  `Multihead mit nv <Multihead mit nv>`__ - Eine Beschreibung für den
   freien nvidia-Treiber nv
-  http://http.download.nvidia.com/freebsd/1.0-8178/README/appendix-d.html
   - NVIDIA Accelerated FreeBSD Driver Set README and Installation Guide

* :ref:`genindex`

Zuletzt geändert: |date|

