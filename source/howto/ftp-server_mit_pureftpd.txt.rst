FTP-Server mit PureFTPd
=======================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

In diesem Artikel wird das Aufsetzen einer FTP-Server-Maschine beschrieben. Auf
diesem System sollen zwei FTP-Serverdienste (im Folgenden "Server" genannt)
laufen. Anonymes FTP soll nicht zugelassen sein. Der eine der beiden Server
soll Clients als Zugang dienen, die über eine feste IP-Adresse verfügen. Der
andere bedient Clients mit dynamischer IP-Adresse. Diese Verbindungen sollen
aus Sicherheitsgründen mit Hilfe von TLS aufgebaut werden. Es soll mit
virtuellen Usern gearbeitet werden.

Vorbereitungen
--------------

Als erstes benötigt man eine Maschine mit einem lauffähigen
OpenBSD-System. Diese sollte voll gepatcht sein.

Da wir beide Server nicht über inetd laufen lassen wollen, schalten wir
diesen in der ``/etc/rc.conf`` ab:

::

   inetd=NO

Weiterhin installieren wir pure-ftpd als paket. Dieses ist auf jedem
OpenBSD-Mirror (z.B. ``ftp.de.openbsd.org``) im Verzeichnis
``/pub/OpenBSD/3.8/packages/i386`` zu finden.

.. note::

  Die Versionsnummern sollten natürlich an die verwendete OpenBSD-Version
  angepasst werden.

Man lädt es auf seine Maschine und installiert es mit

::

   # pkg_add -v /local/path/to/pure-ftpd-VERSION.tgz

In unserem Beispiel verwende ich die Standardversion und verzichte auf
MySQL-, LDAP- oder Postgres-Unterstützung.

Um mit virtuellen Usern arbeiten zu können müssen wir eine reale Gruppe
und einen realen User anlegen:

::

   # groupadd ftpgroup
   # useradd -g ftpgroup -d /dev/null -s /etc ftpuser

Weiterhin benötigen wir ein Verzeichnis, in welchem die Verzeichnisse
der virtuellen Accounts angelegt werden.

::

   # mkdir -p /data/ftp

Funktionsweise
--------------

Die beiden Server werden mit unterschiedlichen Startparametern aus der
``/etc/rc.local`` heraus gestartet.

Die Authentifizierung der User erfolgt anhand einer Datenbank, die dem
jeweiligen Server beim Start übergeben wird. Da es in unserem Beispiel
erwünscht ist, daß sich bestimmte Accounts nur auf einem der beiden
Server anmelden können, müssen wir für jeden der beiden Server eine
eigene Datenbank erzeugen.

Diese Datenbank wird aus einem speziellen passwd-File generiert.

::

   # pure-pw mkdb /etc/pureftpd.pdb -f /etc/pureftpd.passwd

Das passwd-File enthält alle für den Betrieb des Servers notwendigen
Accountdaten. Es wird mit Hilfe des pure-pw-Kommandos manipuliert:

::

   # pure-pw useradd USER -u ftpuser -d /data/ftp/USER -f /etc/pureftpd.passwd
   # pure-pw userdel USER -f /etc/pureftpd.passwd
   # pure-pw usermod USER -r 1.2.3.0/24 -f /etc/pureftpd.passwd
   # pure-pw passwd USER -f /etc/pureftpd.passwd

Accountparameter
~~~~~~~~~~~~~~~~

Es ist möglich, jedem Account spezielle Eigenschaften zuzuweisen. Dies
kann sowohl beim Anlegen des Accounts (``useradd``) als auch später
(``usermod``) erfolgen. Folgende Parameter sind möglich:

