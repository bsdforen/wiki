Pakete deinstallieren
=====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Programme sauber zu deinstallieren ist wegen den Abhängigkeiten nicht
einfach. Bei der Programmdeinstallation ist man stets im Kampf gegen das
Abhängigkeitenwirrwar (Programm x benötigt Programm y, Programm y lässt
sich nicht deinstallieren, da von Programm z benötigt). Diese Anleitung
soll beim Kampf gegen das Abhängigkeitenwirrwar unter FreeBSD helfen.

Programme sauber deinstallieren
-------------------------------

Zwei Methoden für die saubere Programmdeinstallation sollen aufgezeigt
werden:

pkg_deinstall
~~~~~~~~~~~~~

Pkg_deinstall ist im Programmpaket
`ports-mgmt/portupgrade <https://www.google.com/search?q=ports-mgmt/portupgrade&btnI=lucky>`__
enthalten. Pkg_deinstall wird mit:

::

   # pkg_add -r portupgrade

installiert.

Anwendung
^^^^^^^^^

Mit:

::

   # pkg_deinstall -R <Programmname> 

wird das Programm <Programmname> deinstalliert. Die Option "-R" sorgt
dafür, dass zusätzlich alle Programme deinstalliert werden, die nach der
Deinstallation von <Programmname> nicht mehr benötigt werden.

Mit:

::

   # pkg_deinstall -Rn <Programmname>

kann ein Testlauf gestartet werden.

Mit:

::

   # pkg_deinstall --all

werden alle installierten Programme entfernt!

pkg_cutleaves
~~~~~~~~~~~~~

Pkg_cutleaves hat gegenüber pkg_deinstall den Vorteil, dass Sie bei der
Anwendung von pkg_cutleaves automatisch eine Softwareinventarliste
erhalten. Pkg_cutleaves wird mit:

::

   # pkg_add -r pkg_cutleaves

installiert.

Konfiguration
^^^^^^^^^^^^^

Beim Aufruf von:

::

   # pkg_cutleaves -l

werden Ihnen alle installierten Programme aufgelistet, die von keinem
anderen installierten Programm benötigt werden. Erstellen Sie unter
**/usr/local/etc/pkg_leaves.exclude** eine Liste aller Programme, welche
Sie behalten möchten, zum Beispiel:

::

   amarok
   bash
   cabextract

Entfernen Sie jegliche Programmversionsangaben aus der Datei
**/usr/local/etc/pkg_leaves.exclude**!

.. _anwendung-1:

Anwendung
^^^^^^^^^

Dann startet man pkg_cutleaves mit:

::

   # pkg_cutleaves -x

und schon wird nachgefragt, ob Programm xy, welches nicht in der Datei
**/usr/local/etc/pkg_leaves.exclude** aufgeführt wird und nicht von
einem anderen Programm benötigt wird, deinstalliert werden soll.

Praktisch ist dies, wenn man ein Programm nur zum Testen installieren
möchte. Nach dem (nicht überzeugenden) Testen des Programms wird mit:

::

   # pkg_cutleaves -x 

das Programm deinstalliert.

Softwareinventarliste
^^^^^^^^^^^^^^^^^^^^^

Wurde die Datei **/usr/local/etc/pkg_leaves.exclude** wie oben
beschrieben erstellt, so enthält **/usr/local/etc/pkg_leaves.exclude**
eine Inventarliste der aktuell installierten Programme.

Hinweise
--------

make deinstall
~~~~~~~~~~~~~~

Vermeiden Sie die Anwendung von:

::

   # cd /usr/ports/<Programmpfad>
   # make deinstall

aus folgenden Gründen:

make deinstall installiert ein Programm vorbehaltslos. Egal ob das zu
deinstallierende Programm von einem anderen Programm benötigt wird oder
nicht. Zum Beispiel führt:

::

   # cd /usr/ports/devel/gettext
   # make deinstall

dazu, dass nach der Deinstallation von GetText mit "make deinstall" ca.
70% aller Programme nicht mehr laufen!

Üblicherweise wird "make deinstall" eingesetzt, wenn nachher ein "make
install" folgt. Das Programm `portupgrade </anwendungen/portupgrade>`__
verwendet bei einer Programmaktualisierung die beiden Kommandos "make
deinstall" und "make install". Jedoch erstellt portupgrade vor der
Ausführung der beiden make-Kommandos ein Backup der installierten
Programmversion. Schlägt die Programmaktualisierung fehl, so wird das
Backup der alten Programmversion zurückgespielt.

.. note::

  Verzichten Sie aus diesem Grund auf die Verwendung von "make deinstall" und
  setzen Sie `portupgrade </anwendungen/portupgrade>`__ ein!

* :ref:`genindex`

Zuletzt geändert: |date|

