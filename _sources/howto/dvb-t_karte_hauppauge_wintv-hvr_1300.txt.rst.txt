DVB-T Karte Hauppauge WinTV-HVR 1300
====================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Die Karte basiert auf dem Conexant-Chip. Dieser Chip wird vom Treiber
CX88 unterstützt.

Erstellt und getestet wurde die Installation und folgende Anleitung in
der Konfiguration:

| Hauppauge WinTV-HVR 1300
| Chip: Conexant
| Tuner Modul: Philips FMD1216MEX
| Decoder: CX882

| FreeBSD 7.1-BETA2

Treiber und libtuner kompilieren
--------------------------------

Als root die Treiber Module aus den Ports kompilieren via

::

   cd /usr/ports/multimedia/cx88

::

   make install clean

Make erzeugt folgende Kernelmodule in /boot/modules und zeigt das auch
am Ende der korrekten Übersetzung an:

-  cx88.ko
-  cx88video.ko
-  iicdev.ko
-  iicbus.ko
-  cx88i2c.ko
-  cx88audio.ko
-  cx88mpeg.ko
-  cx88ir.ko

| CX88 benötigt libtuner. Also auf gleiche Standardweise
  /usr/ports/multimedia/libtuner kompilieren.

Module installieren
-------------------

**entweder:**

alle Module nach jedem Systemstart nacheinander manuell laden,

::

   kldload cx88

dann mit

::

   kldstat

| prüfen: es sollte cx88.ko in der Liste zeigen.

::

   kldload cx88video

| Lädt neben cx88video.ko auch
| iicdev.ko
| iicbus.ko
| cx88i2c.ko
| dann mit

::

   kldstat

| prüfen: es sollte die Module in der Liste zeigen.

::

   kldload cx88audio

::

   kldload cx88mpeg

::

   kldload cx88ir

und dann mit

::

   kldstat

| prüfen. Alle Module aus der Liste sollten gezeigt werden.
| **oder:**

die /boot/loader.conf für ein automatische Laden bei Systemstart
ergänzen

::

   cx88_load="YES"
   cx88video_load="YES"
   cx88audio_load="YES"
   cx88mpeg_load="YES"
   cx88ir_load="YES"

| Die Module iicdev.ko, iicbus.ko und cx88i2c.ko müssen nicht
  eingetragen werden. Sie werden automatisch von cx88video geladen.

Erkennung der Devices prüfen
----------------------------

| 

::

   pciconf -lv

sollte die drei Devices anzeigen

::

   cx88video0@pci0:4:3:0:  class=0x040000 card=0x96010070 chip=0x880014f1 rev=0x05hdr=0x00
       vendor     = 'Conexant Systems, Inc.'
       device     = '23880 Conexant 23880 Video Capture (NTSC)'
       class      = multimedia
       subclass   = video
   cx88audio0@pci0:4:3:1:  class=0x048000 card=0x96010070 chip=0x881114f1 rev=0x05hdr=0x00
       vendor     = 'Conexant Systems, Inc.'
       device     = 'CX2388x TV Capture Chip'
       class      = multimedia
   cx88mpeg0@pci0:4:3:2:   class=0x048000 card=0x96010070 chip=0x880214f1 rev=0x05hdr=0x00
       vendor     = 'Conexant Systems, Inc.'
       device     = 'CX2388x TV Capture Chip'
       class      = multimedia

| 
| und in der

/var/log/messages

| sollte am Ende der Einträge etwa Folgendes stehen

