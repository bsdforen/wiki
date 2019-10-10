ZFS auf FreeBSD
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Was ist ZFS?
------------

| Seit ZFS im Jahr 2004 das erste Mal von Sun vorgestellt wurde, schlägt
  es Wellen und weckt Begehrlichkeiten. Grund ist vor allem, das ZFS ein
  neuartiges Konzept hat, welches sich wesentlich von dem anderer
  Dateisysteme unterscheidet. Oder anders ausgedrückt: ZFS verändert ein
  fast 40 Jahre altes Konzept grundlegend.
| Auf den ersten Blick fällt vor allem das integrierte Volumenmanagement
  auf. Bisher hatte man einen RAID-Controller oder einen virtuellen
  Volumenmanager, welcher unterhalb des Dateisystems lag. Dieser
  verteilte die Daten über mehrere Platten, sorgte so für
  Ausfallsicherheit und machte einzelne Platten austauschbar. Auch
  konnte er den zur Vefügung stehenden Speicherplatz vergrößern, was
  eine Vergrößerung des Dateisystems nötig machte. Dies Vorgehen ist gut
  und schön und etabliert, dennoch hat es entscheidende Nachteile. Die
  wenigsten Dateisysteme können schrumpfen, so muss der Speicherplatz
  immer gleich bleiben oder ansteigen. Das Vergrößern von Dateisystemen
  läuft meist ineffizient ab, sie werden langsamer oder man verliert
  Speicherplatz. Außerdem weiß das Dateisystem über die
  zugrundeliegenden Platten nicht bescheid, sodass es ihre Eigenarten
  nur unzureichend berücksichtigen kann. Der integrierte Volumenmanager
  von ZFS löst diese Probleme und kann außerdem auch mit anderen
  Dateisystemen verwendet werden. Weiter hat UFS2 inzwischen einige
  gravierende Nachteile. Es beginnt damit, dass die Algorithmen
  ineffizient werden, wenn man sich der vollständigen Belegung des
  Dateisystems nähert. So werden üblicherweise 7% des zur Verfügung
  stehenden Speicherplatzes nicht genutzt. Waren Softupdates vor einigen
  Jahren eine sehr gute Lösung um das Dateisystem auch bei Problemen wie
  Stromausfall oder Absturz konsistent zu halten, lässt sich bei
  modernen Festplattengrößen der Nachteil nicht mehr leugnen, dass ein
  fsck nötig ist. Dies kann zwar im Hintergrund ablaufen, nur bei einer
  größtenteils belegten 750GiB Festplatte kann es schnell mehr als 30
  Minuten dauern. 30 Minuten, in denen es den Server belastet und
  langsam macht. Mit gjournal gibt es inzwischen einen Weg dies Problem
  zum umgehen, besonders elegant ist er jedoch nicht.
| Mit Snapshots verfügt UFS2 über eine Funktion, die die meisten anderen
  Dateisysteme nicht haben. Jedoch sind sie teuer, Snapshots unter UFS2
  benötigen recht lange zum Erstellen, machen das Dateisystem merklich
  langsamer, benötigen viel Platz und sind zudem recht umständlich zu
  handhaben.
| ZFS hat diese Probleme nicht. Durch besondere Mechanismen sind Dinge
  wie fsck im Falle eines Ausfalls nicht mehr nötig, auch ein Journal -
  wie z.B. bei JFS oder XFS - wird nicht benötigt. Snapshots sind in
  Sekundenbruchteilen erstellt, sie benötigen zu Beginn nicht einmal
  zusätzlichen Speicherplatz. Auch das Zurückrollen eines Datensatzes
  (dazu später mehr) auf einen Snapshot ist in wenigen Sekunden und mit
  einem einzigen Kommando möglich. Datensätze könne ohne Dump/Restore
  Tricks blitzschnell kopiert werden, es stehen schnelle Möglichkeiten
  zur Festplattenkompression zur Verfügung und Daten werden auf ihre
  Korrektheit geprüft, sodass im Falle einer defekten Festplatte keine
  bösen Überraschungen auftreten. Zudem beschleunigt die "Copy on Write"
  Technik das Kopieren von Daten innerhalb eines Zpool erheblich,
  verglichen mit zum Beispiel UFS2.

