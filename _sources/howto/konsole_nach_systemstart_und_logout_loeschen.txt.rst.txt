Konsole nach Systemstart und Logout löschen
===========================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Normalerweise bleiben nach dem Systemstart die Meldungen des Boot-Vorgangs auf
der ersten Konsole stehen und das Login-Prompt öffnet sich unterhalb der
Meldungen. Ebenso bleiben die letzten Eingaben des Benutzers sichtbar, wenn
sich dieser ausloggt. Zumindest das letztere Problem lässt sich durch Hacks
unsauber lösen. Im Folgenden wird ein einfacher und sauberer Weg über Getty
gezeigt, die Konsole bei jedem Erscheinen des Login-Prompts zu löschen.

Die folgende Anleitung wurde unter FreeBSD 6.1 erstellt, ist aber auch
mit allen älteren Versionen, beginnend mit 3.0, umsetzbar.

Man öffne die Datei ``/etc/gettytab`` in einem Editor und suche
folgenden Eintrag:

::

   P|Pc|Pc console:\
           :ht:np:sp#115200:

Bei FreeBSD 6.1 befindet er sich in Zeile 164, dies ist im letzten
Drittel der Datei. Man ändere den Eintrag auf Folgendes:

::

   P|Pc|Pc console:\
           :ht:np:sp#115200:\
           :cl=\E[H\E[2J:

Erklärung der Optionen:

-  cl -> „Screen Clear Sequenz“. Gibt an, welche Zeichenfolge genutzt
   werden soll, um den Inhalt des Terminals zu löschen.

-  \\E[H\E[2J -> Die Zeichenfolge, welche an das Terminal gesendet wird.
   Ein simples strg + l, wie es auch jederzeit von Hand ausgeführt
   werden kann.

Nachdem die Datei gespeichert wurde, sind die neuen Einstellungen sofort
wirksam. Ein Neustart von ``getty`` ist nicht erforderlich.

* :ref:`genindex`

Zuletzt geändert: |date|

