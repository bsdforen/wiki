Postfix Antivirus
=================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Postfix wie gewohnt einrichten. Wie das geht wurde sicher schon irgendwo
erwähnt. ;)

.. note::

  Das ganze läuft hier in einer Jail, deshalb ist folgendes zu beachten! Zur
  Verwendung in einer Jail muss ``localhost`` oder ``127.0.0.1`` durch die
  Jail-IP bzw. den Namen der Jail ersetzt werden.

Mails sollen von Postfix an den Virenscanner gegeben werden und von
diesem zurück an Postfix zur weiteren Auslieferung, dazu öffnet das
Programm amavisd standardmässig den Port 10024 (frei einstellbar) und
nimmt auf diesem Port per smtp Mails entgegen. Nach erfolgter Prüfung
wird die Mail auf den standardmässig am Port 10025 lauschenden Postfix
zurück gegeben und weiter verteilt.

Um zum Mailvirenscanner zu gelangen, installiere man sich folgenden
Port:

-  `security/amavisd-new <https://www.google.com/search?q=security/amavisd-new&btnI=lucky>`__

Postfix muss der Virenscanner als "content_filter" bekannt gemacht
werden. Damit wird auch gleich geklärt wie die Mail weitergeleitet
werden soll. In diesem Falle per ``smtp`` an die IP ``192.168.0.10`` an
Port 10024, dahin werden wir ``amavisd`` binden. Beim Anpassen der
/usr/local/etc/postfix/main.cf entweder einen vorhandenen Eintrag
bearbeiten oder eine neue Zeile anlegen Zur Verwendung in meiner Jail
wurde ``localhost`` oder ``127.0.0.1`` durch die Jail-IP ersetzt.

``/usr/local/etc/postfix/main.cf``

::

   content_filter = smtp:192.168.0.10:10024

Um zu veranlassen, das Postfix auch auf Port 10025 die Mail wieder
zurücknimmt muss folgendes in der ``/usr/local/etc/postfix/master.cf``
vorhanden sein. Dies wird mit Installation des amavisd-new Ports
normalerweise automatisch erledigt. Zur Verwendung in einer Jail muss
``localhost`` oder ``127.0.0.1`` durch die Jail-IP ersetzt werden.
``/usr/local/etc/postfix/master.cf``

::

   192.168.0.10:10025 inet   n   -   n   -   -   smtpd -o content_filter=

Einige Zeilen der ``amavisd.conf`` sollte man noch anpassen, als
Grundlage habe ich ``amavisd.conf-sample`` verwendet. Aber auch
``amavisd.conf-default`` und ``amavisd.conf-dist`` sind einen Blick wert
wenn es um zusätzliche Einstellungen geht die in der ``conf.sample``
nicht aufgeführt sind.

``/usr/local/etc/amavisd.conf``

::

   $mydomain = 'otfa.lan';      # (no useful default)
   $myhostname = 'ip10.otfa.lan';  # fqdn of this host, default by uname(3)

Das wars auch schon, ansonsten ist man mit den Vorgaben gut bedient.
Will man ``amavisd`` in einer Jail betreiben, muss man noch die
folgenden Zeilen beachten, da er sonst versucht sich an die IP
``127.0.0.1`` zu binden und NUR dieser Zugriff gibt. Das führt in einer
Jail jedoch zu Problemen. Gleichzeitig sind dies die Zeilen die man
verändern muss, wenn amavisd auf einem anderen Recher als der Postfix
läuft.

::

   $forward_method = 'smtp:[192.168.0.10]:10025'; # Wohin soll die Mail nach der Prüfung (siehe postfix/master.cf)
   $inet_socket_bind = '192.168.0.10';            # An welche IP soll der Port gebunden werden
   @inet_acl = qw( 192.168.0.10 );                # Welche Rechner dürfen Mails zur Prüfung abgeben
   @mynetworks = qw( 192.168.0.0/24 );

Da ich nicht gebrauchen kann, das ausführbare Dateien für Windows nicht
ankommen habe ich die folgenden Zeilen zum Kommentar degradiert und eine
leere "new_RE()" angelegt.

