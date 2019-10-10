make world
==========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Diese Anleitung soll erläutern, wie man das Betriebssystem aktualisiert. Dies
wird im `FreeBSD-Handbuch
<http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/cutting-edge.html>`__
und in der Datei **/usr/src/UPDATING** genau beschrieben.

.. important::

  Besonders wichtig sind die letzten Seiten der Datei **/usr/src/UPDATING**, da
  je nach altem und neuem Betriebssystem verschiedene Schritte notwendig sind!

  Die Angaben in der /usr/src/UPDATING sind den abweichenden Angaben in dieser
  Anleitung oder den abweichenden Angaben `im FreeBSD-Handbuch unbedingt
  vorzuziehen
  <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/makeworld.html>`__!

  Für alle hier notwendigen Aktionen wird die root-Permission
  vorausgesetzt!

Vorbereitungen
--------------

Den Betriebssystem-Quellcode aktualisiert man mit ``csup``, welches ab
FreeBSD 6.2 im Basissystem integriert ist. Wird ein älteres
Betriebssystem einsetzt, muß ``csup`` als Package installiert werden:

::

  # pkg_add -r csup

Wenn csup nicht als Package verfügbar ist, müssen Sie auf CVSup
ausweichen (pkg_add -r cvsup-without-gui).

Da csup mit csvup kompatibel ist, erscheinen im Folgenden u.U.
cvsup-Angaben.

Der Betriebssystem-Quellcode befindet sich üblicherweise im Verzeichnis
**/usr/src**. Bevor wir csup einsetzen können, muss noch die
Konfigurationsdatei mit den Einstellungen für die Aktualisierung des
Verzeichnis **/usr/src** erstellt werden. 

::

  # cp /usr/share/examples/cvsup/standard-supfile /etc/source-supfile

in der Datei /etc/source-supfile die Zeile:

::

   *default host=CHANGE_THIS.FreeBSD.org

ändern auf (``'at``' für austria kann z.B. durch ``'de``' für einen
deutschen Server ersetzt werden, die 2 ist eine fortlaufende Nummer und
kann variieren):

::

   *default host=cvsup2.at.FreeBSD.org

Mit der "tag"-Zeile wird die gewünschte Betriebssystemversion angegeben.

Auf http://www.freebsd.org werden die [http://www.freebsd.org/releases/
aktuelle(n) Betriebssystemversion(en)] aufgelistet. Am 13.5.2006 wurde
FreeBSD 6.1:

::

   *default release=cvs tag=RELENG_6_1

und FreeBSD 5.4 empfohlen:

::

   *default release=cvs tag=RELENG_5_4

Jetzt ist die cvsup-Konfiguration abgeschlossen. Das Verzeichnis
/usr/src wird mit:

::

  # csup /etc/source-supfile

aktualisiert.

Hinweise zur Betriebssystem-Versionswahl
----------------------------------------

Bei allgemeinen Verständnisfragen zu den verschiedenen FreeBSD-Versionen
siehe bitte: `FreeBSD - FAQ <FreeBSD - FAQ>`__

Entwicklungsversionen
~~~~~~~~~~~~~~~~~~~~~

Die Verwendung von Nicht-Release-Betriebssystemversionen (STABLE,
CURRENT) wird nicht empfohlen! Für STABLE und CURRENT siehe bitte:
`http://www.freebsd.ch/doc/de_DE.ISO8859-1/books/handbook/current-stable.html FreeBSD Handbuch: CURRENT / STABLE <http://www.freebsd.ch/doc/de_DE.ISO8859-1/books/handbook/current-stable.html FreeBSD Handbuch: CURRENT / STABLE>`__

Versionsübersicht
~~~~~~~~~~~~~~~~~

Eine Übersicht über alle möglichen, verwendbaren Betriebssystemversionen
findet man unter: `FreeBSD Handbuch: CVS
Tags <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/cvs-tags.html>`__

Release-Tags
~~~~~~~~~~~~

Verwenden Sie keine Release-Tags. Der Tag "RELENG_6_0_RELEASE" enthält
im Gegensatz zu "RELENG_6_0" keine
Betriebssystemsicherheitslücken-"Flicken" (engl. Patches).

Betriebssystem aktualisieren
----------------------------

Einleitung
----------

Die Anleitung orientiert sich an den Angaben in der Datei
<tt>/usr/src/UPDATING</tt> für einen "kleinen" Betriebssystemwechsel,
zum Beispiel FreeBSD 6.0 -> 6.1. Für grosse Betriebssystemwechsel, also
FreeBSD 5.xy auf FreeBSD 6.xy, sind von dieser Anleitung abweichende
Schritte notwendig. Unbedingt in <tt>/usr/src/UPDATING</tt>
nachschlagen!

Backup
~~~~~~

``'make sure you have good level 0 dumps``'

Als erstes ist erstmal ein Backup angesagt, denn bei der
Betriebssystem-Aktualisierung kann man die FreeBSD-Installation wirklich
zerstören! Eine gute Backup-Anleitung findet man unter:
`Backup <Backup>`__

/usr/obj entfernen
~~~~~~~~~~~~~~~~~~

