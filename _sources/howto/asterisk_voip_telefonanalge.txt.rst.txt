Asterisk
========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

In diesem HowTo wird erklärt, wie man Asterisk auf BSD installiert und
eine einfache Telefonanlage mit SIP und IAX Konten erstellt, externe
VoIP Provider einbindet und zwei Asterisk mittels IAX verbindet.

Was kann Asterisk?
------------------

Asterisk kann so gut wie alles, was von einer modernen Telefonanlage im
Business Bereich erwartet wird. Aber auch für kleine private Anlagen ist
Asterisk eine gute Wahl!

-  Music on Hold
-  Konferenzräume
-  Voice Mail
-  Anrufweiterleitung mit/ohne Rücksprache
-  Warteschleifen
-  Agentenlogin (für. z.B HomeOffice Mitarbeiter)
-  Blacklists für unerwünschte Anrufer
-  Manager Interface für Call-Center

Was kann Asterisk nicht?
------------------------

Asterisk unterstützt SRTP und IAX2 (AES128) jedoch kein ZRTP (Zimmermann
RTP). Wer ZRTP nutzen möchte, sollte einen zusätzlichen Kamailio Server
einrichten.

Die Einrichtung eines Kamailio Server ist einfach, wird in diesem HowTo
jedoch nicht behandelt.

Installation von Asterisk
-------------------------

Vorzugsweise installieren wir Asterisk in einer Jail. Wie man Jails
erstellt, wird hier erklärt `jails </howto/jails>`__.

Wir installieren Asterisk aus den Binärpaketen.

<note important> seit Version 13.19.2_1 ist SRTP Support aktiviert. Wer
auf eine frühere Version von Asterisk13 zurück greifen muss und SRTP
nutzen will, muss Asterisk aus den Ports bauen. Siehe Notes:
https://www.freshports.org/net/asterisk13/ </note>

.. code:: tcsh

   export jail=<Pfad zur Jail>

.. code:: tcsh

   pkg -c $jail install -y asterisk13

Konfiguration
-------------

Jail
~~~~

Die Konfiguration unserer Jail sieht wie folgt aus

.. code:: tcsh

   $EDITOR /etc/jail.conf

::

   exec.clean;
   exec.system_user = "root";
   exec.jail_user = "root";
   exec.start += "/bin/sh /etc/rc";
   exec.stop = "/bin/sh /etc/rc.shutdown";
   exec.consolelog = "/var/log/jail_${name}_console.log";
   mount.devfs;
   allow.set_hostname = 0;
   allow.sysvipc = 0;
   path = "/${name}";

   <gib mir einen Namen!> {
            host.hostname = "asterisk.lokal";
            ip4.addr = "re0|192.168.xxx.xxx/32";
            ip4.addr += "re1|192.168.xxx.xxx/32";
            }``

Hier zu sehen sind zwei Netzwerkschnittstellen, in meinem Fall wären das
Realtek-Karten ``re0`` und ``re1``. Diese müssen auf die jeweilige
Hardware und Konfigurationswünsche angepasst werden. Intel wäre ``em0``
und respektive ``em1``.

.. _asterisk-1:

Asterisk
~~~~~~~~

Es gibt eine Menge Files im Ordner ``/usr/local/etc/asterisk/`` und beim
ersten Blick ist man etwas erschrocken. Aber keine Angst, wir werden
hier nur jene Files bearbeiten oder neu erstellen, welche für den
Betrieb von Asterisk relevant sind.

Das sind folgende

-  asterisk.conf
-  modules.conf
-  features.conf
-  musiconhold.conf
-  rtp.conf
-  sip.conf
-  iax.conf
-  extensions.conf

Die fett markierten Dateien werden wir neu erstellen/ändern, alle
anderen nicht markierten bleiben in der ursprünglichen Version im
Ordner. Den Rest dürft ihr löschen.

.. warning:: 

  Im File modules.conf ist aufgelistet welche Module Asterisk beim
  Start lädt. Da standardmässig alle Module geladen werden und wir aber bis auf
  unsere oben angeführten Konfig-Files alle anderen gelöscht haben, erscheinen in
  der Asterisk CLI beim Start einige Fehlermeldungen, dass keine Konfig gefunden
  wurde. Das ist nicht weiter schlimm, wenn Asterisk nach seinen Wünschen
  konfiguriert ist, kann mann die nicht benötigten Module auf **noload** setzen

