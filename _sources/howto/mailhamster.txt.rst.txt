Mailhamster (Mailsammler)
=========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Seit geraumer Zeit plagt mich das Problem, dass meine Mail von
verschiedenen Mailkonten auf verschiedenen Rechnern verteilt liegen, da
man ja nicht immer am selben Rechner sitzt (oder ihn angemacht hat), so
dass es manchmal schwierig ist eine bestimmte Mail zu finden. Auch die
Datensicherung der so verteilten Mails ist aufgrund der dezentralen
Anordnung nicht so trivial. Aus diesem Grunde wollte ich mir eine
zentralisierte Mailablage mit automatischem Abruf der einzelnen
Mailkonten aufbauen. Dieser Artikel beschreibt das Vorgehen zur
Einrichtung der einzelnen Komponenten bis hin zur Gesamtlösung.

Grundlagen / Voraussetzungen
----------------------------

Grundlagen
~~~~~~~~~~

Die Anleihen für mein Setup habe ich den diversen Forenbeiträgen, den
Gesprächen mit rabbit und dem hervoragenden Wiki-Eintrag aus dem Wiki
der bsd-crew.de entnommen und mit meinen persönlichen Erfahrungen
ergänzt.

Folgende Voraussetzungen habe ich für mein Setup vorgefunden:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Ein komplett aufgesetztes FreeBSD-System, Version 6.0-Stable
-  Einen aktuellen Portstree

Weiter sollte dem geneigten Benutzer die Grundkenntnisse im Umgang mit
den Ports und/oder Paketen bekannt sein um die benötigten Programme
installieren zu können. Desweiteren ist der sichere Umgang mit Editor
und Kommandozeile von Vorteil.

Ergänzende Angaben zum System:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Für das Verständnis der Konfigurationsdateien sind folgende
Zusatzangaben gegeben:

-  IP-Adresse des Systems: 192.168.1.12
-  Mailkonto 1: foo
-  Mailkonto 2: bar
-  Benutzer für den die Mails bestimmt sind: hamster

Welche Programm aus den Ports werden benötigt
---------------------------------------------

-  getmail
   (`mail/getmail <https://www.google.com/search?q=mail/getmail&btnI=lucky>`__)
   » unser Mailsammler für die diversen Postkonten
-  procmail
   (`mail/procmail <https://www.google.com/search?q=mail/procmail&btnI=lucky>`__)
   » unser Mailsortierer
-  dovecot
   (`mail/dovecot <https://www.google.com/search?q=mail/dovecot&btnI=lucky>`__)
   » unser IMAP-Server für die lokale Administration

Funktionsweise des Setups
-------------------------

Die Funktionsweise des Setups für den Mailhamster ist wie folgt:
**getmail** klappert alle ihn vorgegebenen Mailkonten ab und empfängt
die Mails. Er übergibt die Mails an **procmail** welches sich um das
Einsortieren der Mails in die Richtige Ordnerstruktur kümmert.
**dovecot** schlussendlich stellt die Schnittstelle zwischen dieser
Ordnerstruktur und dem Mailclient (z.B. Thunderbird, Outlook, Mail.app
etc) dar.

Konfiguration von getmail
-------------------------

Aus Gründen der Einfachheit habe ich mir einen eigenen Benutzer
``getmail`` angelegt. Nicht nur der Übersichtlichkeit halber, sondern
auch um die Konfigurationsdateien zentral abgelegt zu wissen.

::

  # adduser getmail

Im Homeverzeichnis des soeben erstellten Users habe ich einen
versteckten Ordner namens ``getmail`` angelegt:

::

  [getmail@server] ~/> mkdir .getmail

Das Programm **getmail** werden wir nun in einem weiteren Schritt
konfigurieren. Wir wechseln hierzu in das neu erstellte Verzeichnis und
erstellen dort das Konfigurationsfile für das erste Mailkonto.

::

  [getmail@server] ~/.getmail> touch foo.conf

Wir werden diese neu erstellte Datei nun mit der eigentlichen
Konfiguration "beglücken". Wir öffnen die Datei mit einem Editor und
schreiben folgende Einträge rein:

