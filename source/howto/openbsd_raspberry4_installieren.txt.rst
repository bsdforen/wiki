OpenBSD 6.7 auf dem RaspberryPI 4 installieren
==============================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

Bei der Installation von OpenBSD auf dem RaspberryPI 4 sind einige dinge zu beachten.

OpenBSD auf Raspberry 4 (8GB)
---------------------------------------------

Benötigt: 2 µSD Karten, eine für die Firmware, eine für das OpenBSD Image.

Firmware beschaffen
-------------------

::

   wget https://github.com/pftf/RPi4/releases/download/v1.16/RPi4_UEFI_Firmware_v1.16.zip


Firmware auf einem Rechner (in meinem Fall einer mit OpenBSD 6.7) entpacken.
----------------------------------------------------------------------------

::

   unzip RPi4_UEFI_Firmware*.zip


SD Karte No.1 vorbereiten.
--------------------------

Eine kleine SD-Karte reicht, sind nur ein paar MB, die drauf kommen.

Auf der SD-Karte nur eine Partition FAT32 erstellen und das Boot-flag setzen.

Die entpackte Firmware (6 Dateien und der Ordner overlays) auf die SD-Karte kopieren.

Die SD-Karte ist damit bereit, die Firmware auf dem Raspi zu booten. Sie wird jetzt in den µSD-Slot des Raspi gesteckt und dort verbleiben.


SD Karte No.2 vorbereiten.
--------------------------
Diese zweite SD-Karte wird zunächst das Bootimage von OpenBSD bekommen, ich hab das mit miniroot67.fs gemacht.
Gleichzeitig wird die Karte aber das installierte OpenBSD bekommen, also nicht zu klein wählen. Auf 64 GB bekommt man aber locker alles drauf und kann später noch reichlich dazu installieren.
Jetzt also das Bootimage und die entsprechende SHA Datei herunter geladen:

::

   wget https://cdn.openbsd.org/pub/OpenBSD/6.7/arm64/miniroot67.fs
   wget https://cdn.openbsd.org/pub/OpenBSD/6.7/arm64/SHA256.sig

Nach dem sha256 Check dann das heruntergeladene Image auf die SD-Karte packen:

::

   dd if=miniroot67.fs of=/dev/rsd1c bs=1m

So siehts unter OpenBSD aus, bei anderen Unixen kann die Laufwerksbezeichnung natürlich eine andere sein.
Damit ist auch die zweite Karte vorbereitet. Sie kommt nun mittels eines USB-SD-Adapters an einen USB Port des Raspi.

Die Installation
----------------
Ich gehe jetzt davon aus, dass ihr einen Monitor am Raspi habt. Ist das nicht der Fall, muss über einen usb-uart-Adappter eine serielle Verbindung von einem anderen Rechner auf den Raspi aufgebaut werden.
Dieser Adapter wird in einen USB Port eines PC eingesteckt und mit drei Leitungen an die GPIO-Leiste des Raspi angeschlossen: TxD an RxD, RxD an TxD, Gnd an Gnd.
Als Programm ist eine simple Terminalsoftware wie cu geeignet:

::

   cu -l /dev/ttyUSB0 -s 115200

Die Bezeichnung der Schnittstelle ist abhängig vom Betriebssystem eures Rechners, im Beispiel war's ein Linux-Rechner mit ttyUSB0. Ich habs auf einem Rechner mit NetBSD gemacht und da heisst die Schnittstelle ttyU0.

Aber das nur nebenbei, ich gehe ja von einem Raspi mit Monitor aus, da ist das Schnittstellengedöns nicht nötig.
Die Firmaware-SD steckt jetzt also im SD-Slot des Raspi, die OpenBSD-SD hängt an einem USB-Port des Raspi.

Schaltet nun des Raspi ein und er wird von der Firmware SD-Karte booten. Es erscheint zunächst das grosse farbige Quadrat auf dem Bildschirm und danach ein kleines Menü mit

::

   Esc (setup), F1 (shell), ENTER (boot)

