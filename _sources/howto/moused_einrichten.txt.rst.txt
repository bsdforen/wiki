moused einrichten
=================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Unter FreeBSD kann ``moused`` alle Mäuse transparent über das Interface
``/dev/sysmouse`` zur Verfügung stellen. Ein weiterer Zweck von ``moused`` ist
es die Maus auf der Konsole zur Verfügung zu machen.

moused verwenden
----------------

Bei USB-Mäusen wird moused automatisch gestartet, trotzdem gibt es
natürlich allerlei Konfigurationsmöglichkeiten. Beim Einsatz einer
seriellen oder PS/2 Maus (zum Beispiel ein Touchpad in einem Notebook)
muss der ``moused`` erst eingerichtet werden.

Zum Testen reicht oft schon der folgende Befehl:

::

   # moused -p /dev/psm0

Dieses Beispiel aktiviert ``moused`` für eine PS/2 Maus. Der Typ kann
auch angegeben werden, das ist normalerweise aber nicht notwendig. Hat
die Maus nur zwei Tasten, wie das leider bei Touchpads die Regel ist,
hilft der Parameter ``-3``, der die dritte Taste über das gleichzeitige
Drücken der Tasten simuliert.

Um den ``moused`` entsprechend neu zu starten genügen die folgenden
Befehle:

::

   # kill `cat /var/run/moused.pid`
   # moused -p /dev/psm0 -3

moused automatisch starten
--------------------------

Um ``moused`` beim Booten automatisch zu starten genügt die folgende
Zeile in der Datei ``/etc/rc.conf``:

::

   moused_enable="YES"

Der Standardport ist ``/dev/psm0``, was mit der Variable
``moused_device`` jedoch geändert werden kann. Weitere Parameter können
``moused`` mit ``moused_flags`` übergeben werden. Für ein Notebook mit
Touchpad ergibt sich für gewöhnlich also folgende Konfiguration:

::

   moused_enable="YES"
   moused_flags="-3"

Wer zum Testen das automatische Starten für USB Mäuse deaktivieren will
tut das mit der folgenden Zeile:

::

   moused_nondefault_enable="NO"

Das Device für USB-Mäuse heißt ``/dev/ums0`` für die erste Maus.

SysMouse in Xorg
----------------

<note>Mit Xorg 7.4 werden InputDevice Sektionen ignoriert, was all die
schönen Einstellungen zu Nichte macht. Dem kann mit folgendem Eintrag
abgeholfen werden:

::

   Section "ServerFlags"
       Option      "AllowEmptyInput"   "off"
   EndSection

Ist bereits eine **ServerFlags**-Sektion vorhanden, sollte diese
natürlich ergänzt werden.</note>

Um ``/dev/sysmouse`` mit Xorg zu verwenden, was sich vor allem mit
Notebooks empfiehlt damit Mäuse im laufenden Betrieb gewechselt werden
können ohne X neu zu starten, muss das ``ServerLayout`` ergänzt werden
und eine ``InputDevice`` Sektion angelegt werden:

::

   Section "ServerLayout"
       ...
       InputDevice "SysMouse"
       ...
   EndSection


   Section "InputDevice"
       Identifier  "SysMouse"
       Driver      "mouse"
       Option      "Protocol" "SysMouse"
       Option      "Device" "/dev/sysmouse"
       Option      "ZAxisMapping" "4 5"
   EndSection

hochauflösende Mäuse einsetzen
------------------------------

Beim Einsatz hochauflösender Mäuse empfiehlt es sich ``moused`` und
``Xorg`` diesen Umstand bekannt zu machen und die Polling-Rate zu
erhöhen. Die Standard-Polling-Rate für PS/2 ist 80Hz, das maximal
mögliche aber 200Hz. Die Standard-Polling-Rate für USB ist 125Hz, 250,
500 und 1000 sind ebenfalls möglich. Hier ist jedoch zu beachten, dass
Raten jenseits 500Hz durchaus Schäden an der Hardware verursachen
können.

Selbst die Erhöhung der Polling-Rate einer ordinären PS/2 Maus oder
eines Touchpads von 80 auf 200Hz bringt schon einen subjektiv sehr
deutlich gefühlten Präzisionsgewinn. Das liegt daran, dass die mangelnde
Präzision bei der Erfassung schneller Bewegungen durch kleinere
Schrittweiten ausgeglichen wird. Effekte wie Zählerüberläufe bei sehr
schnellen Bewegungen einer Kugelmaus (der Zeiger läuft dann in die
entgegengesetzte Richtung) oder des Positionsverlusts einer optischen
Maus (der Zeiger bleibt bei hohen Geschwindigkeiten stehen) lassen sich
damit deutlich abschwächen. Bei der doppelten Polling-Rate muss die Maus
auch doppelt so schnell bewegt werden damit ein solcher Fehler auftritt.