Anforderungen
-------------

ZFS hat leider relativ hohe Hardwareanforderungen. Um es wirklich sauber
und schnell nutzen zu können, sollte das System unter FreeBSD/amd64
laufen und über mindestens ein Gigabyte RAM verfügen. Der Betrieb unter
FreeBSD/i386 und auch mit weniger als einem Gigabyte RAM ist nur unter
Einschränkungen möglich. Siehe hierzu
http://wiki.freebsd.org/ZFSTuningGuide.

Implementationsstatus
---------------------

ZFS ist mittlerweile stabiler Bestandteil von FreeBSD und kann
bedenkenlos auch für Produktivsysteme eingesetzt werden. Mittlerweile
ist auch das Booten von ZFS möglich. Das in FreeBSD eingesetzte ZFS
basiert auf der OpenZFS-Entwicklung und unterstützt Featureflags. Als
integraler Bestandteil von FreeBSD ist es seit einiger Zeit auch möglich
während der Installation über den Installer ein System komplett auf ZFS
aufsetzen zu lassen.

Zpool und ZFS
-------------

| Bevor wir zu ZFS kommen, werden wir uns mit der integrierten
  Volumenverwaltung Zpool beschäftigen. Grundsätzlich gilt: Ein ZFS kann
  niemals außerhalb eines Zpool existieren, es schwimmt sozusagen auf
  ihm. Andererseits enthält ein Zpool immer mindestens ein ZFS. Ein ZFS
  innerhalb eines Zpools wird zur Abgrenzung gegenüber herkömmlichen
  Dateisystemen nicht als "Dateisystem" bezeichnet, sondern als
  "Dataset" oder auf Deutsch "Datensatz".
| Innerhalb eines Zpool können drei Dinge existieren:

-  Ein Datensatz. Ein Datensatz ist das, was man im Sprachgebrauch als
   ZFS bezeichnet. Ein Datensatz wächst dynamisch, sprich er nimmt
   innerhalb des Pools genau soviel Platz ein, wie die Daten in ihm. Man
   kann ihm eine maximale Größe zuordnen und auch ein mindestmaß an
   Speicherplatz garantieren.
-  Ein Volume. Ein Volume entspricht einem herkömlichen Blockgerät. Es
   wird ein Gerät in /dev angelegt, welches mit einem normalen
   Dateisystem wie UFS2 formatiert werden oder als Swap genutzt werden
   kann. Dabei kann es trotzdem die Vorzüge des Zpools nutzen, wie
   Datenintegritätsprüfungen oder die sehr schnellen und günstigen
   Snapshots.
-  Ein Snapshot. Dies ist der eingefrorene Zustand eines Datensatzes
   oder eines Volumens.

| Ein Zpool beinhaltet nun verschiedene Möglichkeiten einzelne Medien zu
  nutzen. Die einfachste Art ist ein einfacher Pool, welcher auf einer
  einzelnen Festplatte oder sogar in einem Slice auf dieser lebt.
  Darüber steht die Verbindung, welche sich über verschiedene Platten
  erstreckt. Der "Mirror" oder "Spiegel" lebt auf mindestens zwei
  Platten und spiegelt diese - wie gmirror. Die Krönung ist RaidZ. Dies
  spezielle RAID5 muss nach einem Absturz oder ähnlichem nicht wieder
  aufgebaut werden, der Wiederaufbau beim Tausch einer Platte ist zudem
  schneller als bei einem echten RAID5. Und es läuft auch ohne teuren
  Controller sehr schnell.
| RaidZ kennt drei Ebenen. RaidZ1 benötigt mindestens drei Platten, hier
  kann nur eine ausfallen ohne das Datenverlust eintritt. RaidZ2
  benötigt mindestens vier Platten, hier können zwei ausfallen. RaidZ3
  benötigt mindestens fünf Platten und es können bis zu drei ausfallen
  ohne Datenverlust. Die drei Ebenen haben gemeinsam, dass ein RaidZ
  nach der Erstellung nicht mehr vergrößert werden kann.

