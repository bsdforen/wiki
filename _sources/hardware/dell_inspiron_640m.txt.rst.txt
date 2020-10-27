Dell Inspiron 640m
==================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Eine genaue Beschreibung der **Inspiron 6*0m-Reihe** findet auf
www.dell.de

Das Notebook wurde mit **6.2-STABLE-200705, NetBSD 3.0-Release** und
**Kubuntu GNU/Linux Dapper Drake 6.06** getestet. FreeBSD würde am
ausführlichsten getestet.

Wenn ich zu etwas nichts geschrieben haben sollte, dann funktioniert es
wahrscheinlich auf allen drei Systemen von alleine. Um sich mehrmaliges
neubauen des Kernels zu sparen, erstmal ganz durchlesen ;-) .

Prozessor
---------

Der Prozessor ist ein *Intel CoreDuo T2300 CPU* mit 1,66Ghz und zwei
Kernen.

**FreeBSD:** Damit beide Kerne erkannt und genutzt werden können, muss
ein SMP-Kernel von der FreeBSD-CD installiert oder nachträglich "Options
SMP" in den Kernel kompiliert werden. "Options apic" muss dazu auch drin
sein (ist aber in GENERIC schon drin). Mit "sysctl hw.ncpu" lässt sich
die Anzahl der erkannten Prozessoren-/Kerne feststellen.

**NetBSD:** Bei NetBSD reicht es auch einen SMP-Kernel zu installieren.

**GNU/Linux:** Unter Kubuntu kann man auch einfach per adept einen
aktuellen SMP-Kernel installieren.

Sound
-----

Es wurde eine Intel HD-Audio-Karte verbaut.

**FreeBSD:** Karte wird in -current unterstützt. Für ein aktuelles
-stable gibts hier binärtreiber
http://people.freebsd.org/~ariff/BINARY_MODULES/sndkld_releng6_i386_virtual_volume.tar.gz
(einfach nach /boot/kernel/ entpacken und snd_hda_load="YES" in die
/boot/loader.conf)

**NetBSD:** Die Karte wird vom Azalia-Treiber erkannt, hat aber bei mir
nicht funktioniert. Da, der FreeBSD-Treiber ein Port des Azalia-Treibers
ist gehe ich aber davon aus, dass mindestens die selbe Funktionalität
erreicht werden kann.

**GNU/Linux:** Die Karte wird von normalen *snd_ich*-Treiber vollständig
unterstützt.

Video
-----

Es befindet sich eine Intel-Grafikkarte mit i945-Chip in dem Notebook.
Ich habe das Notebook mit einem 14,1" TFT und einer Auflösung von
1440x900 gekauft, es ist auch mit einer Auflösung von 1280x800 zu haben.

**FreeBSD:** Mit aktuellem X-Server (7.2) wird die Karte in -stable und
-current unterstützt. Ob sie unter 6.2-release auch geht, weiß ich
nicht. sysutils/915resolution wird für die korrekte Auflösung benötigt.
(es muss auch beim Systemstart ausgeführt werden, das kann man z.B.
machen indem den Aufruf in das kdm-Script (`KDM <KDM>`__) screibt)

**NetBSD:** Hier läuft die Grafikkarte nur mit dem *vesa*-Treiber, d.h.
es können keine Breitbildaufösungen verwendet werden, 1024x786 wird
gestreckt dargestellt.

**GNU/Linux:** Hier wird der Chip automatisch erkannt, die Auflösungen
müssen aber auch mit 915resolution eingestellt werden. Man sollte auf
jeden Fall die aktuellsten Kernel-Images und xorg-Treiber verwenden, da
sonst Probleme mit der 3D-Beschleunigung bestehen.

Netzwerk
--------

Das Notebook hat:

#. Broadcom 10/100-Ethernet-Netzwerkkarte
#. Intel Pro/Wireless 3945 Gerät
#. Dell TrueMobile 350 Bluetooth 2.0 Modul

**FreeBSD, NetBSD:**

#. funktioniert "von alleine" (unter FreeBSD wird das Gerät als bfe\*
   erkannt).
#. für FreeBSD gibts hier http://bsd-geek.de/FreeBSD/wpi-freebsd.tgz
   semi-stabile Treiber mit denen es klappt. UPDATE: der Treiber
   funktioniert garnicht mehr, die aktuellen Treiber funktionieren auch
   nicht :(
#. nicht getestet.

**GNU/Linux:**

#. funktioniert "von alleine".
#. wird von den semi-proprietären Intel-Treiber unterstützt. Dazu muss
   nicht nur das ipw3945-Paket, sondern auch das \*-restricted-modules
   Paket für die verwendete Kernel installiert werden.
#. nicht getestet.

DVD-Brenner
-----------

**FreeBSD:** Gerät wird erkannt, brennen funktioniert.

**NetBSD:** Gerät wird erkannt, brennen nicht getestet.

**GNU/Linux:** Funktioniert einwandfrei. Es werden übrigens sowohl die
Festplatte, als auch der DVD-Brenner als SCSI-Geräte erkannt (unter den BSDs
ist dies nicht so).

5in1 CardReader
---------------

**FreeBSD:** Wird nicht erkannt.

**NetBSD:** Nicht getestet.

**GNU/Linux:** Wird erkannt und funktioniert. (getestet mit einer SD-Karte)

Media/Front-Keys
----------------

**FreeBSD, GNU/Linux:** Erstellt eine Datei irgendwo und schreibt
folgendes hinein:

::

   keycode 140 = XF86AudioMute
   keycode 174 = XF86AudioLowerVolume
   keycode 176 = XF86AudioRaiseVolume
   keycode 162 = XF86AudioPlay
   keycode 144 = XF86AudioPrev
   keycode 153 = XF86AudioNext
   keycode 164 = XF86AudioStop
   keycode 146 = XF86HomePage

Beim starten von X solltet ihr nun **xmodmap /pfad/zu/datei** auführen
lassen, z.B. durch ein Skript in **~/.kde/Autostart**, dann könnt ihr
die Tasten "ganz normal" (bei KDE-Programmen, durch "Settings->configure
Shortcuts") bestimmten Aktionen zuweisen.

**NetBSD:** Nicht getestet.

Sonstiges
---------

**FreeBSD:** Nicht das Display zuklappen! Es geht zwar aus, aber beim
Aufklappen nicht mehr an!

**Firewire:** Es gibt auch einen Firewire-Anschluss, habe ich aber unter
keinem System getestet.

* :ref:`genindex`

Zuletzt geändert: |date| 
