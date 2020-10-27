PostgreSQL Update auf einen neueren Zweig
=========================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel erklärt den `PostgreSQL </anwendungen/PostgreSQL>`__
Updatevorgang auf einen neueren Zweig.

FreeBSD
-------

Dieses HowTo setzt einen bereits installierten und laufenden PostgreSQL
Server voraus. Im Updateverlauf muss der Server vorübergehend
abgeschaltet werden, eine Downtime ist deshalb leider nicht zu
vermeiden.

In den folgenden Beispielen wird PostgreSQL vom 8.4er Zweig auf den
9.1er Zweig aktualisiert, die Versionsnummern ``84`` und ``91`` müssen
natürlich auf den eigenen Fall angepasst werden.

Versionen
~~~~~~~~~

Bei jedem Wechsel der "higher minor revision", also der zweiten
Versionsnummer muss, ein Backup der Datenbank gemacht werden um die
Daten nach dem Update wieder herzustellen.

-  Backup/Restore erfordert:

   -  8.3.x → 8.4.x
   -  8.4.x → 9.1.x

-  Backup/Restore nicht erfordert:

   -  8.4.8 → 8.4.9
   -  8.4.0 → 8.4.9

Deshalb gibt es für jeden Zweig unter FreeBSD eigene Ports. Der Vorteil
ist, dass dann bei regulären Updates nichts weiter zu beachten ist. Der
Nachteil ist, dass Zweigwechsel von einem veralteten Zweig manuell
erfolgen müssen.

Backup
~~~~~~

Am Anfang des Backups sollten ggf. alle Dienste/Anwendungen, die die
Datenbank verwenden beendet werden. Diese stören das Backup zwar nicht.
Aber Änderungen die nach/während dem Backup gemacht werden sind nach dem
Restore natürlich nicht mehr verfügbar.

Das Backup wird mit folgendem Kommando ausgeführt::

  # pg_dumpall -Upgsql > /usr/local/pgsql/data.84.dump

Danach sollte der Server gestoppt werden::

  # service postgresql stop

Das Standardverzeichnis für die Daten ist ``/usr/local/pgsql/data``, um
den Restore später durchzuführen muss es neu angelegt werden. Es ist
ratsam das Verzeichnis an diesem Punkt einfach umzubenennen::

  # mv /usr/local/pgsql/data /usr/local/pgsql/data.84

Update
~~~~~~

Jetzt kommt das Update der Server-Software. Beispiele sind für
`portmaster </anwendungen/portmaster>`__ und
`pkg_upgrade </anwendungen/pkg_upgrade>`__. Die Parameter für
`portupgrade </anwendungen/portupgrade>`__ sind in diesem Fall identisch
mit denen für ``portmaster``. Auch ``pkg_upgrade`` funktioniert mit den
gleichen Parametern, das Beispiel nutzt jedoch eine kompaktere
Schreibweise.

Update mit ``portmaster``::

  # portmaster -o databases/postgresql91-client databases/postgresql84-client ... 
  # portmaster -o databases/postgresql91-server databases/postgresql84-server

Upgrade mit ``pkg_upgrade``::

  # pkg_upgrade -C databases/postgresql91-server

Wiederherstellung
~~~~~~~~~~~~~~~~~

Nach erfolgtem Software-Update kann die Datenbank nun wieder hergestellt
werden. Als erstes muss die Datenbank dazu neu initialisiert werden:

::

  # service postgresql initdb

Nun kann das Backup wieder importiert werden::

  # service postgresql start 
  # psql -U pgsql -f /usr/local/pgsql/data.84.dump postgres

Ab diesem Punkt ist die Datenbank wieder voll einsatzbereit. Dienste und
Anwendungen, die auf die Datenbank zugreifen können jetzt wieder
gestartet werden.

Aufräumarbeiten
~~~~~~~~~~~~~~~

Wenn alles wie erhofft funktioniert können die alten Daten, also
``data.84`` und ``data.84.dump`` gelöscht werden.

Verweise
--------

-  `databases/postgresql91-server <https://www.google.com/search?q=databases/postgresql91-server&btnI=lucky>`__
   in den FreeBSD Ports
-  Die `PostgreSQL Homepage <http://www.postgresql.org/>`__
-  Die **service(8)** Manpage
-  Die **pg_dumpall(1)** Manpage
-  Die **psql(1)** Manpage

* :ref:`genindex`

Zuletzt geändert: |date|

