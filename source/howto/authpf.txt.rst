authpf
======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

**authpf** stellt eine Erweiterung zu `OpenBSD`_\ s pf (**p**\ acket **f**\
ilter) da, um bestimmten Rechnern den Zugang zu Ports/Diensten/Rechnern zu
erlauben.

Übersicht
---------

Die Installation ist erstaunlich einfach. Zunächst erstellt man die
Datei ``/etc/authpf/authpf.conf``. Nur wenn diese vorhanden ist (und sei
sie leer), läuft authpf überhaupt.

Es gibt zwei Möglichkeiten, authpf zu benutzen. Zum Einen mit

::

   /etc/authpf/users/$USER/authpf.rules

oder aber mit

::

   /etc/authpf/authpf.conf 

Die erste Datei beinhaltet Regeln, die geladen werden, wenn sich der
Nutzer $USER anmeldet. Das ist dann notwendig, wenn bestimmte Nutzer
besondere Rechte haben sollen, etwa wenn einem Benutzer der Zugang zu
einem Portbereich eröffnet werden soll und einem anderen der Zugang
hierzu nicht versperrt bleiben soll, er zusätzlich aber auch noch andere
Ports zur Verfügung gestellt bekommt.

Für alle anderen sich anmeldenden Nutzer wird die letztgenannte Datei
gelesen. Diesen Nutzern wird nur erlaubt, was in /etc/authpf/authpf.conf
steht, natürlich zusätzlich zu den ohnehin erlaubten Dingen aus
/etc/pf.conf.

Zusätzlich ist es noch notwendig, folgendes in die /etc/pf.conf
einzubauen.

::

   nat-anchor "authpf/*"
   rdr-anchor "authpf/*"
   binat-anchor "authpf/*"
   anchor "authpf/*"

Allerdings nur die anchors, die man benötigt. Beispielsweise bewirkt

::

   rdr-anchor "authpf/*"

dass in den authpf Regeln vorhandene rdr-Regeln gelesen werden.

::

   anchor "authpf/*"

wiederum bewirkt, dass Portöffnungsregeln aus den authpf-Regeln gelesen
werden. Die jeweiligen Anweisungen haben in dem entsprechen Teil der
pf.conf zu stehen, also in diesen beiden Beispielen der obere Befehl im
rdr-Teil, der untere Befehl in den Portregeln. Die Reihenfolge der
Regeln an sich liest man am Besten im pf Users Guide nach.

Beispiel
--------

Nehmen wir an, wir möchten auf einem Rechner einen Dienst laufen lassen
(in diesem Fall mal einen etwas überholten, nämlich telnet; bot sich
halt grade so an ;-) ) und nur bestimmten Benutzern Zugang gewähren.
"Vor" diesem Rechner, quasi als Absicherung ins Internet, steht ein
OpenBSD-"Router", auf dem wir pf und authpf laufen lassen möchten.

Zunächst ist natürlich der Dienst auf dem Zielrechner zu starten, in
diesem Falle also Telnet.

authpf anschalten
~~~~~~~~~~~~~~~~~

::

   # mkdir /etc/authpf
   # touch /etc/authpf/authpf.conf

erstellt die notwendige Grundkonstellation auf unserem OpenBSD-Rechner.

User erstellen
~~~~~~~~~~~~~~

Nehmen wir nun an, unser Benutzer, der ganz alleine auf den Rechner mit
dem Telnet-Dienst zugreifen dürfen soll, heisse Erwin. Diesen lieben
Onkel Erwin legen wir zunächst auf dem OpenBSD-Rechner an. Am
einfachsten geht das mit

::

   # adduser erwin

User spezifische konfiguration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Dann erstellen wir für authpf das entsprechende Verzeichnis.

::

   # mkdir /etc/authpf/users
   # mkdir /etc/authpf/users/erwin

Darauf folgt nun das erstellen den entsprechenden

.. _OpenBSD: https://www.openbsd.org

.. Ab hier bitte auf allen Seiten ergänzen

Original Eintrag: 2007/07/18 14:12 kamikaze

* :ref:`genindex`

Zuletzt geändert: |date|

