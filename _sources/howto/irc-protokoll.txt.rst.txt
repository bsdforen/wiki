irc-Protokoll im Mozilla, Seamonkey und Firefox
===============================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Hier wird beschrieben, wie man dem Browser beibringen kann, irc-Links an
`xChat </Anwendungen/xChat>`__ zu liefern. Das Ziel ist es aber auch,
dass sich xChat nicht immer neu Startet, sondern den Link im bestehenden
xChat öffnet.

xchat.sh
--------

Zuerst erstellen wir eine Datei mit dem Namen xchat.sh unter ~/bin:

::

   #!/bin/sh
   /usr/local/bin/xchat -e --url=$1

Einstellungen am Browser
------------------------

-  In der URL-/Adresszeile ``'about:config``' eingeben.
-  Einen Rechtsklick in der Liste machen und ``'Neu/New``' auswählen.
-  Als Typ ``'Boolean``' auswählen und beim
   ``'Eigenschaftsnamen/Preference Name``' folgendes
   ``'network.protocol-handler.external.irc``' und als ``'Wert/Value``'
   dann ``'true``' eingeben.
-  Jetzt erneut in der Liste einen Rechtsklick machen und ``'Neu/New``'
   anwählen.
-  Diesmal als Typen ``'String``' auswählen und als
   ``'Eigenschaftsnamen/Preference Name``' dann
   ``'network.protocol-handler.app.irc``' eingeben.
-  Als ``'Wert/Value``' dann ``'xchat.sh``'. ``'/home/user/bin``' muss
   in der PATH Variable sein!
-  Fertig. Klickt man nun einen irc-link an, wird dieser von xChat
   übernommen.

* :ref:`genindex`

Zuletzt geändert: |date|

