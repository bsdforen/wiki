Manuelle Verwaltung von Festplatten nach neuem GUID Partition Table (GPT)
=========================================================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Im Gegensatz zu dem Artikel `festplatte_einrichten <festplatte_einrichten>`__
werden hier keine klassischen MBR-Partitionen aka (klassisches BIOS) bzw.
BSD-Slices sondern moderne GPT-Partitionen manuell angelegt, wie sie z.B. das
moderne EFI verwenden kann.

Solange nicht von dieser Festplatte gebootet werden muss, kann auch ohne
EFI eine solche Partition angelegt werden, damit z.B.
Größenbeschränkungen der alten MBR-Partitionen umgangen werden können.

FreeBSD unterstützt ab 7.0 dieses Partitionsschema.

GPT-Partitionstabelle anlegen
-----------------------------

In dem Beispiel wird auf der Festplatte ``/dev/da1`` eine
GPT-Partitionstabelle wie folgt erstellt.

.. code:: bash

   gpart create -s gpt da1

Partition darin anlegen
-----------------------

Eine Partition mit der Größe der gesamten Platte anlegen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Auf der Festplatte wird nun eine Partition mit der vollen Größe der
Festplatte angelegt:

.. code:: bash

   gpart add -t efi da1

Die neue Partition wird als Device **``/dev/da1p1``** sichtbar.

Mehrere Partitionen mit vorgegebenen Größe anlegen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

   gpart add -t efi -s 300G da1

Erzeugt eine 300 Gigabyte Partition als ``/dev/da1**p1**`` auf leerer Platte.

.. code:: bash

   gpart add -t efi -s 500G da1

Erzeugt eine zweite 500 Gigabyte Partition als ``/dev/da1**p2**`` im zweiten
Schritt.

Dateisystem erstellen usw.
--------------------------

Das erstellen von Dateisystemen in dieser Partition ist alten
klassischem Artikel
`festplatte_einrichten#dateisystem_erstellen <festplatte_einrichten#dateisystem_erstellen>`__
beschrieben.

Anzeigen der Paritionstabelle
-----------------------------

.. code:: bash

   gpart show da1

oder

.. code:: bash

   gpart list da1

Partition löschen
-----------------

.. code:: bash

   gpart delete -i 1 da1

Hierbei ist die **1** der Index in der Ausgabe *Providers* in der Liste
aus ``gpart list ...``.

Bei **1** wird dann ``/dev/da1**p1**`` gelöscht.

Ändern der Partitionsgröße
--------------------------

FIXME

.. code:: bash

   gpart modify -i ...

Hinweis: Funktionierte unter FreeBSD 7.3 noch nicht.

* :ref:`genindex`

Zuletzt geändert: |date|

