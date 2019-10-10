Postfix Courier OpenLDAP
========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Diese HowTo beschriebt die Installation und Einrichtung eines Mailservers unter
OpenBSD. Die einzelen Komponenten setzen sich wie folgt zusammen

-  Postfix, smtp
-  Courier, imap
-  SASL, zur Authentifizierung
-  OpenLDAP, als Userdatenbank
-  Postgrey, für das [http://de.wikipedia.org/wiki/Greylisting
   Greylisting]
-  amavisd-new, zum Weiterleiten der Mails an den Virenscanner
-  ClamAV, als Virenscanner

.. note::

  Die Grundlagen der einzelnen Komponenten sollten klar sein, insbesondere die
  Funktionsweise von Postfix und OpenLDAP.

.. note::

  Dieses Anleitung sollte man nicht, wie eigentlich alle Anleitungen, einfach
  abtippen. Sondern man ist angehalten, sich selber über die Gefahren eines
  Mailsystems auf dem eigenen System zu informieren. Es kann immer mal
  vorkommen, dass sich Fehler einschleichen können, dies ist auch hier nicht
  auszuschliessen. Diese Anleitung ist nicht getestet und mehr aus dem Kopf
  entstanden ;). Dieses System ist ähnlich auf mehreren Servern im produktiven
  Betrieb.

Installation der Software
-------------------------

**Courier**

::

   # cd /usr/ports/mail/courier-imap
   # env FLAVOR=" no_mysql no_pgsql" make install clean
   # cp -r /usr/local/share/examples/courier-imap/ /etc/

**Cyrus-SASL**

::

   # cd /usr/ports/security/cyrus-sasl2
   # make install clean

**OpenLDAP-Client**

::

   # cd /usr/ports/databases/openldap
   # env FLAVOR="sasl" make install clean

**OpenLDAP-Server**

::

   # cd /usr/ports/databases/openldap
   # env FLAVOR="sasl" SUBPACKAGE="-server" make install clean

**Postfix**

::

   # cd /usr/ports/mail/postfix
   # env FLAVOR="sasl2 ldap" make install clean
   # /usr/local/sbin/postfix-enable

**Postgrey**

::

   # cd /usr/ports/mail/postgrey
   # make install clean

Ports sind manchmal blöde. Daher muss bei Postgrey noch ein bisschen
Hand angelegt werden. Dafür legen wir anfangs ein Gruppe und einen User
für Postgrey an.

::

   # groupadd -g 534 postgrey
   # useradd -s /sbin/nologin -d /var/empty -G 534 -u 534 _postgrey

Danach erstellen wir die Files und Verzeichniss für Postgrey

::

   # touch /etc/postfix/postgrey_whitelist_clients
   # touch /etc/postfix/postgrey_whitelist_recipients
   # mkdir /var/spool/postfix/postgrey
   # chown -R _postgrey:_postgrey /var/spool/postfix/postgrey
   # chown _postgrey:_postgrey /etc/postfix/postgrey_whitelist_clients /etc/postfix/postgrey_whitelist_recipients

**ClamAV**

::

   # cd /usr/ports/security/clamav
   # make install clean

**amavisd-new**

::

   # cd /usr/ports/mail/amavisd-new
   # make install clean

**Anmerkung**

Natürlich kann man auch die entsprechenden Packages installieren.

``/etc/rc.conf.local`` editieren und die flags von sendmail und syslogd
hinzufügen

::

   # vi /etc/rc.conf.local

::

   sendmail_flags="-bd -q30m"
   syslogd_flags="-a /var/spool/postfix/dev/log"

/etc/rc.local konfigurieren
---------------------------

Jetzt noch ``/etc/rc.local`` editieren und die Dienste starten

::

   # vi /etc/rc.local

