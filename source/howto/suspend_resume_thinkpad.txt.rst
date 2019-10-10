Suspend und Resume bei Thinkpads
================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

FreeBSD lässt sich auf Notebook sehr gut installieren und betreiben. Allerdings
gehört zur Notebook-Funktionalität auch "suspend" und "resume". Das bedeutet,
dass entweder durch drücken einer Tastenkombination oder durch Zuklappen des
Bildschirms das Notebook in eine Art Standby-Modus versetzt wird. Durch
Aufklappen des Monitors oder durch Drücken der Taste <Fn> nimmt das Notebook
seine Arbeit wieder auf.  Der Vorteil liegt in der schnellen Verfügbarkeit des
Gerät.

Allerdings ergeben sich unter FreeBSD oftmals Schwierigkeiten
Suspend/Resume zu nutzen, was oft zu Abstürzen oder einfachem Einfrieren
des Geräts führt. Es zeigen sich folgende Stolpersteine:

-  Konfiguration der Grafiktreiber im FreeBSD-Kernel
-  Steuerung der USB-Hardware durch das Kernel-Modul ``acpi_ibm``
-  Trennen und Wiederherstellen einer WLAN-Verbindung
-  Powermanagement der Notebook-Festplatten

Begriffe
--------

-  **ACPI** steht für **A**\ dvanced **C**\ onfiguration and **P**\ ower
   **I**\ nterface und ist ein offener Industriestandard für
   Energieverwaltung unter anderem bei Notebooks. Man darf es nicht mit
   APIC verwechseln, was Advanced Programmable Interrupt Controller
   bedeutet und rein garnichts mit ACPI zu tun hat.
-  **ACPI-Betriebsarten**; ACPI erlaubt sechs verschiedene Ruhezustände
   oder Modi, die in der folgenden Tabelle dargestellt sind:

===== =====================================================================================================================================================================================================================
Modus Beschreibung
===== =====================================================================================================================================================================================================================
S0    System voll funktionsfähig und sofort einsatzbereit.
S1    einfachster Schlafmodus, bei dem nur wenige Funktionen abgeschaltet sind und die CPU angehalten ist
S2    erweiterter Schlafmodus, bei dem weitere Komponenten insbesondere der Cache der CPU abgeschaltet sind
S3    Standby-Modus (STR - "Suspend to RAM", STM - "Suspend to memory"); die meiste Hardware des Notebooks ist abgeschaltet. Der Betriebszustand ist im flüchtigen Speicher gesichert, der vom Akku mit Strom versorgt wird
S4    Ruhezustand ("hibernation" oder "suspend to disk" - "STD"); der Betriebszustand wird auf Festplatte entweder in einer speziellen Partition oder im Swap-Bereich gesichert
S5    Soft-Off-Modus, bei dem das System ausgeschaltet ist, aber das Netzteil oder der Akku liefern Strom und das System kann durch Tastendruck oder durch WOL (Wake on LAN) wieder aktiviert werden
===== =====================================================================================================================================================================================================================

-  **Suspend** beschreibt den ACPI-Moduswechsel von Modus S0 nach Modi
   S1 bis S4. Unter anderem werden hierbei die Register der Grafikkarte
   gesichert oder USB abgeschaltet.
-  **Resume** ist der ACPI-Moduswechsel von S1 bis S4 nach S0. Dabei
   werden unter anderem die Register der Grafikkarte wieder
   zurückgeschrieben oder USB eingeschaltet. Dieser Vorgang ist sehr
   heikel.

Vorbereitung
------------

Aus dem Kernel werden alle Gerätetreiber entfernt, die für die
Kommunikation mit der USB-Hardware zuständig sind. Ebenso muss die
VESA-Unterstützung entfernt werden, da sie sonst für erhebliche
Schwierigkeiten sorgen wird. <note warning>Wer sein Notebook produktiv
einsetzt, sollte auf das ab FreeBSD 10 mögliche Kernel-Mode-Setting
(KMS) verzichten. Die dafür notwendigen Konsolentreiber "newcons"
sollten somit nicht in den Kernel aufgenommen werden</note>

Kernelkonfiguration
~~~~~~~~~~~~~~~~~~~

Bevor ein spezieller Kernel erstellt wird, sollte man sich unbedingt
damit vertraut machen. Dazu leistet das Handbuch wie immer gute Dienste.
Im `Kapitel
neun <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/kernelconfig.html>`__
ist alles notwendige beschrieben.

