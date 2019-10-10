Backup über das Netzwerk
========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Da nicht auf jedem Rechner ein Backupstreamer oder andere Sicherungsgeräte
vorhanden sind, kann man Daten auch über das Netzwerk auf andere im Netz
befindlichen Rechner übertragen. Hier bietet sich **dump** und zum Transport,
zur Verschlüsselung und letztlich auch zur Authentisierung am Zielrechner das
ssh-Protokoll an.

Dump via ssh-tunnel
-------------------

::  

  # dump -La0 -f - /mountpunkt \| gzip -c \| ssh user@ziel "dd of=/pfad/zu/backup.dump"

.. warning::

  Ist der Fremdrechner nicht 100% sicher, kann auch eine verschlüsselte
  variante gewählt werden 

  ::

    # dump -La0 -f - /mountpunkt \| gzip -c \| openssl enc *-bf-cbc* -pass pass:\ *Passwort* \| ssh user@ziel "dd of=/pfad/zu/backup.dump"

Wiederherstellung des Backups
-----------------------------

::  

  # cd /mountpunkt
  # ssh user@ziel "dd if=/pfad/zu/backup.dump" \| gunzip -c \| restore -rf - 
  # rm restoresymtable 

.. warning::

  Ist das Backup verschlüsselt wie bei "Dump via ssh-tunnel" demonstriert, wird
  wie folgt wiederhergestellt 

  ::

    # ssh user@ziel "dd if=/pfad/zu/backup.dump"  \| openssl enc *-bf-cbc* -d -pass pass:\ *Passwort* \| gunzip -c \| restore -rf - 

Trockenübung
------------

Für den ungeübten Benutzer ist es sicher sinnvoll erstmal eine
"Trockenübung" zu machen:

Zunächst ein kleines Test-Image anlegen:

::

  # dd bs=1m count=1024 < /dev/zero > test.image 

und es mit

:: 

  # mdconfig -f test.image 

einbinden. Mit

:: 

  # newfs /dev/md0 

legst du darin ein Dateisystem an. Auf das kannst du dann mal
Probeweiser dein restore loslassen (nimm einen kleinen Dump wie /var
oder /tmp) und es danach mounten.

::

  # mount /dev/md0 mountpunkt 

Sollten keine Fehler auftreten, kann man mit dem Wiederherstellen
beginnen.

Nach der Wiederherstellung des Systems sollte man im vollständig
gemounteten Zustand noch ein:

::

  # chmod 1777 /tmp 

durchführen.

Siehe auch
----------

-  `Backup auf CD-ROM <backup_auf_cd-r>`__

* :ref:`genindex`

Zuletzt geändert: |date| 

