Bootvorgang verschönern/verkürzen
=================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Der Bootvorgang von FreeBSD lässt sich auf verschiedene Arten verschönern und
auch ein wenig verkürzen.

Verzögerung einstellen
----------------------

Fangen wir zuerst mit dem Loader an:

Nachdem der Loader geladen wurde, kann man zwischen verschiedenen
Bootoptionen wählen (ACPI en-/disable, nur Single User Mode etc.).
Standardmässig wird der Eintrag Nr.1 nach 10 Sekunden automatisch
verwendet. Trägt man nun in die Datei ``/boot/loader.conf``:

::

   autoboot_delay="2"

ein, so wird bereits nach 2 Sekunden der Bootvorgang fortgesetzt.

Farbiger Beastie
----------------

Der Beastie im Loadermenu (erst ab FreeBSD 5) ist standardmässig eine
simple, weiße ASCII-Grafik. Mit dem Eintrag:

::

   loader_color="YES"

in der Datei ``/boot/loader.conf`` ist der Beastie ab sofort hübsch
farbig.

In FreeBSD 6 wurde Beastie wieder aus dem Loadermenu verbannt. Um ihn
wiederzubeleben reicht folgender Eintrag in der ``loader.conf``:

::

   loader_logo="beastie"

Splashscreen einrichten
-----------------------

Unter FreeBSD lässt sich ähnlich wie unter Linux oder Windows auch ein
Splashscreen einrichten. Um einen Splashscreen mit einer Auflösung höher
als 320x240 zu verwenden, muss das VESA Modul geladen werden. Je nach
Graphikkarte kann es auch sein, dass dies nicht funktioniert. In diesem
Fall kann leider nur ein Splashscreen mit einer Auflösung von 320x240
Pixeln verwendet werden. Folgende Einträge müssen in der Datei
``/boot/loader.conf`` eingetragen werden:

::

   splash_bmp_load="YES" # Fuer Bitmap Files
   splash_pcx_load="NO" # Fuer PCX Files (Nur ein Eintrag aktivieren)
   vesa_load="YES" # Auf NO setzen wenn von der Graphikkarte nicht unterstützt
   bitmap_load="YES"
   bitmap_name="/boot/splash.bmp" # Name der Bitmap Datei

Danach muss natürlich noch eine Bitmap Datei unter /boot/splash.bmp
abgelegt werden. Falls der VESA Modus funktioniert, können auch Bilder
mit einer Auflösung grösser als 320x240 eingerichtet werden. Wichtig ist
jedoch, dass die Bilder maximal 256 Farben (8Bit) aufweisen und im
Bitmap Format vorliegen! Besitzt man z.B. ein JPEG so öffnet man dies
unter Gimp und schraubt via "Image -> Scale Image" die Auflösung
herunter. Danach kann man mit "Image -> Mode -> Indexed" die Farbtiefe
verringern. Wichtig ist, dass der Eintrag "Generate Optimum Palette"
aktiviert ist und 256 Farben ausgewählt sind. Unter "Dithering Options"
kann man die Dithering Methode auswählen, hier lohnt sich Ausprobieren
um das ansprechendste Ergebnis zu erhalten. Die Datei anschliessend als
Bitmap speichern und nach ``/boot/splash.bmp`` verschieben (benötigt
natürlich root Rechte).

Weitere Optimierungsmöglichkeiten
---------------------------------

Generell sollte man alle nicht benötigten Dienste deaktivieren.
Systemdienste wie z.B. der ssh-Daemon lassen sich via einen Eintrag in
die Datei ``/etc/rc.conf`` deaktivieren (``sshd_enable="NO"``). Hilfe
dazu bietet die Manpage von ``rc.conf`` (``man rc.conf``). Nachträglich
installierte Dienste (z.B. Apache) lassen sich meistens auch auf diesem
Weg deaktivieren. Ansonsten hilft auch unter ``/usr/local/etc/rc.d/``
das entsprechende Startscript nicht ausführbar zu machen
(``chmod -x <script>``). Weiteres Potential für Optimierungen bietet
eine Bereinigung/Entschlackung des Kernels. Generell sollte man nicht
benötigte Devices in der Kernel-Configdatei deaktivieren (einfach ein #
vor den entsprechenden Eintrag stellen), so spart man Zeit beim Booten.
Hilfe zum Kompilieren von einem Kernel findet ihr hier:
`Kernel erstellen </freebsd/Kernel erstellen>`__.

* :ref:`genindex`

Zuletzt geändert: |date|