::

   [options]
   verbose = 0                        # damit wir nicht von Meldungen belästigt werden
   delete = true                      # lösche die Mails die erfolgreich vom Mailkonto abgeholt wurden
   read_all = false                   # damit nur die neuen Mails abgeholt werden
   message_log = ~/.getmail/foo.log   # sollte was sein, wird es ins Log geschrieben

   [retriever]
   type = SimplePOP3Retriever         # diese Mailkonto ist nur ein einfaches POP3-Konto
   server = mail.foo.org              # Adresse des Mailservers
   username = foo                     # Benutzername
   password = werweiss                # Passwort im Klartext

   [destination]
   type = MDA_external                # Gibt an in welches Maildir-Format gespeichert werden soll
   path = /usr/local/bin/procmail     # Gibt an an welches Programm die Mails weitergegeben werden
   arguments = ("-dhamster", )        # Gibt die Argumente für das Programm an, hier der Benutzername hamster

So das wäre mal geschafft. Jetzt noch schnell das zweite Mailkonto
konfigurieren:

::

  [getmail@server] ~/.getmail> cp foo.conf bar.conf

Und dann die Datei ebenfalls im Editor bearbeiten:

::

   [options]
   verbose = 0                        # damit wir nicht von Meldungen belästigt werden
   delete = true                      # lösche die Mails die erfolgreich vom Mailkonto abgeholt wurden
   read_all = false                   # damit nur die neuen Mails abgeholt werden
   message_log = ~/.getmail/bar.log   # sollte was sein, wird es ins Log geschrieben

   [retriever]
   type = SimplePOP3Retriever         # diese Mailkonto ist nur ein einfaches POP3-Konto
   server = mail.bar.org              # Adresse des Mailservers
   username = bar                     # Benutzername
   password = werahnt                 # Passwort im Klartext

   [destination]
   type = MDA_external                # Gibt an in welches Maildir-Format gespeichert werden soll
   path = /usr/local/bin/procmail     # Gibt an an welches Programm die Mails weitergegeben werden
   arguments = ("-dhamster", )        # Gibt die Argumente für das Programm an, hier der Benutzername hamster

Da die Dateien jedoch das Passwort im Klartext enthalten, habe ich die
Rechte noch wie folgt gesetzt:

::

  [getmail@server] ~/.getmail> chmod 640 foo.conf 
  [getmail@server] ~/.getmail> chmod 640 bar.conf

Jetzt ist die Konfiguration von **getmail** eigentlich soweit
abgeschlossen. Wenden wir uns der Konfiguration von **dovecot** zu.

Konfiguration von dovecot
-------------------------

Die Beispielkonfiguration von **dovecot** finden wir nach erfolgreicher
Installation im Verzeichnis ``/usr/local/etc``. Wir wechseln ins
Verzeichnis und kopieren die Beispielkonfiguration.

::

  # cp dovecot.conf.sample dovecot.conf

Wir öffnen nun die Datei in einem Editor und verändern was wir brauchen.
Diese Konfigurationsdatei wie unten gezeigt wird mit der von mir
verwendeten Version mitgeliefert. Spätere Versionen können in der Syntax
unterschiedlich sein. Um die Übersichtlichkeit zu wahren, habe ich nur
die relvanten Änderungen untenstehend aufgelistet.

