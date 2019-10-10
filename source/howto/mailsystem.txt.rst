Das eigene Mailsystem
=====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

In diesem HowTo geht es darum, einmal zu zeigen was man alles mit seinem
Rechner machen kann.  Ich selbst habe nun endlich meinen Rechner zu Hause zu
einem Server umfunktioniert, der eigentlich rund um die Uhr läuft. Außerdem
verfüge ich über eine T-DSL Flatrate und einen DynDNS Account. Ich bin also von
jedem Rechner in der Welt aus erreichbar. Ja und? - Na das eröffnet doch
ungeahnte Möglichkeiten. Ich kann mich von überall in der Welt, aus jedem
System heraus auf meinem Rechner zu Hause einloggen und z.B. meine Emails
lesen. Ganz genau meine Emails, ich hab 4 verschiedene Adressen und die bei 2
verschiedenen Providern, d.h. wenn ich sie über das sehr unkomfortable
Webinterface lesen möchte, muß ich mich laufend ein- und ausloggen. Nicht so
mit der Lösung, die ich euch hier erkläre. Getmail besorgt mir meine Mails, die
ich mir dann z.B. mit dem `Mutt </anwendungen/Mutt>`__ anschauen und mit Hilfe
von Postfix beantworten kann.

Getmail versorgt die Emails
---------------------------

Zu allererst wollen wir unserem Server sagen, dass er uns unsere Emails
von den POP3-Konten holt. Dazu benötigen wir natürlich ein Programm. Das
bekannteste ist wohl Fetchmail, welches ich aber nicht leiden kann, weil
es bei mir ohne ersichtlichen Grund einfach mal ging und dann wieder
nicht ging. Ich habe mich für Getmail entschieden, welches, meiner
Meinung nach, schöner konfigurierbar und vor allem etwas geschwätziger
ist als Fetchmail.

Konfiguration der POP3-Accounts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

  Ich setze mal vorraus, dass ihr wisst, wie ihr Programme aus den
  Ports/Packages eures jeweiligen BSDs installiert.

Folgende Aktionen sollten wir (dann wieder als normaler User) ausführen:

::

   $ mkdir ~/.getmail
   $ vi ~/.getmail/getmailrc

Machen wir uns mal einen Kopf wie die Mails abgelegt werden sollen. Ich
hab es so gemacht, dass ich ein Verzeichnis ~/mail angelegt habe, in dem
dann für alle meine Konten Unterverzeichnisse liegen. Damit diese ins
"Maildir-Format" passen, müssen sie die Unterverzeichnisse new, cur, tmp
aufweisen:

::

   $ mkdir ~/mail/konto1
   $ touch ~/mail/konto1/new
   $ touch ~/mail/konto1/cur
   $ touch ~/mail/konto1/tmp

In meinem Fall hätte ich dann also folgende Struktur:

-  ~/mail

   -  konto1

      -  new
      -  cur
      -  tmp

   -  konto2

      -  new
      -  cur
      -  tmp

   -  usw.

Jetzt schreiben wir in die ``getmailrc`` unsere Konten und die ganzen
Zugangsinformationen hinein.

::

   [options]
   verbose = 1                 # zeig immer schön an was du gerade machst
   delete = True               # lösche die Nachrichten auf dem Server, nachdem du sie abgeholt hast

   [retriever]
   type = SimplePOP3Retriever  # hole die Mails über POP3
   server = $SERVER            # der Mailserver, z.B. pop3.freenet.de
   username = $USER            # hier kommt der Anmeldename für das Konto hin, meist ist das die Mailadresse
   password = $PASS            # Passwort für das Mailkonto

   [destination]
   type=Maildir
   path=~/mail/                # unser jeweiliges Verzeichnis (Maildir) z.B. ~/mail/konto1/
                               # Wichtig: Bei Benutzung von Maildir den Trailing Slash nicht vergessen!

Fertig. Nun noch ein schönes, herzhaftes:

::

   $ getmail

und Getmail rennt los und holt uns unsere Post ab.

