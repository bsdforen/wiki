Torsmo
======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Torsmo (**T**\ yopoyta\ **OR**\ velo **S**\ ystem **MO**\ nitorist) ein
Systemmonitor. Man kann ihn in eine der vier Ecken des Desktops
platzieren. Torsmo ist im Gegensatz zu
`GKrellM </anwendungen/GKrellM>`__ recht einfach aufgebaut und zeigt
dementsprechend einfach die Ausgaben als Text an. Ein großer Vorteil
ist, dass nur eine Library benötigt wird, nämlich die Xlib. Unter
Features ist aufgelistet, welche Informationen Torsmo unter anderem
ausgeben kann. |torsmo.jpg|

Features
--------

-  Kernel Version
-  Uptime
-  Systemzeit
-  Netzwerktraffic
-  Speicher und Swap Auslastung
-  Hostname
-  Prozessortyp
-  Systemname
-  Batteriekapazität
-  Zahl der laufenden Prozesse
-  vorhande E-Mails
-  Dateisysteminformationen

Konfiguration
-------------

Torsmo liest eine Konfigurationsdatei im Homeverzeichnis des jeweiligen
Benutzers aus:

::

    # cat ~/.torsmorc

::

      background yes
      font 6x10
      update_interval 10.0
      own_window no
      minimum_size 280 5
      draw_shades no
      draw_outline no
      draw_borders no
      stippled_borders 8
      border_margin 4
      border_width 1
      
      default_color white
      default_shade_color black
      default_outline_color black
      
      alignment bottom_left
      
      gap_x 12
      gap_y 12
      
      no_buffers yes
      
      uppercase no
      
      TEXT
      $nodename - $sysname $kernel on $machine
      $stippled_hr
      Uptime $uptime
      RAM Usage: $mem/$memmax - $memperc% ${color red}${membar 6}$color
      Swap Usage: $swap/$swapmax - $swapperc% ${color red}${swapbar 6}$color
      CPU Usage: $cpu% ${color red}${cpubar 6}$color
      Processes: $processes  Running: $running_processes
      Networking: Up: ${upspeed eth0} k/s - Down: ${downspeed sis0} k/s
      File systems (used/total):
        / ${fs_used /}/${fs_size /} ${color red}${fs_bar 6 /}$color
        /home ${fs_used /home}/${fs_size /home} ${color red}${fs_bar 6 /home}$color

Weblinks
--------

-  `sysutils/torsmo <https://www.google.com/search?q=sysutils/torsmo&btnI=lucky>`__
-  `torsmo.sourceforge.net <http://torsmo.sourceforge.net/>`__

.. |torsmo.jpg| image:: images/torsmo.jpg
   :class: align-right
   :width: 256px

* :ref:`genindex`

Zuletzt geändert: |date|

