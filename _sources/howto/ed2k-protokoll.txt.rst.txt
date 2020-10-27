ed2k-Protokoll beim Mozilla, Seamonkey und Firefox
==================================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Hier wird beschrieben, wie man dem Browser beibringen kann, ed2k-Links
an `aMule </Anwendungen/aMule>`__ zu liefern.

-  In der URL-/Adresszeile ``'about:config``' eingeben.
-  Einen Rechtsklick in der Liste machen und ``'Neu/New``' auswählen.
-  Als Typ ``'Boolean``' auswählen und beim
   ``'Eigenschaftsnamen/Preference Name``' folgendes
   ``'network.protocol-handler.external.ed2k``' und als ``'Wert/Value``'
   dann ``'true``' eingeben.
-  Jetzt erneut in der Liste einen Rechtsklick machen und ``'Neu/New``'
   anwählen.
-  Diesmal als Typen ``'String``' auswählen und als
   ``'Eigenschaftsnamen/Preference Name``' dann
   ``'network.protocol-handler.app.ed2k``' eingeben.
-  Als ``'Wert/Value``' dann den Pfad zum ed2k-Programm eingeben. Dies
   wäre zum Beispiel ``'/usr/local/bin/ed2k``'.
-  Fertig. Klickt man nun einen ed2k-Link an, wird dieser von aMule
   übernommen.

* :ref:`genindex`

Zuletzt geändert: |date|

