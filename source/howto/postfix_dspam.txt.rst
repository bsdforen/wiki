Postfix DSPAM
=============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

**How-To: Postfix + Courier-IMAP + [PostgreSQL oder MySQL] + amavisd-new
+ Clam-AV + DSPAM + Postfixadmin + Postgrey auf FreeBSD**

Dieses Dokument beschreibt die Installation und Konfiguration eines
Datenbank gestützten Mailservers auf Basis von FreeBSD, der über einen
sehr effektiven Spamfilter verfügt. Die spätere Administration erfolgt
über Postfixadmin und DSPAM's CGI's. Falsch erkannte Spammails können
später an die Adresse spam@ geschickt werden, damit der Filter sie
erneut trainiert. Das selbe gilt für als Spam erkannte "echte" Mails
(false-positives), hierfür gibt es dann notspam@. Das kann man natürlich
auch über das Webinterface von DSPAM machen. Desweiteren arbeitet der
Server mit Postgrey, welches dspam eine Menge Arbeit abnehmen sollte ;-)
Zudem wird mit Postfix Header-Checks und DNSBL-Listen gearbeitet, um den
groben Schmutz schon direkt am MTA abzufangen.

.. note::

  In diesem Artikel sind einige Fehler enthalten! Bitte daher die hier
  gemachten Angaben mit Vorsicht benutzen! Eine Überarbeitung findet später
  statt!

Einleitung
----------

.. note::

  Dieser Artikel beschreibt ein auf FreeBSD angepasstes Vorgehen.  Die
  Installation auf anderen Plattformen wird daher von diesem How-To nicht
  abgedeckt. Diese Installation auf Basis von MySQL wurde bereits einmal
  wissentlich nach diesem How-To erfolgreich durchgeführt. Die Version mit
  PostgreSQL ist durchgeführt, aber noch nicht als funktionierend von einem
  "Dritten" bestätigt.

**Sicherheit**

Das hier angesprochene Mailsystem wurde in den Produktivsystemen in
einer Jail installiert, die auf die IP-Adresse 127.0.0.2 hört. Wer
seinen Mailer nicht in einer Jail installieren möchte, der sollte bitte
die 127.0.0.2 durch 127.0.0.1, bzw localhost ersetzen, ggf. auch durch
die IP des Hosts, bzw. den richtigen Hostnamen (Dabei sollte man etwas
nachdenken! \*g*). Wer seine Passwörter nicht im Klartext über das
Internet übertragen mag, sollte sich SSL Vhost's erstellen. :)

Tip: Erstelle eine Jail, das ist einfach, schnell, sicher und gut. Mit
NAT und RDR in PF kann man dann ggf. den Rest erledigen. In den Ports
ist `ezjail </howto/ezjail>`__ zur schnellen Installation einer Jail zu
finden. Axel Gruners Jail-Howto eignet sich dafür hervorragend, um sich
Schritt für Schritt in Jails einzuarbeiten
http://www.grunix.de/doku/howto/files/jails.pdf.

**Kommentar**

Einige FLAVORS, z.B. amavisd-new mit MySQL- oder PostgreSQL-Support zu
kompilieren macht unter Umständen keinen Sinn, da diese keine
Auswirkungen auf dieses Setup haben. Es bleibt jedem selbst überlassen,
ob man später entsprechende Features wie z.B. Blacklisting in über eine
Datenbank benutzen will.

Im folgendem Text werden Installationswege auf 2 unterschiedlichen
Datenbanken aufgezeigt. Die Qual der Wahl: PostgreSQL oder MySQL? Hier
sollte man das nehmen, womit man sich besser auskennt bzw. entscheiden
ob es um "mal schnell" oder Datenintegrität und benötigte Features geht.

**Vorsicht**

Dieses How-To kann man nicht einfach so runterrattern, ohne sich
gedanken zu machen, was man hier überhaupt tut. Wie schon eben kurz
erwähnt, werden hier 2 Installationswege, und zwar für PostgreSQL oder
MySQL beschrieben. Die eigentliche Installation, egal mit welcher
Datenbank sieht ziemlich gleich aus. Es gibt nur ein paar kleine
Unterschiede, die im Text dann entsprechend markiert sind.

**Dankeschön**

Vielen Dank an Saintjoe für den Live-Test und das "Adden" von weiteren
Features. :-)

**Vorbereitung**

**``/etc/make.conf``**

::

   WITH_BDB_VER=43
   WITH_PGSQL=yes # oder WITH_MYSQL=yes
   CLAMUSER=vscan
   CLAMGROUP=vscan
   WITH_APACHE_SUEXEC=yes
   DSPAM_OWNER=dspam
   DSPAM_GROUP=dspam
   WITH_AUTHDAEMON=yes # für cyrus-sasl2

Installation der Software
-------------------------

**Datenbank-Server**

MySQL::

  # cd /usr/ports/databases/mysql50-server
  # make package clean

oder PostgreSQL::

  # cd /usr/ports/databases/postgresql80-server
  # make package clean

Cyrus-SASL2::

  # cd /usr/ports/security/cyrus-sasl2
  # make package clean

Postfix

Options: SASL2, SPF, TLS, DB43, [MySQL oder PGSQL] und VDA::

  # cd /usr/ports/mail/postfix
  # make package clean

amavisd-new

Options: [MySQL oder PGSQL]::

  # cd /usr/ports/security/amavisd-new
  # make package clean

Clam-AV::

  # cd /usr/ports/security/clamav
  # make package clean

Courier-IMAP

Options: OPENSSL, [AUTH_MYSQL oder AUTH_PGSQL] (Bei Bedarf noch IPV6)::

  # cd /usr/ports/mail/courier-imap
  # make package clean

Apache::

  # cd /usr/ports/www/apache13-modssl
  # make package clean

DSPAM

