devfs
=====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Alle unixoiden Systeme haben ein `/dev <file:///dev>`__ Verzeichnis, worin
"Dateien" liegen, die Geräte im System repräsentieren. Früher wurden diese
Geräte auf den meisten Betriebsystemen mit MAKEDEV manuell erstellt, bei
GNU/Linux und FreeBSD gibt es dafür spezielle Dateisysteme, die einem die
Arbeit abnehmen.

Unter FreeBSD heißt dieses Dateisystem **devfs**, welches inzwischen nicht
einmal mehr in `/etc/fstab <file:///etc/fstab>`__ eingetragen sein muss,
sondern beim Start automatisch eingehangen wird. Unter GNU/Linux wurde
**devfs** durch **udev** ersetzt, was angeblich viele neue Vorteile hat (etwa
dass Geräte frei benennbar sind; z.B. kann man jetzt einer Netzwerkkarte das
Kürzel einer Festplatte geben :-x).  FreeBSD-\ **devfs** unterscheidet sich
auch von **udev**, indem es wirklich *nur die Geräte* erzeugt, die auch
wirklich da sind.  `/dev <file:///dev>`__ ist somit in FreeBSD viel
aufgeräumter als unter GNU/Linux!

/etc/devfs.conf
---------------

| In dieser `Datei <file:///etc/devfs.conf>`__ findet die statische
  Konfiguration von **devfs** statt.

/etc/devfs.rules
----------------

In dieser `Datei <file:///etc/devfs.rules>`__ findet die "dynamische"
Konfiguration von **devfs** statt: Man kann hier Regelsets anlegen, die
beim Start und auf neuangelegte Geräte angewendet werden. Ein "gutes"
Regelset für einen Desktop-PC könnte so aussehen:

::

   [localrules=10]
   add path 'cd*' mode 0660 group operator         # CD devices
   add path 'xpt*' mode 0660 group operator        # required for CD access
   add path 'pass*' mode 0660 group operator       #  ----"----
   add path 'da*s*' mode 0660 group operator       # SCSI-Disks, e.g. USB-Sticks
   add path 'md*' mode 0660 group operator         # virtual FSs, e.g. for .img, .iso
   add path 'video*' mode 0666 group operator  # webcam etc

Die ersten drei Regeln sorgen dafür, dass alle Mitglieder der
Operatorgruppe (da sollte man also drin sein ;-)) und z.B. der `HAL
(Hardware Abstraction
Layer) <http://de.wikipedia.org/wiki/Hardware-Abstraktions-Schicht>`__-Daemon
die CD-Laufwerke benutzen können (`mounten <mounten_als_benutzer>`__,
`brennen <brennen_mit_k3b>`__ etc).

Die vierte Zeile dient dem Setzen der Rechte auf SCSI-Festplatten (z.B.
werden USB-Sticks vom SCSI-Subsystem des `Kernels </kategorie/kernel>`__
verwaltet), damit diese vom Nutzer (und `HAL </kompendium/HAL>`__) ein-
und ausgehangen werden können.

Die letzte Zeile setzt Rechte auf sog. Memory Disks. Das sind z.B.
Geräte, die einer ISO- oder IMG-Datei zugeordnet werden, wenn man das
Dateisystem aus diesen einhängen möchte. Diese Rechtesetzung wird z.B.
benötigt, wenn man, wie im `Wine <Wine>`__-Artikel beschrieben, das
Einhängen von Spiele-CDs in Form von ISOs erleichtern möchte.

Damit diese Regeln beim Starten geladen werden, ist noch folgende Zeile
in `/etc/rc.conf <file:///etc/rc.conf>`__ einzutragen:

::

   devfs_system_ruleset="localrules"

Änderungen wirksam machen
-------------------------

Damit die Änderungen ohne Neustart wirksam werden muss noch **devfs**
neu gestartet werden:

::

  # /etc/rc.d/devfs restart</xterm>

Ruleset anzeigen
----------------

Mit *devfs rule show* werden die aktuellen Rules angezeigt: 

::

  # devfs rule show 
  100 path cd\* group operator mode 660 
  200 path xpt0 group operator mode 660 
  300 path pass\* group operator mode 660 
  400 path da*s\* group operator mode 660 
  500 path md\* group operator mode 0660

Verweise
--------

-  `k3b einrichten </howto/k3b einrichten>`__
-  Manual-Seiten:
   `devfs(8) <http://www.freebsd.org/cgi/man.cgi?query=devfs>`__,
   `devfs.conf(5) <http://www.freebsd.org/cgi/man.cgi?query=devfs.conf>`__,
   `devfs.rules(5) <http://www.freebsd.org/cgi/man.cgi?query=devfs.rules>`__

 * :ref:`genindex`

Zuletzt geändert: |date|