::

   Nov 25 14:09:15 freebsd kernel: cx88video0: <Conexant CX2388x Analog Video> mem 0xd9000000-0xd9ffffff irq 19 at device 3.0 on pci4
   Nov 25 14:09:15 freebsd kernel: iicbus0: <Philips I2C bus> on cx88video0
   Nov 25 14:09:15 freebsd kernel: iicbus0: <unknown card> at addr 0
   Nov 25 14:09:15 freebsd kernel: iicbus0: <unknown card> at addr 0
   Nov 25 14:09:15 freebsd kernel: cx88video0: [FILTER]
   Nov 25 14:09:15 freebsd kernel: cx88video0: [FILTER+ITHREAD]
   Nov 25 14:09:15 freebsd kernel: cx88audio0: <Conexant CX2388x Analog Audio> mem 0xda000000-0xdaffffff irq 19 at device 3.1 on pci4
   Nov 25 14:09:15 freebsd kernel: iicbus1: <Philips I2C bus> on cx88audio0
   Nov 25 14:09:15 freebsd kernel: iicbus1: <unknown card> at addr 0
   Nov 25 14:09:15 freebsd kernel: iicbus1: <unknown card> at addr 0
   Nov 25 14:09:15 freebsd kernel: cx88audio0: [FILTER]
   Nov 25 14:09:15 freebsd kernel: pcm1: <CX2388x PCM interface> on cx88audio0
   Nov 25 14:09:15 freebsd kernel: cx88audio0: [FILTER+ITHREAD]
   Nov 25 14:09:15 freebsd kernel: cx88mpeg0: <Conexant CX2388x MPEG Transport Stream> mem 0xdb000000-0xdbffffff irq 19 at device 3.2 on pci4
   Nov 25 14:09:15 freebsd kernel: iicbus2: <Philips I2C bus> on cx88mpeg0
   Nov 25 14:09:15 freebsd kernel: iicbus2: <unknown card> at addr 0
   Nov 25 14:09:15 freebsd kernel: iicbus2: <unknown card> at addr 0
   Nov 25 14:09:15 freebsd kernel: cx88mpeg0: [FILTER]
   Nov 25 14:09:15 freebsd kernel: cx88mpeg0: [FILTER+ITHREAD]

Scannen und Aufzeichnen mit cx88
--------------------------------

| 
| CX88 bietet neben dem Treiber auch ein kleines Programm zum Scannen
  und Aufzeichnen, das auch interaktive Funktionalitäten ermöglicht.

mit

::

   cx88 -help

| kann man sich die Optionen anzeigen lassen. Ein Manual oder
  Dokumentation ist zum Zeitpunkt der Erstellung dieser kleinen
  Anleitung nicht vorhanden und wird derzeit gerade vom Entwickler
  erarbeitet.
| Scannen im interaktiven Mode mit dem Befehl:

::

   cx88 -d /dev/cx88mpeg0 -c DVBT_EU_UHF:46 -f ~/capture.m2t -x /usr/local/share/examples/cx88/cx88.xml.sample

Achtung: "DVBT_EU_UHF" ist hier wichtig, weil sonst im default im
US-Mode "ATSC" gescannt wird.

| Dahinter, durch ":" getrennt, die Kanalnummer. Kanal 46 ist im
  getesteten Falle eine Kanalnummer aus dem Listing des lokalen DVB-T
  Senders, hier der schöne Bayerische Rundfunk. ;-)
| Nach Befehlseingabe passiert scheinbar erstmal nichts, aber: mit
  Eingabe von "s" kann man interaktiv den Scan starten, kann den
  Signal-Schwellwert setzen, und dann den Scan beobachten.
| Das sieht dann etwa wie folgt aus:

::

   freebsd# cx88 -d /dev/cx88mpeg0 -c DVBT_EU_UHF:46 -f ~/capture.m2t -x /usr/local/share/examples/cx88/cx88.xml.sample
   s
   Enter signal strength threshold:
   30
   Scanning profile DVBT_EU_UHF . . . . 25 (50.0008%) . . . . . . . . . . 36 (50.0008%) . . . . . . . . . 46 (100%) . . . . 
   . . . . . . . . . . . . . . . . . . . Finished scan.
   q

| "q" beendet den interaktiven Modus.
| Aufzeichnen im nicht-interaktiven Mode mit dem Befehl:

::

   cx88 -d /dev/cx88mpeg0 -c DVBT_EU_UHF:46 -f ~/capture.m2t -x /usr/local/share/examples/cx88/cx88.xml.sample -n 2

| Mit der Option -n wird hier 2 Minuten aufgezeichnet und in einer Datei
  capture.m2t abgelegt. Kann man sich dann mit zB VLC anschauen.

* :ref:`genindex`

Zuletzt geändert: |date|

