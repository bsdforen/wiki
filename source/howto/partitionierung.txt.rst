Festplattenpartitionierung unter FreeBSD
========================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Ausgangsituation
----------------

Sie haben eine neue leere Festplatte, die sie ohne Verwendung von
``bsdinstall`` in das System einhängen wollen. Diese wurde in diesem
**Beispiel** z.B. als */dev/ad3* erkannt, was sich über ``dmesg``
herausfinden läßt.

Lösung
------

Folgende Schritte sind dazu notwendig:

-  **ggf. alte Daten auf der Festplatte löschen (Vorsicht!):**

``dd if=/dev/zero of=/dev/ad3 bs=1k count=1``

-  **Zunächst eine Partitionstabelle erstellen (in diesem Fall GPT)**

nicht bootbar

::

   gpart create -s GPT ad3

-  **Darin eine Partition erstellen**

Die Partition nimmt den ganzen Platz auf der Platte ein.

::

   gpart add -a 1m -t freebsd-ufs ad3

-  **Dateisystem erstellen**

Dabei wird die Partition mit einem UFS2-Dateisystem und einem Label
(Namen) versehen um sie später wieder eindeutig zuordnen zu können. In
diesem Beispiel ist der Name *EINEPARTITION*.

::

   newfs -L EINEPARTITION /dev/ad3p1 

-  **Softupdates ggf. einschalten**

::

   tunefs -n enable /dev/ad3p1

\* **ggf. temporär nach /mnt mounten**

::

   mount /dev/ad3p1 /mnt

Dauerhaft kann die Partition über einen neuen Eintrag in der /etc/fstab
gemountet werden. Die hier dargestellten Schritte erzeugen eine
Partition über die gesamte Platte, die nicht bootbar ist. Wenn eine
bootbare Partition benötigt wird, so findet sich im FreeBSD-Handbuch ein
Beispiel dazu.

* :ref:`genindex`

Zuletzt geändert: |date|