Unter Getmail Version 4 kann man für jedes Konto ein eigenes
getmailrc-File schreiben. Wenn man für das erste Konto die Datei
getkonto1rc und für das zweite getkonto2rc erstellt hat, ruft man
getmail so auf:

::

   $ getmail --rcfile getkonto1rc --rcfile getkonto2rc

Postfix verschickt unsere Post
------------------------------

Jetzt geht es ans Post verschicken. Unter Windows z.B. macht das auch
der Mailclient, indem er die Mails über den Server des jeweiligen
Mailanbieters schickt. Selbstverständlich geht das auch mit FreeBSD auf
diesem Wege aber wir lösen das etwas cooler. Wir schicken die Mails über
unseren eigenen Mailserver ... ja genau der, der da unterm Schreibtisch
steht. Auch hierfür benötigen wir wieder ein Progrämmchen.

Standardmäßig bringt FreeBSD Sendmail mit. Das ist allerdings schon ganz
schön betagt und auch nicht sonderlich komfortabel. Ich entschied mich
für Postfix...

Konfiguration
~~~~~~~~~~~~~

Postfix ersetzt also unser altes Sendmail-System. Das muß man dem System
natürlich auch sagen. Dies geschieht unter FreeBSD, indem wir in die
``/etc/rc.conf`` folgendes eintragen:

::

   # Mailing Stuff - Postfix
   sendmail_enable="YES"
   sendmail_flags="-bd"
   sendmail_pidfile="/var/spool/postfix/pid/master.pid"
   sendmail_outbound_enable="NO"
   sendmail_submit_enable="NO"
   sendmail_msp_queue_enable="NO"

So, weiter im Text. Jetzt müssen wir nur noch ein paar kleine Optiönchen
einstellen und das geschieht in der Konfigurationsdatei von Postfix, die
neben anderen Configfiles, an denen ich mich nicht vergriffen habe, in
``/usr/local/etc/postfix`` liegt und den schönen Namen main.cf trägt.
Die Datei ist so gut dokumentiert, dass ich euch einfach mal machen
lasse. Es sollte kein Problem sein z.B. den Hostnamen einzustellen oder
sowas. Ich hab damals eigentlich außer "myhostname" und "mydomain" nix
geändert.

Fertig würd' ich sagen, wir können nun Mails senden und auch welche
empfangen. Nun fehlt nur noch ein schönes Programm, mit dem man das
komfortabel erledigen kann...

Einfacher geht's meistens mit ssmtp
-----------------------------------

Da ich meine Mails ja einfach nur an meinen Provider weitergeben will,
reicht auch ssmtp.

Unter FreeBSD muss man in der ``/usr/local/etc`` die Konfigurationsdatei
umbennen:

::

   $ cd /usr/local/etc/ssmtp/
   $ cp ssmtp.conf.sample ssmtp.conf

Hier die ``ssmtp.conf``:

::

   root=root
   mailhub=smtp.provider.de
   rewriteDomain=provider.de
   hostname=hos.tna.me

Sendmail verschickt unsere Post
-------------------------------

Oft will man alle Emails an einen Smart-Host senden. Dazu braucht man
nur das "Mail-Submision-Program".

**/etc/rc.conf** wird wie folgt gesetzt:

::

   sendmail_enable="NO"
   sendmail_submit_enable="NO"
   sendmail_outbound_enable="NO"
   sendmail_msp_queue_enable="YES"

Falls **/etc/mail/hostname.submit.mc** nicht existiert::

  # cd /etc/mail 
  # make

In **/etc/mail/hostname.submit.mc** wird die IP-Adresse des Smarthosts
geändert. Der Default ist "127.0.0.1".

::

   FEATURE(`msp',`[192.168.1.2]')dnl

Jetzt noch die Konfiguration kompilieren, installieren und den sendmail
mit den neuen Einstellung neu starten::

  # cd /etc/mail
  # make all install restart

* :ref:`genindex`

Zuletzt geändert: |date|