Options: DEBUG, DAEMON, GRAHAM_BAYES, BURTON_BAYES, RPV, TEST_COND, TRUSTED_USERS, [MYSQL50 oder PGSQL], VIRT_USERS, LONG_USERNAMES, LARGE_SCALE, SENDMAIL_LDA und CGI

Achtung: die UID/GID müssen nicht 20000 haben, sie sollten allerdings über 1000 liegen, damit Apache suexec mit dem dspam User funktioniert::

  # pw groupadd dspam -g 20000
  # pw useradd dspam -u 20000 -g dspam -s "/sbin/nologin" -d "/var/db/dspam" -c "DSPAM User"
  # cd /usr/ports/mail/dspam
  # make package clean

Postfixadmin::

  # cd /usr/ports/mail/postfixadmin
  # make package clean

php4-session::

  # cd /usr/ports/www/php4-session
  # make package clean

Folgenden Port bei PostgreSQL zusätzlich installieren:

php4-pgsql::

  # cd /usr/ports/databases/php4-pgsql
  # make package clean

Postgrey::

  # cd /usr/ports/mail/postgrey
  # make package clean

Konfiguration
-------------

Datenbank-Server
----------------

Bei MySQL::

  # echo "mysql_enable=\"YES\"" >> /etc/rc.conf
  # /usr/local/etc/rc.d/mysql-server.sh start
  # mysqladmin password NEWMYSQLPW

Bei PostgreSQL::

  # echo "postgresql_enable=\"YES\"" >> /etc/rc.conf
  # /usr/local/etc/rc.d/010.pgsql.sh initdb
  # cp /usr/local/etc/periodic/daily/502.pgsql /etc/periodic/daily/

/usr/local/pgsql/data/pg_hba.conf ← Entsprechend bearbeiten::

  host    postfix     postfixadmin    127.0.0.2/32      trust
  host    postfix     postfix         127.0.0.2/32      trust
  host    dspam       dspam           127.0.0.2/32      trust

/usr/local/pgsql/data/postgresql.conf ← Entsprechend bearbeiten::

  listen_addresses = '127.0.0.2'

Hinweis: An dieser Stelle sollte man sich die Dokumentation zu PostgreSQL 8
anschauen. Das Tuning eines PostgreSQL Servers ist kein Kinderspiel, wenn man
sich damit nicht auskennt.

PostgreSQL starten::

  # /usr/local/etc/rc.d/010.pgsql.sh start

Cyrus-SASL2
-----------

``/usr/local/lib/sasl2/smtpd.conf`` <- Erstellt diese Datei mit
folgendem Inhalt.

::

   pwcheck_method: authdaemond
   log_level: 3
   mech_list: LOGIN DIGEST-MD5 CRAM-MD5
   authdaemond_path:/var/run/authdaemond/socket

Postfix
-------

Ein SSL Zertifikat für Postfix erstellen::

  # mkdir /usr/local/etc/postfix/ssl
  # cd /usr/local/etc/postfix/ssl
  # openssl req -new -x509 -nodes -out smtpd.pem -keyout smtpd.pem -days 3650

Verzeichniss erstellen und Permissions setzen::

  # mkdir /usr/local/virtual
  # chown postfix:postfix /usr/local/virtual
  # chmod -R 771 /usr/local/virtual

Gruppenangehörigkeit erstellen::

  # pw groupmod courier -m postfix


``/usr/local/etc/postfix/main.cf`` <- Entsprechend bearbeiten.

::

   smtpd_client_restrictions  =
       permit_mynetworks, 
       permit_sasl_authenticated,
       reject_rbl_client dnsbl.sorbs.net,
       reject_rbl_client sbl-xbl.spamhaus.org,
       reject_rbl_client list.dsbl.org,
       permit
   smtpd_helo_restrictions  =
       permit_mynetworks,
       permit_sasl_authenticated,
       reject_invalid_hostname,
       reject_non_fqdn_hostname,
       permit
   smtpd_sender_restrictions  =
       reject_unknown_sender_domain,
       reject_non_fqdn_sender,
       permit_mynetworks,
       permit_sasl_authenticated,
       reject_rhsbl_sender rhsbl.sorbs.net,
       reject_rhsbl_sender dsn.rfc-ignorant.org,
       permit
   smtpd_recipient_restrictions  =
       reject_unknown_recipient_domain,
       reject_non_fqdn_recipient,
       permit_mynetworks,
       permit_sasl_authenticated,
       reject_unauth_destination,
       check_policy_service inet:127.0.0.2:10023,
       permit
   smtpd_data_restrictions  =
       permit_mynetworks,
       reject_unauth_pipelining,
       permit

   smtpd_use_tls = yes
   smtpd_tls_key_file = /usr/local/etc/postfix/ssl/smtpd.pem
   smtpd_tls_cert_file = /usr/local/etc/postfix/ssl/smtpd.pem
   smtpd_tls_CAfile = /usr/local/etc/postfix/ssl/smtpd.pem
   smtpd_tls_loglevel = 1
   smtpd_tls_received_header = yes
   smtpd_tls_session_cache_timeout = 3600s
   tls_random_source = dev:/dev/urandom

   smtpd_sasl_auth_enable = yes
   smtpd_sasl2_auth_enable = yes
   smtpd_sasl_security_options = noanonymous
   smtpd_sasl_local_domain = $myhostname

   header_checks = pcre:/usr/local/etc/postfix/header_checks.pcre

Bei MySQL:

