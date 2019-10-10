runit auf FreeBSD
=================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Runit ist eine Sammlung von kleinen modularen Programmen, die zusammen
eine Alternative zu rcNG (und init) darstellen. Eine Beschreibung der
Vorteile findet sich unter http://smarden.org/runit/benefits.html. Die
Komponenten in runit arbeiten lose gekoppelt zusammen und die offizielle
Dokumentation unter http://smarden.org/runit ist etwas kurz angebunden,
wenn es um die Beschreibung des Zusammenspiels des Komponenten geht. Die
runit Software ist unter FreeBSD als
`Port <http://www.freshports.org/sysutils/runit>`__ verfügbar. Um runit
als Ersatz für rcNG verwenden zu können müssen die runit Programme und
Konfiguration ins Rootdateisystem kopiert werden nach der Installation
des Ports.

Installation des Ports
----------------------

Runit lässt sich entweder aus den Ports installieren mittels:

::

     make -C /usr/ports/sysutils/runit install clean

oder per pkgng:

::

     pkg install sysutils/runit

Anschließend müssen alle Programme die als Teil von runit installiert
wurden nach ``/sbin`` kopiert werden:

::

     pkg query %Fp sysutils/runit | grep '^/usr/local/sbin' | xargs -J % cp -vp % /sbin

Der Bootprozesse mit runit
--------------------------

Ziel dieses Howtos ist es ``/sbin/runit-init`` statt ``/sbin/init`` als
PID 1 zu verwenden um möglichst viele Dienste mit ``runsv`` überwachen
zu können. Um ``runit-init`` verwenden zu können ist ein FreeBSD
spezifisches Skript für jede der vier Stufen in ``runit-init`` nötig.
Die ``runit`` `Manpage <http://smarden.org/runit/runit.8.html>`__
dokumentiert die Aufgaben der Skripte.