.. code:: bash

   # openldap
   if [ -x /usr/local/libexec/slapd ]; then
          echo -n ' slapd';  
          /usr/local/libexec/slapd
   fi

   # courier
   mkdir -p /var/run/courier-imap
   /usr/local/libexec/authlib/authdaemond start
   /usr/local/libexec/imapd.rc start

   # postgrey 
   if [ -x /usr/local/libexec/postgrey ]; then
          echo -n 'Postgrey';
          /usr/local/libexec/postgrey --inet=10023 -d --delay=50 --greylist-text="Policy restrictions; try later" --user=_postgrey --group=_postgrey
   fi

   # clamav
   if [ -x /usr/local/sbin/clamd ]; then
          echo -n 'clamd';
          /usr/local/sbin/clamd
   fi

   # amavisd-new
   if [ -x /usr/local/sbin/amavisd ]; then
          echo -n 'amavisd';
          /usr/local/sbin/amavisd
   fi

OpenLDAP konfigurieren
----------------------

slapd.conf
~~~~~~~~~~

Konfiguration vom OpenLDAP-Server, die folgenden Schemen werden
mindestens benötigt. Das qmail.schema enthält die Attribute
mailMessageStore und mailAlternateAdress, welche wir für die einzelnen
Userkonten benötigen.

Der Schemacheck ist aktiviert, so dass es nicht möglich ist Attribute
anzulegen, welche nicht durch Schemen definiert sind. Als
Datenbankbackend wird ldbm genutzt, es ist aber auch möglich die
BerkelyDB zu nutzen.

::

   include         /etc/openldap/schema/core.schema
   include         /etc/openldap/schema/cosine.schema
   include         /etc/openldap/schema/nis.schema
   include         /etc/openldap/schema/inetorgperson.schema
   include         /etc/openldap/schema/qmail.schema

   pidfile         /var/run/slapd.pid
   argsfile        /var/run/slapd.args
   loglevel        30
   schemacheck     on

   # this is for the postfix authentification
   allow bind_v2

   #######################################################################
   # BDB database definitions
   #######################################################################

   database        ldbm
   suffix          "dc=example,dc=de"
   rootdn          "cn=Manager,dc=example,dc=de"
   # Cleartext passwords, especially for the rootdn, should
   # be avoid.  See slappasswd(8) and slapd.conf(5) for details.
   # Use of strong authentication encouraged.
   rootpw          {MD5}asdfzasdfIMOZnasdof=
   # The database directory MUST exist prior to running slapd AND
   # should only be accessible by the slapd and slap tools.
   Mode 700 recommended.
   directory       /var/openldap-data
   # Indices to maintain
   index   objectClass, eq

qmail.schema
~~~~~~~~~~~~

Das qmail.schema ist nicht unbedingt schnell zu finden und teilweise
werden inkompatible Versionen angezeigt. Aus diesem Grunde wird dieses
Schema hier noch einmal aufgeführt

