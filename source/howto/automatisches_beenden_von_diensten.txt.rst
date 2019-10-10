Automatisches Beenden von Diensten
==================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-netbsd.png

`NetBSD </NetBSD>`__ und `FreeBSD </FreeBSD>`__ teilen sich eine
Init-Implementierung, die vom NetBSD Projekt als Ablösung des weit verbreiteten
System-V Init entwickelt wurde. Statt die korrekte Reihenfolge über
Ausführungsstufen zu realisieren, kennen die Skripte bei **rc(8)** ihre
Abhängigkeiten. Welche Dienste automatisch gestartet werden, wird in einer
Datei Namens ``/etc/rc.conf`` festgelegt. Diese Dienste werden dann auch beim
Herunterfahren oder Reboot automatisch sauber beendet.

Leider ist das für Dienste, die zur Laufzeit gestartet wurden und nicht in der
``rc.conf`` eingetragen sind nicht der Fall. Vor allem auf Entwicklersystemen
bei denen Dienste hauptsächlich zum Ausprobieren gedacht sind, oder in
wechselnden Umgebungen (Notebook) kommt es gelegentlich vor, dass Dienste
manuell gestartet werden und manuell vor dem Herunterfahren gestoppt werden
müssen.

.. warning::

  Mit dem Parameter ``onestart`` können Dienste in ``/etc/rc.d`` und
  ``/usr/local/etc/rc.d`` manuell, ohne Eintrag in die ``rc.conf``.  gestartet
  werden. Entsprechend funktionieren auch ``onestop`` und
  ``onerestart``.

Dynamisch Einträge in der rc.conf
---------------------------------

Die Lösung des Problems sind dynamische Einträge in der ``rc.conf``. Die
Datei rc.conf ist lediglich ein Shell-Skript in dem Variablen für das
rc-System angelegt werden können. Entsprechend können wie in
Shell-Skripten Variablen bedingt erzeugt werden.

Die meisten Dienste legen einen sogenannten PID-File (eine Datei in der
die Process ID des Dienstes hinterlegt ist) an. Oft reicht die Existenz
der Datei als Hinweis einen Dienst implizit zu aktivieren, damit er
später automatisch gestoppt wird. Das folgende Beispiel sorgt dafür das
VPNC, ein Client für Cisco-VPNs, automatisch beendet wird. Das ist
besonders wichtig, da viele Cisco-VPNs konfiguriert sind über einen
längeren Zeitraum keine Logins mehr zuzulassen, wenn eine Sitzung nicht
ordentlich beendet wurde:

.. code:: bash

   test -e /var/run/vpnc/pid && vpnc_enable="YES"

<note warning>Unter NetBSD fällt das ``_enable``-Suffix weg. Der Eintrag
hieße also ``vpnc="YES"``.</note>

Andere Dienste legen leider kein PID-File an, in einem solchen Fall kann
kann ``pgrep`` Abhilfe schaffen:

.. code:: bash

   pgrep -x postgres > /dev/null && postgresql_enable="YES"

Beispiele
---------

Das folgende Beispiel ist die ``/etc/rc.conf`` des Autors (Stand
26.11.2009), die dynamischen Einträge befinden sich am Ende:

.. code:: bash

   ####################
   # Basic settings
   #

   # The hostname.
   hostname="mobileKamikaze.norad"

   # Network configuration.
   ifconfig_bge0="192.168.1.12/16"
   wlans_wpi0="wlan0"

   # German keyboard.
   keymap="german.iso"

   # Device access ruleset from devfs.rules.
   devfs_system_ruleset="localrules"

   # Clear /tmp during boot.
   clear_tmp_enable="YES"

   ####################
   # i386 tree
   #

   ldconfig32_paths="/usr/lib32 /usr/local32/lib /usr/local32/lib/compat/pkg"

   ####################
   # Base services
   #

   # Service mapper (finger).
   inetd_enable="YES"

   # Print spooler.
   lpd_enable="YES"

   # CPU speed stepping.
   powerd_enable="YES"
   powerd_flags="-a adaptive -b minimum"

   # Linux compatibility.
   linux_enable="YES"

   # Deactivate annoying sendmail stuff.
   sendmail_msp_queue_enable="NO"
   sendmail_outbound_enable="NO"
   sendmail_submit_enable="NO"
   sendmail_enable="NO"

   # Activate moused for the touchpad.
   moused_enable="YES"
   # Touchpad tracking from 80Hz to 200Hz.
   moused_flags="-3 -F 200"
   # USB mouse tracking from 125Hz to 1kHz.
   moused_ums0_flags="-F 1000 -r 4000"
   # USB mouse tracking from 125Hz to 1kHz.
   moused_ums1_flags="-F 1000 -r 4000"

   # NFS mounting is nice to have.
   nfs_client_enable="YES"

   ####################
   # Local services
   #

   # My personal automounter, better than HAL. :)
   automounter_enable="YES"

   # My personal wlan daemon.
   wi_device="wlan0"

   # Skip musicpd if the music is missing.
   test -d /home/musicpd/music &&  musicpd_enable="YES"

   # User mode file systems.
   fusefs_enable="YES"
   fusefs_safe="YES"
   fusefs_safe_evil="YES"

   # Time synchronization.
   openntpd_enable="YES"
   #ntpd_enable="YES"
   #ntpd_sync_on_start="YES"

   # Enable wacom USB tablet support.
   #wacom_enable="YES"

   # Interprocess communication daemon.
   dbus_enable="YES"

   # C build distribution via distcc.
   distccd_flags="-a 127.0.0.0/8 --listen 127.0.0.1 -a 192.168.0.0/16 --listen 192.168.1.12 --user distcc --daemon -P /var/run/distccd.pid"

   # The hardware abstraction layer, kernel panic, probably conflicts with
   # automounter.
   #hald_enable="YES"

   # Default vpnc setup.
   : {vpnc_conf="hska1.conf dukath.conf"}
   : {vpnc_conf_dir="/usr/local/etc/vpnc"}

   ####################
   # Clean shutdown of optional daemons.
   #

   test -e /var/run/wi.pid && wi_enable="YES"
   test -e /var/run/vpnc/pid && vpnc_enable="YES"
   test -e /var/run/httpd.pid && apache22_enable="YES"
   test -e /var/run/tor/tor.pid && tor_enable="YES"
   test -e /var/run/privoxy/privoxy.pid && privoxy_enable="YES"
   pgrep -x postgres > /dev/null && postgresql_enable="YES"
   pgrep -x ppp > /dev/null && ppp_enable="YES"

Verweise
--------

-  Die Manual-Pages **rc(8)**, **rc.conf(5)** und **rc.subr(8)**
-  Die Manual-Page **pgrep(1)**

* :ref:`genindex`

Zuletzt geändert: |date|