asterisk.conf
^^^^^^^^^^^^^

Kontrolliert, ob diese beiden Zeilen nicht auskommentiert sind. Wir
wollen Asterisk als User Asterisk laufen lassen.

.. code:: tcsh

   runuser = asterisk              ; The user to run as.
   rungroup = asterisk             ; The group to run as.

rtp.conf
^^^^^^^^

Hier definieren wir nur die RTP-Ports. Bei den meisten SIP-Telefonen
sind diese zwischen 5004-5020 als Standard vordefiniert (im Zweifel also
erstmal ins Menü des Telefons schauen), also sieht unser File so aus.

.. code:: tcsh

   [general]
   rtpstart=5004
   rtpend=5020

.. warning::

  Die Port Range begrenzt **auch** die Anzahl der maximalen parallelen
  Gespräche! Bei Bedarf muss diese beim Asterisk sowie bei den Clients
  angepasst werden. Bsp.: Konferenz, Music-on-hold, Weiterleitung usw.

sip.conf
^^^^^^^^

Hier sind unsere Zugangsdaten für alle SIP Verbindungen hinterlegt. Egal
ob interne Telefone oder externer Provider.

.. code:: tcsh

   [general]

   alwaysauthreject = yes ;Überaus wichtig!!!! Die wichtigste Einstellung überhaupt!
   allowguest = no  ; Wir verbieten "Guest" Registrationen

   bindport = 5060 ; SIP Standard Port
   bindaddr = 0.0.0.0 ; wir lauschen auf allen Adressen

   language=de ; deutsche Ansagen (z.B. Konferenzraum)

   nat = force_rport,comedia  ; für SIP Telefone aus dem WAN

   localnet = 192.168.1.0/255.255.255.0 ; Unser lokales Netz, wo sich die Telefone befinden.
   localnet = 192.168.2.0/255.255.255.0 ; Es können natürlich auch mehrere definiert werden

   register => <benutzer>:<passwort>@<server/ip>nummer ; -> selbe Zugangsdaten wie ext-sip

   [100]; BSD User 100
   disallow = all
   allow = ulaw ; hier wird der **für die Telefone zum Asterisk hin** verwendete Sprachcodec definiert. Kommagetrennt können hier mehrere und/oder andere verwendet werden.
   context = bsdforen ; -> wird in der extensions.conf erläutert!
   secret = <streng geheimes Passwort>
   callerid = "BSD User XYZ" <100>
   type = peer
   host = dynamic
   qualify = yes

   [ext-sip]
   type = peer
   context = von-ext
   username = xxxxx
   fromuser = xxxxx
   secret = xxxxx
   host = xxx.xxx.xxx.xxx
   insecure = port,invite
   nat = force_rport,comedia
   qualify = yes
   canreinvite = no
   dtmfmode = rfc2833

.. warning::

  ``alwaysauthreject = yes`` und ``allowguest = no`` sollten immer gesetzt
  sein! Es gibt eine Menge VoIP Brute Force Tools, welche sonst eruieren
  könnten, welche Nummer auf der Asterisk konfiguriert sind. Mit diesen beiden
  Zeilen denkt der Angreifer, dass er einen Treffer erzielt hat, obwohl diese
  Nebenstelle nicht existiert. Ein Brute Force Angriff auf ein Benutzer Konto
  das nicht existiert, wird keinen Erfolg liefern.

iax.conf
^^^^^^^^

Wie der Name schon sagt, geht es um IAX Konten. IAX ist ein spezielles
Protokoll, welches von Digium, dem Asterisk Erfinder entworfen wurde und
die bekannten Probleme von SIP und NAT nicht aufweist.

IAX2 verwendet im Standard den Port 4569 und wickelt allen Verkehr über
diesen ab, es wird kein zusätzlicher SIP/RTP Port benötigt.

Über das IAX2 Protokoll lassen sich sehr einfach und komfortabel mehrere
Asterisk mit einander verbinden. So ist es möglich ein weit verteiltes
"privates" Telefonnetz aufzubauen und das dazu noch mit AES128
verschlüsselt! Natürlich kann man das ganze noch zusätzlich durch ein
VPN schicken.