Das Entfernen von /usr/obj beschleunigt den ``make buildworld`` Prozess
und erspart einem Ärger mit Abhängigkeiten::

  # cd /usr/obj 
  # chflags -R noschg \* # rm -rf \*

make buildworld
~~~~~~~~~~~~~~~

Man starte die Betriebssystem-Kompilierung mit::

  # cd /usr/src 
  # make buildworld

Die Betriebssystem-Kompilierung dauert etwa zwischen 30 und 60 Minuten,
je nach Leistingsfähigkeit des PCs!

make kernel
~~~~~~~~~~~

Als nächstes muss der Kernel neugebaut werden. Dies dauert etwa 15
Minuten. GENERIC-Kernelbenutzer geben folgendes ein::

  # cd /usr/src 
  # make buildkernel 
  # make installkernel

Benutzer eines angepassten Kernels gehen bitte nach der Anleitung unter
`Kernel erstellen (FreeBSD) <Kernel erstellen (FreeBSD)>`__ vor.

Single-Usermode starten
~~~~~~~~~~~~~~~~~~~~~~~

.. note::

  Sollte man /usr per geli oder gbde verschlüsselt haben sollte man jetzt
  **nicht** neustarten. In diesem Fall können dann nämlich Userland und
  Kernelteile der Verschlüsselung "out-of-sync" werden, welches ein
  entschlüsseln und einbinden von /usr verhindert, welches wiederum das ``make
  installworld`` verhindert.

``'reboot in single user``'

Starten Sie den Computer im Single-Usermode und hängen Sie die
Partitionen mit Lese-/Schreibzugriff ein, wie im Kapitel 19.4.5 des
FreeBSD-Handbuchs beschrieben wird:
[http://www.freebsd.ch/doc/de_DE.ISO8859-1/books/handbook/makeworld.html
FreeBSD Handbuch: Make world]

Kurzgefasst::

  # shutdown -r now 

