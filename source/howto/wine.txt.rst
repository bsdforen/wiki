Wine
====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel soll einen Einblick geben, wie Programme die unter `Wine
<http://www.winehq.org>`__ funktionieren, möglichst komfortabel eingerichtet
werden. Die Installation von Wine wird nicht erklärt. Ein beliebiges
Wine-Programm (z.B. **winecfg**) sollte bereits einmal gestartet worden sein,
damit das Verzeichnis **~/.wine** angelegt ist.

Laufwerke anlegen
-----------------

Vorkonfiguriert sind bei Wine die Laufwerke **c:** das auf
**~/.wine/drive_c** und **z:** das auf **/** zeigt. Eigene Laufwerke
können auch mit **winecfg** angelegt werden, einfacher und flexibler
geht es aber von der Konsole aus.

Um Wine zum Beispiel das CD-Rom Laufwerk zugängig zu machen, wird
folgendermaßen vorgegangen: <xterm> # cd ~/.wine/dosdevices # ln -s
/cdrom 'd:' </xterm>

Einzelne Programme konfigurieren
--------------------------------

Die meisten Programme benötigen eine individuelle Konfiguration, wenn
sie ordentlich funktionieren sollen. Besonders bei den
Grafikeinstellungen hat jedes Programm andere Bedürfnisse.

Wenn also einem Programm vom Standard abweichend konfiguriert werden
soll, muss **winecfg** gestartet werden. Dort muss in der Kategorie
**Applications** der Button **Add application...** angewählt werden. Es
öffnet sich ein Dialog in dem das gewünschten Programm ausgewählt werden
kann.

Von jetzt an ist das Programm in der Liste der Anwendungen verfügbar. Am
Fenstertitel kann ist zu erkennen, für welches Programm wine gerade
konfiguriert wird. Jetzt können die gewünschten Einstellungen
vorgenommen und das Programm versuchsweise gestartet werden. Vorher
jedoch **Apply** und **OK** anwählen. Bis die richtigen Einstellungen
gefunden sind, musste normalerweise einige Male <xterm> # killall wine
</xterm> eingetippt werden. Dann sollte möglichst nur das zu testende
Programm laufen. <xterm> # wine program.exe </xterm>

wine-kthread
------------

Viele Programme laufen nur mit dem alten **kthread**\ ing, dass
inzwischen von **pthread**\ ing abgelöst wurde besser. Um das alte
kthread zur Verfügung zu haben gibt es im offiziellen `FreeBSD
Wiki <http://wiki.freebsd.org/Wine#head-fa5231640e75c26b7fb58f5c47770135771ad1ce>`__
einen Patch.

<xterm> # cd /usr/ports/emulators/wine # fetch -o files/patch-kthread
'http://wiki.freebsd.org/Wine?action=AttachFile&do=get&target=patch-kthread'
</xterm>

Nach einem Neubau und erneuter Installation von Wine steht die
``wine-kthread`` Binary zur Verfügung.

Tipps
-----

-  In **~/bin** können Skripte angelegt werden, die das Mounten von
   CD-Images und den Programmaufruf wegkapseln.
-  Viele Programme erzeugen Bildfehler, wenn die Grafikoption **Enable
   desktop double buffering** nicht aktiviert ist.
-  Programme, die nur in einer niedrigen Auflösung im Vollbildmodus
   laufen, können komfortabel mit der Option **Emulate a virtual
   desktop** in ein Fenster gesperrt werden. Leider unterstützt wine
   seit einigen Versionen nicht mehr, diese Einstellung seperat ist für
   jedes Programm vorzunehmen.

Programme, die Images benötigen
-------------------------------

Wenn mehrere Programme vorhanden sind, die zum Ausführen `Images
mounten </howto/ISO-Images mounten>`__ müssen, (z.B.
`Starcraft </spiele/Starcraft>`__ oder
`Diablo II </spiele/Diablo II>`__), empfiehlt es sich ein Skript
anzulegen, das diese Aufgabe standardisiert. Ein solches Skript kann im
Verzeichnis **~/bin** angelegt werden. Der Nutzer braucht dafür
natürlich die nötigen `Rechte zum
Mounten </howto/Mounten als Benutzer>`__. Dieses Skript funktioniert
unter FreeBSD, es sollte jedoch nicht allzu schwer sein das zu
portieren.

::

   #!/bin/sh
   #
   # winexec
   #
   # This script handles running programs with wine that require images to be
   # mounted. Simply populate the required variables and put
   #
   # . winexec
   #
   # at the end of your script to run a program.
   #
   # version 1.1

   # User line breaks as delimiters.
   IFS='
   '

   # Set default settings.
   {
       # The script name, by default taken from the filename.
       : ${name="$(echo "$0" | grep -Eo '[^/]+$')"}

       # Name wine binary.
       : ${wine_suffix=""}         # Run a different wine binary.
       : ${wine_bin="wine"}            # The wine binary.

       # Set wine directory.


       : ${wine_dir="$HOME/.wine"}     # Configuration folder.

       # Set logfile.
       : ${wine_log="$wine_dir/log.$name"} # Name of the logfile

       # Set available actions.
       : ${exec_default="run"}         # The command that is used
                           # when no command is given.
       : ${exec_actions="showlog mount umount"}# Other available commands.

       # Set command functions.
       : ${run_cmd="winexec_run"}
       : ${run_pre_cmd=""}
       : ${run_after_cmd=""}
       : ${mount_cmd="winexec_mount"}
       : ${mount_pre_cmd=""}
       : ${mount_after_cmd=""}
       : ${umount_cmd="winexec_umount"}
       : ${umount_pre_cmd=""}
       : ${umount_after_cmd=""}
       : ${showlog_cmd="winexec_showlog"}
       : ${test_cmd="winexec_test"}
       : ${cleanup_cmd="winexec_cleanup"}

       # Mount settings.
       : ${mount_images=""}            # A newline separated list
                           # of images.
       : ${mount_nodes=""}         # Set this if you want
                           # device links to /dev.
       : ${mount_devices="$(printf 'd\ne\nf\ng\nh\ni\nj\nk\nl')"}
                           # List of permitted dos devices.
       : ${mount_type="cd9660"}        # The image format.
       : ${mounted_file="mounted.$name"}   # Remember mounted devices.
       : ${loaded_file="loaded.$name"}     # Remember loaded images.

       # Run settings.
       : ${run_binary=""}          # The name of the binary to run.
       : ${run_folder=""}          # The folder from where to run.
       : ${run_parameters=""}          # Parameters for the program.

       # X settings to restore if changed.
       : ${x_get_res_cmd="xrandr | grep \* | grep -Eo '+[0-9]+x[0-9]+'"}
       : ${x_res=$(eval "$x_get_res_cmd")} 
       : ${x_set_res_cmd="xrandr -s $x_res"}
       : ${x_refresh_cmd="xgamma -g 1; xrefresh"}
   }

   # Mounts the images given in 'mount_isos'.
   winexec_mount() {
       eval "$mount_pre_cmd"

       for image in $mount_images; {
           echo "Loading image '$image'."
           image=$(mdconfig -a -t vnode -f "$image")
           echo "$image" >> "$wine_dir/$loaded_file"

           for device in $mount_devices; {
               device_dir="$wine_dir/dosdevices/$device:"
               if [ ! -L "$device_dir" -a ! -e "$device_dir" ]; then
                   echo "Creating device '$device:'."
                   if [ "$mount_nodes" ]; then
                       ln -s "/dev/$image" "$device_dir:"
                   fi
                   mkdir "$device_dir"
                   mount -r -t $mount_type "/dev/$image" "$device_dir"
                   echo "$device" >> "$wine_dir/$mounted_file"
                   break
               fi
           }
       }

       eval "$mount_after_cmd"
   }

   # Unmount images.
   winexec_umount() {
       eval "$umount_pre_cmd"

       for device in $(cat "$wine_dir/$mounted_file" 2> /dev/null); {
           echo "Destroying device '$device:'."
           device_dir="$wine_dir/dosdevices/$device:"
           umount -f "$device_dir"
           rmdir "$device_dir" > /dev/null 2>&1
           rm "$device_dir:" > /dev/null 2>&1
       }
       rm "$wine_dir/$mounted_file" > /dev/null 2>&1

       for image in $(cat "$wine_dir/$loaded_file" 2> /dev/null); {
           echo "Destroying image node '$image'."
           mdconfig -d -u "$image"
       }
       rm "$wine_dir/$loaded_file" > /dev/null 2>&1

       eval "$umount_after_cmd"
   }

   # Show the logfile.
   winexec_showlog() {
       cat "$wine_log"
   }

   # Test weather all required settings are set.
   winexec_test() {
   }

   # Cleanup the environment.
   winexec_cleanup() {
       # Check screnn resolution.
       if [ "$(eval "$x_get_res_cmd")" != "$x_res" ]; then
           eval "$x_set_res_cmd"
       fi

       # A little screen refresh never hurts.
       eval "$x_refresh_cmd"
   }

   # Run the given binary.
   winexec_run() {
       eval "$test_cmd"
       eval "$mount_cmd"

       eval "$run_pre_cmd"

       echo "Running '$run_binary' with '$wine_bin$wine_suffix'."
       cd "$run_folder" && $wine_bin$wine_suffix "$run_binary" $run_parameters > "$wine_log" 2>&1

       eval "$run_after_cmd"

       eval "$cleanup_cmd"
       eval "$umount_cmd"
   }

   # Get the given command.
   command=run
   if [ $1 ]; then
       command=$1
   fi

   # Run the requested action.
   for action in $exec_default $exec_actions; {
       if [ "$command" = "$action" ]; then
           eval $"${action}_cmd"
           exit 0
       fi
   }

   # This happens when a wrong parameter is supplied.
   exit 1

Alternativen
------------

Leider funktioniert bei weitem nicht alles mit Wine. Zumindest dort, wo
Performance nicht ganz so wichtig ist, ist die Verwendung virtueller
Maschinen eine Erwägung wert. Die populärsten Vertreter dieser Kategorie
sind wohl `VMWare </anwendungen/VMWare>`__ (Closed Source) und
`qemu </anwendungen/qemu>`__.

Verweise
--------

-  http://wiki.freebsd.org/Wine - Porting Wine to FreeBSD
-  Die `Wine Homepage <http://winehq.org/>`__.
-  `AppDB <http://appdb.winehq.org/>`__, eine Datenbank der unter Wine
   laufenden Programme.
-  `Spiele mittels Wine </spiele/Hauptseite#Spiele mittels Wine>`__.
-  `emulators/wine <https://www.google.com/search?q=emulators/wine&btnI=lucky>`__
   in den FreeBSD Ports.
-  `emulators/wine <https://www.google.com/search?q=emulators/wine&btnI=lucky>`__
   in Pkgsrc.

* :ref:`genindex`

Zuletzt geändert: |date|

