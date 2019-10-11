Thunderbird
===========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

`Thunderbird <http://www.mozilla.com/thunderbird>`__ ist ein einfach zu
benutzender, mächtiger und konfigurierbarer E-Mail- und
Usenet-News-Client. Gleichzeitig bietet er viele Features. Thunderbird
untertützt IMAP und POP genauso wie HTML Formatierungen im Editor. Wenn
man auf Thunderbird umsteigt, kann man einfach seine alten Konten und
Mails übernehmen.

Features
--------

::

   *RSS-Feeds 
   *Quicksearch
   *Rechtschreibprüfung
   *selbst erstellbare Filter
   *Speicherort definierbar
   *Abfrage zum Download

Konfiguration
-------------

Die Konfiguration erfolgt über das Menü **Bearbeiten** ->
**Einstellungen**. Das Menü ist ähnlich aufgebaut wie bei Firefox. Über
das Konten-Menü **Bearbeiten** -> **Konten** werden Konten angelegt,
Filter eingerichtet und noch vieles mehr.

Links mit einem Browser öffnen
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   *Im Einstellungs-Menü wird der Tab '''Erweitert''' ausgewählt.
   *Im unteren Bereich wird dann '''Konfiguration bearbeiten''' angeklickt. Es erscheint ein neues Fenster, das dem Firefox-'''about:config''' entspricht.
   *Mit der rechten Maustaste wird jeweils eine neue Stringeingabe ausgewählt ('''Neu''' -> '''String'''). Folgende Eingaben sind erforderlich:

================================== ======================
Eigenschaftsname                   Beispiel-String
================================== ======================
network.protocol-handler.app.http  /usr/local/bin/firefox
network.protocol-handler.app.https /usr/local/bin/firefox
network.protocol-handler.app.ftp   /usr/local/bin/firefox
================================== ======================

Es gibt noch einen weiteren Weg, die oben genannten Einstellung
vorzunehmen. Hierzu darf kein Thunderbird laufen. Folgende Einträge sind
in der
`user.js <http://www.mozilla.org/support/thunderbird/edit#user>`__
vorzunehmen:

::

   user_pref("network.protocol-handler.app.http", "/usr/X11R6/bin/firefox");
   user_pref("network.protocol-handler.app.https", "/usr/X11R6/bin/firefox");
   user_pref("network.protocol-handler.app.ftp", "/usr/X11R6/bin/firefox");

ab X.org 7.2 haben sich die Pfade der X11-Anwendungen geändert:

::

   user_pref("network.protocol-handler.app.http", "/usr/local/bin/firefox");
   user_pref("network.protocol-handler.app.https", "/usr/local/bin/firefox");
   user_pref("network.protocol-handler.app.ftp", "/usr/local/bin/firefox");

Sprachpakete
------------

Mit `thunderbird-i18n <http://www.freshports.org/mail/thunderbird-i18n>`__ ist
eine bequeme Installation von Sprachpaketen möglich. Standardmäßig werden die
Sprachen zh-CN de fr ja ru it es-ES installiert. Will man nur deutsch haben,
genügt ein

::

   echo THUNDERBIRD_I18N=de >> /etc/make.conf

Mit ``THUNDERBIRD_I18N=all`` werden alle verfügbaren Sprachen
installiert: bg ca cs da de el en-GB es-AR es-ES eu fi fr gu-IN he hu it
ja ko mk nb-NO nl pa-IN pl pt-BR ro ru sk sl sv-SE tr zh-CN

Ändern kann man die Sprache dann über Extras --> Andere Sprachen

`Übersicht über
Ländercodes <http://de.wikipedia.org/wiki/ISO-3166-1-Kodierliste>`__

Rechtschreibprüfung
-------------------

Einfach
`http://www.freshports.org/mail/thunderbird-dictionaries/ thunderbird-dictionaries <http://www.freshports.org/mail/thunderbird-dictionaries/ thunderbird-dictionaries>`__
installieren. Im Optionsmenü kann man auswählen, für welche Sprachen
Rechtschreibprüfungen installiert werden.

Add-ons
-------

Für Thunderbird sind viele `Erweiterungen, Themes und Suchplugins
<https://addons.mozilla.org/thunderbird/>`__ verfügbar.

Auch in die Ports werden derzeit nahezu täglich interessante
Erweiterungen für Thunderbird und andere Gecko basierte Produkte
hinzugefügt. Der Name der Ports beginnt meist mit
```xpi-`` <http://www.freshports.org/search.php?stype=name&method=match&query=xpi-&num=500&orderby=category&orderbyupdown=asc&search=Search>`__.

Ob man die Erweiterungen über die Ports oder die Mozilla Seite
installiert ist Geschmackssache. Bei einer Installation über die Ports
werden die Erweiterungen für alle Benutzer und alle Gecko basierten
Produkte (Firefox, Thunderbird u.a.) installiert.

Wenn man die Erweiterungen aus den Ports installiert hat, muss man, nach
einer Neuinstallation von Thunderbird, folgendes ausführen:

::

   # cd /usr/ports/www/xpi-adblock/ && make relink-all

Dies ist unabhängig davon, ob man xpi-adblock installiert hat oder
nicht. Alternativ kann man auch jedes andere Verzeichnis einer
Erweiterung nehmen. ``'Das Ergebnis ist dasselbe!``'

Enigmail und Lightning
~~~~~~~~~~~~~~~~~~~~~~

Unter FreeBSD sind Enigmail (GPG Add-On) und Lightning (Kalender),
direkt über die Konfiguration des Thunderbird Ports zu aktivieren.

GnuPG
~~~~~

Mit
`enigmail-thunderbird <http://www.freshports.org/mail/enigmail-thunderbird/>`__
gibt es eine komfortable Möglichkeit seine E-Mails zu signieren und zu
verschlüsseln. Ein Verwaltung der Schlüssel ist hiermit auch möglich.

Moztraybiff
~~~~~~~~~~~

[STRIKEOUT:Ein kleines Icon im Systemtray, das auf neue E-Mails
hinweist,
bietet]\ `moztraybiff <http://www.freshports.org/mail/moztraybiff>`__\ [STRIKEOUT:.]
Port wurde gelöscht, da nur bis TB 2.x

Siehe auch
----------

-  `Tipps für
   Thunderbird <http://www.mozilla.org/support/thunderbird/tips>`__

* :ref:`genindex`

Zuletzt geändert: |date|

