Majordomo Postfix
=================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Diese Anleitung beschreibt die Installation des Mailinglisten-Managers
`Majordomo <http://www.greatcircle.com/majordomo/>`__ auf einem mit Postfix
betriebenen Mailserver.

Installation von Majordomo
--------------------------

.. note::

  Die Installation wurde auf einem OpenBSD-Server getestet und sie wird auch so
  beschrieben. Prinzipiell sollte sie auf anderen BSD-Systemen genauso
  ablaufen, evtl. sind einige Pfadangaben nicht so vorzufinden und müssen
  angepasst werden.

Majordomo ist nicht als Package verfügbar, daher wird es ueber die Ports
installiert.

::

   # cd /usr/ports/mail/majordomo
   # make install clean

Dies sollte ohne Probleme und Fehlermeldungen den Port installieren.

Konfiguration von Majordomo
---------------------------

Für eine einfache Standardinstallation muss in der majordomo.cf nur ein
Parameter verändert werden.

::

   # vi /etc/majordomo/majordomo.cf

::

   $whereami = "example.com";

$whereami ist so anzupassen, dass es den Hostnamen der Maschine
verwendet.

Konfiguration von Postfix
-------------------------

Um das Beispiel und die Konfiguration recht einfach zu halten und zu
erklären wird mit Hash-Maps gearbeitet. Es ist dementsprechend auch
möglich alle Daten in eine Datenbank, wie MySQL oder auch in einem
Verzeichnissdienst wie OpenLDAP zu speichern.

In der ``main.cf`` von Postfix müssen nur zwei Parameter angepasst
werden, *virtual_alias_maps* und *alias_maps*. Mit virtual maps und
virtual mailboxes ist es nicht möglich Mailinglisten-Manager zu nutzen,
daher muss der Umweg über die ``alias_maps`` gemacht werden.

::

   # vi /etc/postfix/main.cf

::

   virtual_alias_maps = ..., hash:/etc/postfix/majordomo_virtual_aliases
   alias_maps = ..., hash:/etc/postfix/majordomo_aliases

Die Grundeinstellung für Majordomo werden in den Hash-Maps vorgenommen.

::

   # vi /etc/postfix/majordomo_virtual_aliases

::

   majordomo@example.com             majordomo
   majordomo-owner@example.com       mailinglistenmanager@example.com
   owner-majordomo@example.com       mailinglistenmanager@example.com

::

   # vi /etc/postfix/majordomo_aliases

::

   majordomo                         "|/usr/local/lib/majordomo/wrapper majordomo"

*majordomo-owner@example.com* und *owner-majordomo@example.com* leiten
wir an die Email-Adresse des Mailinglisten-Managers weiter. Also die
Person, die die Mailinglisten-Server betreut. *majordomo@example.com*
wird an majordomo in der Map majordomo_aliases weitergeleitet. Dort wird
der wrapper dann ausgeführt.

Mit postmap werden schließlich die Hash-Maps erstellt.

::

   # /usr/local/sbin/postmap hash:/etc/postfix/majordomo_virtual_aliases
   # /usr/local/sbin/postmap hash:/etc/postfix/majordomo_aliases

Dies erstellt die Dateien ``/etc/postfix/majordomo_virtual_aliaeses.db``
und ``/etc/postfix/majordomo_aliases``, die von Postfix gelesen werden
können.

Jetzt muss noch nur die Konfiguration von Postfix neugeladen werden dies
kann mit einem

::

   # /usr/local/sbin/postfix reload

geschehen.

Anmerkung zu OpenBSD
~~~~~~~~~~~~~~~~~~~~

Da Postfix in der Standardkonfiguration unter OpenBSD unter den
Privilegien, des Users nobody laufen, müssen die Permissions des
wrappers angepasst werden. Dies kann mit einem

::

   # chmod 666 /usr/local/lib/majordomo/wrapper

durchgeführt werden.

Damit ist die Installation von Majordomo abgeschlossen und man kann zum
Testen eine Mail an majordomo@example.com schicken, dieser sollte einen
jetzt eine Antwort zurück senden.

Anlegen einer Mailingliste
--------------------------

Um eine Mailingliste anzulegen müssen lediglich die beiden Hash-Maps
erweitert und zwei Dateien im Listenverzeichniss angelegt werden. In
einer Datei werden alle Listenmitgleider gespeichert und die zweiten
enthält die Beschreibung der Liste.

Als erstes wird eine leere Datei im Listenverzeichniss erstellt. Der
Name dieser Datei legt auch gleichzeitig den Namen der Mailingliste
fest. In diesem Beispiel soll die Liste foo heissen und wird über
*foo@example.com* zu erreichen sein.

::

   # touch /var/spool/majordomo/lists/foo

Danach folgt die Datei mit der Listenbeschreibung

::

   # vi /var/spool/majordomo/lists/foo.info

::

   Dies ist eine Test-Mailingliste.

::

   # vi /etc/postfix/majordomo_virtual_aliases

::

   foo@example.com             foo
   foo-request@example.com     foo-request
   foo-list@example.com        foo-list
   foo-owner@example.com       betreuer@example.com
   owner-foo@example.com       betreuer@example.com

Dies ist die Standardkonfiguration für eine Mailingliste ohne Digest und
Archiv. foo-owner und owner-foo werden an den Betreuer/Moderator der
Mailingliste weitergeleitet. Der Rest wird an die passenden Einträge von
majordomo_aliases weitergeleitet.

::

   # vi /etc/postfix/majordomo_aliaes

::

   foo                   "|/usr/local/lib/majordomo/wrapper resend -l foo foo-list"
   foo-request           :include:/var/spool/majordomo/lists/foo
   foo-list              "|/usr/local/lib/majordomo/wrapper -l foo"

Um bei ausgehenden Mails majordomo Kommandos abzufangen wird diese
Konfiguration gewählt.

Abschliessend müssen noch die Hash-Maps aktualisiert werden.

::

   # /usr/local/sbin/postmap hash:/etc/postfix/majordomo_virtual_aliases
   # /usr/local/sbin/postmap hash:/etc/posftix/majordomo_aliases

Um sich jetzt zu subscriben muss eine Mail an majordomo@example.com mit
dem Text *subscribe foo* im body geschickt werden.

Weitere Einstellungen zur Mailingliste kann man unter
``/var/spool/majordomo/lists/foo.config`` vornehmen.

Majordomo Kommandos
-------------------

Informationen ueber weitere/alle Majordomo Kommandos findet man unter:

-  http://lims.taratec.com/majordomo.html
-  http://lists.uoregon.edu/mdhelp.html
-  http://www.educationaldevelopment.net/elt2/majrdomo.htm

* :ref:`genindex`

Zuletzt geändert: |date|