Runit kann zwar die Bootzeit eines Systems durch parallelen Start von
Diensten reduzieren jedoch ist es zuviel Aufwand die rcNG Startskripte
vollständig zu ersetzen ohne gewohnte Flexibilität zu verlieren. Dieses
Howto verwendet ``/etc/rc`` um das System zu starten, deaktiviert dabei
die "Zeitfresser" in ``/etc/rc.d/*`` und verlegt sie nach später im
Bootprozess als runit Dienste.

::

     #!/bin/sh -e
     
     # Install the runit startup scripts
     
     mkdir -p /tmp/runit
     mkdir -p /tmp/runit/runsvdir
     mkdir -p /tmp/runit/runsvdir/log
     
     cat >/tmp/runit/1 << EOF
     #!/bin/sh
     
     export PATH=/sbin:/bin:/usr/local/sbin:/usr/local/bin:/usr/games:/usr/sbin:/usr/bin
     trap : 2
     trap : 3
     
     if ! sh /etc/rc autoboot
     then
             echo '/etc/rc/ failed. Press <enter> for emergency shell...'
             read input
             sh -p
             exec reboot
     fi
     
     touch   /etc/runit/stopit
     chmod 0 /etc/runit/stopit
     EOF
     
     cat >/tmp/runit/2 << EOF
     #!/bin/sh
     
     export PATH=/sbin:/bin:/usr/local/sbin:/usr/local/bin:/usr/games:/usr/sbin:/usr/bin
     
     exec runsv /etc/runit/runsvdir
     EOF
     
     cat >/tmp/runit/3 << EOF
     #!/bin/sh
     
     exec 2>&1
     
     export PATH=/sbin:/bin:/usr/local/sbin:/usr/local/bin:/usr/games:/usr/sbin:/usr/bin
     
     echo 'Waiting for services to stop...'
     
     sv -w 180 force-stop /etc/sv/*
     sv        exit       /etc/sv/*
     
     if [ -x /etc/runit/reboot ]; then
     echo 'Reboot...'
             exec reboot
     else
             echo 'Shutdown...'
             exec halt -p
     fi
     EOF
     
     cat >/tmp/runit/ctrlaltdel << EOF
     #!/bin/sh
     
     export PATH=/sbin:/bin:/usr/local/sbin:/usr/local/bin:/usr/games:/usr/sbin:/usr/bin
     
     TIMEOUT=14
     MSG="System is going down in $TIMEOUT seconds..."
     
     touch     /etc/runit/stopit
     chmod 100 /etc/runit/stopit && echo "$MSG" | wall
     sleep "$TIMEOUT"
     EOF
     
     cat >/tmp/runit/runsvdir/run << EOF
     #!/bin/sh -e
     
     echo 'Starting runsvdir.'
     
     exec 2>&1 runsvdir -P /service
     EOF
     
     cat >/tmp/runit/runsvdir/log/config << EOF
     s100000
     n10
     N5
     t86400
     EOF
     
     cat >/tmp/runit/runsvdir/log/run << EOF
     #!/bin/sh -e
     
     REAL_PATH=`realpath .`
     SERVICE_PATH=${REAL_PATH%/*}
     SERVICE_NAME=${SERVICE_PATH##*/}
     
     LOG_DIR=/var/log/$SERVICE_NAME
     LOG_MODE=0755
     LOG_USER=svlogd
     LOG_GROUP=svlogd
     LOG_RAM=$((32 * 1024 * 1024))
     LOG_FILES=128
     LOG_PROCS=1024
     
     install -d -o "$LOG_USER" -g "$LOG_GROUP" -m "$LOG_MODE" "$LOG_DIR"
     install    -o root        -g wheel        -m 0444        config "$LOG_DIR"
     
     exec chpst                          \
             -u "$LOG_USER:$LOG_GROUP"   \
             -m "$LOG_RAM"               \
             -o "$LOG_FILES"             \
             -p "$LOG_PROCS"             \
             svlogd -t "$LOG_DIR"
     EOF
     
     install -d -o root -g wheel -m 0755                 /etc/runit
     install    -o root -g wheel -m 0555 /tmp/runit/1    /etc/runit
     install    -o root -g wheel -m 0555 /tmp/runit/2    /etc/runit
     install    -o root -g wheel -m 0555 /tmp/runit/3    /etc/runit
     install    -o root -g wheel -m 0555 /tmp/ctrlaltdel /etc/runit
     
     install -d -o root -g wheel -m 0755                         /etc/runit/runsvdir
     install    -o root -g wheel -m 0555 /tmp/runit/runsvdir/run /etc/runit/runsvdir
     
     install -d -o root -g wheel -m 0755                                /etc/runit/runsvdir/log
     install    -o root -g wheel -m 0555 /tmp/runit/runsvdir/log/run    /etc/runit/runsvdir/log
     install    -o root -g wheel -m 0555 /tmp/runit/runsvdir/log/config /etc/runit/sunsvdir/log
     
     rm /tmp/runit/1 /tmp/runit/2 /tmp/runit/3 /tmp/ctrlaltdel
     rm /tmp/runit/runsvdir/run /tmp/runit/runsvdir/log/run /tmp/runit/runsvdir/log/config
     rmdir /tmp/runit

Wie auch daemontools bringt runit ein eignes Loggingkonzept mit. Der
eigentliche Dienst wird von ``runsv`` im Vordergrund gestarted. Seine
Standardausgabe ist eine Pipe zu seinem Logging Prozess. Der Logging
Prozess ist optional. Das starten im Vordergrund (ohne das bei doppelte
Forken eines Daemons) ist nicht optional. Der Logging Prozess sollte
unter einem eigenen User laufen. Nun legen wir diesen user an:

::

     pw group add svlogd ...
     pw user add svlogd ...
     pw lock svlogd

Getty...

* :ref:`genindex`

Zuletzt geändert: |date|