Ihr drückt ESC und kommt damit ins UEFI-Menü.
Mit den Pfeiltasten navigiert ihr zu "Boot Manager", entert euch herein und wählt als Bootoption

UEFI Generic Mass Storage Device

Abhängig davon, welchen USB-SD-Adapter ihr benutzt, kann die Bezeichnung auch etwas abweichen.

Wählt also mit ENTER das USB Mass Storage Device aus. Danach startet der Rechner durch und bootet von der Installations-SD-Karte.

Auf dem Bildschirm erscheint nun der Boot-prompt des Installationsimages. Jetzt heisst es, schnell zu sein, denn am BOOT muss eingetippt werden: set tty fb0

Bedenkt, dass noch keine deutsche Tastatur geladen ist, also beim "y" aufpassen. Und seid ihr zu langsam, bootet der Rechner auf die serielle Schnittstelle und der Monitor bleibt schwarz.
Nach einem ENTER erscheint der BOOT prompt erneut und jetzt könnt ihr mit einem weiteren ENTER den Bootvorgang starten.

Jetzt läuft die komplette Bootsequenz eines OpenBSD-Rechners auf dem Bildschirm durch.
Danach ist die bekannte Auswahl des Installationsprogrammes zu sehen:

::

   Welcome to the OpenBSD/arm64 6.7 installation program.
   (I)nstall, (U)pgrade, (A)utoinstall or (S)hell?

Die jetzt ablaufende Installation dürfte den meisten hier bekannt sein, da gibt es nicht viel zu erklären.
Passt nur bei der Auswahl des Installationsmediums auf, dass die SD-Karte am USB-Adapter gewählt wird und nicht die Firmware-SD-Karte.
Bei meinen Installationen war es immer sd0.
Während der Installation wird übrigens nicht das Tastaturlayout abgefragt, sodass bis zum Schluss auf x/y geachtet werden muss.

Achtung! Solltet ihr nicht das vorgeschlagene Partitionslayout übernehmen wollen, sondern ein Custum-Layout erstellen, dann darf die vorhandene winzige MS-DOS-Partition nicht gelöscht werden. Die wurde beim Schreiben des Image auf dem Datenträger erstellt und wird zum Bootvorgang benötigt. Sind auch nur ein paar KB.

Eine Liste der OpenBSD-Mirrors wird nicht angeboten, ich hab dann cdn.openbsd.org angegeben.

Nach ca. 15 Minuten ist die Installation abgeschlossen und das Installationsprogramm schlägt einen Neustart vor. Machen wir.

Der normale Bootvorgang
-----------------------
Beide SD-Karten verbleiben ab jetzt dauerhaft im Raspi: Firmware-Karte im SD-Slot, OpenBSD-Karte am USB-Port.
Der Raspi fährt also wieder hoch, es erscheint das bunte Quadrat, dann das Menü der Firmware. Wieder mit ESC das Setup wählen,
dann das USB Mass Storage Device als Bootmedium auswählen.
Anschliessend erscheint der BOOT-Prompt von OpenBSD und hier muss ebebnfalls bei jedem Bootvorgang "set tty fb0" eingeben werden. Sonst geht die Ausgabe wieder komplett auf die serielle Schnittstelle und der Bildschirm bleibt dunkel.
Abgesehen von diesen beiden Eingaben habt ihr jetzt ein schönes, stabiles und schnelles Betriebssystem auf dem Raspi.

Ob die beiden manuellen Eingaben automatisiert werden können, versuche ich herauszufinden. Und dann würde ich noch gern NetBSD auf dem Raspi haben.

Beim Schreiben dieser Zeilen habe ich parallel die komplette Installation auf einer 128GB SD-Karte durchgezogen. Hat alles prima funktioniert.


Diese Anleitung wurde vom Benutzer `berni51 <https://www.bsdforen.de/members/berni51.74111/>`_ erstellt und im Forum `gepostet <https://www.bsdforen.de/threads/netbsd-9-auf-arm64-raspberry-4-geht-das.35774/#post-321032>`_ und anschließend ins Wiki übertragen.


* :ref:`genindex`

Zuletzt geändert: |date|

