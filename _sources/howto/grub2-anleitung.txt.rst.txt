GRUB2-Anleitung
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Es gibt etliche Anleitungen für die Benutzung von GRUB2 unter FreeBSD, die aber
praktisch alle nicht über die Angabe einer Handvoll Befehle für ein
Standard-Setup hinausreichen und/oder noch Altlasten von GRUB1 mitbringen.

In dieser kleinen Anleitung will ich GRUB2 ein wenig genauer unter die
Lupe nehmen; GRUB1 werde ich jedoch mit keinem weiteren Wort erwähnen.
Auch soll dies nicht eine komplette Installationsanleitung sein mit
Partitionierung etc. pp.; dazu könnte man vielleicht den Artikel von
User "nukama" unter dem Titel „\ `MultiBoot mit
Grub2 </wiki/user/nukama>`__\ “ konsultieren.

Das FreeBSD-Paket beinhaltet eine Info-Datei (``info grub``), die
`Dokumentation <http://www.gnu.org/software/grub/manual/grub.html>`__
auf GRUBs Homepage ist ein wenig aktueller; die Dokumentationen von
`Ubuntu <https://help.ubuntu.com/community/Grub2>`__ oder `Arch
Linux <https://wiki.archlinux.org/index.php/GRUB2>`__ sind teilweise
sogar noch besser (Ubuntu führt auch \*BSD auf), aber natürlich auf die
betreffende Linux-Distribution fokussiert. 

.. warning::

  Die GRUB2-Version in den Ports (1.98_1) ist veraltet und kann den Kernel der
  neuesten Release (9.1) nicht direkt booten. Sobald ein neuerer Port zur
  Verfügung steht, werde ich diesen Hinweis entfernen. Vorerst ist nur
  Chainloading möglich.

Erste Schritte
--------------

| Zuerst will natürlich der Port installiert sein, und zwar
  `sysutils/grub2 <https://www.google.com/search?q=sysutils/grub2&btnI=lucky>`__.
| Wenn man diese Anleitung für einen Linux-basierten GRUB verwenden
  will, kann man eine Überraschung erleben. Manche Distributionen
  verwenden gepatchte GRUB-Pakete, denen manchmal die Unterstützung für
  \*BSD und zugehörige Dateisysteme wie UFS einfach fehlt. Nur mit dem
  Original-GRUB aus dem FreeBSD-Port (oder unter Linux von Hand
  installiert) ist man auf der sicheren Seite.

| GRUB2 gehorcht einer Konfigurationsdatei ``/boot/grub/grub.cfg`` (so
  der Standardpfad), aber um deren Syntax dem unbedarften Nutzer zu
  ersparen und sie vollautomatisch zu erzeugen, werden Tools
  bereitgestellt, die sich auf eine sehr einfache
  Meta-Konfigurationsdatei ``/usr/local/etc/default/grub`` stützen. In
  den gängigen Linux-bezogenen Anleitungen lautet ihr Pfad übrigens
  ``/etc/default/grub``, das geht in FreeBSD nicht.
| Für den Anfang, um überhaupt eine Vorstellung einer ``grub.cfg`` zu
  bekommen, empfehle ich die Verwendung dieser Automatik. Weil der
  FreeBSD-Port aber doch etwas mit heißer Nadel gestrickt ist, sollte
  man die so erzeugte ``grub.cfg`` dann auf jeden Fall manuell
  nachbearbeiten.

Meine ``/usr/local/etc/default/grub`` sieht so 

::

  GRUB_DISTRIBUTOR="FreeBSD Release 9.0"
  GRUB_DEFAULT=1
  GRUB_SAVEDEFAULT=false
  GRUB_TERMINAL=console
  #GRUB_GFXMODE=1024x768
  #GRUB_FONT_PATH=
  GRUB_TIMEOUT=9
  #GRUB_HIDDEN_TIMEOUT=5
  GRUB_DISABLE_OS_PROBER=false
  GRUB_DISABLE_RECOVERY=true
  GRUB_INIT_TUNE="2000 128 1 192 1 288 1 432 1 648 1 972 1 1458 1 2187 1"``