Administration eines ZPool
--------------------------

Die Administration eines ZPool ist nicht sonderlich kompliziert, jeder
der schon mit ähnlichen Systemen wie gmirror oder auch Raid-Controllern
gearbeitet hat, dürfte schnell Ergebnisse erzielen. Aber auch für
Anfänger sollte es sich schnell erschließen.

Anlegen eines Zpool
~~~~~~~~~~~~~~~~~~~

Angelegt wird ein ZPool mit dem Kommando

::

   zpool create -m mountpunkt name art gerät1 gerät2 spare gerät3 gerät4

| Mit der Option -m wird ein Mountpunkt angebeben. Das benötigte
  Verzeichnis erstellt Zpool automatisch und hängt das Wurzelverzeichnis
  des Zpools dort ein. Wird diese Option nicht angegeben, wird im
  Wurzelverzeichnis des Systems ein Verzeichnis mit dem Namen des Zpool
  angelegt und dieser dort eingehängt.
| Art ist die Art des ZPool. Dies kann "mirror" sein - der Spiegel.
  Dieser benötigt mindestens zwei Geräte. Alternativ kann "raidz1",
  "raidz2" oder "raidz3" angegeben werden, welche sich wie oben erklärt
  verhalten. "raidz1" benötigt mindestens drei Geräte, "raidz2"
  mindestens vier und "raidz3" mindestens fünf Geräte. Wird diese Option
  nicht angegeben, bedeckt der Zpool eine oder mehrere Platten per
  simpler Verkettung (Stripes).
| Die Option "spare" und die dahinter angebebenen Geräte sind
  Ersatzgeräte. Wenn eines der Hauptgeräte verschwindet, zum Beipsiel
  durch Defekt oder per Hotplug wird eines dieser dem Zpool hinzugefügt
  und der mit diesem neu aufgebaut. Diese Option kann nur mit "mirror"
  oder "raidz" verwendet werden.
| Die Art des Gerätes ist egal, solange es ein Blockgerät ist. Möglich
  sind also rohe Festplatten, was empfohlen ist, aber auch Slices,
  GEOM-Geräte und ähnliches. Es sind sogar Dateien als Geräte möglich,
  doch dies ist nur zu Testzwecken zu empfehlen.

Löschen eines Zpool
~~~~~~~~~~~~~~~~~~~

Gelöscht wird ein Zpool mit dem Kommando

::

   zpool destroy name

Dies Kommando hängt alle Datensätze aus, exportiert den Zpool und
zerstört ihn. Dies geschieht ohne eine Sicherheitsnachfrage und alle
Daten gehen dabei verloren!

Entfernen eines Gerätes aus einem ZPool
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ein Gerät wird aus einem ZPool entfernt mit

::

   zpool remove gerät

Das Entfernen kann einige Zeit dauern, da die Daten erst umkopiert
werden müssen. Außerdem muss der Pool alle seine Daten auch ohne das zu
entfernende Gerät aufnehmen können. Aus einem RaidZ kann kein Gerät
entfernt werden.

Hinzufügen eines Gerätes zu einem ZPool
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ein Gerät wird so in einen Zpool eingefügt

::

   zpool attach name gerät

Das Gerät steht sofort zur Verfügung, ein Neuaufbauen ist nicht
notwenig. Zu beachten ist, dass man zu einem RaidZ kein gerät hinzufügen
kann und ein Spiegel zwei neue Geräte benötigt.

Ein Gerät ersetzen
~~~~~~~~~~~~~~~~~~

Wer ein defektes oder zu kleines Gerät ersetzen möchte, kann dies mit
diesem Kommando machen

::

   zpool replace name altes_gerät neues_gerät

