portsnap
========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

*portsnap* dient zum effizienten Aktualisieren des Portstrees (gewöhnlich unter
**/usr/ports**). Seit 6.0-RELEASE ist das Programm *portsnap* Teil des
Basissystems von FreeBSD, in früheren Version kann es aus den Ports installiert
werden.

Einzelplatz
-----------

Portsnap ist denkbar einfach zu bedienen, beim ersten mal genügt der
Befehl

::

   # portsnap fetch extract

um einen aktuellen Portstree nach **/usr/ports** zu entpacken. Vorher
sollte man sicherstellen, dass das Verzeichnis **/usr/ports** existiert
und **leer** ist.

Jedes weitere Update erfolgt mit einem

::

   # cd /usr/ports && make update

oder

::

   # portsnap fetch update

oder auch von cron aus

::

   portsnap cron update

*portsnap* sorgt automatisch für einen aktuellen und passenden INDEX.

Mehrplatz
---------

Da *portsnap* das http-Protokoll verwendet, kann ein gewöhnlicher
http-Proxy mit Caching verwendet werden, um mehrere Rechner mit
*portsnap* zu versorgen. Dies stellt laut dem Entwickler (Colin
Percival) die effizienteste Methode dar, ein komplettes Spiegeln der
Daten ist somit unnötig. Um *portsnap* einen Proxy benutzen zu lassen,
setzt man die Umgebungsvariable **HTTP_PROXY** auf den gewünschten
(lokalen) Proxy.

* :ref:`genindex`

Zuletzt geändert: |date|