Gleich ob 2,3 oder 4 parallele Gespräche, alle werden über den selben
IAX Kanal abgewickelt,es ist also sehr ressourcenschonend.

Wir nehmen an, wir haben zwei Asterisk Server, Server-A und Server-B.

.. code:: tcsh

   [general]
   bindport=4569
   bindaddr=192.168.xxx.xxx
   externhost=xxx.xxx.xxx.xxx
   bandwidth=high
   allow=all
   language=de

   [Asterisk-Server-A]
   type=peer
   username=Asterisk-Server-B
   secret=<gemeinsames Passwort>
   host=<IP Server B>
   auth=md5,rsa
   trunk=yes
   qualify=yes
   encryption=yes
   forceencryption=yes
   context=bsdforen ;-> wird in der extensions.conf erläutert!

Und hier die Konfig von Server-B

.. code:: tcsh

   [general]
   bindport=4569
   bindaddr=192.168.xxx.xxx
   externhost=xxx.xxx.xxx.xxx
   bandwidth=high
   allow=all
   language=de

   [Asterisk-Server-B]
   type=peer
   username=Asterisk-Server-A
   secret=<gemeinsames Passwort>
   host=<IP Server A>
   auth=md5,rsa
   trunk=yes
   qualify=yes
   encryption=yes
   forceencryption=yes
   context=bsdforen ;-> wird in der extensions.conf erläutert!

Wie man erkennen kann, sind die Passwörter bei beiden gleich und die
Benutzernamen ausgekreuzt. So kann man mit einem Block den eingehenden
sowie ausgehenden Verkehr abwickeln. Wer aus Sicherheitsgründen dies
nicht möchte, kann auch für Ein/Ausgehend jeweils eine eigene Konfig
erstellen und die Anmeldedaten anpassen.

Hat jemand ein IAX Softphone wie Zoiper im Einsatz, dann benötigt man
einen Eintrag in der iax.conf der wie folgt aussieht. Bis heute ist mir
leider kein IAX Client bekannt, der verschlüsselte Anrufe unterstützt.

.. code:: tcsh

   [200]
   username =<benutzer>
   secret = <passwort>
   disallow = all
   allow = ulaw
   allow = alaw
   context = bsdforen ;-> wird in der extesions.conf erläutert!
   type = peer
   host = dynamic
   callerid = "BSD Foren-Mama" <200>
   qualify = yes

extensions.conf
^^^^^^^^^^^^^^^

Jetzt geht es los, wir wollen telefonieren und dazu muss Asterisk
wissen, wohin

.. code:: tcsh

   [general]
   static = yes
   writeprotect = no
   clearglobalvars = yes
   language=de ; Deutsche Sprachprompts

   [bsdforen]
   exten => _10X,1,Dial(SIP/${EXTEN},600,r)
   exten => _10X,n,Hangup()

   exten => _20X,1,Dial(IAX2/${EXTEN},600,r)
   exten => _20X,n,Hangup()

   exten => _30X,1,Dial(IAX2/${EXTEN},60,r)
   exten => _30X,n,Dial(SIP/100,60,r)
   exten => _30X,n,Hangup()

   exten => _7XX,1,Dial(IAX2/Asterisk-Server-B/${EXTEN},120,r)
   exten => _7XX,n,Hangup

   exten => _00[1-9].,1,Dial(SIP/${EXTEN:1}@ext-sip,120,r) ; **Behandelt KEINE Notrufnummer!!!!!!!!**
   exten => _00[1-9].,n,Hangup()


   [von-ext]
   exten => 0041123456,1,Dial(SIP/100,120,rR)
   exten => 0041123456,n,Hangup()

Hier sollte unser Konstrukt an Konfigurationen in der iax.conf /
sip.conf schlüssig werden. Wie zu sehen ist, haben wir die Sektion
[bsdforen] welche bei jedem Account in der IAX/SIP Konfig als context
gesetzt wurde. Asterisk sucht also hier nach einer Wahlregel.

