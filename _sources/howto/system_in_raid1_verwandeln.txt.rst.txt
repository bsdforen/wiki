System in RAID1 verwandeln
==========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Über RAID-Systeme können Festplatten in einem "Bündel" zusammen gepackt
werden. Je nach RAID-Level kann somit die Performace gesteigert oder die
Ausfallsicherheit der Festplatten erhöht werden. Die Beschreibung der
verschiedenen RAID-Level ist in entsprechender Lektüre zu finden. Im
Weiteren wird erläutert wie ein ohne RAID installiertes FreeBSD mittels
systemeigenen Modulen (GEOM_MIRROR) zu einem Software-RAID (Level1)
umgewandelt werden kann.

Ausgangssituation
-----------------

RAID1 dient der Erhöhung der Ausfallsicherheit von Festplatten indem die
Daten gespiegelt (mirroring) werden. Die Lesezugriffe können durch RAID1
beschleunigt werden, die Schreibzugriffe werden jedoch nicht schneller.

Das installierte System soll der gezeigten Struktur entsprechen:

::

   /dev/ad4s1a on /
   /dev/ad4s1e on /tmp
   /dev/ad4s1f on /usr
   /dev/ad4s1d on /var

Als zweite RAID-Platte zusätzlich zu ``'ad4``' soll die Platte ``'ad8``'
dienen.

BACKUP
------

Zunächst muss ein Backup des Systems erzeugt werden!! Die Umwandlung in
ein RAID ist nicht ohne Risiken für die auf der Festplatte gespeicherten
Daten!

Erzeugen des Mirrors
--------------------

Da der Mirror auf einer Boot-Platte erzeugt werden soll wird dieser
Schritt über ein Live-System durchgeführt. Das Umwandeln einer Partition
in einen RAID ist ein kritischer Eingriff in die Datenstruktur einer
Festplatte und sollte nicht im aktiven Betrieb dieser Platte erfolgen!

Ins Live-System booten
----------------------

Es wird ein aktuelles FreeBSD-Livesystem gebootet.

Nach der neuerlichen Kontrolle ob die Devicenamen stimmen(!!) wird nun
ein neues GEOM_MIRROR-Device angelegt. Das Device trägt den Namen
``'gm``' und erhält das Laufwerk ``'/dev/ad4``' als erstes
RAID-Laufwerk. Somit ist sicher gestellt, dass alle später hinzu
kommenden Laufwerke die Daten von ``'ad4``' übernehmen.

::

  # gmirror label -vb round-robin gm0 /dev/ad4

Anschließend wird ``gmirror`` geladen. Hiermit wird das RAID
initialisiert und das benötigte Kernelmodul geladen::

  # gmirror load

Nun sollte das RAID unter /dev/mirror erzeugt sein.

Anpassen der loader.conf
------------------------

Damit das System mit dem Spiegel starten kann muss noch die
``/boot/loader.conf`` angepasst werden. Hierzu ist das Device zu
mounten. Anschließend wird ``/boot/loader.conf`` wie folgt ergänzt um
das notwendige Kernelmodul beim Systemstart zu laden. Ohne dieses kann
das System nicht auf den RAID-Verband zugreifen.

::

   geom_mirror_load="YES"

Anpassen der fstab
------------------

Da der RAID unter dem Namen ``/dev/mirror/gm0`` angelegt wurde ist nun
noch die ``/etc/fstab`` wie folgt anzupassen.

Sei dies ist originale ``/etc/fstab``

::

   /dev/ad4s1b     none                      swap      sw      0   0
   /dev/ad4s1a     /                         ufs       rw      1   1
   /dev/ad4s1e     /tmp                      ufs       rw      2   2
   /dev/ad4s1f     /usr                      ufs       rw      2   2
   /dev/ad4s1d     /var                      ufs       rw      2   2

so ist sie folgend anzupassen:

::

   /dev/mirror/gm0s1b     none               swap      sw      0   0
   /dev/mirror/gm0s1a     /                  ufs       rw      1   1
   /dev/mirror/gm0s1e     /tmp               ufs       rw      2   2
   /dev/mirror/gm0s1f     /usr               ufs       rw      2   2
   /dev/mirror/gm0s1d     /var               ufs       rw      2   2

Booten in den Mirror
--------------------

Nachdem alle Änderungen durchgeführt wurden kann nun in den Mirror
gestartet werden. Hierzu wird der Rechner einfach neu gestartet.

Spiegelplatten hinzufügen
-------------------------

Im neu gestarteten System sollten nun die Systempartitionen über
``/dev/mirror`` laufen.

Um nun die zweite Platte ``ad8`` in den RAID aufzunehmen (und erst
wirklich den RAID herzustellen) wird diese nun hinzugefügt::

  # gmirror insert gm0 /dev/ad8

Nun wird die neue Platte an den RAID-Verband ansynchronisiert. Auf
diesem Weg können später noch mehr Platten hinzugefügt werden. Um den
Status des RAIDs und der Synchronisierung zu sehen kann
``'gmirror status``' verwendet werden.

Solange die Festplatte noch synchronisiert wird sieht die Ausgabe von
``'gmirror status``' wie folgt aus

::

  # gmirror status
        Name    Status  Components
  mirror/gm0  DEGRADED  ad4
                        ad8

Ist die Synchronisierung beendet und die Platten befinden sich im
normalem Spiegelbetrieb wird der Status ``'COMPLETE``' zurück geliefert.

::

  # gmirror status
        Name    Status  Components
  mirror/gm0  COMPLETE  ad4
                        ad8

* :ref:`genindex`

Zuletzt geändert: |date|