::

   relay_domains = proxy:mysql:/usr/local/etc/postfix/sql_relay_domains_maps.cf
   transport_maps = mysql:/usr/local/etc/postfix/sql_transport_maps.cf, pcre:/usr/local/etc/postfix/transport.pcre
   virtual_alias_maps = mysql:/usr/local/etc/postfix/sql_virtual_alias_maps.cf
   virtual_gid_maps = static:125
   virtual_mailbox_base = /usr/local/virtual
   virtual_mailbox_domains = mysql:/usr/local/etc/postfix/sql_virtual_domains_maps.cf
   virtual_mailbox_limit = 51200000
   virtual_mailbox_maps = $transport_maps, mysql:/usr/local/etc/postfix/sql_virtual_mailbox_maps.cf
   virtual_minimum_uid = 125
   virtual_transport = virtual
   virtual_uid_maps = static:125
   virtual_create_maildirsize = yes
   virtual_mailbox_extended = yes
   virtual_mailbox_limit_maps = mysql:/usr/local/etc/postfix/sql_virtual_mailbox_limit_maps.cf
   virtual_mailbox_limit_override = yes  
   virtual_maildir_limit_message = Sorry, the user's maildir has overdrawn his diskspace quota, please try again later.
   virtual_overquota_bounce = yes

Bei PostgreSQL:

::

   relay_domains = proxy:pgsql:/usr/local/etc/postfix/sql_relay_domains_maps.cf
   transport_maps = pgsql:/usr/local/etc/postfix/sql_transport_maps.cf, pcre:/usr/local/etc/postfix/transport.pcre
   virtual_alias_maps = pgsql:/usr/local/etc/postfix/sql_virtual_alias_maps.cf
   virtual_gid_maps = static:125
   virtual_mailbox_base = /usr/local/virtual
   virtual_mailbox_domains = pgsql:/usr/local/etc/postfix/sql_virtual_domains_maps.cf
   virtual_mailbox_limit = 51200000
   virtual_mailbox_maps = $transport_maps, pgsql:/usr/local/etc/postfix/sql_virtual_mailbox_maps.cf
   virtual_minimum_uid = 125
   virtual_transport = virtual
   virtual_uid_maps = static:125
   virtual_create_maildirsize = yes
   virtual_mailbox_extended = yes
   virtual_mailbox_limit_maps = pgsql:/usr/local/etc/postfix/sql_virtual_mailbox_limit_maps.cf
   virtual_mailbox_limit_override = yes
   virtual_maildir_limit_message = Sorry, the user's maildir has overdrawn his diskspace quota, please try again later.
   virtual_overquota_bounce = yes

``/usr/local/etc/postfix/master.cf`` <- Entsprechend bearbeiten.

Auskommentieren:

::

   smtp      inet  n       -       n       -       -       smtpd
   pickup    fifo  n       -       n       60      1       pickup
   cleanup   unix  n       -       n       -       0       cleanup
   local     unix  -       n       n       -       -       local

Anhängen:

::

   smtp                 inet      n      -      n      -      -      smtpd
     -o cleanup_service_name=pre-cleanup
     -o content_filter=smtp-amavis:[[127.0.0.2]]:10024
     
   pickup               fifo      n      -      n      60     1      pickup
     -o cleanup_service_name=pre-cleanup

   smtp-amavis          unix      -      -      n      -      2      lmtp
     -o smtp_send_xforward_command=yes

   127.0.0.2:10025      inet      n      -      n      -      -      smtpd
     -o cleanup_service_name=pre-cleanup
     -o content_filter=dspam:dummy
     -o local_recipient_maps =
     -o relay_recipient_maps =
     -o smtpd_restriction_classes =
     -o smtpd_client_restrictions =
     -o smtpd_helo_restrictions =
     -o smtpd_sender_restrictions =
     -o smtpd_recipient_restrictions=permit_mynetworks,reject
     -o mynetworks=127.0.0.0/8
     -o strict_rfc821_envelopes=yes
     -o smtpd_error_sleep_time=0
     -o smtpd_soft_error_limit=1001
     -o smtpd_hard_error_limit=1000

   127.0.0.2:10026      inet      n      -      n      -      -      smtpd
     -o local_recipient_maps =
     -o relay_recipient_maps =
     -o smtpd_restriction_classes =
     -o smtpd_client_restrictions =
     -o smtpd_helo_restrictions =
     -o smtpd_sender_restrictions =
     -o smtpd_recipient_restrictions=permit_mynetworks,reject
     -o mynetworks=127.0.0.0/8
     -o strict_rfc821_envelopes=yes
     -o smtpd_error_sleep_time=0
     -o smtpd_soft_error_limit=1001
     -o smtpd_hard_error_limit=1000

   dspam                unix   -   n   n   -   -   pipe
     flags=Rhq user=dspam argv=/usr/local/bin/dspam --mode=teft --user global \
     --deliver=innocent,spam --feature=chained,noise -i -f ${sender} -- ${recipient}

   dspam-spam           unix   -   n   n   -   -   pipe
     flags=Rhq user=dspam argv=/usr/local/bin/dspam --mode=teft --user global \
     --class=spam --source=error ${sender} --deliver=spam

   dspam-notspam        unix   -   n   n   -   -   pipe
     flags=Rhq user=dspam argv=/usr/local/bin/dspam --mode=teft --user global \
     --class=innocent --source=error ${sender} --deliver=innocent

   cleanup              unix      n      -      n      -      0      cleanup
     -o header_checks =
     -o mime_header_checks =
     -o nested_header_checks =
     -o body_checks =

   pre-cleanup          unix      n      -      n      -      0      cleanup
     -o canonical_maps =
     -o sender_canonical_maps =
     -o recipient_canonical_maps =
     -o masquerade_domains =
     -o always_bcc =
     -o sender_bcc_maps =
     -o recipient_bcc_maps =

   local                unix      -      n      n      -      -      local
     -o content_filter =
     -o myhostname=HOSTNAME
     -o local_recipient_maps =
     -o relay_recipient_maps =
     -o mynetworks=127.0.0.0/8
     -o mynetworks_style=host
     -o smtpd_restriction_classes =
     -o smtpd_client_restrictions =
     -o smtpd_helo_restrictions =
     -o smtpd_sender_restrictions =
     -o smtpd_recipient_restrictions=permit_mynetworks,reject