Patch für acpi_ibm
~~~~~~~~~~~~~~~~~~

Bei manchen Thinkpad-Modellen kommt es vor, dass bei einem Resume die
USB-Ports nicht mehr aktiv werden. Teilweise liegt dies an
BIOS-Einstellungen. Dankenswerter Weise wurde für diesen Missstand ein
Patch entwickelt: `[patch] USB after second suspend/resume on
ThinkPads. <http://lists.freebsd.org/pipermail/freebsd-current/2014-June/050721.html>`__
Allerdings ist dieser Patch für FreeBSD-CURRENT gedacht, aber er
funktioniert unter FreeBSD 10 ohne Probleme.

Für die weitere Verwendung wird der Kode von der Webseite in eine Datei
kopiert und beispielsweise in ``'/tmp``' abgelegt. Im Verzeichnis
``'/usr/src/sys/dev/acpi_support``' wird der Patchkode auf die Datei
``'acpi_ibm.c``' angewandt: <xterm> # cd /usr/src/sys/dev/acpi_support #
patch acpi_ibm.c <path to patchfile> </xterm> Anschließend wird der
Kernel neu kompiliert und installiert: <xterm> # cd /usr/src # make
buildkernel KERNCONF=<Kernelkonfiguration> # make installkernel
KERNCONF=<Kernelkonfiguration> </xterm>

Konfigurationsdateien
---------------------

Die weitere Konfiguration erfolgt in ``'/boot/loader.conf``' und
``'/etc/sysctl.conf``'.

Module laden /boot/loader.conf
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In dieser Datei werden die Module eingetragen, die weitere Gerätetreiber
dem Kernel zur Verfügung stellen. In diesem Fall sind es das ACPI-Modul
für IBM-Rechner, womit sich die speziellen ACPI-Funktionen ansprechen
lassen. Desweiteren wird das für Intel-Grafikkarten notwendige
Kernelmodul geladen. Schließlich werden die für USB wichtigen Module
gezogen. Wer Firewire nutzen möchte, trägt dieses Kernelmodul ebenfalls
in diese Datei ein.

::

   # ACPI-Module fuer Thinkpads
   acpi_ibm_load="YES"

   # Grafik Intel GM45 (TP T400)
   i915_load="YES"

   # USB
   usb_load="YES"
   ehci_load="YES"
   uhci_load="YES"
   ums_load="YES"
   u3g_load="YES"
   umass_load="YES"
   ukbd_load="YES"

Konfiguration /etc/sysctl.conf
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Damit Suspend/Resume die richtigen ACPI-Modi nutzt, wird diese
Konfigurationsdatei entsprechend ergänzt. Das Zuklappen des Notebooks
beziehungsweise das Drücken der Sleep-Taste sollen den Rechner in den
Standby-Modus (S3-Mode) schicken. Der Einausschalter schaltet das Gerät
hingegen vollständig ab.

::

   hw.acpi.reset_video=0
   hw.acpi.lid_switch_state=S3
   hw.acpi.sleep_button_state=S3
   hw.acpi.power_button_state=S5
   hw.acpi.sleep_delay=3
   hw.acpi.verbose=1
   hw.syscons.sc_no_suspend_vtswitch=0
   dev.acpi_ibm.0.events=1

Systemskripte
-------------

Die folgenden drei Systemskripte werden einerseits beim Bootvorgang
aufgerufen und andererseits beim Wechsel in den Suspend-Modus
beziehungsweise in den Resume-Modus aufgerufen. Alle drei Skripte müssen
ergänzt werden, um eine einwandfreie Funktionalität zu erreichen.

Skript /etc/rc.local
~~~~~~~~~~~~~~~~~~~~

Das beim Booten aufgerufene Skript wird um ein Kommando ergänzt. Damit
erreicht man, dass das Powermanagement der Festplatte(n) abgeschaltet
wird.

::

   #!/bin/sh

   # Powermanagement deaktivieren
   /sbin/camcontrol cmd ada0 -a "EF 85 00 00 00 00 00 00 00 00 00 00"

Skript /etc/rc.suspend
~~~~~~~~~~~~~~~~~~~~~~

Das Skript, das beim Wechsel in den Suspend-Modus aufgerufen wird,
ergänzt man um einige Zeilen. Zunächst werden WLAN und der Maus-Daemon
terminiert. Anschließend werden alle USB-Module aus dem Speicher
entfernt, damit beim späteren Resume USB sauber initialiert wird.

