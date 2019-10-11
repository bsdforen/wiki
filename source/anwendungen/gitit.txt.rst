gitit
=====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

gitit ist ein schön einfaches `Wiki <Wiki>`__. Es basiert auf die
Anwendung von `haskell <haskell>`__.

Klassischer Weise wird `git <git>`__ und markdown angewendet.

gitit verfolgt den Ansatz standardmäßige Programme zur
Versionsverwaltung und einfache standardmäßige Auszeichnungssprache zur
Grundlage zu machen. Somit ist es vergleichbar mit `ikiwiki <ikiwiki>`__
(`perl <perl>`__) und `gollum <gollum>`__ (`ruby <ruby>`__).

Installation
------------

Nachfolgende Installation wurde in einer "geklickten" `Jail <Jail>`__
(`FreeBSD <FreeBSD>`__) bei einem aktuellen (2017-09-11)
`FreeNAS <FreeNAS>`__ 11 erfolgreich vorgenommen.

--------------

optionales pauschales Aktualisieren der bereits installierten Pakete

.. code:: bash

   pkg upgrade -y

--------------

.. code:: bash

   pkg install hs-gitit -y

optionales pauschales Erstellen eines Verzeichnisses, wo die für die
Dateien zum Betrieb abgelegt werden (Nachfolgend ist das beispielsweise
das Verzeichnis ``/usr/local/www/gitit/``.)

.. code:: bash

   mkdir -p /usr/local/www/gitit ; cd /usr/local/www/gitit/

erstmaliges Starten des Dienstes

.. code:: bash

   cd /usr/local/www/gitit/ && gitit

.. code:: bash

   Could not read mime types file: /etc/mime.types
   /etc/mime.types: openFile: does not exist (No such file or directory)
   Using defaults instead.

.. code:: bash

   Created repository in wikidata
   Added Front Page.page to repository
   Added Help.page to repository
   Added Gitit User’s Guide.page to repository
   Created static/css/custom.css
   Created static/img/logo.png
   Created templates/footer.st

Beenden des Dienstes

   durch beliebiges Abbrechen des Prozesses

.. code:: bash

   ^C

Erzeugen einer beliebig bezeichneten Datei für die Konfiguration von
gitit, welche "ordnungsgemäß" optional in einem beliebigen Verzeichnis
abgelegt wird (Nachfolgend ist das beispielsweise die Datei
``gitit.conf``. Nachfolgend ist das beispielsweise das Verzeichnis
``/usr/local/etc/``.)

.. code:: bash

   gitit --print-default-config > /usr/local/etc/gitit.conf

Vornehmen von Anpassungen in der Datei für die Konfiguration von gitit

.. code:: bash

   $EDITOR /usr/local/etc/gitit.conf

(optional) berichtigendes Anpassen in der Datei für die Konfiguration
von gitit für den Ort der Ablage der Datei ``mime.types``

.. code:: bash

   $EDITOR /usr/local/etc/gitit.conf

.. code:: bash

   #mime-types-file: /etc/mime.types           
   mime-types-file: /usr/local/share/cups/mime/mime.types

(optional berichtigendes) Anpassen in der Datei für die Konfiguration
von gitit für das Bereitstellen am üblichen Port für http, ohne einen
zusätzliche Dienst als web server

.. code:: bash

   $EDITOR /usr/local/etc/gitit.conf

.. code:: bash

   #port: 5001
   port: 80

(erneutes) Starten des Dienstes

.. code:: bash

   cd /usr/local/www/gitit/ && gitit -f /usr/local/etc/gitit.conf

rc.d
----

Es gibt bisher (2017-09-12) kein Skript für rc.d (im "ordnungsgemäßen"
Verzeichnis ``/usr/local/etc/rc.d/``) nach der Installation.

einzeilig
~~~~~~~~~

hässlich, aber erst einmal was auf die Schnelle Vielleicht mag das wer richtig
richtig machen, also vielleicht gar gleich direkt für *rc.d*.

Erzeugen
^^^^^^^^

aka ``init``

.. code:: bash

   mkdir -p /usr/local/www/gitit && gitit --print-default-config > /usr/local/etc/gitit.conf && cd /usr/local/www/gitit/ && gitit -f /usr/local/etc/gitit.conf && killall gitit

Starten
^^^^^^^

aka ``start``

.. code:: bash

   cd /usr/local/www/gitit/ && gitit -f /usr/local/etc/gitit.conf &

Stoppen
^^^^^^^

aka ``stop``

.. code:: bash

   killall gitit

Status
^^^^^^

aka ``status``

.. code:: bash

   ps aux | grep gitit | grep -v grep

* :ref:`genindex`

Zuletzt geändert: |date|