Alle SIP Konten sind im Bereich 100-109 und alle IAX Konten im Bereich
200-209 zu finden. Natürlich kann auch ein IAX User auf einen SIP
Anschluss anrufen und umgekehrt.

Die Zahl 600 ist ein TIMEOUT, nach dieser Zeit wird der Anruf zu Schritt
2 geleitet, also dem Hangup(). Das **r** bedeutet RING, setzt man ein
**m** so wird MOH <Music on Hold> gestartet. Das MOH Modul muss
natürlich konfiguriert und dementsprechende Sound Files auf der Anlage
abgelegt sein. Bitte denkt daran, dass ihr GEMA-freie Musik dort
verwendet. MOH wird nämlich rechtlich als music-on-demand verstanden.

Im dritten Beispiel ruft ein User eine Nummer im 300er Kreis,
beantwortet nach 60s niemand diesen Anruf, so wird der Anrufer auf die
Nummer 100 geleitet, ist auch dort nach 60s keiner erreichbar, wird er
beendet.

Im vierten Block definieren wir, dass alle Nummern im Bereich von
700-799 per IAX an den Asterisk-Server-B geleitet werden. Asterisk-B
braucht also eine Wahlregel für diese Nummern!

Im fünften Block behandeln wir unsere ausgehenden Gespräche. Zu sehen
ist 00 am Anfang, was bedeutet wir müssen für externe Anliegen eine NULL
vorwählen. Mit **EXTEN:1** teilen wir Asterisk mit, dass er diese Null
abschneiden soll, beim Provider kommt also die richtige Nummer an.

.. warning::

  Bitte vergesst nicht für die Notrufnummer einen Block zu definieren! Beim
  jeweiligen Anbieter muss dieser dann auf den richtigen Standort des
  Anschlusses geroutet werden. Leider gibt es derzeit noch keine Testnummer um
  zu prüfen, ob das Routing korrekt funktioniert.

.. code:: tcsh

   exten => 133,1,Dial(SIP/${EXTEN}@ext-sip,600,r)

In diesem Beispiel würde man die 133 (Polizei Österreich) über den
externen SIP Anbieter routen. Natürlich ist es auch möglich den Block
für ausgehende Gespräche zu modifizieren, sodass er Notrufe absetzen
kann. Da in vielen Firmen jedoch die Null für extern Standard ist, und
nicht immer gewährleistet werden kann, dass nicht ein "fremder" einen
Notruf absetzten muss, würde ich **immer** die Wahlregel einbauen!

Im Block [von-ext] behandeln wir unsere externe Rufnummer, hier die
0041123456. Ruft jemand diese Nummer an, wird er an die 100
weitergeleitet, ist auch dort nach 60s keiner erreichbar, wird er
beendet.

.. code:: tcsh

   [von-ext]
   exten => 0041123456,1,Dial(SIP/100,60,r)
   exten => 0041123456,n,Hangup()

.. warning::

  Ist der SIP User **offline** so wird das TIMEOUT NICHT berücksichtigt und
  sofort zum nächsten Schritt weitergeleitet!

Starten der Jail für Asterisk
-----------------------------

Jail starten

.. code:: tcsh

   jexec -c asterisk

Im Übrigen könnte die Jail wieder beenden (entfernt) werden.

.. code:: tcsh

   jexec -r asterisk

Konfiguration der Jail für Asterisk
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Wir setzen folgenden Eintrag in der JAIL /etc/rc.conf!

.. code:: tcsh

   asterisk_enable="YES"

oder manuell mit

.. code:: tcsh

   jexec <JID> asterisk

Asterisk CLI
^^^^^^^^^^^^

Wir verbinden uns mit der Asterisk Command Line.

.. code:: tcsh

   jexec <JID> asterisk -r

Ist beim Starten von Asterisk kein Fehler aufgetreten sollte die Ausgabe
in etwa so aussehen.

   ``Asterisk 13.19.0, Copyright (C) 1999 - 2014, Digium, Inc. and others.
   Created by Mark Spencer <markster@digium.com>
   Asterisk comes with ABSOLUTELY NO WARRANTY; type 'core show warranty' for details.
   This is free software, with components licensed under the GNU General Public
   License version 2 and other licenses; you are welcome to redistribute it under
   certain conditions. Type 'core show license' for details.
   =========================================================================
   Running as user 'asterisk'
   Running under group 'asterisk'
   Connected to Asterisk 13.19.0 currently running on asterisk (pid = 1299)
   asterisk*CLI>``