-  -d : chroots the user (-D doesn't so)
-  -m : updates the /etc/pureftpd.pdb -> IMPORTANT
-  -t : download bandwidth
-  -T : upload bandwidth
-  -n : max number of files
-  -N : max Mbytes
-  -q : upload ratio
-  -Q : download ratio
-  -r : allowclienthost/mask,allowclienthost/mask (mask is short)
-  -R : denyclienthost/mask,denyclienthost/mask
-  -i : allowlocalhost/mask,allowlocalhost/mask
-  -I : denylocalhost/mask,denylocalhost/mask
-  -y : max number of concurrent sessions
-  -z : hhmm-hhmm (time restriction)

Das Ergebnis kann man sich mit

::

   # pure-pw show USER -f /etc/pureftpd.passwd 

ansehen. Alle Werte können mit ``-T`` zurückgesetzt werden.

.. note::

  Nachdem Änderungen im passwd-File vorgenommen wurden, muss eine neue
  Datenbank generiert werden:

  ::

    # pure-pw mkdb /etc/pureftpd.pdb -f /etc/pureftpd.passwd

  Man kann auf diesen Schritt verzichten, wenn bei den Manipulationen am
  passwd-File der Schalter ``-m`` benutzt wurde. Dieser Schalter erzeugt
  aber stets eine Standard-Datenbank. Da ich in unserem Fall mehrere
  Datenbanken erzeugen möchte, will ich selbige explizit angeben. Deshalb
  verzichte ich auf die Benutzung von ``-m`` und generiere meine
  Datenbanken händisch mit ``pure-pw mkdb``.

TLS-Zertifikate
---------------

Um TLS benutzen zu können, benötigt der Server ein Zertifikat. Der
DoItYourself-Weg:

::

   # mkdir -p /etc/ssl/private
   # openssl req -x509 -nodes -newkey rsa:1024 -keyout \
    /etc/ssl/private/pure-ftpd.pem \
    -out /etc/ssl/private/pure-ftpd.pem
   # chmod 600 /etc/ssl/private/*.pem

Serverstart
-----------

Ich benutze für den Start der beiden Server jeweils ein eigenes
Startscript, das aus ``/etc/rc.local`` heraus aufgerufen wird.

Script für den Server ohne erzwungenes TLS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   #!/bin/sh
   params=" "
   # listen on these address
   #params=" $params -S 192.168.1.1,21"
   #
   # maximum number of clients allowed
   params=" $params -c 100"
   #
   # ipv4 only
   params=" $params -4"
   #
   # prohibit dot file writing
   #params=" $params -x"
   #
   # prohibit dot file reading
   #params=" $params -X"
   #
   # show main intro
   params=" $params -F /usr/local/homegrown/ftp/INTRO.txt"
   #
   # chroot everyone
   params=" $params -A"
   #
   # daemonize the server
   params=" $params -B"
   #
   # limit simultan clients from one ip
   params=" $params -C 2"
   #
   # auth only - no anonymous
   params=" $params -E"
   #
   # max idletime in minutes
   params=" $params -I 3"
   #
   # stop uploads if partition is more than 95% full.
   params=" $params -k 90"
   #
   # prevents DoS: show only x files an y subdirs on command ls
   #params=" $params -L 100:2"
   #
   # use uploadscript function
   #params=" $params -o"
   #
   # enable bandwidth limitation in kilobytes/second (upload:download)
   params=" $params -T 128:128"
   #
   # don't allow UID's below x to log in
   params=" $params -u 100"
   #
   # limits the connections per user (authenticated:anonymous)
   params=" $params -y 2:0"
   #
   # sets the range of ports opened by the server for passive FTP
   params=" $params -p 50000:50100"
   #
   # defines authentication
   params=" $params -l puredb:/etc/pureftpd.pdb"
   #
   # logs in apache format
   params=" $params -O clf:/var/log/ftp/transfers.log"
   #
   # logs in stat format
   #params=" $params -O stats:/var/log/ftp/stats.log"
   #
   # creates virtual users home dir
   params=" $params -j"
   # 
   # sets the level of ssl authentication
   #params=" $params --tls=0"  # no ssl
   params=" $params --tls=1"  # both
   #params=" $params --tls=2"  # ssl only
   #
   ftpd="/usr/local/sbin/pure-ftpd"
   uploadd="/usr/local/sbin/pure-uploadscript"
   uploadscript=/usr/local/homegrown/ftp/uploadscript.sh
   #
   $ftpd $params &
   #$uploadd -B -r $uploadscript &
   # ATTENTION: uploadscript function was disabled
   # uploadscript does not detect any uploaded file
   exit $?

.. note::

  -  Wenn kein Interface bzw. Port angegeben wird, benutzt der Server Port
     21.
  -  Die ``dot file``-Parameter sollte man vorsichtig handhaben, da einige
     Clients dots zum markieren im Upload befindlicher Files benutzen.
  -  Wenn ein Intro verwendet werden soll, muss natürlich eines gebaut
     werden...
  -  Die Uploadscript-Funktion wurde durch mich nicht genutzt, da ich
     feststellen musste, dass in der durch mich verwendeten Version nicht
     jeder Upload registriert wurde.

Script für den Server mit erzwungenem TLS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   #!/bin/sh
   params=" "
   #
   # listen on these address
   params=" $params -S 1.2.3.4,65021"
   #
   # maximum number of clients allowed
   params=" $params -c 100"
   #
   # ipv4 only
   params=" $params -4"
   #
   # prohibit dot file writing
   #params=" $params -x"
   #
   # prohibit dot file reading
   #params=" $params -X"
   #
   # show main intro
   params=" $params -F /usr/local/homegrown/ftp/INTRO.SSL.txt"
   #
   # chroot everyone
   params=" $params -A"
   #
   # daemonize the server
   params=" $params -B"
   #
   # limit simultan clients from one ip
   params=" $params -C 2"
   #
   # auth only - no anonymous
   params=" $params -E"
   #
   # max idletime in minutes
   params=" $params -I 3"
   #
   # stop uploads if partition is more than 95% full.
   params=" $params -k 90"
   #
   # prevents DoS: show only x files an y subdirs on command ls
   #params=" $params -L 100:2"
   #
   # use uploadscript function
   #params=" $params -o"
   #
   # enable bandwidth limitation in kilobytes/second (upload:download)
   params=" $params -T 128:64"
   #
   # don't allow UID's below x to log in
   params=" $params -u 100"
   #
   # limits the connections per user (authenticated:anonymous)
   params=" $params -y 5:0"
   #
   # sets the range of ports opened by the server for passive FTP
   params=" $params -p 50101:50200"
   #
   # defines authentication
   params=" $params -l puredb:/etc/pureftpd.ssl.pdb"
   #
   # logs in apache format
   params=" $params -O clf:/var/log/ftp/transfers.ssl.log"
   #
   # logs in stat format
   #params=" $params -O stats:/var/log/ftp/stats.log"
   #
   # creates virtual users home dir
   params=" $params -j"
   #
   # sets the level of ssl authentication
   #params=" $params --tls=0"  # no ssl
   #params=" $params --tls=1"  # both
   params=" $params --tls=2"  # ssl only
   #
   ftpd="/usr/local/sbin/pure-ftpd"
   uploadd="/usr/local/sbin/pure-uploadscript"
   uploadscript=/usr/local/homegrown/ftp/uploadscript.sh
   #
   $ftpd $params &
   #$uploadd -B -r $uploadscript &
   # ATTENTION: uploadscript function was disabled
   # uploadscript does not detect any uploaded file
   # 
   exit $?

Eintrag in /etc/rc.local
~~~~~~~~~~~~~~~~~~~~~~~~

::

   # pure-ftp stuff
   if [ -x /usr/local/homegrown/ftp/runftpd.sh ]; then
          echo -n ' pureftpd';    /usr/local/homegrown/ftp/runftpd.sh
   fi
   if [ -x /usr/local/homegrown/ftp/runftpd.ssl.sh ]; then
          echo -n ' pureftpd.ssl';        /usr/local/homegrown/ftp/runftpd.ssl.sh
   fi

Firewalleinstellungen
---------------------

Wir benutzen bpf als Firewall. Das folgende sollte in die
``/etc/pf.conf`` eingetragen werden:

::

   table <FTP_CLIENT_T> \
          { 1.2.3.0/24 }
   table <FTP_CLIENT2_T> \
          {1.2.4.5/32 }
   pass  in  on { $EXT0 } proto { tcp, udp } \
        from { <FTP_CLIENT_T>, <FTP_CLIENT2_T> } \
        to { $EXT0IP } \
        port { 21 } \
        keep state
   pass  in  on { $EXT0 } proto { tcp, udp } \
        from any \
        to { $EXT0IP } \
        port { 65021 } \
        keep state
   pass in on { $EXT0 } proto { tcp, udp } \
        from any \
        to { $EXT0IP } \
        port 49999 >< 50201 \
        keep state

Natürlich sollte die Firewall in der ``/etc/rc.conf`` enabled sein:

::

   pf=YES

Clients
-------

Für den TLS-losen Server eignet sich jeder anständige Client. Will man
allerdings TLS benutzen, bedarf es TLS-unterstützender Clients. Unter
Windows habe ich die besten Erfahrungen mit CoreFTP gemacht. Der hat
alles, was der gemeine User braucht. Für weitere Empfehlungen
diesbezüglich wäre ich sehr dankbar.

Quellen
-------

-  http://www.pureftpd.org

* :ref:`genindex`

Zuletzt geändert: |date|

