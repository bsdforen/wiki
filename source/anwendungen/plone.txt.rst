Plone 5
=======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Installation Plone 5
--------------------

Aktuell (2015-10-22) ist bei `FreeBSD <FreeBSD>`__ noch die bisherige
Version `#Plone 4 <#Plone 4>`__ in den Ports. Dennoch kann "zum Spielen"
die neue Version Plone 5 installiert werden.

   https://wiki.stura.htw-dresden.de/index.php/Diskussion:Server/Plone#Installation_Plone_5

Nachfolgende Installation wurde in einer "geklickten" `Jail <Jail>`__
(`FreeBSD <FreeBSD>`__) bei einem aktuellen (2015-10-22)
`FreeNAS <FreeNAS>`__ 9.3 erfolgreich vorgenommen.

--------------

optionales pauschales Aktualisieren der bereits installierten Pakete

.. code:: bash

   pkg upgrade -y

--------------

optionales Erstellen eines Verzeichnisses, wo die für die Dateien zur
Installation abgelegt werden (Nachfolgend ist das beispielsweise das
Verzeichnis ``/usr/local/install/plone``.)

.. code:: bash

   mkdir -p /usr/local/install/plone ; cd /usr/local/install/plone

Herunterladen und entpacken vom sogenannten ``UnifiedInstaller``

.. code:: bash

   fetch http://launchpad.net/plone/5.0/5.0/+download/Plone-5.0-UnifiedInstaller.tgz

.. code:: bash

   tar -xvf Plone-5.0-UnifiedInstaller.tgz 

--------------

optionales Prüfen der Verfügbarkeit von Bibliotheken, `die Plone
benötigt <http://docs.plone.org/manage/installing/requirements.html#libraries>`__

.. code:: bash

   ls /usr/lib/libz.so ; ls /usr/local/lib/libexpat.so ; ls /usr/lib/libssl.so

Installieren der noch fehlenden Bibliotheken, `die Plone
benötigt <http://docs.plone.org/manage/installing/requirements.html#libraries>`__

.. code:: bash

   pkg install -y jpeg readline libxml2 libxslt

--------------

Erstellen eines Verzeichnisses, wo die Instanz für Plone installiert
wird (Nachfolgend ist das beispielsweise das Verzeichnis
``/usr/local/www/plone``.)

.. code:: bash

   mkdir -p /usr/local/www/plone

Installieren von anscheinend für den sogenannten ``UnifiedInstaller``
doch noch benötigte Programme

.. code:: bash

   pkg install -y sudo 

Wechseln in das Verzeichnis vom sogenannten ``UnifiedInstaller``

.. code:: bash

   cd /usr/local/install/plone/Plone-5.0-UnifiedInstaller 

Installieren von Plone in das dafür erstellte Verzeichnis
(Vorangegangenen wurde beispielsweise das Verzeichnis
``/usr/local/www/plone`` verwendet.)

.. code:: bash

   ./install.sh --target=/usr/local/www/plone standalone

Bei der Installation von Plone wird, da dazu keine Parameter übergeben
wurden, ein administrativer Account für Zope (Rechte der Gruppe
``Manager``) mit dem Namen ``admin`` und einem zufälligen Passwort
erstellt. Die Zugangsdaten (Name und Passwort) werden bei der
Installation ausgegeben.

Anzeigen lassen der der Datei mit den Zugangsdaten vom administrativen
Account für Zope (Rechte der Gruppe ``Manager``) (Vorangegangenen wurde
beispielsweise das Verzeichnis ``/usr/local/www/plone`` verwendet.)

.. code:: bash

   cat /usr/local/www/plone/zinstance/adminPassword.txt

--------------

Wechseln in das Verzeichnis, wo die Instanz für Plone installiert wurde
(Vorangegangenen wurde beispielsweise das Verzeichnis
``/usr/local/www/plone`` verwendet.)

.. code:: bash

   cd /usr/local/www/plone/zinstance

Starten der Instanz Plone

.. code:: bash

   ./bin/plonectl start

Stoppen der Instanz Plone

.. code:: bash

   ./bin/plonectl stop

Status der Instanz Plone anzeigen lassen

.. code:: bash

   ./bin/plonectl status

rc.d Plone 5
------------

   https://wiki.stura.htw-dresden.de/index.php/Server/Jails/SRS18#rc.d_scripting_Plone

.. code:: bash

   $EDITOR /usr/local/etc/rc.d/plone

.. code:: bash

   #!/bin/sh
   # PROVIDE: plone
   # REQUIRE: LOGIN
   # KEYWORD: shutdown

   . /etc/rc.subr

   name="plone"
   rcvar=plone_enable

   start_cmd="${name}_start"
   stop_cmd="${name}_stop"
   restart_cmd="${name}_restart"
   status_cmd="${name}_status"

   extra_commands="status"

   load_rc_config ${name}
   #: ${plone_enable:="NO"}

   plone_stop()
   {
       /usr/local/www/plone/zinstance/bin/plonectl stop
   }

   plone_status()
   {
       /usr/local/www/plone/zinstance/bin/plonectl status
   }

   plone_start()
   {
         /usr/local/www/plone/zinstance/bin/plonectl start
   }

   plone_restart()
   {
       /usr/local/www/plone/zinstance/bin/plonectl restart
   }

   run_rc_command "$1"

.. code:: bash

   chmod 540 /usr/local/etc/rc.d/plone

.. code:: bash

   service plone onestart

.. code:: bash

   echo 'plone_enable="YES"' >> /etc/rc.conf

.. code:: bash

   service plone start

.. code:: bash

   service plone status

Siehe auch
----------

-  https://wiki.stura.htw-dresden.de/index.php/Server/Plone

* :ref:`genindex`

Zuletzt geändert: |date|