Das Kommando ``sip show peers`` zeigt uns nun die registrierten Sip User
an.

.. code:: tcsh

   asterisk*CLI> sip show peers
   Name/username             Host                          Dyn Forcerport Comedia    ACL Port     Status      Description                      
   100/xxx                   192.168.xxx.xxx               D  Yes        Yes         A  5060     OK (35 ms)                                   
   101/xxx                   xxx.xxx.xxx.xxx               D  Yes        Yes            63464    OK (94 ms)                                   
   102/xx                   xxx.xxx.xxx.xxx                D  Yes        Yes            18556    OK (41 ms)                                   
                                   
   3 sip peers [Monitored: 3 online, 0 offline Unmonitored: 0 online, 0 offline]

Das selbe Kommando gibt es auch für IAX2, ``iax2 show peers``

.. code:: tcsh

   asterisk*CLI> iax2 show peers
   Name/Username    Host                         Mask                         Port           Status      Description                     
   <Name>  xxx.xxx.xxx.xxx                       (S)  255.255.255.255         4569  (T) (E)  OK (48 ms)                                  
   <Name>  xxx.xxx.xxx.xxx                       (S)  255.255.255.255         4569  (T) (E)  OK (36 ms)                                  
    iax2 peers [3 online, 3 offline, 0 unmonitored]

Das ``(E)`` bei Status bedeutet, encrypted.

.. code:: tcsh

   asterisk*CLI> sip show registry
   Host                                    dnsmgr Username       Refresh State                Reg.Time                 
   xxx.xxx.xxx.xxx:5060                    N      xxxxxxxxxx         105 Registered           Sun, 08 Apr 2018 14:07:51
   1 SIP registrations.

Das Kommando ``core show channels`` zeigt alle aufgebauten
Gesprächsverbindungen an.