::

   #
   # qmail-ldap (20030901) ldapv3 directory schema
   #
   # The offical qmail-ldap OID assigned by IANA is 7914
   #
   # Created by: David E. Storey <dave@tamos.net>
   # Modified and included into qmail-ldap by Andre Oppermann <opi@nrg4u.com>
   # Schema fixes by Mike Jackson <mjj@pp.fi>
   # Schema fixes by Christian Zoffoli (XMerlin) <czoffoli@xmerlin.org>
   #
   #
   # This schema depends on:
   # - core.schema
   # - cosine.schema
   # - nis.schema
   #

   # Attribute Type Definitions

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.1 NAME 'qmailUID'
    DESC 'UID of the user on the mailsystem'
    EQUALITY numericStringMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.36 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.2 NAME 'qmailGID'
    DESC 'GID of the user on the mailsystem'
    EQUALITY numericStringMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.36 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.3 NAME 'mailMessageStore'
    DESC 'Path to the maildir/mbox on the mail system'
    EQUALITY caseExactIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.4 NAME 'mailAlternateAddress'
    DESC 'Secondary (alias) mailaddresses for the same user'
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} )

   #
   # mailQuota format is no longer supported from qmail-ldap 20030901 on,
   # user mailQuotaSize and mailQuotaCount instead.
   #
   #attributetype ( 1.3.6.1.4.1.7914.1.2.1.5 NAME 'mailQuota'
   # DESC 'The amount of space the user can use until all further messages get bounced.'
   # SYNTAX 1.3.6.1.4.1.1466.115.121.1.44 SINGLE-VALUE )
   # 
     
   attributetype ( 1.3.6.1.4.1.7914.1.2.1.6 NAME 'mailHost'
    DESC 'On which qmail server the messagestore of this user is located.'
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} SINGLE-VALUE)
    
   attributetype ( 1.3.6.1.4.1.7914.1.2.1.7 NAME 'mailForwardingAddress'
    DESC 'Address(es) to forward all incoming messages to.'
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.8 NAME 'deliveryProgramPath'
    DESC 'Program to execute for all incoming mails.'
    EQUALITY caseExactIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.9 NAME 'qmailDotMode'
    DESC 'Interpretation of .qmail files: both, dotonly, ldaponly, ldapwithprog'
    EQUALITY caseIgnoreIA5Match
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{32} SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.10 NAME 'deliveryMode'
    DESC 'multi field entries of: nolocal, noforward, noprogram, reply'
    EQUALITY caseIgnoreIA5Match
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{32} )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.11 NAME 'mailReplyText'
    DESC 'A reply text for every incoming message'
    EQUALITY caseIgnoreMatch
    SUBSTR caseIgnoreSubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{4096} SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.12 NAME 'accountStatus'
    DESC 'The status of a user account: active, noaccess, disabled, deleted'
    EQUALITY caseIgnoreIA5Match
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.14 NAME 'qmailAccountPurge'
    DESC 'The earliest date when a mailMessageStore will be purged'
    EQUALITY numericStringMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.36 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.15 NAME 'mailQuotaSize'
    DESC 'The size of space the user can have until further messages get bounced.'
    EQUALITY integerMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.16 NAME 'mailQuotaCount'
    DESC 'The number of messages the user can have until further messages get bounced.'
    EQUALITY integerMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.2.1.17 NAME 'mailSizeMax'
    DESC 'The maximum size of a single messages the user accepts.'
    EQUALITY integerMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 SINGLE-VALUE )

   #
   # qmailGroup attributes
   #

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.1 NAME 'dnmember'
    DESC 'Group member specified as distinguished name.'
    EQUALITY distinguishedNameMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.2 NAME 'rfc822member'
    DESC 'Group member specified as normal rf822 email address.'
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.3 NAME 'filtermember'
    DESC 'Group member specified as ldap search filter.'
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{512} )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.4 NAME 'senderconfirm'
    DESC 'Sender to Group has to answer confirmation email.'
    EQUALITY booleanMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.5 NAME 'membersonly'
    DESC 'Sender to Group must be group member itself.'
    EQUALITY booleanMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.6 NAME 'confirmtext'
    DESC 'Text that will be sent with sender confirmation email.'
    EQUALITY caseIgnoreMatch
    SUBSTR caseIgnoreSubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{4096} SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.7 NAME 'dnmoderator'
    DESC 'Group moderator specified as Distinguished name.'
    EQUALITY distinguishedNameMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.8 NAME 'rfc822moderator'
    DESC 'Group moderator specified as normal rfc822 email address.'
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.9 NAME 'moderatortext'
    DESC 'Text that will be sent with request for moderation email.'
    EQUALITY caseIgnoreMatch
    SUBSTR caseIgnoreSubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{4096} SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.10 NAME 'dnsender'
    DESC 'Allowed sender specified as distinguished name.'
    EQUALITY distinguishedNameMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.11 NAME 'rfc822sender'
    DESC 'Allowed sender specified as normal rf822 email address.'
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} )

   attributetype ( 1.3.6.1.4.1.7914.1.3.1.12 NAME 'filtersender'
    DESC 'Allowed sender specified as ldap search filter.'
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{512} )


   #
   # qldapAdmin Attributes
   #
    
   attributetype ( 1.3.6.1.4.1.7914.1.4.1.1 NAME 'qladnmanager'
    DESC ''
    EQUALITY distinguishedNameMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 )

   attributetype ( 1.3.6.1.4.1.7914.1.4.1.2 NAME 'qlaDomainList'
    DESC ''
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} )

   attributetype ( 1.3.6.1.4.1.7914.1.4.1.3 NAME 'qlaUidPrefix'
    DESC ''
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.4.1.4 NAME 'qlaQmailUid'
    DESC ''
    EQUALITY integerMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.4.1.5 NAME 'qlaQmailGid'
    DESC ''
    EQUALITY integerMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.4.1.6 NAME 'qlaMailMStorePrefix'
    DESC ''
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.4.1.7 NAME 'qlaMailQuotaSize'
    DESC ''
    EQUALITY integerMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.4.1.8 NAME 'qlaMailQuotaCount'
    DESC ''
    EQUALITY integerMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.4.1.9 NAME 'qlaMailSizeMax'
    DESC ''
    EQUALITY integerMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 SINGLE-VALUE )

   attributetype ( 1.3.6.1.4.1.7914.1.4.1.10 NAME 'qlaMailHostList'
    DESC ''
    EQUALITY caseIgnoreIA5Match
    SUBSTR caseIgnoreIA5SubstringsMatch
    SYNTAX 1.3.6.1.4.1.1466.115.121.1.26{256} )


   # Object Class Definitions

   objectclass ( 1.3.6.1.4.1.7914.1.2.2.1 NAME 'qmailUser'
    DESC 'QMail-LDAP User'
    SUP top
    AUXILIARY
    MUST ( mail )
    MAY ( uid $ mailMessageStore $ homeDirectory $ userPassword $
          mailAlternateAddress $ qmailUID $ qmailGID $
          mailHost $ mailForwardingAddress $ deliveryProgramPath $
          qmailDotMode $ deliveryMode $ mailReplyText $
          accountStatus $ qmailAccountPurge $
          mailQuotaSize $ mailQuotaCount $ mailSizeMax ) ) 

   objectclass ( 1.3.6.1.4.1.7914.1.3.2.1 NAME 'qmailGroup'
    DESC 'QMail-LDAP Group'
    SUP top
    AUXILIARY
    MUST ( mail $ mailAlternateAddress $ mailMessageStore )
    MAY ( dnmember $ rfc822member $ filtermember $ senderconfirm $
          membersonly $ confirmtext $ dnmoderator $ rfc822moderator $
          moderatortext $ dnsender $ rfc822sender $ filtersender) )

   objectclass ( 1.3.6.1.4.1.7914.1.4.2.1 NAME 'qldapAdmin'
    DESC 'QMail-LDAP Subtree Admin'
    SUP top
    AUXILIARY
    MUST ( qlaDnManager $ qlaDomainList $ qlaMailMStorePrefix $
           qlaMailHostList )
    MAY ( qlaUidPrefix $ qlaQmailUid $ qlaQmailGid $ qlaMailQuotaSize $
          qlaMailQuotaCount $ qlaMailSizeMax ) )