::

   ###############
   ##$banned_filename_re = new_RE(
   ## qr'^UNDECIPHERABLE$',  # is or contains any undecipherable components
   #
   #  # block certain double extensions anywhere in the base name
   #  qr'\.[^./]*\.(exe|vbs|pif|scr|bat|cmd|com|cpl|dll)\.?$'i,
   #
   ## qr'[{}]',      # curly braces in names (serve as Class ID extensions - CLSID)
   #
   #  qr'^application/x-msdownload$'i,                  # block these MIME types
   #  qr'^application/x-msdos-program$'i,
   #  qr'^application/hta$'i,
   #
   ## qr'^message/partial$'i,                           # rfc2046 MIME type
   #
   ## qr'^message/external-body$'i,                     # rfc2046 MIME type
   ##    (btw, note that allowing 'message/external-body' is probably no worse
   ##    than allowing mail with HTML and/or allowing a user to browse the web)
   #
   ## [ qr'^\.(Z|gz|bz2)$'           => 0 ],  # allow any in Unix-compressed
   #  [ qr'^\.(rpm|cpio|tar)$'       => 0 ],  # allow any in Unix-type archives
   ## [ qr'^\.(zip|rar|arc|arj|zoo)$'=> 0 ],  # allow any within such archives
   #
   #  qr'.\.(exe|vbs|pif|scr|bat|cmd|com|cpl)$'i, # banned extension - basic
   ## qr'.\.(ade|adp|app|bas|bat|chm|cmd|com|cpl|crt|emf|exe|fxp|grp|hlp|hta|
   ##        inf|ins|isp|js|jse|lnk|mda|mdb|mde|mdw|mdt|mdz|msc|msi|msp|mst|
   ##        ops|pcd|pif|prg|reg|scr|sct|shb|shs|vb|vbe|vbs|
   ##        wmf|wsc|wsf|wsh)$'ix,  # banned ext - long
   #
   ## qr'.\.(mim|b64|bhx|hqx|xxe|uu|uue)$'i,  # banned extension - WinZip vulnerab.
   #
   #  qr'^\.(exe-ms)$',                       # banned file(1) types
   ## qr'^\.(exe|lha|tnef|cab|dll)$',         # banned file(1) types
   #);
   ###############
   $banned_filename_re = new_RE();

Beim gewünschten (und installiertem) Virenscanner die Kommentarzeichen
entfernen, hier nur als Beispiel ClamAV. Zu finden unter
`security/clamav <https://www.google.com/search?q=security/clamav&btnI=lucky>`__.

::

   ['ClamAV-clamd',
     \&ask_daemon, ["CONTSCAN {}\n", "/var/run/clamav/clamd"],
     qr/\bOK$/, qr/\bFOUND$/,
     qr/^.*?: (?!Infected Archive)(.*) FOUND$/ ],

Mails bei denen ein Virus gefunden wird, werden nach ``/var/virusmails``
gesichert.

Ein erfolgreicher Start sieht in ``/var/log/maillog`` in etwa so aus,
wichtig sind die Zeilen der erkannten Virenscanner. Manuell wird mit

::

   # /usr/local/sbin/amavisd

gestartet.

``/var/log/maillog``

