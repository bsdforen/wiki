Acer Aspire One
===============

.. |date| date::

.. warning::

  Die Inhalte sind arg veraltet!
  Die Inhalte gründen nicht auf dem Stand von `FreeBSD 11/12`
  Dennoch können Inhalte richtig sein.

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Hier folgen einige Informationen zur Installation von FreeBSD 8.x STABLE auf
einem Acer Aspire One (A150X). Einige Einstellungen sollten auch mit anderen
BSDs funktionieren.

Status
------

Folgende Komponenten werden unterstützt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  LAN re0
-  Touchpad (synaptics)
-  Sound
-  WLAN ath0 über Madwifi (siehe unten)

Noch nicht getestet
~~~~~~~~~~~~~~~~~~~

-  Webcam
-  Cardreader

loader.conf
-----------

Um Sound und das Touchpad nutzen zu können, müssen folgende Einträge in
**/boot/loader.conf** eingefügt werden:

::

   snd_hda_load="YES"
   hw.psm.synaptics_support=1

rc.conf
-------

So könnte die **/etc/rc.conf** aussehen:

::

   hostname="<Rechnername>"
   wlans_ath0="wlan0"
   ifconfig_re="DHCP"      # LAN über DHCP
   ifconfig_wlan0="WPA DHCP"   # WLAN über DHCP

   keymap="german.iso"     # Deutsches Tastaturlayout
   keyrate="fast"

   dbus_enable="YES"
   hald_enable="YES"
   polkitd_enable="YES"
   [...]

Xorg-Konfiguration
------------------

Dies ist die an FreeBSD angepasste **/etc/X11/xorg.conf**-Datei der
original Linux-Installation:

::

   # Xorg configuration created by system-config-display
   Section "ServerFlags"
   #   Option "DontZap" "yes"
   #   Option "DontVTSwitch" "yes"
   EndSection

   Section "ServerLayout"
       Identifier     "Default Layout"
       Screen      0  "Screen0" 0 0
   #   InputDevice    "Mouse0" "CorePointer"
   #   InputDevice    "Synaptics Mouse" "AlwaysCore"
       InputDevice "Synaptics_Touchpad"    "CorePointer"

       InputDevice    "Keyboard0" "CoreKeyboard"
   EndSection

   Section "InputDevice"
       Identifier  "Keyboard0"
       Driver      "kbd"
       Option      "XkbModel" "pc105"
           Option      "XkbLayout" "de,gb,us(euro)"
           Option      "XkbOptions" "grp:alt_shift_toggle"
   EndSection

   Section "InputDevice"
       Identifier  "Synaptics_Touchpad"
       Driver      "synaptics"

       Option      "Device"        "/dev/psm0"
       Option      "Protocol"      "psm"

       Option      "LeftEdge"      "1700"
       Option      "RightEdge"     "5300"
       Option      "TopEdge"       "1700"
       Option      "BottomEdge"        "4200"

       Option      "FingerLow"     "25"
       Option      "FingerHigh"        "30"

       Option      "MaxTapTime"        "180"
       Option      "MaxTapMove"        "220"

       Option      "VertScrollDelta"   "100"
       Option      "HorizScrollDelta"  "100"

       Option      "MinSpeed"      "0.06"
       Option      "MaxSpeed"      "0.06"
       Option      "AccelFactor"       "0.0010"

       Option      "ScrollButtonRepeat"    "100"
       Option      "UpDownScrolling"   "on"
       Option      "UpDownRepeat"      "on"
       Option      "LeftRightScrolling"    "on"
       Option      "LeftRightRepeat"   "on"

       # "SHMConfig on" seems good works with synclient(1).  But this
       # options is insecure.  I recommended "off" as default.
       Option      "SHMConfig"     "off"

       # If you use circular touchpad, uncomment them.
   #   Option      "CircularPad"       "on"
   #   Option      "CircularScrolling" "on"
   #   Option      "CircScrollDelta"   "0.5"
   EndSection

   Section "InputDevice"
       Identifier  "Mouse0"
       Driver      "mouse"
       Option      "Protocol" "IMPS/2"
       Option      "Device" "/dev/input/mice"
       Option      "ZAxisMapping" "4 5"
       Option      "Emulate3Buttons" "no"
   EndSection

   Section "Monitor"
       Identifier  "Monitor0"
       Modeline  "1024x600" 48.96 1024 1064 1168 1312 600 601 604 622 -HSync +VSync
   #   Option  "Above" "Monitor1"
   EndSection

   Section "Device"
       Identifier  "Videocard0"
       Driver      "intel"
   #   Option      "monitor-LVDS" "Monitor0"
   #   Option      "monitor-VGA" "Monitor1"
       Option      "Clone" "true"
       Option  "MonitorLayout" "LVDS,VGA"
       BusID   "PCI:0:2:0"
   #   Screen  0
   EndSection

   Section "Screen"
       Identifier "Screen0"
       Device     "Videocard0"
       Monitor     "Monitor0"
       DefaultDepth     24
       SubSection "Display"
           Viewport   0 0
           Depth     24
           Modes    "1024x600" "800x600" "640x480"
            Virtual 1024 768
       EndSubSection
   EndSection

WLAN ath0
---------

Der Atheros Chipssatz des One kann mittels madwifi genutzt werden. Dazu
wird ein Snapshot des `madwifi-Projekts <http://madwifi-project.org/>`__
heruntergeladen:
http://snapshots.madwifi-project.org/madwifi-hal-0.10.5.6/. Der
http://snapshots.madwifi-project.org/madwifi-hal-0.10.5.6/madwifi-hal-0.10.5.6-r3879-20081204.tar.gz
wird in diesem Beispiel verwendet.

Das Archiv wird in einem beliebigen Ordner entpackt:

::

  $ su - 
  Password:
  cd <beliebiger Ordner> 
  tar xvfz madwifi-hal-0.10.5.6-r3879-20081204.tar.gz

Jetzt die notwendigen Dateien aus dem Madwifi-Ordner in den ``/usr/src/``-Ordner
kopieren:

::

  cd madwifi-hal-0.10.5.6-r3879-20081204 
  cp -R * /usr/src/sys/contrib/dev/ath/

Nun kann der Kernel neu gebaut und installiert werden. Nach
erfolgreichem Bau des Kernels und der Module den One neustarten. Der
eingebaute WLAN-Chip sollte nun erkannt und benutzbar sein.

* :ref:`genindex`

Zuletzt geändert: |date| 