LDAP-Struktur
~~~~~~~~~~~~~

::

           base
            |
      -------------
      |           |
    system       user
      |
   postfix
      |
    -----
    |   |
   eu  de

Unter base liegen system und user. In system liegt postfix und darin die
jeweiligen Domains geordnet nach TLDs (Top Level Domains). Denn ist
auszugegehen, dass der Mailserver für einige Domains genutzt werden
soll. In user liegen die ganzen Daten der einzelnen Userkonten.

Diese Abbildung muss eventuell an die eigenen Bedürfnisse angepasst
werden und sollte nicht einfach übernommen werden. Ein wichtiger Schritt
bei der Benutzung von LDAP ist die passende Baumstruktur für seinen
individuellen Anwendungsfall zu entwickeln.

**Anlegen der LDAP Hierarchie**

**Initialisieren**

::

   dn: dc=example, dc=de
   objectClass: top
   objectClass: organization
   objectClass: dcObject
   dc: example
   o: example.de

Die oberste Struktur im Verzeichnisbaum wird angelegt

**Systemabschnitt**

::

   dn: ou=system, dc=example, dc=de
   ou: system
   objectClass: organizationalUnit

Es wird eine Systemabschnitt angelegt, der bei Bedarf noch weitere
Unterbäume enthalten kann. Dieser kann auch wegfallen.