``/usr/local/etc/postfix/transport.pcre`` <- Erstellt diese Datei mit
folgendem Inhalt.

::

   /^spam@(.*)$/       dspam-spam:${1}
   /^notspam@(.*)$/    dspam-notspam:${1}

``/usr/local/etc/postfix/header_checks.pcre`` <- Erstellt diese Datei
mit folgendem Inhalt.

::

  -  Following will block mails with Asian and Cyrillic charsets which is almost spam.
  /^Content-Type:.*charset="?(big5|euc-jp|euc-kr|euc-tw|gb2312|iso-2022-jp|koi8|ks_c_5601-1987|windows-1251)"?/   REJECT Sorry, we do not accept messages in the ${1} character set.
  /^(From|Subject): .*=\?(big5|euc-jp|euc-kr|euc-tw|gb2312|iso-2022-jp|koi8|ks_c_5601-1987|windows-1251)\?/   REJECT Sorry, we do not accept messages in the ${2} character set.

    -  Following will block mails with potential virus infected attachements.
    - /^Content-(Type|Disposition):.*(file)?name=.*\.(ade|adp|asd|bas|bat|chm|cmd|com|cpl|crt|dbx|dll|exe|hlp|hta|js|jse|lnk|ocx|pif|scr|shb|shm|shs|vb|vbe|vbs|vbx|vxd|wsf|wsh)/ REJECT Sorry, we do not accept .${3} file types.

    -  These are headers used to track some spam messages.
  /^X-Spam-Flag: YES/     WARN SpamAssassin Confirmed Spam Content
  /^Bel-Tracking: .*/     REJECT Confirmed spam. Go away.
  /^Hel-Tracking: .*/     REJECT Confirmed spam. Go away.
  /^Kel-Tracking: .*/     REJECT Confirmed spam. Go away.
  /^BIC-Tracking: .*/     REJECT Confirmed spam. Go away.
  /^Lid-Tracking: .*/     REJECT Confirmed spam. Go away.

    -  Following will block mails marked as junk.
  /^Precedence: junk/     REJECT Confirmed spam. Go away.

    -  Emails with eronious dates (or dates far in the past) will appear at the top or bottom of your mail client.
  /^Date: .* 19[[0-9]][[0-9]]/    REJECT UBE Header - Past Date #1
  /^Date: .* 200[[0-4]]/      REJECT UBE Header - Past Date #2

    -  This filter will block subjects that contain ISO specifications.
    - /^Subject: .*\=\?ISO/       REJECT We don't accept strange character sets.

    -  This will block messages that do not have an address in the From: header.
  /^From: <>/         REJECT You need to specify a return address, otherwise we will not accept your email.

    -  This will block messages that do not have an address in the Return-Path:  header.
  /^Return-Path: <>/      REJECT You need to specify a return address, otherwise we will not accept your email.

    -  Following is a listing of known mass mailer programs.
  /^X-Mailer: .*0001/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*007 Direct Email Easy/                REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*2\.0-b55-VC_IPA/                  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Advanced Mass Sender/             REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Aristotle/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Aureate Group Mail/               REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Avalanche/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Caretop 2604/                 REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Copia emailFacts/                 REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Crescent Internet Tool/               REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*CyberCreek/                   REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*DMailer/                      REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Delphi Mailing System/                REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*DiffondiCool/                 REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Direct Email/                 REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Dynamic Opt-In Emailer/               REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*E-Access/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*E-Mail Delivery Agent/                REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*E-mail sender/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Emailer Platinum/                 REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Entity/                       REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*EVAMAIL/                      REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Extractor/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Floodgate/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*GMail2/                       REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*GOTO Software Sarbacane/              REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*GoldMine/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*GreenRider/                   REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*GRMessageQueue/                   REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Inet_Mail_Out/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*JiXing .{0,30}Design By JohnnieHuang/     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Mail Bomber/                  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Mail Sender/                  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*MailKing/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*MailPro/                      REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*MailWorkZ/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*MailWorkz/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*MailXSender/                  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Mailloop/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*MassE-Mail/                   REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*MaxBulk.Mailer/                   REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Microsoft CDO/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Microsoft Outlook Express 4.72.1712.3/        REJECT Sorry, your mailer was identified as spam mailing software.
  /^X-Mailer: .*Microsoft Outlook Express 5.00.2919.6900 DM/  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*MMailer v3\.0/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Mozilla 4.55/                 REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*MultiMailer/                  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*NetMasters SMTP/                  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*News Breaker Pro/                 REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Out[[Ll]]ook Express 3\.14159/            REJECT Sorry, your mailer was identified as spam mailing software.
  /^X-Mailer: .*Opt-In Lightning/                 REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*PLAUZIUM/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*PersMail/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Power CGI Bulk/                   REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*PowerCampaign/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Prospect Mailer/                  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*RoryMAILER/                   REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*SmartMailer/                  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Sparc12/                      REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*StormPort/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*StormPost/                    REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*Super-Duper-FastMail/             REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*SuperMail-2/                  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*THOR/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*bulk/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*charset(89)/                  REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*commercialmail/                   REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*demography opalescent/                REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*diffondi/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*e-Merge/                      REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*eGroups Message Poster/               REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*jfmailer/                     REJECT Sorry, your mailer was identified as mass mailer program.
  /^X-Mailer: .*jpfree Group Mail Express/            REJECT Sorry, your mailer was identified as mass mailer program.

    -  Some Wordcombinations
  /^Subject: .*Free Money/        REJECT UBE Header - Rule #1

    -  Following Will Block Spams With Many Spaces In The Subject.
  /^Subject: .*            /      REJECT UBE Header - 12 Spaces

    -  Following will block spams with $ values in the subject
  /^Subject: \$[[0-9]][[0-9]]*$/      REJECT Sorry, your mail was identified as spam.