Die Daten werden dann umkopiert, was eine Weile dauern kann. Zu beachten
ist, dass im Spiegel oder RaidZ das neue Gerät mindestens so groß wie
das alte sein muss. Im RaidZ und im Mirror kann zusätzlicher
Speicherplatz nicht verwendet werden bis alle Festplatten durch größere
ausgetauscht wurden und der Zpool dazu angestoßen wird den größeren
Platz zu nutzen.

Sonstiges
~~~~~~~~~

Dies waren die wichtigsten Kommandos. Desweiteren ist es möglich mittels

::

   zpool list name

| sich Informationen zu einem Pool anzeigen zu lassen. Wird kein Name
  angegeben, werden alle Pools aufgelistet. Für weitere
  Informationsmöglichkeiten wird an dieser Stelle auf die Manpage
  zpool(1) verwiesen, da diese sehr umfangreich sind und den Rahmen
  dieser Einführung sprengen würden.
| Ein Zpool muss aus dem System ausgehängt werden, bevor er in Form
  seiner Geräte in eine andere Maschine gebracht wird. Auch muss man ihn
  Aushängen, damit er für das System unsichtbar wird und man z.B. die
  einzelnen Platten mit dd(1) behandeln kann. Dies geschieht mit

::

   zpool export name

Daraufhin werden alle Datensätze ausgehängt und der Pool geschlossen.
Wieder einfügen kann man ihn mittels

::

   zpool import name

Eine Liste aller für den Import zur Verfügung stehenden Zpools kann man
mit dem Kommando

::

   zpool import

angezeigt werden.

Für weitere Möglichkeiten, wie dem Abschalten von Geräten innerhalb
eines Pools oder dem Aktualisieren auf eine neuere ZFS Version sei
wieder auf die Handbuchseite zpool(1) verwiesen.

ZFS
---

Das Konzept
~~~~~~~~~~~

| Ein oder mehrere ZFS schwimmen wie oben erwähnt auf einem Zpool. Wir
  werden und hier an die Notation von Sun halten und sie als Datensätze
  bezeichnen. Jeder Zpool beinhaltet mindestens ein ZFS, dies befindet
  sich im Hauptverzeichnis diesem. Hat man zum Beispiel den Zpool
  "Banane" mit Standardoptionen erstellt, ist dieser in /banane
  eingehängt. Was man in /banane sieht, ist jedoch nicht der Zpool
  selbst, sondern das zwingend enthaltene ZFS. Ein Aufruf von mount(8)
  bestätigt dies.
| Wenn man mit den Standardeinstellungen arbeitet, werden alle
  Datensätze innerhalb eines Zpool relativ zum Wurzelverzeichnis von
  diesem angelegt. Es ist jedoch möglich sie auch an anderen Stellen im
  System einzuhängen, wodurch sich jedoch nur ihre logische
  Repräsentation ändert, in Wirklichkeit befinden sie sich immer noch
  relativ zur Wurzel des Zpool.

Anlegen eines Datensatzes
~~~~~~~~~~~~~~~~~~~~~~~~~

Mit dem Kommando

::

   zfs create pool/pfad/zu/datensatz

wird ein neuer Datensatz erstellt und an der angebenen Position relativ
zur Wurzel des Zpool eingehängt. Eventuell fehlende Verzeichnisse werden
dabei automatisch erstellt.

Löschen eines Datensatzes
~~~~~~~~~~~~~~~~~~~~~~~~~

Mit dem Kommando

::

   zfs destroy pool/pfad/zu/datensatz

wird der gewählte Datensatz ausgehängt und zerstört. Die in ihm
enthaltenen Daten sowie abhängende Snapshots werden zerstört.

<note tip> Sollten sich auf einem vollen zfs Datensatz mal keine Dateien
mehr löschen lassen, mit dem Hinweis: No space left on device.

Kann man mit:

::

   cp /dev/null <file auf dem vollen Datensatz>

eine Datei löschen um anschließend wieder regulär dateien mir rm löschen
zu können. </note>

Verschieben und umbennen
~~~~~~~~~~~~~~~~~~~~~~~~

Natürlich kann man einen Datensatz verschieben und umbenennen. Gibt man

