Hardware-Guide
==============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel ist noch nicht fertig. Genauergesagt fehlt noch das worum es
eigentlich geht undzwar die meisten Tabellen mit der Treibersituation für
Hardware bestimmter Hersteller sortiert nach Hardware-Rubrik! Hilf mit oder
warte ab :-D

Allgemeines
-----------

| Dieser Guide soll einen Überblick über "Hardware und die BSDs" geben.
  Dabei geht es nicht um die Frage "Läuft Grafikkarte X Modell Y
  Revision Z mit #BSD Version Q?" (dazu siehe `Unterstütze
  Hardware </Hardware/Unterstützte_Hardware>`__), sondern allgemeiner
  darum welche Hardware-Hersteller in welchem Umfang BSD unterstützen
  und darum was man sich kaufen soll wenn man gerade dabei ist ;-)
| Dabei geht es sowohl um pragmatische Aspekte, aber auch um Themen wie
  Codefreiheit, Hardware-Dokumentation und BLObs.
| Ich habe die Hersteller in drei Gruppen unterteilt an denen wir messen
  können wie gut/schlecht sie die BSDs und andere Freie Betriebsysteme
  unterstützen.

keine Dokumentation, keine Treiber
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| Diese Hardware läuft offensichtlich überhaupt nicht. Wenn man etwas
  aus dieser Kategorie hat ist man aufgeschmissen. Hersteller solcher
  Hardware sollte man aus Prinzip meiden.
| Hersteller dieser Gruppe sind in den Tabellen schwarz markiert.

keine Dokumentation, BLOB-Treiber
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| BLOb steht für **B**\ inary\ **L**\ anguage\ **Ob**\ ject, Treiber die
  vorkompiliert kommen und keine freie Software sind. Was ist daran
  schlimm?, mag sich einer fragen...
| So einiges! Ersteinmal wird die Freiheit des Benutzers erheblich
  eingeschränkt, es ist nicht länger möglich das *komplette*
  Betriebsystem zu verändern, ein Teil entzieht sich deiner Kontrolle,
  du bist somit dem Hersteller auf Gedeih und Verderb ausgeliefert.
| "Ist mir egal, hauptsache es geht!" mag mancheiner erwiedern, aber
  auch Pragmatiker sollten vor BLObs gewarnt sein:

::

   *BLObs kommen oft mit EULAS, die selbst das "einfache" Weiterverbreiten verbieten und in vielen Fällen rechtliche Probleme verursachen können.
   *BLObs sind nicht vertrauenwürdig. Es läuft schließlich Code in deiner Kernel, den du nicht kennst, es könnte ein Staats- oder Industrie-Trojaner sein oder Code nachladen, der sowas macht.
   *BLObs sind oft einfach schlechte Software. Im Gegensatz zu freien Treibern wird der Code von sehr wenigen Leuten gelesen und programmiert. Diese habe oft außerdem einfach nicht sehr viel Ahnung von freien Betriebsystemen da sie nicht zu der Community der Kernel- und Treiberprogrammierer gehören. Bugs im Treiber können die Sicherheit des Systems gefährden und es unstabil machen, wenn dies passiert dauert es oft lange bis der Hersteller das Problem behebt.
   *BLOb-Treiber gibt es nur für bestimmte (freie) Betriebsysteme, nie für alle. Außerdem gibt es sie nur für bestimmte Rechnerarchitekturen, so dass sowohl Hardware- als auch Software-Wechsel erschwert oder unmöglich gemacht werden.
   *BLOb-Treiber sind oft unpraktisch da sie nicht ins Betriebsystem integriert werden. So müssen BLOb-Treiber getrennt aktualisiert und gewartet werden, es kann Versionskonflikte geben.
   *Manche Programmierer behaupten, es sei besser wenn ein Hersteller garkeine Treiber anbietet als wenn er BLObs veröffentlicht, so hätten freie Alternativen (siehe unten) bessere Chancen unterstützt zu werden (größere Nachfrage)

| Wenn irgendmöglich sollte man auch diese Hersteller kategorisch
  ablehnen. Das soll nicht heißen, dass jemand der seine Hardware
  versucht zum laufen zu kriegen deswegen ein schlechter Mensch oder
  ähnliches ist, aber wer sein freies und stabiles Betriebsystem mag der
  achtet beim Kauf neuer Hardware darauf, dass sie keine BLObs benötigt.
| Hersteller dieser Gruppe sind in den Tabellen rot markiert.

Dokumentation oder freie Treiber
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| Diese Hersteller sind die "Guten" ;-) Entweder wird Dokumentation
  veröffentlicht oder der Hersteller entwickelt - manchmal sogar in
  Zusammenarbeit mit Programmierern aus den Projekten - freie Treiber.
  Diese Treiber werden dann fest in die freien Betriebsysteme
  integriert, die Hardware wird somit oft automatisch erkannt und gut
  unterstützt.
| Hersteller dieser Gruppe sind in den Tabellen grün markiert.

freie Treiber durch Reverse-Engineering
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| Wenn ein Hersteller zur I. oder II. Kategorie gehört wird oft versucht
  durch Reverse-Engineering freie Treiber zu entwickeln. Dies ist sehr
  viel Arbeit und z.T. auch mit rechtlichen Risiken für Programmierer
  verbunden. Auch wenn die Hersteller dadurch nicht besser sind und auch
  die Qualität dieser Treiber oft nicht mit auf Dokumentation
  basierenden Treiber mithalten kann (und aus beiden Gründen nach
  Möglichkeit Hersteller der Gruppe III bevorzugt werden sollten), so
  kann es schon ein wichtig Kriterium sein, besonders wegen den
  praktischen Vorteilen gegenüber BLOb und dem Mangel an Alternativen.
| Hardware dieser Hersteller ist gelb markiert.

Firmware
~~~~~~~~

| Neben Treibern, die *Software* sind, gibt es noch *Firmware*. Der Name
  (*firm* von engl. fest) impliziert es handele sich um etwas zwischen
  Hardware und Software. Das stimmt so nicht ganz. Technisch gesehen ist
  Firmware auch Software, mit dem einzigen Unterschied, dass sie auf dem
  Prozessor eine Chipsatzes der Hardware läuft und nicht auf dem
  Hauptprozessor (CPU).
| Firmware läuft also weder im Kernel-, noch im Userspace, womit
  eigentlich fast alle praktischen Nachteile von Unfreiheit
  verschwinden. Deswegen und weil es in vielen Bereichen noch garkeine
  freie Firmware gibt wird auf Firmware nicht gesondert eingeganen
  (außer in Einzelfällen).
| Trotzdem sollte man sich darüber im Klaren sein, dass EULAS die
  Distribution von Firmware beeinträchtigen können und das "moralische"
  Problem weiterbesteht etwas vorgesetzt zu bekommen, das man nicht
  verändern *darf* (und praktisch nicht kann) obwohl man es *könnte*
  ohne, dass jemand zu Schaden kommt (falls man auf sowas wie Freiheit
  wert legt ;-) ).

Hardware
--------

Grafikkarten (2D-Support)
~~~~~~~~~~~~~~~~~~~~~~~~~

Grafikkarten (3D-Support)
~~~~~~~~~~~~~~~~~~~~~~~~~

Für mehr Informationen zum Thema 3D-Beschleunigung mit freien Treibern:

https://wiki.archlinux.org/index.php/Hardware_video_acceleration

LAN
~~~

TODO

Mainboard / Chipsätze
~~~~~~~~~~~~~~~~~~~~~

TODO

Soundkarten
~~~~~~~~~~~

TODO

WLAN
~~~~

| Siehe dazu den
  `Treibervergleich <https://en.wikipedia.org/wiki/Comparison_of_Open_Source_Wireless_Drivers>`__
  der englischen Wikipedia.
| TODO: vielleicht aus der Wikipedia-Tabelle eine mehr
  Hersteller-basierte und übersichtlichere Tabelle machen.

* :ref:`genindex`

Zuletzt geändert: |date| 