Es werden nur Variablen gesetzt, die den Skripten im Verzeichnis
``/usr/local/etc/grub.d`` dann die Informationen liefern, um geeignete
Befehle in die ``grub.cfg`` zu schreiben. Das Kapitel *"Configuration"*
in der Dokumentation listet die möglichen Variablen auf, manche
Linux-Distributionen fügen denen noch selbstgestrickte hinzu.

Der Aufruf

::

   grub-mkconfig -o /boot/grub/grub.cfg

erstellt die eigentliche Konfigurationsdatei. Bevor wir uns an deren
Nachbearbeitung machen, nun ein paar allgemeine Erklärungen.

Bootkonzepte
------------

Der Bootloader besteht, ähnlich wie der von FreeBSD, aus mehreren
Stufen. Die erste paßt in einen 512-Byte-Block (ist sogar noch etwas
kleiner, da sie ggf. Platz für eine Partitionstabelle lassen muß), und
ihr Sinn und Zweck besteht einzig und allein darin, zu wissen, wo die
zweite Stufe liegt und wie sie geladen werden kann. Jene ist typisch
20-30 kB groß, hat als Datei den Namen ``core.img`` und kann nicht aus
einem richtigen Dateisystem herausgelesen werden, weil der Code der
ersten Stufe bei weitem nicht die Intelligenz für ein Dateisystem
mitbringt. [1]_

**Installationsmöglichkeiten**

Wo findet sich nun Platz für GRUBs ``core.img``? Ein FreeBSD-User hat im
wesentlichen drei Möglichkeiten:

#. in einer eigenen, dedizierten Partition (bzw. Slice) ohne Dateisystem
   oder sonstigen Inhalt. Dies wird normalerweise bei
   GPT-partitionierten Festplatten angewandt und ist wohl die sauberste,
   eleganteste Lösung. Bei Festplatten nach dem MBR-Schema könnte man
   theoretisch einen eigenen Slice oder sogar eine Unterpartition
   innerhalb eines BSD-Labels dafür verwenden, aber so etwas habe ich
   noch nie gesehen.
#. in dem für Bootcode vorgesehenen Leerraum eines BSD-Labels. Dieser
   ist mit 64 kiB leicht groß genug; man hat großzügig in die Zukunft
   geplant, obwohl der FreeBSD-eigene Bootcode ja nur 8 kiB umfaßt.
   Es macht auch keinen Unterschied, ob die ganze Festplatte
   BSD-partitioniert ist (aka dangerously dedicated) oder ob das BSDsche
   Partitionierungsschema nur in einen MBR-Slice hineingeschachtelt
   wurde, was ja bis vor kurzem der Standardweg für eine
   BSD-Installation war.
#. im MBR und den folgenden Sektoren einer MBR-partitionierten
   Festplatte. Dies ist der Standardweg für die Verwendung von GRUB auf
   MBR-Festplatten. Mit Ausnahme des MBR bleibt nämlich die erste Spur
   ungenutzt, d. h. wird keinem Slice zugeteilt. Eine Spur umfaßt
   meistens 63 Sektoren, d. i. ca. 31 kiB. GRUB ist natürlich so
   gestrickt, daß er die fest verplanten Teile des MBR,
   Partitionstabelle und Magic number, unangetastet läßt.

Eigentlich kann das Core image in jedem staubigen Winkel einer
Datenträgerpartitionierung stehen, sofern es sich dort von
vorgeschalteten Bootmanagern nur *irgendwie* hervorholen läßt.

**Bootmöglichkeiten**

Und wie genau wird FreeBSD nun von GRUB gebootet? Auch dafür finden sich
drei Möglichkeiten:

#. Chainloading: Wenn der FreeBSD-eigene Bootloader vollständig da ist –
   also nicht im obigen Fall 2 – ruft GRUB einfach den Slice bzw. die
   GPT-Partition mit diesem Bootloader auf und legt alles weitere
   vertrauensvoll in dessen Hände. Das ist der bequemste Weg, und GRUB
   bedarf keiner FreeBSD-spezifischen Konfiguration.