Nach dem Hochfahren den Single-Usermode wählen (# boot -s). 

::

  # /sbin/fsck -p 
  # /sbin/mount -u / 
  # /sbin/mount -a -t ufs 
  # /sbin/swapon -a


.. warning::

  **/tmp** darf nicht mit der Option "noexec" (in **/etc/fstab**) eingebunden
  werden!!

.. note::

  Zeigt ``date``\ (1) die falsche Zeit und eine falsche Zeitzone an, kann das
  mit dem Kommando ``tzsetup``\ (8) korrigiert werden. Viele Dienste
  funktionieren später nur dann korrekt, wenn die Zeit richtig eingestellt
  ist.

Jetzt ist man nur noch etwa 20 Minuten vom Starten des neuen
Betriebssystems entfernt!

Betriebssystem-Installationsvorbereitungen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Vor der Betriebssysteminstallation müssen noch einige Vorbereitungen
getroffen werden::

  # /usr/sbin/mergemaster -p

make installworld
~~~~~~~~~~~~~~~~~

Jetzt kann es losgehen mit der Installation der neuen
Betriebssystemdateien::

  # cd /usr/src 
  # make installworld

mergemaster
-----------

::

  # /usr/sbin/mergemaster -i 

Mit mergemaster werden die alten, bereits bestehenden Konfigurationsdateien mit
den Vorgaben des neuinstallierten Betriebssystems abgeglichen.

.. note::

  Beim Einsatz von mergemaster ist Vorsicht geboten!

  Mergemaster vergleicht die alten Konfigurationsdateien mit den neuen
  Vorgaben. Und fragt bei Änderungen , ob es die alte Konfigurationsdatei
  durch die neue Vorgabe austauschen soll. Dadurch können bei falscher
  Wahl Einstellungen verloren gehen!

Betriebssystem-Aktualisierung abschliessen
------------------------------------------

::

  # /sbin/reboot

Nach Abschluss der mergemaster-Arbeiten startet man den Computer neu.
Aus eigener Erfahrung wird empfohlen, nochmals in den Single-Usermode zu
starten und folgende Kontroll-/Aufräumarbeiten durchzuführen::

  # /sbin/fsck 
  # /sbin/mount -u / 
  # /sbin/mount -a -t ufs 
  # cd /usr/src 
  # make clean

Abschluss
~~~~~~~~~

Nachdem man auch dies geschafft hat, kann man die neue
Betriebssystemversion nach dem Drücken von: <Ctrl-D> geniessen!

Betriebssystem-Sicherheitslücken
--------------------------------

Betriebssystem-Sicherheitslücken sollten Sie so rasch wie möglich
schliessen. Den Stand der installierte Sicherheitslücken-"Flicker"
(engl. Patches) erkennen Sie mit dem Befehl:

::

  # uname -r 6.0-RELEASE-p5

Das Beispiel ist die Release-Version von FreeBSD 6.0 (6.0-RELEASE) mit
dem Patch-Level 5 (-p5). Der Patch-Level sagt aus, wieviele Patches für
die verwendete Release-Version eingespielt sind. Im Beispiel sind es
fünf Patches.

Eine Übersicht und mehr Informationen zu den bekannten
Betriebssystem-Sicherheitslücken finden Sie unter:

-  http://www.freebsd.org/ -> rechts unten
-  http://www.freebsd.org/security/
-  http://www.de.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/security-advisories.html

freebsd-update
~~~~~~~~~~~~~~

Das im Port security/freebsd-update enthaltene Programm freebsd-update
ermöglicht eine halbautomatische Einspielen von
Betriebssystem-Sicherheitslücken-"Flicker", vergleichbar mit dem Dienst
"Windows Update" der Microsoft-Betriebssysteme. Für Informationen zu
freebsd-update sehen Sie bitte:

-  http://www.daemonology.net/freebsd-update/
-  http://www.taosecurity.com/keeping_freebsd_up-to-date.html
-  http://www.daemonology.net/freebsd-update/binup.pdf
-  http://www.daemonology.net/blog/

.. note::

  ``freebsd-update`` kann nur mit dem originalen GENERIC- oder SMP-Kernel
  verwendet werden!

Installation
~~~~~~~~~~~~

FreeBSD-update wird mit: **pkg_add -r freebsd-update** installiert. Ab
FreeBSD 6.2 ist freebsd-update Teil des Betriebssystems und muss nicht
mehr zusätzlich installiert werden!

Anwendung
~~~~~~~~~

Nun sollen die Betriebssystemlöcher gesucht und gestopft werden (falls
möglich)::

  # freebsd-update fetch 
  # freebsd-update install

Jetzt noch Neustarten und schon sollten alle bisher bekannten
Betriebssystem-Sicherheitslücken geschlossen sein, falls eine
Release-Betriebssystemversion verwendet wird, keine `Stack
Protection <http://www.trl.ibm.com/projects/security/ssp/>`__ aktiviert
ist und nur der GENERIC-Kernel verwendet wird!

Kontrollieren Sie nach dem Neustart den Patchlevel:

::

  # uname -r 6.2-RELEASE-p2

Benutzer von Multiprozessoren-Systemen können auch von freebsd-update
profitieren: `FreeBSD-Security:
2005-June/002975 <http://lists.freebsd.org/pipermail/freebsd-security/2005-June/002975.html>`__

Problemfälle
~~~~~~~~~~~~

Es kann vorkommen, dass sich freebsd-update weigert, ein
Sicherheitsupdate einzuspielen. Es weichen einige lokale Dateien von den
originalen Release-Dateien ab. Wenn Sie sich ganz sicher sind, dass Sie
den GENERIC-Kernel verwenden und keine "make world"-Spezialoptionen in
/etc/make.conf gesetzt haben, so können Sie freebsd-update zur
Installation des Sicherheitsupdates zwingen. Dabei werden die vom
Sicherheitsloch betroffenen Dateien ohne Vorbehalt überschrieben:

freebsd-update --branch crypto fetch freebsd-update install

``Setzen Sie die Option "--branch crypto" nur im Problemfall ein!``

Für mehr Informationen zur Option "--branch crypto" siehe bitte:
``freebsd-update`` und `FreeBSD-Questions:
2005-June/091380 <http://lists.freebsd.org/pipermail/freebsd-questions/2005-June/091380.html>`__

.. _make-world-1:

Make World
----------

Sie können freebsd-update aus einem der obengenannten Gründen nicht
einsetzen, was nun? Ihnen bleibt nicht anderes übrig, als mit ``csup``
die mit dem entsprechenden Patch versehenen Betriebssystem-Quellcode
herunterzuladen und das Betriebssystem zu aktualisieren:

::

  # csup -g /etc/source-supfile

Kontrollieren Sie, ob der Betriebssystem-Quellcode unter **/usr/src**
auch den Patch enthält ``more /usr/src/UPDATING`` sollte mit dem
entsprechenden Patch-Eintrag versehen sein. Zum Beispiel:

::

   20051215:
           The setkey(8) utility was moved from /usr/sbin/setkey to /sbin/setkey.
           You may want to update scripts which depend on its location.

   20051108:
           rp(4)'s device files now contain the unit number.
           Uses of {cua,tty}R[0-9a-f] should be replaced by {cua,tty}R0[0-9a-f].

   20051101:
           FreeBSD 6.0-RELEASE

Aktualisieren Sie das Betriebssystem wie im Kapitel: `FreeBSD - Make
World: Betriebssystem
aktualisieren <FreeBSD - Make World#Betriebssystem aktualisieren>`__
beschrieben!

Unter `BSD-Hacks-Kapitel
67 <http://www.oreilly.de/catalog/bsdhks/chapter/hack67.pdf>`__ wird
eine Lösung vorgestellt, welche die obengenannten Schritte automatisch
ausführt.

Siehe auch
----------

-  http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/cvsup.html
-  http://www.onlamp.com/lpt/a/6543 - Make world bei vielen Rechnern
   lässt sich stark beschleunigen durch ein "FreeBSD Build System"
-  http://www.daemonology.net/freebsd-upgrade-6.0-to-6.1/ - Binären
   Upgrade durchführen

* :ref:`genindex`

Zuletzt geändert: |date|