::

   zfs rename pool/pfad/zu/datensatz pool/neuer/pfad/zu/datensatz 

ein verschiebt man den Datensatz innerhalb der ZFS-Hirachie.

Ändern von Optionen
~~~~~~~~~~~~~~~~~~~

Jeder Datensatz hat eine Unmenge von Einstellungsmöglichkeiten. Hier
werden nur die wichtigsten durchgesprochen werden, für weitere sei auf
die Manpage zfs(1) verwiesen. Gesetzt werden Einstellungen mit

::

   zfs set option=wert pool/pfad/zu/datensatz

und mit

::

   zfs get option pool/pfad/zu/datensatz

kann man sich den aktuellen Wert einer Variable ausgeben lassen.
Wichtige Variablen sind:

-  quota=size gibt die maximale Größe des Datensatzes an. Als
   Modifikatoren sind k, m und g für Kilobyte, Megabyte und Gigabyte
   erlaubt.
-  reservation=size gibt den garantierten Speicherplatz für einen
   Datensatz an. Modifikatoren wie oben.
-  mountpoint=pfad Verzeichnis abhängig zum Wurzelverzeichnis des
   Systems, in dem der Datensatz eingehängt wird.
-  sharenfs=on|off muss eingeschaltet werden, um den Datensatz per NFS
   freizugeben. Eine temporäre Freigabe kann mittels "zfs share
   pool/pfad/zu/datensatz" gemacht werden. Vorraussetzung ist, dass der
   NFS-Server in /etc/rc.conf aktiviert wurde.
-  compression=off|lzjb|gzip schaltet die Kompression ein. Dies betrifft
   nur neue Daten. lzjb ist schnell, gzip komprimiert gut.
-  atime=on|off gibt an, ob die "Accesstime" beim Zugriff auf eine Datei
   neu gesetzt wird. Das Abschalten macht den Datensatz merklich
   schneller, kann aber auch viele Programme verwirren.
-  readonly=on|off macht wenn gesetzt den Datensatz unmodifizierbar.
-  jailed=on|off macht den Datensatz aus einem Jail heraus verwaltbar.

Die geänderten Optionen sind sofort wirksam, weitere Dinge neben dem
Setzen sind nicht notwendig.

Snapshots und Klone
-------------------

Einen wesentlichen Vorteil bietet ZFS durch seine - wie oben besprochen
günstigen - Snapshots und die eng mit diesen verbundenen Klone, welche
es erlauben komplette Datensätze in wenigen Sekunden zu kopieren. ZFS
Snapshots sind deutlich günstiger als ihre Kollegen vom UFS2, weshalb
man ohne bedenken auch sehr viele von ihnen parallel nutzen kann.

Erstellen eines Snapshots
~~~~~~~~~~~~~~~~~~~~~~~~~

Ein Snapshot wird mit dem Kommando

::

   zfs snapshot datensatz@name 

erstellt. Datensatz ist hierbei der Datensatz, von dem ein Snapshot
erstellt werden soll und Name ist entsprechend der Name des Snapshots.
Dies Benennungsschema ist vorgegeben, ein Snapshot heißt immer
ursprung@eigener name. Der Snapshot "gutes_beispiel" von /banane/test1
wäre also /banane/test1@gutes_beispiel.

Löschen eines Snapshots
~~~~~~~~~~~~~~~~~~~~~~~

Snapshots werden exat wie Datensätze gelöscht.

::

   zfs destroy datensatz@name

Dies ist nicht möglich, wenn von dem Snapshot noch Dinge wie Klone
abhängig sind.

Zurückrollen eines Snapshots
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Mittels

::

   zfs rollback datensatz@name

wird der per Snapshot gesicherte Datensatz innerhalb von Sekunden auf
den Zustand des Snapshots zurückgesetzt. Alle Änderungen an ihm sind
unwiderbringlich verloren. Der Snapshot wird dabei nicht gelöscht.

Zugriff auf Snapshots
~~~~~~~~~~~~~~~~~~~~~

Die Snapshots können mit

::

   zfs list