.. code:: tcsh

   asterisk*CLI> core show channels
   Channel                           Location       State   Application(Data)
   IAX2/Asterisk-Server-B-30687      (None)         Up      AppDial((Outgoing Line))
   SIP/XXX-00000007                  700@privat:1   Up      Dial(IAX2/SEC1/650,80,m(spezial)
   2 active channels
   1 active call
   8 calls processed

Hier zu sehen ist ein bestehender IAX2 Kanal welcher über Extension SEC1
aufgebaut wurde. Wohin der Anruf auf der zweiten Asterisk geleitet wird,
ist nicht ersichtlich.

Der Anrufer, hier XXX versucht die Nummer 700 zu erreichen, wir haben in
der extensions.conf definiert, dass alle 7XX Nummer über den IAX2 Peer
(Asterisk-Server-B) geroutet werden. Dieser Versucht nun das Gespräch zu
vemitteln, folglich muss auf dem zweiten Asterisk eine Wahlregel für den
Bereich 700 erstellt sein.

nützliche Infos
---------------

Mit dem Kommando ``core show help`` kann sich Hilfe für die
verschiedenen Asterisk Module ausgegeben werden. Hier ein Auszug.

.. code:: tcsh

   sip show channel               -- Show detailed SIP channel info
   sip show domains               -- List our local SIP domains
   sip show history               -- Show SIP dialog history
   sip show inuse [all]           -- List all inuse/limits

Nun kann man mit dem Kommando ``core show help sip show channel`` alle
Information zur Benutzung dieses Kommandos ausgegeben lassen.

.. code:: tcsh

   asterisk*CLI> core show help sip show channel
   Usage: sip show channel <call-id>
          Provides detailed status on a given SIP dialog (identified by SIP call-id).

Ändert man während dem Betrieb Konfigurationsfiles, so können diese mit

.. code:: tcsh

   sip reload     # sip.conf neu laden
   iax2 reload    # iax.conf neu laden
   core reload    # gesamte Konfiguration neu laden

.. warning::

  Manche Konfigurationsänderungen erfordern einen kompletten NEUSTART von
  Asterisk. Ist man nicht sicher, einfach ein "core stop now" absetzen und
  anschliessend mit "jexec <JID> asterisk" neustarten. Laufende Gespräche
  werden beim Neuladen von Konfigurationen nicht unterbrochen.

Möchte man Asterisk beenden, so wird der Befehl

.. code:: tcsh

   core stop now

abgesetzt.

VoIP Anbieter unterscheiden sich oft im Ablauf der SIP Registrierung.
Die meisten stellen Konfigurationsbeispiele für die gebräuchlichsten
Telefonanlagen zur Verfügung, also im Fehlerfall mal die Webseite des
Anbieters besuchen und nach Beispielen suchen.

Auch bei der Signalisation von eingehenden Gesprächen gibt es
Unterschiede. Hier ein Beispiel wie der Schweizer Anbieter e-fon.ch
Gespräche vermittelt. Wir haben eine Registrierung am e-fon Server
jedoch beliebig viele Durchwahlnummern.

.. code:: tcsh

   [EXTERN] ; Hier kommen die Anrufe rein

   exten => s,1,Set(Durchwahlnummer=${SIP_HEADER(X-Number)})
   exten => s,n,Goto(TELEFON,${Durchwahlnummer},1)
   exten => s,n,Hangup()

   [TELEFON]

   exten => XXXXXXX100,1,Dial(SIP/100,20,r)
   exten => XXXXXXX100,n,Answer()
   exten => XXXXXXX100,n,Verbose(1,Mon-Sam Ansage)
   exten => XXXXXXX100,n,Playback(/usr/local/share/asterisk/ansage/Ansage)
   exten => XXXXXXX100,n,Hangup

   exten => XXXXXXX200,1,Dial(SIP/200,20,r)
   exten => XXXXXXX200,n,Answer()
   exten => XXXXXXX200,n,Verbose(1,Mon-Sam Ansage)
   exten => XXXXXXX200,n,Playback(/usr/local/share/asterisk/ansage/Ansage)
   exten => XXXXXXX200,n,Hangup

Der Ablauf ist relativ einfach,

-  wir setzen eine Variable <Durchwahlnummer>
-  springen in den Context [TELEFON] mit der entsprechenden
   Durchwahlnummer

.. warning:: 

  Es gibt keine Regel die eine nicht existierende Durchwahl behandeln würde.
  Der Anruf wird in so einem Fall abgebrochen.

Blacklist
~~~~~~~~~

In der heutigen Zeit wird man immer öfter dazu gezwungen eine Rufnummer
als "böse" zu markieren und natürlich ist auch das kein Problem für
Asterisk. Mit ein paar einfachen Ergänzungen im Wählplan
"extensions.conf" kann man jede Nummer blacklisten.

::

   [blacklist]
   exten => _*4X[0-9].,1,Answer()
   exten => _*4X[0-9].,n,Set(DB(blacklist/${EXTEN:2})=1)
   exten => _*4X[0-9].,n,SayPhonetic(OK)
   exten => _*4X[0-9].,n,Hangup()

   exten => _*5X[0-9].,1,Answer()
   exten => _*5X[0-9].,n,DBdeltree(blacklist/${EXTEN:2})
   exten => _*5X[0-9].,n,SayPhonetic(OK)
   exten => _*5X[0-9].,n,Hangup()

Dieser Block setzt oder löscht eine Nummer aus der Blacklist. Im
nächsten Schritt wird bei eingehenden Anrufen kontrolliert ob sie auf
der Blacklist vermerkt sind.

::

   [incoming]
   exten => <externe Nummer>,1,GotoIf(${DB_EXISTS(blacklist/${CALLERID(num)})}?blacklisted,s,1)
   exten => <externe Nummer>,n,Dial(SIP/XXX,120,rR)

fällt diese Kontrolle positiv aus, so greift die Zeile "GotoIf" und wir
springen in den Block "blacklisted".

::

   [blacklisted]
   exten => s,1,NoOp("This number is Blacklisted +++++++++++++++++++++++++++++")
   exten => s,n,Hangup

Externe Einwahl
~~~~~~~~~~~~~~~

Möchte man eine **interne** Leitung von aussen, dann kann man folgendes
in der extensions.conf eintragen. Der Anrufer muss eine
Zahlenkombination eingeben, danach erhält er den internen Wählton und
kann je nach Konfiguration jede interne sowie externe Rufnummer wählen.

Wir überprüfen ob es sich beim eingehenden Anruf um unsere Nummer
handelt. Ist dies der Fall verschieben wir ihn in den "AUTH" Block.
Danach fragt Asterisk nach einem Passwort und gibt uns anschliessend
eine interne Leitung.

::

   [incoming]
   exten => <externe Rufnummer>,1,GotoIf($["${CALLERID(number)}" = "<meine Nummer>"]?AUTH,s,1)]
   exten => <externe Rufnummer>,n,Dial(SIP/XXX,60,r)

   [AUTH]
   exten => s,1,Authenticate(<passwort>)
   exten => s,n,DISA(no-password,privat)
   exten => s,n,Hangup