``/usr/local/etc/postfix/sql_virtual_alias_maps.cf`` <- Erstellt diese
Datei mit folgendem Inhalt.

::

   user = postfix
   password = POSTFIXSQLPW
   hosts = 127.0.0.2
   dbname = postfix
   table = alias
   select_field = goto
   where_field = address

``/usr/local/etc/postfix/sql_virtual_domains_maps.cf`` <- Erstellt diese
Datei mit folgendem Inhalt.

::

   user = postfix
   password = POSTFIXSQLPW
   hosts = 127.0.0.2
   dbname = postfix
   table = domain
   select_field = description
   where_field = domain
   additional_conditions = and backupmx = '0' and active = '1'

``/usr/local/etc/postfix/sql_virtual_mailbox_maps.cf`` <- Erstellt diese
Datei mit folgendem Inhalt.

::

   user = postfix
   password = POSTFIXSQLPW
   hosts = 127.0.0.2
   dbname = postfix
   table = mailbox
   select_field = maildir
   where_field = username
     - additional_conditions = and active = '1'

``/usr/local/etc/postfix/sql_virtual_mailbox_limit_maps.cf`` <- Erstellt
diese Datei mit folgendem Inhalt.

::

   user = postfix
   password = POSTFIXSQLPW
   hosts = 127.0.0.2
   dbname = postfix
   table = mailbox
   select_field = quota
   where_field = username
     - additional_conditions = and active = '1'

``/usr/local/etc/postfix/sql_relay_domains_maps.cf`` <- Erstellt diese
Datei mit folgendem Inhalt.

::

   user = postfix
   password = POSTFIXSQLPW
   hosts = 127.0.0.2
   dbname = postfix
   table = domain
   select_field = domain
   where_field = domain
   additional_conditions = and backupmx = '1'

``/usr/local/etc/postfix/sql_transport_maps.cf`` <- Erstellt diese Datei
mit folgendem Inhalt.

::

   user = postfix
   password = POSTFIXSQLPW
   hosts = 127.0.0.2
   dbname = postfix
   table = domain
   select_field = transport
   where_field = domain
   additional_conditions = and backupmx = '1'

amavisd-new
-----------

::

  # cp /usr/local/etc/amavisd.conf-sample /usr/local/etc/amavisd.conf

``/usr/local/etc/amavisd.conf`` <- Entsprechend bearbeiten.

::

   $mydomain = 'example.org';      # (no useful default)
   $myhostname = 'hostname.example.org';  # fqdn of this host, default by uname(3)
   $forward_method = 'smtp:[[127.0.0.2]]:10025';  # where to forward checked mail
   $notify_method = 'smtp:[[127.0.0.2]]:10026';   # where to submit notifications
   @bypass_spam_checks_maps  = (1);  # uncomment to DISABLE anti-spam code
   $inet_socket_bind = '127.0.0.2'; # limit socket bind to loopback interface
   @inet_acl = qw(127.0.0.2);  # allow SMTP access only from localhost IP
   @mynetworks = qw( 127.0.0.0/8 );
   $final_virus_destiny      = D_DISCARD;  # (defaults to D_DISCARD)
   $final_banned_destiny     = D_DISCARD;  # (defaults to D_BOUNCE)
   $warnvirussender = 1; # (defaults to false (undef))
   $warnbannedsender = 1;        # (defaults to false (undef))
   $virus_admin = "virus-admins\@$mydomain";

   [['ClamAV-clamd',
     \&ask_daemon, [["CONTSCAN|{}\n", "/var/run/clamav/clamd"]],
     qr/\bOK$/, qr/\bFOUND$/,
     qr/^.*?: (?!Infected Archive)(.*) FOUND$/ ]],

Clam-AV
-------

Manchmal sind Ports doof. :-)) ::

  # chown -R vscan:vscan /var/run/clamav 
  # chown -R vscan:vscan /var/db/clamav 
  # chown -R vscan:vscan /var/log/clamav

``/usr/local/etc/clamd.conf`` <- Erstellt diese Datei mit folgendem
Inhalt.

::

   LogSyslog
   LogVerbose
   LogFacility LOG_MAIL
   PidFile /var/run/clamav/clamd.pid
   DatabaseDirectory /var/db/clamav
   LocalSocket /var/run/clamav/clamd
   StreamMaxLength 10M
   User vscan
   ScanMail
   ScanArchive

``/usr/local/etc/freshclam.conf`` <- Erstellt diese Datei mit folgendem
Inhalt.

::

   LogSyslog
   LogVerbose
   LogFacility LOG_MAIL
   DatabaseOwner vscan
   Checks 12
   DatabaseMirror db.de.clamav.net

Courier-IMAP
------------

``/usr/local/etc/authlib/authdaemonrc`` <- Entsprechend bearbeiten.

Bei MySQL folgenden Wert ändern:

::

   authmodulelist="authmysql"

Bei PostgreSQL folgenden Wert ändern:

::

   authmodulelist="authpgsql"

Bei MySQL:

``/usr/local/etc/authlib/authmysqlrc`` <- Erstellt diese Datei mit
folgendem Inhalt.