::

   ## Dovecot 1.0 configuration file

   # Protocols we want to be serving:
   #  imap imaps pop3 pop3s
   protocols = imap

   # IP or host address where to listen in for connections. It's not currently
   # possible to specify multiple addresses. "*" listens in all IPv4 interfaces.
   # "[::]" listens in all IPv6 interfaces, but may also listen in all IPv4
   # interfaces depending on the operating system.  If you want to specify ports
   # for each service, you will need to configure these settings inside the
   # protocol imap/pop3 { ... } section, so you can specify different ports
   # for IMAP/POP3.
   #listen = *
   listen = 192.168.1.12

   # Disable LOGIN command and all other plaintext authentications unless
   # SSL/TLS is used (LOGINDISABLED capability). Note that 127.*.*.* and
   # IPv6 ::1 addresses are considered secure, this setting has no effect if
   # you connect from those addresses.
   #disable_plaintext_auth = yes
   disable_plaintext_auth = no

   # chroot login process to the login_dir. Only reason not to do this is if you
   # wish to run the whole Dovecot without roots.
   # http://wiki.dovecot.org/Rootless
   login_chroot = yes

   # User to use for the login process. Create a completely new user for this,
   # and don't use it anywhere else. The user must also belong to a group where
   # only it has access, it's used to control access for authentication process.
   # Note that this user is NOT used to access mails.
   # http://wiki.dovecot.org/UserIds
   login_user = dovecot

   # Should each login be processed in it's own process (yes), or should one
   # login process be allowed to process multiple connections (no)? Yes is more
   # secure, espcially with SSL/TLS enabled. No is faster since there's no need
   # to create processes all the time.
   login_process_per_connection = yes

   # Show more verbose process titles (in ps). Currently shows user name and
   # IP address. Useful for seeing who are actually using the IMAP processes
   # (eg. shared mailboxes or if same uid is used for multiple accounts).
   verbose_proctitle = yes

   # Valid GID range for users, defaults to non-root/wheel. Users having
   # non-valid GID as primary group ID aren't allowed to log in. If user
   # belongs to supplementary groups with non-valid GIDs, those groups are
   # not set.
   first_valid_gid = 0
   #last_valid_gid = 0

   # ':' separated list of directories under which chrooting is allowed for mail
   # processes (ie. /var/mail will allow chrooting to /var/mail/foo/bar too).
   # This setting doesn't affect login_chroot or auth_chroot variables.
   # WARNING: Never add directories here which local users can modify, that
   # may lead to root exploit. Usually this should be done only if you don't
   # allow shell access for users. See doc/configuration.txt for more information.
   #valid_chroot_dirs =
   valid_chroot_dirs = ~/mail/

   #   default_mail_env = maildir:/var/mail/%1u/%u/Maildir
   #   default_mail_env = mbox:~/mail/:INBOX=/var/mail/%u
   #   default_mail_env = mbox:/var/mail/%d/%n/:INDEX=/var/indexes/%d/%n
   #
   #default_mail_env = mbox:/var/mail/%u
   #default_mail_env = maildir:~/mail/ #alt wird durch mail_location ersestzt
   mail_location = maildir: ~/mail/:INBOX=%h/mail/.INBOX

   # Copy mail to another folders using hard links. This is much faster than
   # actually copying the file. This is problematic only if something modifies
   # the mail in one folder but doesn't want it modified in the others. I don't
   # know any MUA which would modify mail files directly. IMAP protocol also
   # requires that the mails don't change, so it would be problematic in any case.
   # If you care about performance, enable it.
   #maildir_copy_with_hardlinks = no
   maildir_copy_with_hardlinks = yes

   auth default {
     # Space separated list of wanted authentication mechanisms:
     #   plain digest-md5 cram-md5 apop anonymous
     mechanisms = plain

Nachdem die Konfiguration abgeschlossen wurde, können wir **dovecot**
mit einem beherzten

::

  # dovecot

starten. Um die Konnektivität zu prüfen können wir uns via **telnet**
mal probeweise verbinden. 

.. note::

  Achtung: Eine kleine Stolperfalle bietet sich hier: Alle Befehle an
  **dovecot** müssen mit einem x beginnen, so z.B. x login USER
  PASSWORT!

::

  [hamster@server] ~/> telnet localhost 143 Trying ::1... Trying
  127.0.0.1... Connected to localhost. Escape character is '^]'. \* OK
  Dovecot ready. x login USER PASSWORT x OK Logged in. x logout \* BYE
  Logging out x OK Logout completed. Connection closed by foreign host.
  [hamster@server] ~/>

Wenn das soweit geklappt hat, dann können wir den letzten Schritt wagen.

Konfiguration von procmail
--------------------------

Jetzt geht es noch darum die ankommende Mail, welche **getmail** abholt
in die verschiedenen Postfächer einzusortieren, damit **dovecot** etwas
zu arbeiten hat. Dafür müssen wir noch **procmail** konfigurieren. Die
geschieht mittels einer Konfigurationsdatei im HOME-Verzeichnis des
Users.

Wir legen nun eine solche Datei an.

::

  [hamster@server] ~/> touch .procmailrc

Wir werden diese neu erstellte Datei nun mit der eigentlichen
Konfiguration "beglücken". Wir öffnen die Datei mit einem Editor und
schreiben folgende Einträge rein:

::

   MAILDIR=$HOME/mail               # das Mailverzeichnis
   LOGFILE=$HOME/log/procmail.log   # dahin sollen die Meldungen
   VERBOSE=on                       # Sag uns was abgeht

   :0                               # Beginn einer Regel
   * ^TO_.*foo\.org                 # alle Mails an foo.org werden in
   .INBOX.foo/                      # die INBOX.foo einsortiert

   :0                               # Beginn der Regel
   * ^TO_.*bar\.org                 # alle Mails an bar.org werden in
   .INBOX.bar/                      # die INBOX.bar einsortiert

   :0                               # Beginn der Regel
   .INBOX.spam/                     # Alle Mails die nicht einsortiert werden konnten, werden hierhin verschoben

So, das war es auch schon fast. Ja fast, denn die Datei bekommt auch
noch ein

::

  [hamster@server] ~/> chmod 640 .procmailrc

Jetzt ist die Konfiguration von **procmail** eigentlich soweit
abgeschlossen. Wenden wir uns mal dem ersten Testlauf zu.

Der erste Test
--------------

Jetzt gilt es ernst, der erste Testlauf steht an und dann stellt sich
heraus ob alle Komponenten soweit funktionieren so wie wir uns das
vorgestellt haben. Wir loggen uns als User ``getmail`` ein und starten
getmail mal zu einem Lauf.

::

  [getmail@server] ~/> getmail -v --rcfile ~/.getmail/foo.conf --rcfile ~/.getmail/bar.conf

Es werden einige Meldungen über den Bildschirm flitzen und am Ende wird,
wenn alles gut gegangen ist, der Prompt auf uns warten. Wir können uns
jetzt mal als User ``hamster`` anmelden und schauen ob was in die
INBOXEN eingegangen ist.

::

  [hamster@server] ~/mail/> ls -la
  drwx------  11 hamster  wheel    512 Dec  8 14:14 ./
  drwxr-xr-x   8 hamster  wheel    512 Dec  6 21:43 ../
  drwx------   5 hamster  wheel    512 Dec  8 14:33 .Deleted Messages/
  drwx------   5 hamster  wheel    512 Dec  8 15:49 .INBOX.bar/
  drwx------   5 hamster  wheel    512 Dec  8 15:48 .INBOX.foo/
  drwx------   5 hamster  wheel    512 Dec  6 20:55 .INBOX.spam/
  drwx------   5 hamster  wheel    512 Dec  8 14:14 .Trash/
  drwx------   2 hamster  wheel    512 Dec  6 20:49 cur/
  -rw-------   1 hamster  wheel    144 Dec  6 20:50 dovecot.index
  -rw-------   1 hamster  wheel  10272 Dec  4 16:17 dovecot.index.cache
  -rw-------   1 hamster  wheel    148 Dec  6 20:50 dovecot.index.log
  drwx------   2 hamster  wheel    512 Dec  6 20:49 new/
  -rw-------   1 hamster  wheel     43 Dec  8 14:14 subscriptions
  drwx------   2 hamster  wheel    512 Dec  6 20:49 tmp/
  [hamster@server] ~/mail/>

Das sieht doch schon gut aus. Doch, woher weiss denn Procmail dass der
User ``hamster`` die Mails von foo und bar bekommt? Wir erinnern uns,
dass wir in den jeweiligen **getmail** Konfigurationsdateien am Ende den
User vorgegeben haben.

Abschliessende Arbeiten
-----------------------

Jetzt fehlt uns eigentlich nur noch dass wir

::

   *a) die Mails in regelmässigen Abständen von unseren Mailkonten abrufen lassen und
   *b) etwas zum Anzeigen der Mails