**Struktur für die Domains**

::

   dn: ou=postfix, ou=system, dc=example, dc=de
   ou: postfix
   objectClass: organizationalUnit

**Ablage der Domains**

::

   dn: dc=eu, ou=postfix, ou=system, dc=example, dc=de
   associatedDomain: example.eu
   dc: eu
   objectClass: dNSDomain
   objectClass: domainRelatedObject

   dn: dc=de, ou=postfix, ou=system, dc=example, dc=de
   associatedDomain: example.de
   associatedDomain: example2.de
   dc: de
   objectClass: dNSDomain
   objectClass: domainRelatedObject

Wie man hier sieht, sind die einzelnen TLDs in getrennten Strukturen.
Dies dient zur besseren Übersicht. Man kann natürlich alle Domains auch
in einer Struktur verwalten.

**Struktur für die Userverwaltung**

::

   dn: ou=user, dc=bossk, dc=de
   ou: user
   objectClass: organizationalUnit

**Anlegen der einzelnen User**

::

   dn: cn=foo, ou=user, dc=example, dc=de
   userPassword:{MD5}CY9rzasdfsaP56ZDJi04d==
   givenName: foo
   sn: bar
   mailMessageStore: /var/mail/foo/
   mail: admin@example.de
   mailAlternateAddress: foo@example2.de
   mailAlternateAddress: bar@example.eu
   uid: foo@example2.de
   accountStatus: active
   cn: foo
   objectClass: top
   objectClass: person
   objectClass: inetOrgPerson
   objectClass: qmailUser
   objectClass: organizationalPerson

Das userPassword wird mit dem Tool ``/usr/local/sbin/slappasswd``
erstellt. Es kann pro Account nur einmal mail existieren, weitere
eMail-Adressen des Accounts können unter mailAlternateAddress
eingestellt werden. Über den accountStatus können Accounts deaktiviert
werden. Die uid wird für die Anmeldungen an Postfix und Courier genutzt.
Der Rest dürfte eindeutig sein.

Courier-IMAP konfigurieren
--------------------------

**/etc/courier-imap/imapd**

::

   # vi /etc/courier-imap/imapd

::

   ADDRESS=<IP auf der, der Server laufen soll>

Sollte man einstellen, wenn der Server mehrer IP's hat

**/etc/courier-imap/authdaemonrc**

::

   # vi /etc/courier-imap/authdaemonrc

::

   authmodulelist="authldap"

Diese Einstellung kann man machen. Die default Einstellung funktioniert
auch.

**/etc/courier-imap/authldaprc**

::

   vi /etc/courier-imap/authldaprc

::

   LDAP_SERVER             localhost 
   LDAP_PORT               389
   LDAP_PROTOCOL_VERSION   3
   LDAP_BASEDN             ou=user,dc=example,dc=de
   LDAP_TIMEOUT            15
   LDAP_MAIL               uid
   LDAP_FILTER             (accountStatus=active)
   LDAP_GLOB_UID           _postfix
   LDAP_GLOB_GID           _postfix
   LDAP_HOMEDIR            mailMessageStore
   LDAP_MAILDIR            mailMessageStore
   LDAP_DEFAULTDELIVERY    defaultDelivery
   LDAP_FULLNAME           cn
   LDAP_CRYPTPW            userPassword
   LDAP_DEREF              never

**Anmerkung** Natürlich kann und sollte man sich auch den Rest der
Einstellungen anschauen und ggf. seinen Anforderungen anpassen.

ClamAV konfigurieren
--------------------

Die Dateien ``/etc/freshclam.conf`` und ``/etc/clamd.conf`` entsprechend
seinen Präferenzen anpassen. In diesem Beispiel wird der Daemon von
ClamAV gestartet und amavisd-new übergibt diesem die Mails zum scannen.
Dafür müssen in der ``/etc/clamd.conf`` ein paar Änderungen durchgeführt
werden.

Die nachfolgenden Zeilen müssen angepasst bzw. auskommentiert werden.

::

   PidFile /var/run/clamav/clamd.pid 
   LocalSocket /var/run/clamav/clamd