::

   # WLAN-Access-Point abmelden
   /usr/sbin/wpa_cli terminate

   # Maus-Daemon anhalten
   /etc/rc.d/moused stop

   # USB-Module entfernen
   kldunload ehci
   kldunload uhci
   kldunload umass
   kldunload u3g
   kldunload usb

Skript /etc/rc.resume
~~~~~~~~~~~~~~~~~~~~~

Dieses Skript wird beim Auswachen des Notebooks aufgerufen. Dabei werden
zuächst die USB-Module geladen und damit auch USB initialisiert. Enbenso
startet der Maus-Daemon und das WLAN nimmt die Arbeit wieder auf.
Wichtig ist, das beim Resume das Powermanagement der Festplatte(n) zu
deaktivieren.

::

   # USB-Module laden
   kldload ehci
   kldload uhci
   kldload umass
   kldload u3g
   kldload usb

   # Maus-Daemon neu starten
   /etc/rc.d/moused restart

   # Mit WLAN-Access-Point verbinden
   /usr/sbin/wpa_cli reassociate

   # Powermanagement deaktivieren
   /usr/sbin/camcontrol cmd ada0 -a "EF 85 00 00 00 00 00 00 00 00 00 00"

Ergänzungen
-----------

-  **ATi-Grafikchip - Suspend und Resume funktioniert!** In vielen
   Thinkpads ist neben einem Intel-Grafikchip auch eine ATi Mobility
   Radeon HD xxxx verbaut. Durch das Kernelmodul ``'radeon.ko``' ist es
   möglich, dass XOrg Zugriff auf die Hardware erhält. Dazu einfach das
   Modul in ``'/boot/loader.conf``' eintragen und das Module für den
   Intel-Grafikchip auskommentieren - man weiß nie! Außerdem sind aus
   der Konfiguration für XOrg in ``'/etc/X11/xorg.conf``' alle
   Intel-spezifischen Einträge auszukommentieren.

Fazit
-----

Es ist nicht trivial, Suspend und Resume unter FreeBSD zu konfigurieren.
Auf manchen Gerät muss nichts gemacht werden, auf anderen dagegen geht
nichts. Die Thinkpads liegen irgendwo dazwischen und wer glaubt, bei
Linux sei alles besser, der sollte sich mal nicht täuschen!

"Es wäre so einfach, wenn sich die Hersteller an die von ihnen selbst
gesetzten Standards halten würden"

Man-Pages
---------

`kldload(8) <http://www.freebsd.org/cgi/man.cgi?query=kldload&apropos=0&sektion=0&manpath=FreeBSD+10.0-RELEASE&arch=default&format=html>`__
`kldunload(8) <http://www.freebsd.org/cgi/man.cgi?query=kldunload&apropos=0&sektion=0&manpath=FreeBSD+10.0-RELEASE&arch=default&format=html>`__
`wpa_cli(8) <http://www.freebsd.org/cgi/man.cgi?query=wpa_cli&apropos=0&sektion=0&manpath=FreeBSD+10.0-RELEASE&arch=default&format=html>`__
`camcontrol(8) <http://www.freebsd.org/cgi/man.cgi?query=camcontrol&apropos=0&sektion=0&manpath=FreeBSD+10.0-RELEASE&arch=default&format=html>`__
`moused(8) <http://www.freebsd.org/cgi/man.cgi?query=moused&apropos=0&sektion=0&manpath=FreeBSD+10.0-RELEASE&arch=default&format=html>`__
`sysctl.conf(5) <http://www.freebsd.org/cgi/man.cgi?query=sysctl.conf&apropos=0&sektion=0&manpath=FreeBSD+10.0-RELEASE&arch=default&format=html>`__
`loader.conf(5) <http://www.freebsd.org/cgi/man.cgi?query=loader.conf&apropos=0&sektion=0&manpath=FreeBSD+10.0-RELEASE&arch=default&format=html>`__
`rc.local(8) <http://www.freebsd.org/cgi/man.cgi?query=rc.local&apropos=0&sektion=0&manpath=FreeBSD+10.0-RELEASE&arch=default&format=html>`__

Danksagung
----------

.. note::

  Mein Dank gilt Yamagi, der mit Rat und Tat zur Seite stand.


* :ref:`genindex`

Zuletzt geändert: |date|