Punkt b) ist relativ einfach, da praktisch jedes Mailprogramm
sogenannten Serverbasierte Mailkonten unterstützt. Leider haben auch
hier gewisse Programme Stärken und Schwächen. Die von mir getesteten
Mailprogramme wie Thunderbird, Outlook und Mail.app vom Aplle kamen sehr
gut damit zurecht. Um die Mails lokal am Server zu lesen, kann **mutt**
verwendet werden.

Punkt a) ist ebenfalls relativ einfach zu lösen. Wir loggen uns als User
``getmail`` ein und editieren seine ``crontab``, die Datei für
regelmässige Tätigkeiten.

::

  [getmail@server] ~/> crontab -e

Im nun erscheinenden Editor tragen wir folgende Befehlskette ein:

::

   */5 * * * * /usr/local/bin/getmail --rcfile ~/.getmail/foo.conf --rcfile ~/.getmail/bar.conf > /dev/null 2>&1

So, das wars. Nun werden die Mailkonten alle fünf Minuten abgegrast und
die neuen Nachrichten in die INBOXEN weitergeleitet.

Ich hoffe, dieser Artikel konnte ein wenig Licht in die Sache mit der
zentralen Ablage von Mails bringen. Sicherlich noch ausbaufähig, aber
für mich funktioniert es so wie es soll.

* :ref:`genindex`

Zuletzt geändert: |date|

