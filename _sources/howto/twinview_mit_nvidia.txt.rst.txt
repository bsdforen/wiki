TwinView mit dem nVidia-Treiber für FreeBSD, Linux und Solaris
==============================================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dies Tutorial beschreibt die grundlegende Einrichtung von TwinView in
Zusammenarbeit dem nVidia-Treiber für FreeBSD. Es richtet sich dabei an
Einsteiger mit grundlegenden Kenntnissen über die Konfiguration des X-Servers.

TwinView ist eine von nVidia entwickelte Technik, um zwei Monitore unter
X zu einem Screen einfach verbinden zu können. X kennt dabei nur einen
Monitor und Screen, die Verteilung und ein Großteil der Konfiguration
wird von dem Treiber übernommen. Ebenso stellt dieser eine entsprechende
Xinerama-Erweiterung zur Verfügung, sodass Programme die Bildschirme
optimal nutzen können.

Anforderungen
-------------

Benötigt werden eine nVidia-Grafikkarte mit 2 Monitorausgängen. Sie muss
zudem TwinView unterstützen, das ist bei den meisten Modellen nach der
GeForce 2 der Fall. Im Zweifel kann es helfen, die README.linux des
Treibers einzusehen. Als Betriebsystem FreeBSD, wobei sich dies Tutorial
auch ohne größere Probleme unter Linux oder Solaris umsetzten lassen
sollte. Der nVidia-Treiber sollte in einer aktuellen Version installiert
und das Kernelmodul geladen sein. Ich gehe hier davon aus, dass X
bereits mit dem Treiber konfiguriert ist und funktioniert.

--------------

Welcher Monitor ist der erste und welcher der zweite?
-----------------------------------------------------

Um dies festzustellen, schließt man beide Monitore an der der
Grafikkarte an und startet den Rechner neu. Leider erkennen viele
Grafikkarten den zweiten Monitor nur bei einem Neuaufruf des GrafikBIOS.
Nun wird X normal mittels ``startx`` gestartet. Der Monitor, auf dem das
Bild erscheint ist der erste, der andere folglich der zweite. Zum
Wechseln tauscht man einfach die Anschlüsse an der Grafikkarte um.

Die Konfiguration des X-Servers
-------------------------------

Zuerst sollten die Spezifikationen der Monitore herausgesucht und bereit
gelegt werden. Dann erstellt man eine Sicherheitskopie der
/etc/X11/xorg.conf und öffnet das Original in einem Editor. Interessant
ist an dieser Stelle nur die Sektion „Device“. Alle folgenden Änderungen
werden an dieser vorgenommen. Dort fügt man folgende Zeilen hinzu:

::

   Option „TwinView“

Dies aktiviert TwinView.

::

   Option "SecondMonitorHorizSync" "50-160"

Dies sind die Werte für die horizontale Bildwiederholungsrate des
zweiten Monitors. Diese sollten auf jeden Fall korrekt angegeben werden,
da sonst die Gefahr eines Schadens am Monitor besteht!

::

   Option "SecondMonitorVertRefresh" "30-96"

Dies sind die Werte für die horizontale Bildwiederholungsrate des
zweiten Monitors. Diese sollten auf jeden Fall korrekt angegeben werden,
da sonst die Gefahr eines Schadens am Monitor besteht!

::

   Option "MetaModes" "1024x768,1280x960"

Diese Zeile ist die wichtigste. Sie legt die Auflösungen der einzelnen
Monitore fest. Links steht die Auflösung für den ersten Monitor, rechts
die des zweiten. Es können mehrere Paare angegeben werden, die Trennung
erfolgt durch Komma. Soll einer der beiden Monitore nicht genutzt
werden, trägt man anstelle einer Auflösung „NULL“ ein. Um eine in der
Sektion „Monitor“ definierte ModeLine zu nutzen, wird der Name dieser
anstelle der Auflösung eingetragen.

Die Sektion "Device" sollte jetzt in etwa so aussehen:

::

   Section "Device"
           Identifier  "Card0"
           Driver      "nvidia"
           Option      "TwinView"
           Option      "MetaModes" "1280x960,1280x960"
           Option      "SecondMonitorHorizSync"     "50-160"
           Option      "SecondMonitorVertRefresh"   "30-96"
   EndSection

Nun muss noch der erste Monitor korrekt in der Sektion Monitor
konfiguriert werden. Wie dies geschieht findet sich in diversen anderen
Tutorials oder in den entsprechenden Dokumentationen des verwendeten
X-Servers.

Mittels ``startx`` wird X nun gestartet. Erscheint auf beiden Monitoren
ein Bild ist alles in Ordnung und die Konfiguration von TwinView
abgeschlossen. Bricht X hingegen mit einem Fehler ab, oder das Bild
erscheint auf nur einem Monitor, liegt ein Fehler oder Problem vor.

--------------

Problemsuche
~~~~~~~~~~~~

Ist die Konfiguration in Ordnung?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Es sollte alles noch einmal auf Tippfehler überprüft werden. Ein
beliebter Fehler ist das Vergessen von Anführungsstrichen.

Sind Modelines richtig benannt?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Modelines dürfen nicht den Namen als MetaMode angegebender Auflösungen
haben, dies verwirrt den Treiber. Stattdessen sollten andere eindeutige
Bezeichnungen gewählt werden, wie z.B. „monitor1“ oder „iiyama“.

Analyse der X.log
^^^^^^^^^^^^^^^^^

Bei nicht lösbaren Fehlern kann es helfen die Logfiles zu lesen und nach
Fehlern hin zu durchsuchen. Diese müssen nicht unbedingt mit (EE)
beginnen, es kann sich auch um Hinweise handeln.

Lesen der README.Linux
^^^^^^^^^^^^^^^^^^^^^^

In dieser stehen noch eine Reihe weiterer Optionen und
Konfigurationsmöglichkeiten, die in seltenen Fällen benötigt werden.

--------------

Besonderen Dank an chkbsd aus #bsdforen.de, er hatte die entscheidende
Idee für die Modelines.

* :ref:`genindex`

Zuletzt geändert: |date|

