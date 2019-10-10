Revision Control System
=======================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Autodokumentation mit RCS
-------------------------

Das Revision Control System (RCS) dient zur Revisionskontrolle von
Files. RCS automatisiert das Speichern, Holen, Dokumentieren,
Identifizieren und Zusammenführen von Änderungen (Revisionen). RCS wird
vor allem bei Textdateien mit vielen Änderungen gebraucht, z.B.
Quellcode, Dokumentation und Konfigurationsdateien.

Dazu wird mit dem Befehl::

  # ci filename

eine Datei dem RCS übergeben (``ci`` = check in). Soll die Datei bearbeitet
werden, muss sie ausgecheckt werden (``co`` = check out)::

  # co filename

Alle Änderungen, die an der Datei vorgenommen wurden, werden in der Datei
filename,v gespeichert.

Die wichtigsten Befehle
~~~~~~~~~~~~~~~~~~~~~~~

Handbuch zu RCS ``rcs`` Einführung zu RCS ``rcsintro`` check in::

  # ci

check out::

  # co

Revisionskommentarlog::

  # rlog

Revisionslog::

  # rcsdiff

Anwendung
---------

Am Beispiel der Startdatei von X ``~/.xinitrc`` werde ich versuchen das
Prinzip zu erklären.

In meinem Heimatverzeichnis habe ich mehrere Konfigurationsdateien. Ich
erzeuge mit ``# mkdir ~/RCS`` darin ein Verzeichnis RCS.
Dieses Verzeichnis dient der Ordnung. Alle Dateien, die ich dem RCS
übergebe, erhalten eine zugehörige Revisionsdatei. Diese
Revisionsdateien werden alle in ``./RCS`` gespeichert, wenn es
existiert.

Nun übergebe ich die ``.xinitrc-Datei`` mit ``# ci -u
~/.xinitrc`` dem RCS. Mit dem Parameter ``-u`` sorge ich dafür,
dass eine editierbare Kopie im Originialverzeichnis bleibt. RCS fordert
mich auch zur Beschreibung der Datei auf, diese kann mehrzeilig sein,
abgeschlossen wird sie mit einem Punkt . auf einer einzelnen Zeile.

Will ich die ``.xinitrc-Datei`` ändern, so muss ich mit ``# co -l
~/.xinitrc`` die Datei auschecken, mit dem Parameter ``-l`` sorge
ich dafür, dass nur ich die Datei ändern kann (lock).

Nachdem ich mit meinem Lieblingseditor ``.xinitrc`` geändert und
geschrieben habe, muss ich die Datei wieder einchecken ``# ci -u
~/.xinitrc`` RCS fordert mich nun zur Beschreibung meiner
Änderungen auf, diese kann mehrzeilig sein, abgeschlossen wird sie mit
einem Punkt . auf einer einzelnen Zeile.

Wenn ich die Kommentare zu meinen Änderungen sehen will, zeigt mir
``# rlog ~/.xinitrc`` alle Kommentare samt Revisionsnummern
an.

Wenn ich meine Änderungen sehen will,zeigt ``# rcsdiff -r1.1 -r1.2
~/.xinitrc`` mir alle Änderungen zwischen den Revisionen an (hier
zw. 1.1 und 1.2).

Um eine frühere Revision wiederherzustellen, ``# co -rx.x
~/.xinitrc`` wobei x durch die jeweilige Revisionsnummer zu
ersetzen ist.

RCS-Variablen
-------------

Beispiele:

::

    $Id$ Dateiname, Revisionsnummer, Datum, Zeit, Autor, Status
    $Header$ Genau wie $Id$ nur mit voller Pfandangabe
    $Log$ Zeigt die letzten Log Meldungen an, die beim check-in abgefragt werden

Mehr unter ``co``.

Möchte man die Uhrzeit in den Variablen auf die lokale Zeit gesetzt
haben, so gibt man beim check-in noch ``-zLT`` mit an ``# ci -u
-zLT`` Noch einfacher geht es, wenn man RCSINIT exportiert:
``# export RCSINIT=-zLT``

Diese Variablen kann ich in jede Datei, die ich dem RCS übergebe,
schreiben. Das sieht für $Id$ folgendermaßen aus:

::

   $Id: index.html,v 1.10 2005/10/12 13:22:58 simon Exp $

rcsedit
-------

`sysutils/rcsedit <https://www.google.com/search?q=sysutils/rcsedit&btnI=lucky>`__
erleichtert die Arbeit mit RCS. Einfach ``# rcsedit <Datei>``
eingeben und die Datei wird ausgecheckt, ein Editor geöffnet und nach
dem Schließen des Editors kann man seine Änderungen eingeben.

* :ref:`genindex`

Zuletzt geändert: |date|

