Automount
=========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Anders als man es beispielsweise von Windows kennt, muss man bei \*BSD-Systemen
CDROM/DVD im Laufwerk mounten. Wenn eine CDROM/DVD ins Laufwerk gelegt wird,
weiß das System von dieser Interaktion des Benutzers noch nichts. Der Benutzer
muß dem System zunächst mitteilen, daß ein Medium ins Laufwerk gelegt wird.
Dies geschieht in der Regel mit:

::

  mount -t cd9660 /dev/cdrom /mnt

In diesem Artikel wird beschrieben, wie man das automatische Erkennen
von CDROM/DVD usw. aktivieren kann.

Vorbereitung
------------

Zuerst stellt man sicher, dass man die entsprechenden devices auch
mounten kann. Dann trägt man sich die entsprechenden Mountpoints in die
``/etc/fstab`` ein:

::

   /dev/acd0c /mnt/cdrom cd9660 ro,noauto,nosuid 0 0
   /dev/fd0c /mnt/floppy msdos rw,noauto 0 0

Nun legen wir die beiden benötigten Verzeichnisse zum mounten an:
<xterm> # cd /mnt # mkdir cdrom # mkdir floppy </xterm> Wenn wir nun als
root <xterm> # mount /mnt/cdrom </xterm> und <xterm> # umount /mnt/cdrom
</xterm> ausführen, sollte das CD-ROM gemountet und wieder entmountet
werden.

AMD
---

In der ``/etc/rc.conf`` aktivieren wir nun den Automount Daemon (AMD)
und den auch noch benötigten Portmapper portmap:

::

   portmap_enable="YES"      # Run the portmapper service (YES/NO).
   amd_enable="YES"
   amd_flags="-a /mnt/.amd_mnt -c 10 -w 5 -l syslog /mnt/host /etc/amd.map"

Dann legen wir das Verzeichnis ``/mnt/.amd_mnt`` an: <xterm> # mkdir -p
/mnt/.amd_mnt </xterm> Und noch die Verzeichnisse
``/mnt/host/localhost/cdrom`` und ``/mnt/host/localhost/floppy``:
<xterm> # mkdir -p /mnt/host/localhost/cdrom # mkdir -p
/mnt/host/localhost/floppy </xterm> Und noch zwei Symlinks, auf die wir
später zugreifen werden: <xterm> # cd / # ln -s
/mnt/host/localhost/cdrom cdrom # ln -s /mnt/host/localhost/floppy
floppy </xterm>

Nun fehlen nur noch ein paar Einträge in der ``/etc/amd.map``:

::

   /defaults type:=host;fs:=${autodir}/${rhost}/host;rhost:=${key}
   * opts:=rw,grpid,resvport,vers=2,proto=udp,nosuid,nodev

   localhost type:=auto;fs:=${map};pref:=${key}/

   localhost/cdrom type:=program;fs:=/mnt/cdrom;\
   mount:="/sbin/mount mount /mnt/cdrom";\
   unmount:="/sbin/umount umount /mnt/cdrom"

   localhost/floppy type:=program;fs:=/mnt/floppy;\
   mount:="/sbin/mount mount /mnt/floppy";\
   unmount:="/sbin/umount umount /mnt/floppy"

Nach einem Neustart des Systems (oder starten der Daemonen per Hand)
sollte es nun funktionieren.

Die Statusausgabe des AMD nach dem Start:

::

  # amq /root "root" tremor:(pid118)
  /mnt/host toplvl /etc/amd.map /mnt/host

Wenn man ein:

::

  # cd /cdrom

ausführt, sollte eine eingelegte CD automatisch gemountet und der Inhalt bereit
für den Zugriff sein.

Der automount funktioniert aber nur, wenn die beiden Symlinks (/cdrom
oder /floppy) oder die Verzeichnisse ``/mnt/host/localhost/cdrom`` bzw.
``/mnt/host/localhost/floppy``, berührt werden.

Erweiterungsvorschlag
---------------------

Legt einen Symlink von ``/usr/share/skel/`` nach ``/cdrom`` und
``/floppy``. Dann hat ihn auch jeder User automtisch bei Erstellung.

* :ref:`genindex`

Zuletzt geändert: |date|

