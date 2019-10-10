GnuPG
=====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Die Sicherheit der eigenen Daten ist ein wichtiges Thema. Bei der
Übertragung von Mails wird keinerlei Verschlüsselung oder ähnliches
angewendet. Daher ist eine E-Mail so ungeschützt wie eine Postkarte.

Durch Verschlüsseln und Signieren von E-Mails und Dateien kann die
Sicherheit erhöht, um nicht zu sagen, erst geschaffen werden. Genauere
Informationen zum Thema Verschlüsselung, Signierung und GnuPG sind hier
nicht gegeben; diese sollten an anderer Stelle zusammengetragen werden.

GnuPG installieren
------------------

GnuPG wird mit root-Rechten installiert.

::

  % cd /usr/ports/security/gnupg 
  % make install clean

Anschliessend wird noch ein Port zur Abfrage des Kennworts installiert.
Hierbei handelt es sich um einer der folgenden:

-  ``security/pinentry-curses`` (für Konsolen Anwender)
-  ``security/pinentry-q4`` (für KDE4 Anwender)
-  ``security/pinentry-gtk2`` (für Gnome2 Anwender)

Schlüssel erzeugen
------------------

Sollte bereits ein gültiges Schlüsselpaar vorliegen kann dieses
importiert werden. Ist dies nicht der Fall wird ein neues Schlüsselpaar
erzeugt. Die Folgenden Befehle werden als normaler Benutzer ausgeführt.

::

  % gpg --gen-key

Es folgt eine Abfrage nach der zu erstellenden Schlüsselart. 

::

  gpg (GnuPG/MacGPG2) 2.0.12; Copyright (C) 2009 Free Software Foundation,
  Inc.
  This is free software: you are free to change and redistribute it.
  There is NO WARRANTY, to the extent permitted by law.
  
  Bitte wählen Sie, welche Art von Schlüssel Sie möchten:
     (1) RSA und RSA (voreingestellt)
     (2) DSA und Elgamal
     (3) DSA (nur unterschreiben/beglaubigen)
     (4) RSA (nur signieren/beglaubigen)
  Ihre Auswahl?

Hier sollte nach eigenem Ermessen gewählt werden (Notfalls gilt
``"Voreinstellung wählen"``).

::

  RSA-Schlüssel können zwischen 1024 und 4096 Bit lang sein.
  Welche Schlüssellänge wünschen Sie? (2048)

Ein Schlüssel kann verschiedene Längen aufweisen. Standard ist hierbei
2048 Bit. Ein längerer Schlüssel ist potenziell sicherer, bedarf aber
auch mehr Rechenleistung. Es sollte kein Schlüssel unter einer Länge von
2048 Bit erstellt werden.

::

  Bitte wählen Sie, wie lange der Schlüssel gültig bleiben soll.
           0 = Schlüssel verfällt nie
        <n>  = Schlüssel verfällt nach n Tagen
        <n>w = Schlüssel verfällt nach n Wochen
        <n>m = Schlüssel verfällt nach n Monaten
        <n>y = Schlüssel verfällt nach n Jahren
  Wie lange bleibt der Schlüssel gültig? (0)

Anschliessend wird noch eine Gültigkeitsdauer festgelegt. Es ist
sicherer einen Schlüssel nur begrenzt gültig zu machen und die
Gültigkeitsdauer nachträglich zu verlängern, als einen Schlüssel ewig
gültig zu haben der am Ende ggf. kompromittiert wurde. Ein Paar Monate,
bis ein Jahr sollten realistisch sein.

Anschliessend werden noch die persönlichen Daten abgefragt und die
Passphrase eingefordert. Hiermit wird das Schlüsselpaar erstellt, was
einige Zeit in Anspruch nehmen kann.

Widerrufszertifikat erstellen
-----------------------------

Zunächst sollte ein Widerrufszertifikat erstellt werden. Mit Hilfe
dieses Zertifikats ist es möglich einen Schlüssel als ungültig zu
erklären. Sollte man seinen Schlüssel verlieren oder sein Kennwort
vergessen ist solch ein Zertifikat die einzige Möglichkeit dies zu tun,
da für die Erzeugung sowohl Kennwort als auch privater Schlüssel
notwendig sind.

Um die Schlüssel-ID des eigenen Schlüssels zu erfahren wird folgender
Befehl verwendet::

  % gpg -k

Die Ausgabe sieht dann ähnlich der folgenden aus::

  pub
  4096R/1A2B3C4D 2009-12-05 [expires: 2011-01-01] uid Ernst Mustermann
  Ernst@Mustermann.org sub 4096R/2B4D6F8A 2009-12-05 [expires: 2011-01-01]

In diesem Fall wäre die Schlüssel-ID: 1A2B3C4D

Mit dem Aufruf::

  % gpg -a -o revokation --gen-revoke 1A2B3C4D

Natürlich mit der entsprechenden Schlüssel-ID

Somit wurde eine Datei mit dem Namen ``revokation`` erstellt welche das
Widerrufszertifikat enthält. Diese Datei ist sicher aufzubewahren!

Privaten Schlüssel exportieren
------------------------------

Der private Schlüssel ist immer sicher und geheim zu halten. Um ihn
jedoch zu sichern (z.B. auf einem USB-Stick im Tresor) muss man ihn
natürlich irgendwie exportieren. 

.. warning::

  Der private Schlüssel darf nicht in fremde Hände fallen da somit der
  Fremde Dateien und Mails mit dem "gestohlenen" Schlüssel unterzeichnen und
  entschlüsseln könnte.  Das Fehlen der Passphrase ist dann nur noch ein
  geringes Problem.

::

  % gpg --export-secret-key -a > secret.key

Privaten Schlüssel einspielen
-----------------------------

Wird ein privater Schlüssel wie im vorigen Absatz gesichert kann er
natürlich auf einem anderen System wieder eingespielt werden. Dies
erfolgt mit folgendem Befehl::

  % gpg --import secret.key

Schlüssel verlängern
--------------------

Ist ein Schlüssel einmal abgelaufen kann er nicht mehr zum Signieren
oder Verschlüsseln verwendet werden. Um nicht immer einen neuen
Schlüssel anlegen zu müssen kann die Gültigkeit des alten Schlüssels
verlängert werden.

Hierzu wird der Schlüssel zum Bearbeiten geöffnet::

  % gpg --edit-key MAILADRESSE

(Tipp: Mit ``help`` erfährt man welches Befehle zum Bearbeiten eines
Schlüssels zur Verfügung stehen.)

Nun wird die Verlängerungsfunktion gewählt.

::

  % gpg> expire

Nun wird entsprechend der angezeigten Syntax die weitere Laufzeit
eingegeben.

Zum Ende wird die Änderung noch gespeichert::

  % gpg> save

* :ref:`genindex`

Zuletzt geändert: |date|

