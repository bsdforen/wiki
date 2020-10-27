Autologin auf der Konsole
=========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-netbsd.png

Ein automatsches Einloggen auf der Konsole wird sicher für den ein oder
anderen wünschenswert sein und ist mit wenigen Änderungen am System vom
Administratorkonto aus einzurichten:

Mit einem Editor die Datei /etc/gettytab öffnen und an ihrem Ende
folgendes hinzufügen:

::

   benutzername:\
   :al=benutzername:ht:np:sp#115200:

wobei das Wort "benutzername" natürlich durch den tatsächlichen
Benutzernamen, der automatisch eingeloggt werden soll zu ersetzen ist.

Als zweiter Schritt ist die Datei /etc/ttys zu öffnen. Wenn das
automatische Einloggen auf der ersten Konsole erfolgen soll muss in
diesem Fall normalerweise die erste nicht mit einer # auskommentierte
Zeile folgendermassen angepasst werden:

Bei NetBSD ändert man die Zeile

::

   console "/usr/libexec/getty Pc"         vt100   on  secure

dann in

::

   console "/usr/libexec/getty benutzername"         vt100   on  secure

Bei FreeBSD ändert man die Zeile

::

   ttyv0 "/usr/libexec/getty PC"   cons25  on     secure

dann in

::

   ttyv0 "/usr/libexec/getty benutzername"   cons25  on     secure

Nach einem Neustart des Rechners sollte "benutzername" auf der ersten
Konsole eingeloggt sein.

* :ref:`genindex`

Zuletzt geändert: |date| 