.. warning::

  Auch wenn solche externen Zugänge mit Passwörter abgesichert werden, gibt es
  gewisse Risiken, deshalb empfiehlt es sich immer die eingehenden Rufnummer zu
  überprüfen!

Öffnungszeiten im Dialplan
~~~~~~~~~~~~~~~~~~~~~~~~~~

Möchte man die Geschäftszeiten definieren könnte der Wählplan so
aussehen.

-  MO-FR 08:00 - 12:30 & 13:30 - 17:00 geöffnet
-  SA 08:00-15:00 geöffnet

Trifft ein Anruf ausserhalb dieser Zeiten ein, wird ein Audiofile
**wir_sind_nicht_da** abgespielt

::

   [von-voip-provider]
   exten => _AAAAAAXXX,1,GotoIfTime(08:00-12:30|mon-thu|*|*?open,${EXTEN},1)
   exten => _AAAAAAXXX,n,GotoIfTime(13:30-16:59|mon-thu|*|*?open,${EXTEN},1)
   exten => _AAAAAAXXX,n,GotoIfTime(08:00-14:59|fri|*|*?open,${EXTEN},1)
   exten => _AAAAAAXXX,n,Answer()
   exten => _AAAAAAXXX,n,Playback(/usr/local/share/asterisk/**wir_sind_nicht_da**)
   exten => _AAAAAAXXX,n,Hangup()


   [open]
   exten => ,1,Dial(SIP/${EXTEN:-3},25,rtT);
   exten => ,n,Hangup();

Während den Öffnungszeiten leiten wir Anrufe an [open] und der
entsprechenden Extension [{EXTEN:-3}] weiter. Die -3 bedeutet, wir
verwenden nur die letzten 3 Stellen.

Sicherheit
----------

Angriffe auf VoIP Anlagen sind sehr populär, es gibt daher einige
Vorsichtsmaßnahmen welche zu beachten sind. Wird die Anlage privat
betrieben, stellt sich die Frage ob es ausreicht, einen prepaid
Anschluss zu verwenden. Ist dies der Fall, hat man bereits eine weitere
Sicherheitsbarriere geschaffen.

Der wichtigste Punkt ist natürlich die Wahl der Passwörter. Die meisten
Angriffe die ich beobachten konnte, basieren auf primitive Brute-Force
Attacken, welche durch starke Passwörter abgewehrt werden können.

Astersik bietet auch die Möglichkeit, durch eine ACL (Access Control
List) IP White- und Blacklists zu definieren.

Dazu wird die Datei acl.conf wie folgt erstellt

::

   [extern]
   permit=8.8.8.0/255.255.255.0

In der sip.conf / iax.conf wird die ACL wie folgt einem SIP Konto
zugewiesen

::

   [peer1]
   ......
   ......
   ......
   acl=extern

.. warning::

  **peer1** kann sich nur registrieren, wenn er sich im richtigen Subnetz
  8.8.8.0/24 befindet. Dies bedingt natürlich, dass er eine statische IP
  Adresse besitzt.

Debugging
---------

Gibt es Probleme mit SIP oder IAX Konten so kann man das Debugging in
der Asterisk CLI aktivieren.

.. code:: tcsh

   sip set debug peer 500

oder für den gesamten SIP Traffic

.. code:: tcsh

   sip set debug on

* :ref:`genindex`

Zuletzt geändert: |date|