angezeigt werden. Um auf sie zuzugreifen wechselt man in das unsichtbare
Verzeichnis .zfs um Wurzelverzeichnis des Datensatzes. So befindet sich
unser oben angelegter Snapshot in
/banane/test1/.zfs/snapshot/gutes_beispiel/. Das das Verzeichnis
unsichtbar ist bedeutet, dass Programme wie ls(1) oder die
Autovervollständigung der Shell es nicht sehen können!

Klone erstellen
~~~~~~~~~~~~~~~

Klone sind nichts anderes als beschreibbare Snapshots. Zum derzeitigen
Zeitpunkt können sie lediglich von Snapshots erstellt werden, nicht
jedoch von Datensätzen. Daher muss man von einem Datensatz erst einen
Snapshot erstellen, welchen man dann "klonen" kann. Dieser Snapshot
bildet die Grundlage für den Klon, dieser ist von ihm abhängig. Dieser
Snapshot kann daher nicht gelöscht werden, solange Klons zu ihm
existieren.

::

   zfs clone snapshot pool/pfad/zu/klon

erstellt einen Klon und hängt ihm am angebenen Pfad relativ zur Wurzel
des Zpool ein. Dort kann er wie ein normaler Datensatz behandelt werden.
Leider können Klone zur Zeit nicht auf den Datensatz, von dem der ihm
zugrundeliegende Snapshot stammt, zurückgerollt werden.

Klone in Datensätze umwandeln
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ein Klon kann jederzeit zu einem vollständigen Datensatz umgewandelt
werden.

::

   zfs promote pool/pfad/zu/klone

Hiernach ist der Klon ein normaler Datensatz und kann als solcher
verwendet werden. Man kann den Datensatz allerdings nicht wieder in
einen Klon zurückwandeln. Die Abhängigkeit zum Snapshot, welche der Klon
hatte, verschwindet bei dieser Operation.

Andere Dateisysteme auf Zpool
-----------------------------

Neben Datensätzen, Snapshots und Klonen kann auf einem Zpool wie oben
bereits besprochen auch ein Volume erstellt werden. Dieser taucht in
/dev/zvol als ein normales Gerät auf und kann mit jedem anderen
Dateisystem formatiert werden. So kann jede Dateisystem auf die Vorteile
des Zpool zurückgreifen. Mounten muss man allerdings herkömmlich, auch
dateisystemspezifische Dinge wie fsck kann der Zpool nicht ersetzen.

Erstellen eines Volumes
~~~~~~~~~~~~~~~~~~~~~~~

Ein Volume wird wie ein Datensatz mit

::

   zfs create -V größe pool/name

erstellt. Wobei Name der Name des Volumens ist und auch sein Gerät in
/dev/zvol so heißt. Ein Volume muss immer im Wurzelverzeichnis des Zpool
angelegt werden.

Warten eines Volumes
~~~~~~~~~~~~~~~~~~~~

Ein Volume kann wie ein normaler Datensatz behandelt werden. Es können
Snapshots erstellt und zurückgerollt werden, ebenso kann man wie bereits
erklärt Klone nutzen. Sogar diverse Optionen können wie oben beschrieben
eingestellt werden. Auch das Löschen läuft wie bei normalen Datensätzen
ab.

Swap auf Zpool
~~~~~~~~~~~~~~

Theoretisch ist es natürlich auch möglich ein Volume mittels swapon(8)
als Swap zu nutzen. Dies ist derzeit allerdings sowohl unter FreeBSD als
auch unter Solaris sehr problematisch. Das Problem ist, dass das System
auf die Swap schreiben möchte, wenn es nur noch wenig Arbeitsspeicher
zur Verfügung hat. Zpool benötigt aber sehr viel Arbeitsspeicher für
seine Daten. Nun kann es dazu kommen, dass der Speicherbedarf von Zpool
schneller steigt, als das System freiräumen kann. Es kommt zum Absturz.
Daher ist von Swap auf Zpool dringenst abzuraten!

* :ref:`genindex`

Zuletzt geändert: |date|

