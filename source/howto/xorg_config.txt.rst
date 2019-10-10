Xorg Konfigurieren
==================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Konfigurationsdateien gehören in */usr/local/etc/X11/xorg.conf.d*/.
Konfigurationsdateien für sind `Xorg </anwendungen/Xorg>`__ sind
optional.

server.conf

::

   Section "ServerFlags"
       Option "AutoAddDevices" "Off"
   EndSection

kbd.conf

::

   Section "InputDevice"
       Identifier "kbd0"
       Driver     "kbd"
           Option     "XkbLayout"  "de"
   EndSection

mouse.conf

::

   Section "InputDevice"
       Identifier "mouse0"
       Driver     "mouse"
       Option     "Device"               "/dev/sysmouse"
       Option     "AccelerationProfile"  "-1"
       Option     "AccelerationScheme"   "none"
   EndSection

video.conf

::

   # Configure video driver
   Section "Device"
       Identifier "video0"
       Driver     "scfb"              # Optional, usually auto-detect works
       Option     "ShadowFB" "Off"    # Bearable performance with vesa and
                                      # scfb, but introduces mouse flicker
   EndSection

   # Configure monitor DPI
   Section "Monitor"
       Identifier  "monitor0"
       DisplaySize 309.93 174.34      # In mm, used to calculate DPI
   EndSection

   # Tie video card to monitor
   Section "Screen"
       Identifier "screen0"
       Device     "video0"
       Monitor    "monitor0"
   EndSection

* :ref:`genindex`

Zuletzt geändert: |date|