::

     - DEFAULT_DOMAIN      example.org
   MYSQL_SOCKET        /tmp/mysql.sock
   MYSQL_USERNAME      postfix
   MYSQL_PASSWORD      POSTFIXSQLPW
   MYSQL_DATABASE      postfix
   MYSQL_OPT       0
   MYSQL_USER_TABLE    mailbox
   MYSQL_CRYPT_PWFIELD password
   MYSQL_UID_FIELD     '125'
   MYSQL_GID_FIELD     '125'
   MYSQL_LOGIN_FIELD   username
   MYSQL_HOME_FIELD    '/usr/local/virtual'
   MYSQL_NAME_FIELD    name
   MYSQL_MAILDIR_FIELD maildir
   MYSQL_QUOTA_FIELD   quota
     - MYSQL_PORT      0
   MYSQL_SERVER        127.0.0.2

Bei PostgreSQL:

``/usr/local/etc/authlib/authpgsqlrc`` <- Erstellt diese Datei mit
folgendem Inhalt.

::

     - DEFAULT_DOMAIN      example.com
   PGSQL_HOST      /tmp
   PGSQL_PORT      5432
   PGSQL_USERNAME      postfix
   PGSQL_PASSWORD      POSTFIXSQLPW
   PGSQL_DATABASE      postfix
     - PGSQL_OPT       0
   PGSQL_USER_TABLE    mailbox
   PGSQL_CRYPT_PWFIELD password
   PGSQL_UID_FIELD     '125'
   PGSQL_GID_FIELD     '125'
   PGSQL_LOGIN_FIELD   username
   PGSQL_HOME_FIELD    '/usr/local/virtual'
   PGSQL_NAME_FIELD    name
   PGSQL_MAILDIR_FIELD maildir
   PGSQL_QUOTA_FIELD   quota

