portupgrade
===========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Portupgrade ist die wahrscheinlich am weitesten verbreitete Anwendung, um unter
FreeBSD Ports und Pakete zu aktualisieren. Dieser Artikel gibt nur kurze
Beispiele, der tatsächliche Umfang von Portupgrade ist deutlich größer. Die
Portupgrade Manpage **portupgrade(1)** ist ein guter Ausgangspunkt, um die
Materie zu vertiefen.

Verwandte Artikel
-----------------

Wer sich für diesen Artikel interessiert, sollte gegebenenfalls auch die
folgenden Artikel lesen:

-  `bsdadminscripts <bsdadminscripts>`__ sind eine Skriptsammlung, die
   unter anderem dabei hilft Ports mit **make -jN** zu kompilieren.
-  `distcc <distcc>`__ erlaubt das verteilte Kompilieren von C/C++
   Programmen auf mehreren Rechnern.
-  `Paketsysteme </howto/Paketsysteme>`__ ist ein allgemeiner Artikel zu
   den BSD-Paketsystemen.
-  `pkg_cutleaves <pkg_cutleaves>`__ ist ein Programm, das Pakete
   listet, die keine Abhängigkeiten von anderen Paketen sind und sie auf
   Wunsch löscht.
-  `portsnap <portsnap>`__ ist ein Programm, um den Ports-Tree zu
   aktualisieren.

Programme installieren
----------------------

Um mit Portupgrade Programme zu installieren, reicht ein einfaches

::

   # portinstall PROGRAMM

wobei **PROGRAMM** entweder der Name des Programms, oder ein Pfad in den
Ports, zum Beispiel **www/firefox**, sein kann.

::

   # portinstall -P PROGRAMM

Dieser Befehl versucht eine Paketinstallation. Nur wenn kein aktuelles
Paket verfügbar ist, fällt Portupgrade auf das Selbstkompilieren zurück.

::

   # portinstall -PP PROGRAMM

Mit diesem Befehl wird das Programm nur installiert, wenn ein aktuelles
Paket vorhanden ist.

Programme aktualisieren
-----------------------

Äquivalent zu Portinstall können auch bei Updates die Parameter **-P**
oder **-PP** verwendet werden.

::

   # portupgrade PROGRAMM

Mit diesem Befehl wird das angegebene Programm auf den Stand der lokalen
Ports-Kollektion gebracht. Manche Programme installieren
Shared-Libraries, die beim Update auf eine neue Version gelöscht werden.
Diese Libraries sichert Portupgrade in **/usr/local/lib/compat/pkg**.
Auf diese Weise wird sichergestellt, dass Programme die mit diesen alten
Versionen verlinkt wurden, weiterhin funktionieren.

Dieses Verhalten kann mit dem Parameter **-u** unterbunden werden. Das
macht dann allerdings eine intensivere Pflege von **/etc/libmap.conf**
notwendig. Manchmal müssen dann sogar mit

::

   # portupgrade -fur PROGRAMM

alle Abhängigkeiten neu gebaut werden. Wer immer diese Variante wählt,
geht auf Nummer Sicher. Der Parameter **-f** erzwingt den Neubau auch
dann, wenn ein Programm nicht veraltet ist.

Das Hinterlegen alter Libraries in **/usr/local/lib/compat/pkg** ist
eine zuverlässige und äußerst praktische Lösung. Leider kommt es
gelegentlich vor, dass Libraries Sicherheitslücken haben. Das sollte
vorher mit `portaudit <portaudit>`__ geprüft werden und betroffene
Pakete sollten dann auf jeden Fall mit dem Parameter **-u** aktualisiert
werden.

Um für Konsistenz in den Paketen zu sorgen, werden Programme
normalerweise mit dem Befehl

::

   # portupgrade -rR PROGRAMM

aktualisiert. Der Parameter **-r** sorgt dafür, dass abhängige Pakete
auch alle auf den neusten Stand gebracht werden, **-R** sorgt dafür,
dass Pakte von denen **PROGRAMM** abhängt, ebenfalls aktualisiert
werden.

Portupgrade ermöglicht es auch alle Pakete zu aktualisieren.

::

   # portupgrade -a

Die Parameter **-r** und **-R** sind hier unnötig, da mit dem Parameter
**-a** Pakete sowieso in der Reihenfolge der Abhängigkeiten aktualisiert
werden.

Portupgrade kann auch eine Liste der zu aktualisierenden Pakete
anzeigen, ohne die Aktualisierung direkt gleich vorzunehmen.

::

   # portupgrade -an

Programme deinstallieren
------------------------

Zur Deinstallation ist pkg_deinstall die präferierte Methode und einem
**make deinstall** oder **pkg_delete** vorzuziehen.

::

   # pkg_deinstall PROGRAMM

Um auch Pakete zu deinstallieren, die als Abhängigkeiten mitinstalliert
wurden, lässt sich der Parameter **-R** verwenden.

::

   # pkg_deinstall -R PROGRAMM

Pakete, die auch Abhängigkeiten von anderen sind, werden dabei nicht
deinstalliert. Auch Build-Dependencies (Pakete, die nur zum kompilieren,
nicht zum verwenden, benötigt werden) werden nicht angetastet.

Verweise
--------

-  Die Portupgrade `Homepage <http://wiki.freebsd.org/portupgrade>`__.
-  `ports-mgmt/portupgrade <https://www.google.com/search?q=ports-mgmt/portupgrade&btnI=lucky>`__
   in den FreeBSD Ports.
-  `portmaster </anwendungen/portmaster>`__ - Alternative zu portupgrade

* :ref:`genindex`

Zuletzt geändert: |date|