#. Übergabe an ``/boot/loader``: Man könnte dies als halbes Chainloading
   bezeichnen. Stufe 1 und 2 des FreeBSD-Bootloaders werden nicht
   gebraucht, deshalb verträgt sich diese Methode auch mit dem obigen
   Fall 2; die eigentliche Arbeit erledigt aber immer noch der
   „\ ``loader``\ “ von FreeBSD.
#. direktes Laden der Systemdateien: GRUB muß nun die ganzen Aufgaben
   des FreeBSD-Loaders selbst übernehmen, insbesondere das Verarbeiten
   der Konfigurationsdateien ``device.hints`` und ``loader.conf``. Im
   einzelnen:

   -  den Kernel laden, ggf. mit Kernelparametern z. B. für serielle
      Konsole;
   -  die "früh" zu ladenden Kernelmodule laden, die sonst in der
      ``loader.conf`` aufgeführt sind, z. B. den NVidia-Grafiktreiber;
   -  die ``device.hints`` dem Kernel verabreichen;
   -  desgleichen für die in der ``loader.conf`` aufgeführten
      Kernel-Umgebungsvariablen und Sysctls.

Methode 3 ist natürlich die komplizierteste; aber weil sie die größte
Flexibilität bietet, ist sie diejenige, welche bei Ausführung von
``grub-mkconfig`` durch die Skripte des FreeBSD-Ports automatisch
eingerichtet wird.

Konfiguration
-------------

Wie oben angegeben, sollte man eine Prä-Konfigurationsdatei
``/usr/local/etc/default/grub`` anlegen und dann das Shellskript
aufrufen:

::

   grub-mkconfig -o /boot/grub/grub.cfg

Anschließend editiert man die erzeugte Datei:

::

   vi /boot/grub/grub.cfg

| Dabei ist zu beachten, daß ``grub-mkconfig`` die Datei
  schreibgeschützt anlegt. Entweder entfernt man den Schreibschutz
  (``chmod u+w /boot/grub/grub.cfg``) oder überzeugt den Editor beim
  Abspeichern, sie trotzdem zu überschreiben (in vi ``:w!``).
| Auf die vielfältigen Konfigurationsmöglichkeiten von GRUB2 will ich
  hier nicht eingehen; das Internet ist voll von Linux-Dokumentation.
  Ich habe u. a. die Teile mit „\ ``load_env``\ “ und „\ ``save_env``\ “
  eliminiert, weil ich von GRUB das immergleiche Verhalten erwarte und
  mir nicht wohl dabei ist, wenn er bei jedem Boot etwas in eine nicht
  gemountete Partition schreibt. Die ``insmod``-Befehle sind alle
  überflüssig, weil GRUB immer von selbst weiß, wann er welches Modul
  hinzufügen muß. Die ``locale``-Datei, z. B. für "de", hat im
  FreeBSD-Port den Pfad
  ``/usr/local/share/locale/de/LC_MESSAGES/grub.mo``. Sie soll ggf. nach
  ``/boot/grub/locale/de/LC_MESSAGES/grub.mo`` kopiert werden, dgl. für
  andere Sprachen.
| Was ich hier beschreibe, sind nur die FreeBSD-spezifischen
  ``menuentry``-Abschnitte. Statt des einen, der nun schon automatisch
  in der ``grub.cfg`` angelegt wurde, will ich drei Menüeinträge zeigen,
  die alle ein und dieselbe FreeBSD-Installation booten, und zwar
  entsprechend den oben genannten Möglichkeiten 1, 2 und 3.