Da das Verzeichnis ``/var/run/clamav`` noch nicht existiert wird es
angelegt und die Berechtigungen werden gesetzt

::

   # mkdir /var/run/clamav && chown -R _clamav:_clamav /var/run/clamav

Damit die Virendatenbanken von ClamAV laufend aktualisiert werden
empfiehlt es sich einen Cronjob für freshclam anzulegen.

::

   # crontab -l
   # clamav aktualisierung
   0       */4     *       *       *       /usr/local/bin/freshclam >/dev/null 2>&1

Postfix konfigurieren
---------------------

.. note::

  Es wird nur auf die Einstellung, die für die Verbindung zum LDAP-Server
  benötigt eingegangen. Postfix ist ein komplexes Programm, daher sollte man
  sich damit auseinander setzen. Mehr Informationen zur Konfiguration findet
  man auch in dem Artikel `Postfix Dspam <Postfix Dspam>`__ oder im Netz und in
  der gängigen Fachliteratur.

**/etc/postfix/main.cf**

::

   # vi /etc/postfix/main.cf

::

   mydestination = localhost, ldap:acceptdomains
   acceptdomains_server_host = localhost
   acceptdomains_server_port = 389
   acceptdomains_bind = no
   acceptdomains_search_base = ou=postfix,ou=system,dc=example,dc=de
   acceptdomains_query_filter = (associatedDomain=%s)
   acceptdomains_result_attribute = associatedDomain

Mit diesen Einträgen, werden die Domains gesucht.

::

   local_transport = virtual
   virtual_mailbox_base = /
   virtual_mailbox_maps = ldap:ldapvirtual
   virtual_uid_maps = static:507
   virtual_gid_maps = static:507
   virtual_mailbox_limit = 0
   ldapvirtual_server_host = localhost
   ldapvirtual_server_port = 389
   ldapvirtual_bind = no
   ldapvirtual_search_base = ou=user,dc=example,dc=de
   ldapvirtual_query_filter= (&(|(mail=%s)(mailAlternateAddress=%s))(|(accountStatus=active)(accountStatus=shared)))
   ldapvirtual_result_attribute = mailMessageStore

Hier wird der Speicherplatz für den passenden Account gesucht.
``Wichtig`` ist, dass die virtual_uid_maps und virtual_gid_maps an den
\_postfix-Account des Systems angepasst wird. Standard unter OpenBSD ist
507.

::

   virtual_alias_maps = ldap:ldapalias
   ldapalias_server_host = localhost
   ldapalias_server_port = 389
   ldapalias_bind = no
   ldapalias_search_base = ou=user,dc=example,dc=de
   ldapalias_query_filter = (&(|(mail=%s)(mailAlternateAddress=%s))(|(accountStatus=active)(accountStatus=shared)))
   ldapalias_result_attribute = mail

Jetzt fügen wir die Restrictions hinzu

::

   smtpd_recipient_restrictions = permit_mynetworks,
                                 permit_sasl_authenticated,
                                 reject_unauth_pipelining,
                                 reject_unauth_destination,
                                 reject_invalid_hostname,
                                 reject_non_fqdn_hostname,
                                 reject_non_fqdn_sender,
                                 reject_non_fqdn_recipient,
                                 reject_rbl_client relays.ordb.org,
                                 reject_rbl_client list.dsbl.org,
                                 reject_rbl_client opm.blitzed.org,
                                 reject_rbl_client sbl-xbl.spamhaus.org,
                                 reject_rhsbl_sender dsn.rfc-ignorant.org,
                                 check_policy_service inet:127.0.0.1:10023
                                 permit

Es sinnig (fast) alle restrictions in die smtpd_recipient_restrictions
zu setzen, da die Restrictions bei der Standardkonfiguration nach dem
RCPT TO: ausgeführt werden und nicht wie allgemein angenommen bei dem
entsprechenden Vorgang. Im Klartext: Die smtpd_helo_restrictions werden
nicht beim HELO geprüpft sondern auch erst nach dem RCPT TO:

to be continued...

* :ref:`genindex`

Zuletzt geändert: |date|

