ssh
===

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Tipps und Tricks zum Umgang mit OpenSSH unter \*BSD und Linux. Ein SSH-Client
für Windows ist `PuTTY <putty.txt>`__.

sshd zum Starten verfügbar machen
---------------------------------

FreeBSD
~~~~~~~

.. code:: bash

   sysrc sshd_enable="YES"

oder in die Datei ``/etc/rc.conf``

::

   sshd_enable="YES"

eintragen.

OpenBSD
~~~~~~~

In die Datei ``/etc/rc.conf.local`` oder die Datei ``/etc/rc.conf``

::

   sshd_flags=""

eintragen.

NetBSD
~~~~~~

In die Datei ``/etc/rc.conf``

::

   sshd=YES

eintragen.

DragonFlyBSD
~~~~~~~~~~~~

In die Datei ``/etc/rc.conf``

::

   sshd_enable="YES"

eintragen.

Alternativ sollte auch die Version von NetBSD funktionieren.

Einloggen über SSH ohne Passworteingabe
---------------------------------------

.. note::

  Das Einloggen ohne Passwort kann mitunter ein Sicherheitsrisiko
  darstellen.

Sinn der Sache
~~~~~~~~~~~~~~

Viele haben zu Hause (oder sonstwo) mehrere Rechner, auf denen ein
SSH-Daemon läuft - sei es zur Verwaltung des Rechners oder sonstigem.
Jedoch kann es, wenn man sich oft auf den Maschinen einloggen muss,
mitunter recht nervig werden, jedes Mal das Passwort einzutippen. Der
Austausch der Public-Keys schafft hier Abhilfe.

Vorgehen
~~~~~~~~

In meinem Beispiel arbeite ich jetzt erst einmal mit dem imaginären User
``bob``, welcher einen Account sowohl auf ``rechner1.domain.bla`` als
auch auf ``rechner2.domain.bla`` hat.

-  ``rechner1.domain.bla`` ist der Rechner, an dem bob sitzt
-  auf ``rechner2.domain.bla`` loggt er sich mehrmals täglich per SSH
   ein.

Der erste Schritt ist, für ``bob`` auf ``rechner1.domain.bla ein``
Key-Pair zu erstellen (hierbei ist der Witz bei der Frage nach dem
Passwort keines einzugeben, sondern einfach mit Return zu bestätigen):

::

  bob@rechner1.domain.bla ~$ ssh-keygen -t rsa
  Generating public/private rsa key pair.
  Enter file in which to save the key (/home/bob/.ssh/id_rsa):↵
  Created directory '/home/bob/.ssh'.
  Enter passphrase (empty for no passphrase):↵
  Enter same passphrase again:↵
  Your identification has been saved in /home/bob/.ssh/id_rsa.
  Your public key has been saved in /home/bob/.ssh/id_rsa.pub.
  The key fingerprint is:
  1f:66:ea:96:83:16:6c:aa:be:1c:71:56:55:5b:e9:da bob@rechner1.domain.bla


Als nächster Schritt loggt ``bob`` sich auf ``rechner2.domain.bla`` ein,
und überprüft, ob das Verzeichnis ``~/.ssh`` in seinem Homedirectory
exisitiert. Wenn nicht - erstellen. Ausloggen. Dann, als ``bob`` an
``rechner1.domain.bla`` in das Verzeichnis ``~/.ssh`` wechseln und
folgendes eingeben:

::

  $ cat id_rsa.pub | ssh bob@rechner2.domain.bla "cat >> .ssh/authorized_keys"

Man wird jetzt ein letztes Mal nach dem Passwort gefragt. Wenn das alles
erledigt ist, kann ``bob`` sich von ``rechner1.domain.bla`` einfach mit

::

  $ ssh bob@rechner2.domain.bla

auf ``rechner2.domain.bla`` ohne Passwort einloggen.

Alternativen
~~~~~~~~~~~~

-  `ssh-agent <http://www.phil.uu.nl/~xges/ssh/>`__
-  Login per pam_ssh

Vorgehensweise bei ssh-agent nach MrFixit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Man muss den Passphrase nur einmal eingeben, dann wird der Schlüssel
dekodiert und im Speicher gehalten. Solange bis man den ssh-agent wieder
beendet (also spätestens beim reboot). Da man das ganze mit pam_ssh
verbandeln kann, kann man sich sogar die Passworteingabe für den
normalen xdm/login-Login sparen.

Hier nochmal mein Setup:

-  ``/etc/pam.d/xdm``
-  die pam_ssh Sachen auskommentiert und want_agent gesetzt.
-  Login per ``xdm`` mit user/passphrase (ich muss also zum Anmelden nur
   die Passphrase meines priv. Schluessels eingeben und auch nur einmal)
-  Schnelle ssh-logins bis ich mich aus X wieder abmelde

ssh-multiplexing
~~~~~~~~~~~~~~~~

Viele Leute öffnen eine ssh-session, geben 2-3 Befehle ein und melden
sich wieder ab. Mit ssh-multiplexing, das seit `OpenBSD <OpenBSD>`__ 3.6
verfügbar ist, kann man aber direkt so arbeiten::

  $ ssh server ps aux
  $ ssh server ls /root
  $ ssh server /usr/local/etc/rc.d/foo.sh stop

Siehe auch
----------

-  `Ssh-blocker </howto/Ssh-blocker>`__
-  `SSH in Chroot </howto/SSH in Chroot>`__
-  `PuTTY: Ein Windows-SSH-Client </howto/PuTTY>`__

* :ref:`genindex`

Zuletzt geändert: |date|