.. code:: bash

   menuentry "FreeBSD 9.0 (1)" --class freebsd --class bsd --class os {
           echo            Chainloading slice hd0,2 ...
           chainloader     (hd0,2)+1
   }

   menuentry "FreeBSD 9.0 (2)" --class freebsd --class bsd --class os {
           set root=hd0,2,a
           search --no-floppy --set --fs-uuid 42869b9f5cddfbea
           echo                    Loading FreeBSD '"loader"' from $root ...
           kfreebsd                /boot/loader
   }

   menuentry "FreeBSD 9.0 (3)" --class freebsd --class bsd --class os {
           set root=hd0,2,a
           search --no-floppy --set --fs-uuid 42869b9f5cddfbea
           echo                    Loading FreeBSD 9.0 from $root ...
           kfreebsd                /boot/kernel/kernel -v
           kfreebsd_module_elf     /boot/modules/nvidia.ko
           kfreebsd_module_elf     /boot/kernel/portalfs.ko
           kfreebsd_loadenv        /boot/device.hints
           set kFreeBSD.vfs.root.mountfrom=ufs:/dev/ada0s3
           #set kFreeBSD.vfs.root.mountfrom.options=rw
           #set kFreeBSD.hint.acpi.0.disabled="1"
           set kFreeBSD.debug.acpi.disabled="timer acad video"
           set kFreeBSD.kern.hz=100
           set kFreeBSD.hw.pci.do_power_nodriver=3
   }

zu (1): Beim Chainloading braucht man keine Variable ``$root``, weil die
Angabe von Partition bzw. Slice im Kommando ``chainloader`` selbst
erfolgen kann.

| zu (2): Die Variable ``$root`` wird zweimal mit einem Wert belegt, und
  zwar einmal ausdrücklich und einmal über das ``search``-Kommando.
  ``search`` ist vorzuziehen, sofern man die UUID des Dateisystems weiß,
  weil sich die genaue Partitionsbezeichnung „\ ``hd0,2,a``\ “ bei
  Änderung der Partitionierung oder Hinzufügen (in Hardware) eines
  weiteren Laufwerks ändern kann. Der ausdrückliche Wert dient nur als
  Rückfalloption, falls das ``search``-Kommando versagt.
| Die UUID erfragt man
  mit\ ``grub-probe -t fs_uuid -d /dev/ada0s2``\ Die Binärdatei
  ``/boot/loader`` wird überraschenderweise mit demselben Kommando
  geladen wie in (3) auch der FreeBSD-Kernel. Man könnte sagen:
  ``/boot/loader`` wird von GRUB als Kernel betrachtet. Wenn der
  „\ ``loader``\ “ dann läuft, erledigt er alle weiteren Aufgaben:
  Kernel und Kernelmodule laden, ``device.hints`` und ``loader.conf``
  befolgen etc. Auch das Menü zeigt er an.

zu (3): Beim Laden des Kernels mit ``kfreebsd`` können dem Kernel
Kommandozeilenoptionen übergeben werden. Folgende werden von GRUB 2.00
erkannt, soweit ich dem Quellcode entnehmen konnte. Weil ich einige
davon nicht verstehe, zitiere ich sie mit den englischen
Textbausteinen:\ ``"dual"    'D'   "Display output on all consoles."
"serial"  'h'   "Use serial console."
"askname" 'a'   "Ask for file name to reboot from."
"cdrom"   'C'   "Use CD-ROM as root."
"config"  'c'   "Invoke user configuration routing."
"kdb"     'd'   "Enter in KDB on boot."
"gdb"     'g'   "Use GDB remote debugger instead of DDB."
"mute"    'm'   "Disable all boot output."
"nointr"  'n'
"pause"   'p'   "Wait for keypress after every line of output."
"quiet"   'q'
"dfltroot"'r'   "Use compiled-in root device."
"single"  's'   "Boot into single mode."
"verbose" 'v'   "Boot with verbose messages."`` Angegeben werden kann
die Langform (``-``\ ``-verbose``, unbedingt mit doppeltem Bindestrich!)
oder Kurzform (``-v``) wie beim Kernel oben.

| Die gewöhnlichen Kernelmodule ``*.ko`` lädt man mit
  ``kfreebsd_module_elf``. GRUB will den vollen Verzeichnispfad, daher
  muß man feststellen, ob ein Modul in ``/boot/modules`` oder
  ``/boot/kernel`` steht.
| Es gibt auch einen Befehl ``kfreebsd_module`` (ohne ``_elf``); ich bin
  nicht sicher, wozu er dient FIXME, eventuell für nicht ausführbare
  Module wie Ramdisk-Images o. ä. Er findet normalerweise keine
  Verwendung.