Bei 125Hz ist selbst bei einer Maus, die mit 16Bit Positions-Daten statt
den üblichen 8Bit ein solcher Fehler mit einer schnellen Bewegung
problemlos herbeizuführen. Solche Mäuse haben oft eine höhere Auflösung,
was den Effekt natürlich verstärkt. Heutzutage schwanken die Abtastraten
gewöhnlicher Mäuse zwischen 400 und 800DPI. High-End-Mäuse tasten
Bewegungen mit bis zu 4000DPI ab. Das bedeutet über 15 Abtastschritte
pro Millimeter. Den daraus resultierenden Problemen kann nur mit einer
erhöhten Polling-Rate entgegengewirkt werden.

Das folgende Beispiel ist eine sinnvolle Konfiguration in der Datei
``/etc/rc.conf`` für den Betrieb eine Razer BoomSlang 2000 an einem
Notebook. Bei der BoomSlang handelt es sich quasi um die Mutter der
Präzisionsmäuse, die noch als Kugelmaus 2000DPI mit 16Bit erfasst, zu
einer Zeit als optische Mäuse 400 bis 500DPI boten. Es hat viele Jahre
gedauert, bis optische Mäuse die Präzision dieser Kugelmaus erreicht
haben. Das Ende der Fahnenstange liegt heute bei 4000DPI (Razer
Lachesis).

<note warning>Die im folgenden Beispiel verwendete Polling-Rate von
``1000``\ Hz kann bei minderwertiger Hardware zu Schäden führen.</note>

::

   moused_enable="YES"
   moused_flags="-F 200 -3"
   moused_ums0_flags="-F 1000 -r 2000"

Die zweite Zeile gibt mit dem Parameter ``-F`` die Polling-Rate
``200``\ Hz für das über PS/2 angeschlossene Touchpad an. Für die erste
USB-Maus gelten die Parameter der dritten Zeile. Hier sind die
Polling-Rate ``1000``\ Hz und mit ``-r`` die Auflösung ``2000``\ DPI
gewählt. Die Auflösung hat keinen Einfluss auf die Kommunikation mit der
Hardware sondern dient lediglich zur Berechnung der Mausgeschwindigkeit.

Damit diese Werte auch bei den Anwendungen ankommen sollte auch in der
Xorg.conf die Auflösung und die Abtastrate angepasst ist. Die
Polling-Rate von Xorg sollte das doppelte betragen um sicherzustellen,
dass jede gelesene Bewegung sofort bei Xorg ankommt. Moused summiert die
Schrittweiten zwar auf, so dass keine Information verloren geht, aber
die Antwortzeiten werden durch eine höhere Rate natürlich
verschlechtert.

::

   Section "InputDevice"
       Identifier  "SysMouse"
       Driver      "mouse"
       Option      "Protocol" "SysMouse"
       Option      "Device" "/dev/sysmouse"
       Option      "ZAxisMapping" "4 5"
       Option      "SampleRate" "2000"
       Option      "Resolution" "2000"
       Option      "AccelerationScheme"    "none"
   EndSection

Die Option ``SampleRate`` gibt hier die Polling-Rate an. ``Resolution``
die Auflösung der Maus in DPI. Auch hier gilt, dass letzteres nur zur
Berechnung der Zeigergeschwindigkeit herangezogen wird.

Die Option ``AccelerationScheme`` sollte mit Bedacht auf ``none``
gesetzt werden. Das ganze bedeutet dass Rohe Mausbewegungen direkt
umgesetzt werden. Normalerweise verwendet Xorg relativ komplexe
Mausbeschleunigungsalgorithmen um Benutzern niedrig aufgelöster Mäuse
ein angenehmes Arbeitsgefühl zu geben. Programme wie ioQuake3 können
jedoch nicht direkt auf die rohen Mausdaten zugreifen sondern enthalten
stattdessen die von Xorg manipulierten Daten, was in 3D-Shootern zu
Übersteuern führt. Das Abschalten der Beschleunigung ist hier die
einzige Möglichkeit das volle Maß an Präzision zu erreichen, verringert
aber je nach Hardware aber den X-Komfort.

Bei 3D-Shootern gilt die Regel, dass die Maus Hardwareseitig möglichst
die volle Auflösung im schnellst-möglichen Polling-Intervall liefern
sollte. Die Treibereinstellungen sollten so gewählt sein, dass keinerlei
Manipulation des Signals stattfindet und die Mausgeschwindigkeit (in der
Regel als Sensitivity bezeichnet) dem Shooter überlassen wird. Der Autor
verwendet eine Razer Lachesis bei 4000DPI mit einer Polling-Rate von
1000Hz und einer Sensitivity von 0.75 in ioQuake3 (die
Standardeinstellung ist 5).

Verweise
--------

-  Die **moused(8)** Manual Page.
-  http://www.x.org/wiki/Development/Documentation/PointerAcceleration/
   - X.Org Entwickler-Dokumentation zu Mausbeschleunigung

* :ref:`genindex`

Zuletzt geändert: |date|

