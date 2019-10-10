Jails
=====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

In diesem Artikel wird kurz erklärt was Jails sind, wofür sie nützlich
sind. Weiterhin wird die Schritt-für-Schritt-Installation von Jails
beschrieben.

Was sind Jails?
---------------

Das FreeBSD Handbuch fasst es ganz passend zusammen:

   Jails setzen auf dem traditionellen chroot(2)-Konzept auf und
   verbessern es auf unterschiedlichste Art und Weise. In einer
   traditionellen chroot(2)-Umgebung sind Prozesse auf den Bereich des
   Dateisystems beschränkt, auf den sie zugreifen können. Der Rest der
   Systemressourcen (wie zum Beispiel eine Reihe von Systembenutzern,
   die laufenden Prozesse oder das Netzwerk-Subsystem) teilen sich die
   chroot-Prozesse mit dem Host-System. Jails dehnen dieses Modell nicht
   nur auf die Virtualisierung des Zugriffs auf das Dateisystem, sondern
   auch auf eine Reihe von Benutzern, das Netzwerk-Subsystem des
   FreeBSD-Kernels und weitere Bereiche aus. Eine ausführlichere
   Übersicht der ausgefeilten Bedienelemente zur Konfiguration einer
   Jail-Umgebung finden Sie im Abschnitt Abschnitt 15.5 des Handbuchs.

`Quelle im
Handbuch <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/jails-intro.html>`__

Installation von Jails
----------------------

Der hier beschriebene Weg zeigt wie man händisch Jails anlegt.
Alternativ können Jails auch automatisiert angelegt werden. Dies wird im
Artikel `ezjail <ezjail>`__ beschrieben.

Vorbereitung
~~~~~~~~~~~~

Bevor es mit der eigentlich Installation los geht, müssen zu erst einmal
die Rahmenbedingungen gegeben werden. Dazu gehört das sich im Hostsystem
keine Dienste an einen Port aller IPs binden und das es eine Alias IP
Adresse auf einem Interface gibt. 

::

  % sockstat -4 

zeigt alle lauschenden Dienste an. Falls sich Dienste z.B. an \*:22 (sshd)
binden, müssen diese so eingestellt werden, dass sie nur auf der IP des
Hostsystem lauschen.

Damit das neue Jail auch eine IP hat wird einem Interface ein Alias
gegeben: 

::

  ifconfig em0 alias 192.168.1.101

Um dieses Alias auch über Systemneustarts hinweg zu erhalten wird es in die
/etc/rc.conf eingetragen

::

   ifconfig_em0_alias0="192.168.1.101"

("em0" ist der Name des Interfaces, alias0 ist eine fortlaufende
Benennung alias0..alias1... usw)

Build a world
~~~~~~~~~~~~~

Ein wichtiger Bestandteil eines Jails ist die Welt im Gefängnis. Diese
erstellen wir klassisch mit::

  cd /usr/src && make buildworld

Jetzt müssen wir uns entscheiden wo das Jail liegen soll: Typischer
Weise ist dies unter /urs/jails. Aber auch /var/jails oder /home/jails
ist üblich. Praktisch kann ein Jail überall dort liegen wo es genügend
Speicherplatz hat. Aus Sicherheitsgründen sollte man aber auch darauf
achten, dass nur Root Zugriff auf die Jails hat.

::

  make installworld DESTDIR=/usr/jails/testjail 
  make distribution DESTDIR=/usr/jails/testjail 
  mount -t devfs devfs /usr/jails/testjail/dev

Der letzte Befehl lässt das /dev-Verzeichnis innerhalb der Jail
erscheinen. Das ist nicht zwingend notwendig, allerdings kann es gerade
beim Installationsprozess zu Verwirrungen kommen wenn z.B. kein
/dev/urandom vorhanden ist.

Theoretisch ist das Jail jetzt schon fertig "installiert". Mit ::

  jail /usr/jails/testjail testjail.example.com 192.168.1.101

landet man in der Root-Shell des Jails. Hier müssen noch ein paar
Änderungen vor genommen werden, bevor das Jail komplett ist. Am besten
legt man eine leere /etc/fstab an, es soll schon zu Problemen gekommen
sein wenn keine vorhanden war::

  touch /etc/fstab