Mit ``kfreebsd_loadenv`` liest man eine Datei mit Umgebungsvariablen für
den Kernel ein. Das ist geeignet für ``device.hints``, nicht aber für
``loader.conf``. Jene enthält nämlich neben Umgebungsvariablen bzw.
Sysctls noch einige andere Anweisungen:

-  zu ladende Kernelmodule,
-  Anweisungen für den FreeBSD-Bootloader selbst, z. B. bzgl. Menü oder
   Beastie-Bild,
-  Anweisungen für Kerneloptionen, die GRUB ja direkt übergibt, siehe
   oben.

Entweder schreibt man die Umgebungsvariablen und Sysctls also in eine
separate Datei, die man dann wie die ``device.hints`` einliest:

::

   kfreebsd_loadenv /boot/loader.conf.rest

| ... oder man setzt sie einzeln mit GRUBs ``set``-Befehl. Dazu muß
  ihnen ein „\ ``kFreeBSD.``\ “ (beachte die Großschreibung)
  vorangestellt werden.
| Im Beispiel oben habe ich zweiteres getan mit einigen Einstellungen,
  die u. a. dem
  `Energiesparen <https://wiki.freebsd.org/TuningPowerConsumption>`__
  dienen.

Bei verschiedenen Anleitungen im Internet findet sich für die
Pfadangaben in ``kfreebsd`` und den anderen Befehlen eine Syntax der Art

::

   kfreebsd /foo/bar@/boot/kernel/kernel

| Was man dort meist verschweigt, ist, daß der Teil vor dem Zeichen "@"
  nur für das Dateisystem ZFS gilt. Ein ZPool enthält i. a. mehrere
  Verzeichnisbäume, und mit ``/foo/bar@`` wählt man einen bestimmten
  aus, also: Dataset ``bar`` innerhalb von Dataset ``foo`` – und dort
  bitte den Pfad ``/boot/kernel/kernel``!
| Für UFS gibt es diese Syntax nicht.

Wenn etwas schiefläuft und das System so schnell rebootet, daß man die
Fehlermeldung nicht erkennt, hilft ein Befehl'' sleep 7 ''am Ende des
``menuentry``. Damit gewinnt man 7 s Zeit.

Schluß
------

Ist alles vollständig, so wird der Bootcode installiert mit

::

   grub-install /dev/ada0

... hier z. B. in den MBR – oder mit ``/dev/ada0s2`` in einen Slice
anstelle des FreeBSD-Bootcodes. Für GRUB-Versionen vor 2.00 beta2 sollte
man die Option -``-no-floppy`` angeben, weil sonst auch nach einer
Floppy gesucht wird, was die Installation verlangsamt und u. U. sogar
blockiert. 

.. note::

  Einen Wunsch noch: Könnte jemand dieses HowTo bzgl. der anderen BSDs
  ergänzen? Was, außer dem einfachen Umschreiben der Befehle
  (``kfreebsd-->knetbsd``), ist dafür notwendig?

Wenn man GRUB erst einmal beherrscht, kann man richtig abgehobene Sachen damit
machen, z. B.  Boot übers Netzwerk oder aus einem Image. Der Grund, GRUB
einzusetzen, sind ja gerade die exotischen Setups, die der FreeBSDsche
Bootloader nicht kann.

Viel Spaß beim Experimentieren!

.. [1]
   Linux-Dateisysteme lassen, im Gegensatz etwa zu FreeBSDs UFS, den
   ersten Block einer Partition für die erste Stufe eines Bootloaders
   frei. Dort, statt im MBR, kann die erste Stufe von GRUB untergebracht
   werden und so eingerichtet, daß sie die Datei ``core.img`` über
   Blocklists aus der Partition herausliest, ohne das Dateisystem zu
   verstehen. So funktionierte auch der gute alte LILO. Natürlich darf
   diese Datei dann in Linux nicht verschoben oder irgendwie manipuliert
   werden, ohne GRUB jedesmal zu aktualisieren, weshalb diese Methode
   fehleranfällig ist.

* :ref:`genindex`

Zuletzt geändert: |date|

