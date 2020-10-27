Tor (OpenBSD)
=============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

Tor ist ein Programm, mit dem man z.B. im Web oder im IRC anonym unterwegs sein
kann. In diesem Artikel wird erklärt, wie man Tor unter OpenBSD aufsetzt.

Tor allgemein
-------------

Installieren
~~~~~~~~~~~~

Tor kann einfach per pkg_add installiert werden:

::

   # pkg_add tor

Tor beim booten starten
~~~~~~~~~~~~~~~~~~~~~~~

Dazu muss die **/etc/rc.local** editiert werden:

::

   if [ -x /usr/local/bin/tor ];
   then
       echo -n ' tor';
       /usr/local/bin/tor -f /etc/tor/torrc
   fi

Konfigurieren
~~~~~~~~~~~~~

<note> Folgende Schritte müssen bis einschlieslich OpenBSD 4.0
unternommen werden. Bei nachfolgenden Versionen ist die torrc schon
vorkonfiguriert! </note>

Damit Tor beim booten in den Hintergrund *forked*, muss Folgendes in der
**/etc/tor/torrc** aktiviert werden:

::

   RunAsDaemon 1

Als nächstes muss ein Verzeichnis für Tors Daten in der **torrc**
festgelegt werden:

::

   DataDirectory /var/spool/tor

Dann muss dieses Verzeichnis noch erstellt und dem **Benutzer \_tor**
müssen die Rechte dafür gegeben werden:

::

   # mkdir /var/spool/tor
   # chmod 0700 /var/spool/tor
   # chown _tor:_tor /var/spool/tor

Tor als Client
--------------

Mit den vorrangegangen Konfigurationsschritten kann Tor als Client
benutzt werden. Um z.B. anonym mit `SILC </anwendungen/SILC>`__
unterwegs zu sein, brauchen wir vorher noch das Packet **dsocks**:

::

   # pkg_add dsocks

Wenn wir SILC nun wie folgt starten:

::

   $ dsocks-torify.sh silc

wird SILC über Tor eine Verbindung zum SILC Server aufbauen.

Verweise
--------

-  `Tor Homepage <http://www.tor.eff.org>`__
-  `Tor Wiki mit vielen Anleitungen und Infos zu
   Tor <http://wiki.noreply.org/noreply/TheOnionRouter/>`__
-  `Thread auf ports@ zum Thema
   DataDirectory <http://marc.theaimsgroup.com/?t=115757341400001&r=1&w=2>`__

* :ref:`genindex`

Zuletzt geändert: |date|