Wer möchte, kann natürlich auch SSL für den imapd bzw pop3d benutzen.
Dazu müssen 2 Dateien in **/usr/local/etc/courier-imap/** angepasst
werden:

In den Dateien pop3d.cnf/imapd.cnf (als Vorlage
pop3d.cnf.dist/imapd.cnf.dist benutzen) den [ req_dn ] Abschnitt
anpassen:

::

   [ req_dn ]
   C=countryName Two letters!
   ST=stateOrProvinceName
   L=localityName
   O=organizationName
   OU=OrganizationalUnitName
   CN=ÖFFENTLICHE_IP_DES_HOSTS_ODER_DER_JAIL
   emailAddress=emailAddress

Danach::

  # /usr/local/sbin/mkimapdcert 
  # /usr/local/sbin/mkpop3dcert

ausführen.

Apache
------

(Beispiel für die Einrichtung der DSPAM CGI's und Postfixadmin mit
VHost's)

``/usr/local/etc/apache/httpd.conf`` <- Entsprechend bearbeiten.

Suchen:

::

     -  AddHandler cgi-script .cgi

Ersetzen:

::

   AddHandler cgi-script .cgi

Anhängen:

::

   NameVirtualHost xxx.xxx.xxx.xxx:80

   <VirtualHost xxx.xxx.xxx.xxx:80>
       DocumentRoot "/usr/local/www/data/org/example/dspam/html"
       ServerName dspam.example.org
       ServerAdmin webmaster@example.org
       ErrorLog /usr/local/www/data/org/example/dspam/logs/dspam.example.org-error_log
       CustomLog /usr/local/www/data/org/example/dspam/logs/dspam.example.org-access_log common

       User dspam
       Group dspam
           
       <Directory "/usr/local/www/data/org/example/dspam/html"> 
           Options FollowSymLinks ExecCGI
           AllowOverride None
           Order deny,allow
           Deny from all   
           AuthType Basic
           AuthName "DSPAM Control Center"
           AuthUserFile /usr/local/www/data/org/example/dspam/etc/.htpasswd
           Require valid-user  
           Satisfy Any
       </Directory>
   </VirtualHost>

   <VirtualHost xxx.xxx.xxx.xxx:80>
       DocumentRoot "/usr/local/www/data/org/example/postfixadmin"
       ServerName postfixadmin.example.org
       ServerAdmin webmaster@example.org
       ErrorLog /usr/local/www/data/org/example/postfixadmin/logs/postfixadmin.example.org-error_log
       CustomLog /usr/local/www/data/org/example/postfixadmin/logs/postfixadmin.example.org-access_log common

       <Directory "/usr/local/www/data/org/example/postfixadmin/admin">
           Options FollowSymLinks
           AllowOverride None
           Order deny,allow
           Deny from all
           AuthType Basic
           AuthName "Postfixadmin"
           AuthUserFile /usr/local/www/data/org/example/postfixadmin/etc/.htpasswd
           Require valid-user
           Satisfy Any
       </Directory>
   </VirtualHost>

Verzeichnisse für Postfixadmin und DSPAM CGI's vorbereiten:

::

   # cd /usr/local/www/data
   # mkdir -p org/example
   # cd org/example
   # cp -R ../../../vhosts/dspam .
   # mkdir dspam/html
   # mv dspam/* dspam/html/
   # mkdir dspam/etc
   # mkdir dspam/logs
   # chmod 555 dspam
   # cd dspam
   # chmod 550 etc
   # chown www:dspam etc
   # chmod 555 html
   # chown dspam:dspam html
   # cd html
   # chown -R dspam:dspam *
   # chmod 444 *.*
   # chmod 554 *.cgi
   # chmod 555 templates
   # chmod 444 templates/*
   # cp admins.sample admins
   # cp configure.pl.sample configure.pl
   # cp default.prefs.sample default.prefs
   # ln -s /usr/local/www/data/org/example/dspam/html/default.prefs /var/db/dspam/default.prefs

Bei MySQL:

::

   # cd /usr/local/www/data/org/example/
   # cp -R ../../../postfixadmin .
   # mkdir postfixadmin/etc
   # mkdir postfixadmin/logs
   # chown -R root:www postfixadmin
   # rm postfixadmin/admin/.htaccess

Bei PostgreSQL:

Die aktuelle Version von Postfixadmin 2.10 funktioniert nicht
einwandfrei mit PostgreSQL, eine gepatchte Version gibt es hier:

::

   # cd ~
   # fetch ftp://ftp.logos-bg.net/pub/Nikola/Postfixadmin/postfixadmin-2.1.0.tar.gz
   # tar xvfz postfixadmin-2.1.0.tar.gz
   # cd /usr/local/www/data
   # mkdir -p org/example
   # cd org/example
   # cp -R ~/postfixadmin-2.1.0 postfixadmin
   # mkdir postfixadmin/etc
   # mkdir postfixadmin/logs
   # chown -R root:www postfixadmin
   # rm postfixadmin/admin/.htaccess

.htpasswd's erstellen:

::

   htpasswd -c /usr/local/www/data/org/example/postfixadmin/etc/.htpasswd postfixadmin
   htpasswd -c /usr/local/www/data/org/example/dspam/etc/.htpasswd root
   htpasswd /usr/local/www/data/org/example/dspam/etc/.htpasswd global

DSPAM
-----

Datenbank erstellen:

Bei MySQL:

::

   # mysql -u root -p
   # mysql> create database dspam;
   # mysql> grant all on dspam.* to dspam@127.0.0.2 identified by 'DSPAMSQLPW';
   # mysql> quit;
   # cd /usr/local/share/examples/dspam/mysql/
   # mysql dspam -u dspam -p < mysql_objects-4.1.sql
   # mysql dspam -u dspam -p < virtual_users.sql

Bei PostgreSQL:

::

   # su - pgsql
   # createuser -P dspam
   # createdb dspam
   # psql dspam
   dspam=# \i /usr/local/share/examples/dspam/pgsql/pgsql_objects.sql
   dspam=# \i /usr/local/share/examples/dspam/pgsql/virtual_users.sql
   dspam=# GRANT ALL ON # dspam_neural_data,dspam_neural_decisions,dspam_preferences,dspam_signature_data,dspam_stats,dspam_token_data,dspam_virtual_uids,dspam_virtual_uids_seq TO dspam;
   dspam=# alter table "dspam_token_data" alter "token" set statistics 200; analyze;
   dspam=# alter table "dspam_signature_data" alter "signature" set statistics 200; analyze;
   dspam=# alter table "dspam_neural_data" alter "node" set statistics 200; analyze;
   dspam=# alter table "dspam_neural_decisions" alter "signature" set statistics 200; analyze;
   dspam=# alter table "dspam_token_data" alter "innocent_hits" set statistics 200; analyze;
   dspam=# alter table "dspam_token_data" alter "spam_hits" set statistics 200; analyze;
   dspam=# \q

Datei kopieren:

Bei MySQL:

::

   # cp purge-4.1.sql /var/db/dspam/

Bei PostgreSQL:

::

   # cp purge.sql /var/db/dspam/

CRON-Jobs erstellen:

Bei MySQL:

::

   0       0       *       *       *       dspam   /usr/local/bin/dspam_logrotate -a 30 /var/db/dspam/system.log `find /var/db/dspam/data -name "*.log"`
   0       0       *       *       *       dspam   /usr/local/bin/mysql -u dspam -p'DSPAMMYSQLPW' dspam < /var/db/dspam/purge-4.1.sql

Bei PostgreSQL:

::

   0       0       *       *       *       dspam   /usr/local/bin/dspam_logrotate -a 30 /var/db/dspam/system.log `find /var/db/dspam/data -name "*.log"`

CRON rehashen:

::

   # kill -HUP `cat /var/run/cron.pid`

Dateien bearbeiten / erstellen:

::

   # cp /usr/local/etc/dspam.conf-sample /usr/local/etc/dspam.conf

``/usr/local/etc/dspam.conf``

Bei MySQL folgendes verändern:

::

   MySQLServer     /tmp/mysql.sock
   MySQLPort
   MySQLUser       dspam
   MySQLPass       DSPAMSQLPW
   MySQLDb         dspam
   MySQLCompress   true

Bei PostgreSQL folgendes verändern:

::

   PgSQLServer     127.0.0.2
   PgSQLPort       5432
   PgSQLUser       dspam
   PgSQLPass       DSPAMSQLPW
   PgSQLDb         dspam

Berechtigungen setzen:

::

   # chown -R dspam:dspam /var/db/dspam
   # chown dspam:dspam /usr/local/etc/dspam.conf

``/usr/local/www/data/org/example/dspam/html/configure.pl``

::

     - !/usr/local/bin/perl

     -  Default DSPAM enviroment
   $CONFIG{'DSPAM_HOME'}   = "/var/db/dspam";
   $CONFIG{'DSPAM_BIN'}    = "/usr/local/bin";
   $CONFIG{'DSPAM'}        = $CONFIG{'DSPAM_BIN'} . "/dspam";
   $CONFIG{'DSPAM_STATS'}  = $CONFIG{'DSPAM_BIN'} . "/dspam_stats";
   $CONFIG{'DSPAM_ARGS'}   = "--deliver=innocent --class=innocent " .
                             "--source=error --user %CURRENT_USER% -d %u";
   $CONFIG{'TEMPLATES'}    = "./templates";        # Location of HTML templates
   $CONFIG{'ALL_PROCS'}    = "ps auxw";            # use ps -deaf for Solaris
   $CONFIG{'MAIL_QUEUE'}   = "mailq | grep '^[[0-9,A-F]]' | wc -l";

     -  Default DSPAM display
   $CONFIG{'HISTORY_SIZE'} = 200;          # Number of items in history
   $CONFIG{'MAX_COL_LEN'}  = 50;           # Max chars in list columns
   $CONFIG{'SORT_DEFAULT'} = "Date";       # Show quarantine by "Date" or "Rating"
   $CONFIG{'3D_GRAPHS'}    = 1;
   $CONFIG{'LOCAL_DOMAIN'} = "HOSTNAME";

     -  Add customized settings below
   $CONFIG{'LOCAL_DOMAIN'} = "example.org";

   $ENV{'PATH'} = "$ENV{'PATH'}:$CONFIG{'DSPAM_BIN'}";

     -  Autodetect filesystem layout and preference options
     - $CONFIG{'AUTODETECT'} = 1;

     -  Or, if you're running dspam.cgi as untrusted, it won't be able to auto-detect
     -  so you will need to specify some features manually:
   $CONFIG{'AUTODETECT'} = 0;
   $CONFIG{'LARGE_SCALE'} = 1;
   $CONFIG{'DOMAIN_SCALE'} = 0;
   $CONFIG{'PREFERENCES_EXTENSION'} = 1;

   $CONFIG{'DSPAM_CGI'} = "dspam.cgi";

     -  Configuration was successful
   1;

Trainingscorpus downloaden, entpacken und DSPAM trainieren:

::

   # cd /root/
   # fetch http://wiki.bsdforen.de/files/dspam_sa_trainer-bsdforen.tar.gz
   # tar xvfz dspam_sa_trainer-bsdforen.tar.gz
   # cd dspam_sa_trainer
   # ./publiccorpus.pl global

Postfixadmin
------------

Bei MySQL:

``/usr/local/www/data/org/example/postfixadmin/DATABASE_MYSQL.TXT`` <-
Entsprechend bearbeiten.

::

   INSERT INTO user (Host, User, Password) VALUES ('127.0.0.2','postfix',password('POSTFIXSQLPW'));
   INSERT INTO db (Host, Db, User, Select_priv) VALUES ('127.0.0.2','postfix','postfix','Y');
     -  Postfix Admin user & password
   INSERT INTO user (Host, User, Password) VALUES ('127.0.0.2','postfixadmin',password('POSTFIXADMINSQLPW'));
   INSERT INTO db (Host, Db, User, Select_priv, Insert_priv, Update_priv, Delete_priv) VALUES ('127.0.0.2', 'postfix', 'postfixadmin', 'Y', 'Y', 'Y', '
   Y');
   FLUSH PRIVILEGES;
   GRANT USAGE ON postfix.* TO postfix@127.0.0.2;
   GRANT SELECT, INSERT, DELETE, UPDATE ON postfix.* TO postfix@127.0.0.2;
   GRANT USAGE ON postfix.* TO postfixadmin@127.0.0.2;
   GRANT SELECT, INSERT, DELETE, UPDATE ON postfix.* TO postfixadmin@127.0.0.2;

Datenbank erstellen:

::

   # mysql -u root -p < /usr/local/www/data/org/example/postfixadmin/DATABASE_MYSQL.TXT
   # rm /usr/local/www/data/org/example/postfixadmin/DATABASE_MYSQL.TXT

``/usr/local/www/data/org/example/postfixadmin/config.inc.php`` <-
Entsprechend bearbeiten.

::

   $CONF[['database_type']] = 'mysql';
   $CONF[['database_host']] = '127.0.0.2';
   $CONF[['database_user']] = 'postfixadmin';
   $CONF[['database_password']] = 'POSTFIXADMINSQLPW';
   $CONF[['database_name']] = 'postfix';
   $CONF[['database_prefix']] = //;

Bei PostgreSQL:

::

   # su - pgsql
   # createuser -P postfixadmin
   # createuser -P postfix
   # createuser -P vacation
   # createdb postfix
   # psql postfix
   postfix=# \i /root/postfixadmin-2.1.0/DATABASE_PGSQL.TXT
   postfix=# GRANT ALL ON admin,alias,domain,domain_admins,log,mailbox,vacation,vacation_notification TO postfixadmin;
   postfix=# GRANT SELECT ON alias,domain,mailbox TO postfix;
   postfix=# \q
   logout

Verändert diese Werte:

``/usr/local/www/data/org/example/postfixadmin/config.inc.php`` <-
Entsprechend bearbeiten.

::

   $CONF[['database_type']] = 'pgsql';
   $CONF[['database_host']] = '127.0.0.2';
   $CONF[['database_user']] = 'postfixadmin';
   $CONF[['database_password']] = 'POSTFIXADMINSQLPW';
   $CONF[['database_name']] = 'postfix';
   $CONF[['database_prefix']] = //;

REST
----

Die ``/etc/rc.conf`` um folgende Werte erweitern:

::

   clamav_clamd_enable="YES"
   amavisd_enable="YES"
   sendmail_enable="YES"
   sendmail_flags="-bd"
   sendmail_pidfile="/var/spool/postfix/pid/master.pid"
   sendmail_outbound_enable="NO"
   sendmail_submit_enable="NO"
   sendmail_msp_queue_enable="NO"
   apache_enable="YES"
   courier_authdaemond_enable="YES"
   courier_imap_pop3d_enable="YES"
   courier_imap_imapd_enable="YES"
   courier_imap_imapd_ssl_enable="YES" # Nur wenn SSL benutzt werden soll
   courier_imap_pop3d_ssl_enable="YES" # Nur wenn SSL benutzt werden soll
   postgrey_enable="YES"

Jetzt die Dämonen starten und GO! :-)

Fehler
------

Hier wurde ein komplexes Mailsystem eingerichtet. Spätere Veränderungen
können auch Fehler enthalten!

Wir wollen dieses How-To stets erweitern und funktionsfähig halten,
postet eure Bugs bitte im Forum oder sucht in
``irc.freenode.net / #bsdforen.de`` erfahrene Hilfe.

Hinweis des Bären: in der /etc/login.conf hinzufügen / ändern:
:maxproc=500: (gegen Fork Bomben) wäre noch sinnvoll. (danach cap_mkdb
/etc/login.conf nicht vergessen)

* :ref:`genindex`

Zuletzt geändert: |date|