Außerdem sollte ein Passwort für den Rootbenutzer vergeben werden und
ein weiterer, unprivilegierter Benutzer angelegt werden::

  passwd 
  adduser

Jetzt das Jail mittels 'exit' wieder verlassen.

Auf dem Hostsystem
~~~~~~~~~~~~~~~~~~

Nach diesen Schritten wird dem Hostsystem beigebracht das es ein Jail
beheimatet. Dazu wird in die /etc/rc.conf folgendes eingetragen:

::

   jail_enable="YES"
   jail_list="testjail"

Der Eintrag "jail_list" enthält eine durch Leerzeichen getrennte Liste
aller auf dem System zu startenden Jails. Für jedes Jail muss auch sein
Pfad, sein Hostname und seine IP bekannt sein:

::

   jail_testjail_rootdir="/usr/jails/testjail"
   jail_testjail_hostname="testjail.example.com"
   jail_testjail_ip="192.168.1.101"
   # für /dev braucht man auch diese Zeile:
   jail_testjail_devfs="YES"

Damit auch die Namensauflösung in den Jails funktioniert muss dort eine
funktionierende /etc/resolv.conf liegen. Einfach aus dem Hostsystem
kopieren::

  cp /etc/resolv.conf /usr/jails/testjail/etc/resolv.conf

Jails administrieren
--------------------

Starten
~~~~~~~

::

  /etc/rc.d/jail start

Wenn nur einzelne Jails gestartet werden sollen::

  /etc/rc.d/jail start jailname

Stoppen
~~~~~~~

Funktioniert genau so wie das starten, "stop" anstatt "start".

::

  /etc/rc.d/jail stop jailname

Alle Jails eines Systems anzeigen lassen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Um zu sehen welche Jails auf dem System laufen::

  jls

Einen Befehl ausführen
~~~~~~~~~~~~~~~~~~~~~~

::

  jexec [-u username \| -U username] jail command ...

Um z.B. eine Shell in einer Jail zu bekommen::

  jexec testjail /bin/sh

Sysctl-Knöpfe für Jails
-----------------------

::

  security.jail.set_hostname_allowed (1/0) 

Normalerweise kann Root in einer Jail den Hostname setzten. Wenn der Knopf auf
0 gesetzt wird nicht mehr.

::

  security.jail.socket_unixiproute_only (0/1)

Wenn andere Protokolle als IP und Unix Sockets genutzt werden dürfen sollte man
dieses Knöpfchen drehen.

::

  security.jail.allow_raw_sockets (0/1)

Wenn man aus einer Jail heraus "ping" oder "traceroute" verwenden will braucht
man dafür "rohe Sockel".

::

  security.jail.sysvipc_allowed (0/1)

Manche Programm brauchen wirklich dieses "shared memory". Prominentes Beispiel
ist PostgreSQL.

::

  security.jail.enforce_statfs (2/1/0)

Bei 2 können Programm innerhalb der Jail nur die Mountpunkte in ihrer Jail
sehen. Bei 1 können Programm innerhalb der Jail die Mountpunkte in ihrer Jail
sehen und sehen zusätzlich noch wo im Hostsystem ihre Jail liegt. Bei 9 können
Programm innerhalb der Jail alle Mountpunkte des Systems sehen!

Jail Tools
----------

Diese Liste verweist auf nützliche Tools, die einem den Umgang mit Jails
erleichtern.

::

   *[[Freshports>sysutils/ezjail]] - Ein Script zum automatisierten Anlegen von Jails
   *[[Freshports>sysutils/jailadmin]] - Tool zum administrieren von Jails
   *[[Freshports>sysutils/jailctl]] - Ein weiteres Toolset inkl. Backup von Jails
   *[[Freshports>sysutils/jailer]] - Tools im Jail
   *[[Freshports>sysutils/jailutils]] - Tools zu Starten/Herunterfahren und Anzeigen von Jails

Siehe auch
----------

-  `ezjail </howto/ezjail>`__

* :ref:`genindex`

Zuletzt geändert: |date|