::

   Feb  4 15:12:15 ip10 amavis[687]: starting.  /usr/local/sbin/amavisd at ip10.otfa.lan amavisd-new-2.2.1 (20041222), Unicode aware
   Feb  4 15:12:15 ip10 amavis[687]: Perl version               5.008005
   Feb  4 15:12:17 ip10 amavis[689]: Module Amavis::Conf        2.034
   Feb  4 15:12:17 ip10 amavis[689]: Module Archive::Tar        1.23
   Feb  4 15:12:17 ip10 amavis[689]: Module Archive::Zip        1.14
   Feb  4 15:12:17 ip10 amavis[689]: Module BerkeleyDB          0.26
   Feb  4 15:12:17 ip10 amavis[689]: Module Compress::Zlib      1.33
   Feb  4 15:12:17 ip10 amavis[689]: Module Convert::TNEF       0.17
   Feb  4 15:12:17 ip10 amavis[689]: Module Convert::UUlib      1.04
   Feb  4 15:12:17 ip10 amavis[689]: Module DBI                 1.46
   Feb  4 15:12:17 ip10 amavis[689]: Module DB_File             1.809
   Feb  4 15:12:17 ip10 amavis[689]: Module MIME::Entity        5.417
   Feb  4 15:12:17 ip10 amavis[689]: Module MIME::Parser        5.417
   Feb  4 15:12:17 ip10 amavis[689]: Module MIME::Tools         5.417
   Feb  4 15:12:17 ip10 amavis[689]: Module Mail::Header        1.66
   Feb  4 15:12:17 ip10 amavis[689]: Module Mail::Internet      1.66
   Feb  4 15:12:17 ip10 amavis[689]: Module Mail::SpamAssassin  3.000002
   Feb  4 15:12:17 ip10 amavis[689]: Module Net::Cmd            2.26
   Feb  4 15:12:17 ip10 amavis[689]: Module Net::DNS            0.48
   Feb  4 15:12:17 ip10 amavis[689]: Module Net::SMTP           2.29
   Feb  4 15:12:17 ip10 amavis[689]: Module Net::Server         0.87
   Feb  4 15:12:17 ip10 amavis[689]: Module Razor2::Client::Version 2.67
   Feb  4 15:12:17 ip10 amavis[689]: Module Time::HiRes         1.59
   Feb  4 15:12:17 ip10 amavis[689]: Module Unix::Syslog        0.100
   Feb  4 15:12:17 ip10 amavis[689]: Amavis::DB code        loaded
   Feb  4 15:12:17 ip10 amavis[689]: Amavis::Cache code     loaded
   Feb  4 15:12:17 ip10 amavis[689]: Lookup::SQL code       NOT loaded
   Feb  4 15:12:17 ip10 amavis[689]: Lookup::LDAP code      NOT loaded
   Feb  4 15:12:17 ip10 amavis[689]: AMCL-in protocol code  loaded
   Feb  4 15:12:17 ip10 amavis[689]: SMTP-in protocol code  loaded
   Feb  4 15:12:17 ip10 amavis[689]: ANTI-VIRUS code        loaded
   Feb  4 15:12:17 ip10 amavis[689]: ANTI-SPAM  code        loaded
   Feb  4 15:12:17 ip10 amavis[689]: Unpackers  code        loaded
   Feb  4 15:12:17 ip10 amavis[689]: Found $file       at /usr/bin/file
   Feb  4 15:12:17 ip10 amavis[689]: Found $arc        at /usr/local/bin/arc
   Feb  4 15:12:17 ip10 amavis[689]: Found $gzip       at /usr/bin/gzip
   Feb  4 15:12:17 ip10 amavis[689]: Found $bzip2      at /usr/bin/bzip2
   Feb  4 15:12:17 ip10 amavis[689]: Found $lzop       at /usr/local/bin/lzop
   Feb  4 15:12:17 ip10 amavis[689]: Found $lha        at /usr/local/bin/lha
   Feb  4 15:12:17 ip10 amavis[689]: Found $unarj      at /usr/local/bin/unarj
   Feb  4 15:12:17 ip10 amavis[689]: Found $uncompress at /usr/bin/uncompress
   Feb  4 15:12:17 ip10 amavis[689]: Found $unfreeze   at /usr/local/bin/unfreeze
   Feb  4 15:12:17 ip10 amavis[689]: Found $unrar      at /usr/local/bin/unrar
   Feb  4 15:12:17 ip10 amavis[689]: Found $zoo        at /usr/local/bin/zoo
   Feb  4 15:12:17 ip10 amavis[689]: Found $pax        at /bin/pax
   Feb  4 15:12:17 ip10 amavis[689]: Found $cpio       at /usr/bin/cpio
   Feb  4 15:12:17 ip10 amavis[689]: Found $ar         at /usr/bin/ar
   Feb  4 15:12:17 ip10 amavis[689]: Found $rpm2cpio   at /usr/local/bin/rpm2cpio.pl
   Feb  4 15:12:17 ip10 amavis[689]: Found $cabextract at /usr/local/bin/cabextract
   Feb  4 15:12:17 ip10 amavis[689]: No $ripole,       not using it
   Feb  4 15:12:17 ip10 amavis[689]: No $dspam,        not using it
   Feb  4 15:12:17 ip10 amavis[689]: Using internal av scanner code for (primary) ClamAV-clamd
   Feb  4 15:12:17 ip10 amavis[689]: Found primary av scanner H+BEDV AntiVir or CentralCommand Vexira Antivirus at /usr/bin/antivir
   Feb  4 15:12:17 ip10 amavis[689]: Found secondary av scanner ClamAV-clamscan at /usr/local/bin/clamscan
   Feb  4 15:12:17 ip10 amavis[689]: Creating db in /var/amavis/db/; BerkeleyDB 0.26, libdb 3.3

Anfangs bietet es sich an auf einer extra Konsole ``amavisd`` nicht als
daemon und im debug-Modus laufen zu lassen.

::

   # /usr/local/sbin/amavisd debug

Hier werden z.B. Fehler des Virenscanners mit angezeigt und wenn die
Installation nicht funktioniert sollte man das dort auch bis zum Fehler
zurückverfolgen können.

Läuft alles wie geplant, kann man in seiner ``/etc/rc.conf`` auch den
Start von amavisd erlauben

``/etc/rc.conf``

::

   amavisd_enable="YES"

Viel Spaß damit. --`XPectIT </wiki/user/XPectIT>`__ 16:59, 4. Feb 2005
(CET)

* :ref:`genindex`

Zuletzt geändert: |date|

