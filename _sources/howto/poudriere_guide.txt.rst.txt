Poudriere Guide
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Einführung
----------

Das Ports-System ist eines der größten Vorteile von FreeBSD für
Benutzer, die Flexibilität und Kontrolle über ihre Software wünschen. Es
ermöglicht Administratoren das einfache Erstellen und Verwalten von
quellbasierten Installationen mithilfe eines Systems, das robust und
vorhersehbar ist.

Während die Vorteile dieser Funktion groß sind, sind einige der am
häufigsten gegen die portsbasierte Verwaltung erhobenen Beschwerden die
Zeit und die Ressourcen, die für die Kompilierung der einzelnen
Softwareprogramme erforderlich sind. Dies wird noch problematischer,
wenn Sie eine große Anzahl von Servern verwalten, von denen jeder seine
eigenen Ports kompiliert. Während FreeBSD-Pakete eine Alternative
bieten, die die Installation beschleunigt, opfert sie die Kontrolle, die
Ports gewähren.

Um dieses Problem zu beheben, können Administratoren eine Anwendung
namens `poudriere <https://www.freshports.org/ports-mgmt/poudriere/>`__
verwenden, um benutzerdefinierte Pakete zu erstellen und zu verwalten.
Während es technisch so angelegt ist, dass es Pakete für eine Vielzahl
von Architekturen erstellt, wird Poudrière oft als eine
Paketerstellungsumgebung verwendet, um Pakete für eine gesamte
Infrastruktur von FreeBSD-Servern zu erstellen und zu hosten.

Installieren Sie die erforderlichen Ports
-----------------------------------------

Zu Beginn werden wir alle benötigten Ports installieren.

Zuerst müssen wir Poudriere selbst installieren.

.. code:: bash

   pkg install poudriere

Schließlich wollen wir auch einen Webserver installieren. Dies wird zwei
Zwecken dienen. Erstens wird dies die Methode sein, mit der unsere
Maschinen die Pakete herunterladen können, die wir kompilieren werden.
Zweitens bietet Poudriere eine Weboberfläche, so dass wir den
Build-Prozess verfolgen und Protokolle überwachen können. In unserem
Fall verwenden wir den NGINX Webserver:

.. code:: bash

   pkg install nginx

Erstellen Sie ein SSL-Zertifikat und einen Schlüssel
----------------------------------------------------

Wenn wir Pakete mit Poudrière erstellen, wollen wir sie mit einem
privaten Schlüssel signieren können. Dies stellt sicher, dass all unsere
Maschinen die erstellten Pakete legitimieren und dass niemand die
Verbindung zum Build-Rechner abfängt, um schädliche Pakete zu versenden.

Wir werden sicherstellen, dass wir ein SSL-Verzeichnis haben, das zwei
Unterverzeichnisse enthält, die keys und certs genannt werden. Wir
können dies in einem Befehl tun, indem wir Folgendes eingeben:

.. code:: bash

   mkdir -p /usr/local/etc/ssl/{keys,certs}

Unser privater Schlüssel, der geheim gehalten werden muss, wird im
Schlüsselverzeichnis abgelegt. Dies wird verwendet, um die Pakete zu
signieren, die wir erstellen werden.Wir können das Verzeichnis sperren,
damit Benutzer ohne Root- oder Sudoprivilegien nicht mit dem Verzeichnis
oder seinen Inhalten interagieren können:

.. code:: bash

   chmod 0600 /usr/local/etc/ssl/keys

Als nächstes erzeugen wir einen 4096-Bit-Schlüssel namens
``poudriere.key`` und platzieren ihn in unserem ``keys`` Verzeichnis,
indem Sie Folgendes eingeben:

.. code:: bash

   openssl genrsa -out /usr/local/etc/ssl/keys/poudriere.key 4096

Nachdem der Schlüssel generiert wurde, können Sie ein öffentliches
Zertifikat daraus erstellen, indem Sie Folgendes eingeben:

.. code:: bash

   openssl rsa -in /usr/local/etc/ssl/keys/poudriere.key -pubout -out /usr/local/etc/ssl/certs/poudriere.cert

Wir haben jetzt die SSL-Komponenten, die wir benötigen, um Pakete zu
signieren und die Signaturen zu verifizieren. Später werden wir unsere
Clients so konfigurieren, dass sie das generierte Zertifikat für die
Paketverifizierung verwenden.

Konfigurieren von Poudrière
---------------------------

Jetzt wo wir unser SSL Zertifikat und Schlüssel haben, können wir mit
der Konfiguration von Poudriere beginnen.

Die Hauptkonfigurationsdatei befindet sich unter
``/usr/local/etc/poudriere.conf``. Öffnen Sie diese Datei:

.. code:: bash

   nano /usr/local/etc/poudriere.conf

Die Poudriere-Konfigurationsdatei ist sehr gut kommentiert und die
meisten Einstellungen müssen vordefiniert sein. Wir werden einige
spezifische Änderungen vornehmen, aber die Mehrheit davon intakt lassen.

Wenn Sie als Dateisystem UFS verwenden muss die folgende Option
auskommentiert werden:

.. code:: bash

   NO_ZFS=yes

Wenn Ihr Server andererseits ZFS verwendet, können Sie poudriere für die
Verwendung eines bestimmten Pools konfigurieren, indem Sie die Option
ZPOOL festlegen. In diesem Pool können Sie den Stamm angeben, den
poudriere für Pakete, Protokolle usw. mit der Option ZROOTFS verwenden
soll. Beachten Sie, dass diese beiden Optionen nicht festgelegt werden
sollten, wenn die Option NO_ZFS auf "yes" gesetzt ist:

.. code:: bash

   # NO_ZFS=yes
   ZPOOL=tank
   ZROOTFS=/poudriere

Beim Erstellen von Software verwendet Poudriere eine Art von Jail, um
das Build-System vom Hauptbetriebssystem zu trennen. Als nächstes müssen
wir einen gültigen Host ausfüllen, wo der Build-Rechner die Software
herunterladen kann, die er für die Jails benötigt. Dies wird über die
FREEBSD_HOST-Option konfiguriert.

Diese Option sollte bereits vorhanden sein, obwohl sie derzeit nicht auf
einen gültigen Host festgelegt ist. Sie können dies auf den Standardpfad
ftp://ftp.freebsd.org ändern oder einen engeren Spiegel verwenden, wenn
Sie einen kennen:

.. code:: bash

   FREEBSD_HOST=ftp://ftp.freebsd.org

Als nächstes wollen wir sicher sein, dass unser Datenverzeichnis
innerhalb der Poudrière-Wurzel korrekt eingestellt ist. Dies wird mit
der Option POUDRIERE_DATA gesteuert und sollte standardmäßig eingestellt
werden. Wir werden die Option jedoch auskommentieren, um sicherzugehen:

.. code:: bash

   POUDRIERE_DATA=${BASEFS}/data

Die nächsten Optionen, die wir auskommentieren sollten, sind die
Optionen CHECK_CHANGED_OPTIONS und CHECK_CHANGED_DEPS. Die erste Option
weist poudriere an, Pakete neu zu erstellen, wenn sich die Optionen
dafür geändert haben. Die zweite Option weist tells poudriere an, Pakete
neu zu erstellen, wenn sich Abhängigkeiten seit der letzten Kompilierung
geändert haben.

Beide Optionen existieren in der Form, die wir in der
Konfigurationsdatei haben wollen. Wir müssen sie nur auskommentieren:

.. code:: bash

   CHECK_CHANGED_OPTIONS=verbose
   CHECK_CHANGED_DEPS=yes

Als nächstes werden wir poudriere auf den SSL-Schlüssel zeigen, den wir
erstellt haben, damit Pakete während des Builds signiert werden können.
Die zur Angabe verwendete Option heißt PKG_REPO_SIGNING_KEY. Heben Sie
die Markierung dieser Option auf, und ändern Sie den Pfad so, dass er
den Speicherort des SSL-Schlüssels widerspiegelt, den Sie zuvor erstellt
haben:

.. code:: bash

   PKG_REPO_SIGNING_KEY=/usr/local/etc/ssl/keys/poudriere.key

Schließlich können wir die URL_BASE-Zeichenfolge auf den Domänennamen
oder die IP-Adresse festlegen, unter der Ihr Server erreichbar ist. Dies
wird von poudriere verwendet, um Links in der Ausgabe zu erstellen, auf
die geklickt werden kann. Sie sollten das Protokoll einschließen und den
Wert mit einem Schrägstrich beenden:

.. code:: bash

   URL_BASE=http://server_domain_or_IP/

Wenn Sie mit den Änderungen fertig sind, speichern und schließen Sie die
Datei.

Erstellen der Build-Umgebung
----------------------------

Als nächstes müssen wir unsere Build-Umgebung konstruieren. Wie bereits
erwähnt, wird Poudriere Ports in einer isolierten Umgebung mit Jails
bauen.

Für unsere Zwecke sieht unser Jails-Aufbaubefehl so aus:

.. code:: bash

   poudriere jail -c -j 11-2x64 -v 11.2-RELEASE

Das wird eine Weile dauern, also seid geduldig. Wenn Sie fertig sind,
können Sie das installierte Jail sehen, indem Sie Folgendes eingeben:

.. code:: bash

   poudriere jail -l

.. code:: bash

   JAILNAME        VERSION         ARCH  METHOD TIMESTAMP           PATH
   11-2x64 11.2-RELEASE-p0 amd64 ftp    2018-01-06 20:43:48 /usr/local/poudriere/jails/11-2x64

Sobald Sie eine Jail erstellt haben, müssen wir eine Ports-Struktur
installieren.

Wir können das Flag -p verwenden, um unseren Ports-Baum zu benennen. Wir
werden unseren Baum HEAD nennen, da er genau die Verwendung dieses
Baumes (den "Kopf" oder den aktuellsten Punkt des Baumes) zusammenfasst.
Wir werden es regelmäßig aktualisieren, damit es der aktuellsten Version
der verfügbaren Ports-Struktur entspricht:

.. code:: bash

   poudriere ports -c -p HEAD

Auch diese Prozedur wird eine Weile dauern, da die gesamte
Ports-Struktur abgerufen und extrahiert werden muss. Wenn dies
abgeschlossen ist, können Sie unseren Ports-Baum anzeigen, indem Sie
Folgendes eingeben:

.. code:: bash

   poudriere ports -l

Nachdem dieser Schritt abgeschlossen ist, haben wir nun die Strukturen,
um unsere Ports zu kompilieren und Pakete zu erstellen. Als nächstes
können wir damit beginnen, unsere Liste von Ports zusammenzustellen, um
die Optionen zu erstellen und zu konfigurieren, die wir für jede
Software benötigen.

Erstellen einer Port-Building-Liste und Festlegen von Port-Optionen
-------------------------------------------------------------------

Wir werden eine Liste von Ports erstellen, die wir direkt Poudriere
übergeben können.

Die Datei sollte die Port-Kategorie gefolgt von einem Schrägstrich und
dem Port-Namen auflisten, um ihre Position in der Ports-Struktur
widerzuspiegeln:

.. code:: bash

   port_category/first_port
   port_category/second_port
   port_category/third_port

Alle erforderlichen Abhängigkeiten werden ebenfalls automatisch
erstellt. Sie müssen also nicht die gesamte Abhängigkeitsstruktur der
Ports aufspüren, die Sie installieren möchten.Sie können diese Datei
manuell erstellen, aber wenn Ihr Basissystem bereits die meiste Software
erstellen möchte, können Sie diese Liste mit portmaster erstellen.

Bevor Sie dies tun, ist es in der Regel eine gute Idee, alle nicht
benötigten Abhängigkeiten von Ihrem System zu entfernen, um die
Portliste so sauber wie möglich zu halten. Sie können dies tun, indem
Sie Folgendes eingeben:

.. code:: bash

   pkg autoremove

Danach können wir eine Liste der Software, die wir explizit auf unserem
Build-System installiert haben, erhalten:

.. code:: bash

   pkg query -e "%a==0" "%o" | sort -d | tee /usr/local/etc/poudriere.d/port-list 

Überprüfen Sie die Liste. Wenn es Ports gibt, die Sie nicht hinzufügen
möchten, entfernen Sie die zugehörige Zeile. Dies ist auch eine
Gelegenheit, zusätzliche Ports hinzuzufügen, die Sie möglicherweise
benötigen.

Wenn Sie zum Erstellen Ihrer Ports bestimmte make.conf-Optionen
verwenden, können Sie für jede Jail innerhalb Ihres Verzeichnisses
/usr/local/etc/poudriere.d eine Datei make.conf erstellen. Zum Beispiel
können wir für unser Jail eine make.conf-Datei mit diesem Namen
erstellen:

.. code:: bash

   nano /usr/local/etc/poudriere.d/11-2x64-make.conf

Ich persönlich erstelle lieber eine globale make.conf die für alle Jails
gültig ist.

.. code:: bash

   nano /usr/local/etc/poudriere.d/make.conf

Im Inneren können Sie alle Optionen angeben, die Sie beim Erstellen
Ihrer Ports verwenden möchten. Wenn Sie beispielsweise keine
Dokumentation, Beispiele, Unterstützung für die Muttersprache oder
X11-Unterstützung erstellen möchten, können Sie Folgendes festlegen:

.. code:: bash

   OPTIONS_UNSET+= DOCS NLS X11 EXAMPLES

Sie können aber auch
`Default_Versions <https://wiki.freebsd.org/Ports/DEFAULT_VERSIONS>`__
angeben. In meinem Fall schaut es so aus:

.. code:: bash

   DEFAULT_VERSIONS+= gcc=7 linux=c7 mysql=10.3m php=7.2 samba=4.7 ssl=openssl

In diesem Fall werden alle Pakete und/oder deren Abhängigkeiten mit der
angegebenen Version gebaut.

Danach können wir jeden unserer Ports konfigurieren, der Dateien mit den
ausgewählten Optionen erstellt.

Sie können alles konfigurieren, das noch nicht mit dem Befehl options
konfiguriert wurde. Wir sollten sowohl den Port-Baum, den wir erstellt
haben (mit der Option -p), als auch das Jail, für das wir diese Optionen
festlegen (mit der Option -j), übergeben. Wir müssen auch die Liste der
Ports angeben, die wir mit der Option -f konfigurieren möchten.

.. code:: bash

   poudriere options -j 11-2x64 -p HEAD -f /usr/local/etc/poudriere.d/port-list

Sie sehen einen Dialog für jeden der Ports in der Liste und alle
Abhängigkeiten, für die keine entsprechenden Optionen im Verzeichnis
-options festgelegt sind. Die Angaben in Ihrer Datei make.conf werden in
den Selektionsbildern vorausgewählt. Wählen Sie alle Optionen aus, die
Sie verwenden möchten.

Wenn Sie die Optionen für Ihre Ports in Zukunft neu konfigurieren
möchten, können Sie den obigen Befehl mit der Option -c erneut
ausführen. Dadurch werden Ihnen alle verfügbaren Konfigurationsoptionen
angezeigt, unabhängig davon, ob Sie in der Vergangenheit eine Auswahl
getroffen haben:

.. code:: bash

   poudriere options -c -j 11-2x64 -p HEAD -f /usr/local/etc/poudriere.d/port-list

Die Ports bauen
---------------

Jetzt sind wir endlich bereit, mit dem Aufbau von Ports zu beginnen.

Geben Sie Folgendes ein, um Ihre Jail zu aktualisieren:

.. code:: bash

   poudriere jail -u -j 11-2x64

Geben Sie Folgendes ein, um Ihre Ports-Struktur zu aktualisieren:

.. code:: bash

   poudriere ports -u -p HEAD

Sobald dies abgeschlossen ist, können wir den Build-Prozess starten.

**Hinweis:** Dies kann ein sehr langer Prozess sein. Wenn Sie über SSH
mit Ihrem Server verbunden sind, empfehlen wir Ihnen, das ``screen``
Paket zu installieren und eine Sitzung zu starten:

.. code:: bash

   pkg install screen

   rehash

   screen

Um den Build zu starten, müssen wir nur den Befehl bulk verwenden und
auf alle unsere einzelnen Teile zeigen, die wir konfiguriert haben. Wenn
Sie die Werte in diesem Handbuch verwendet haben, sieht der Befehl
folgendermaßen aus:

.. code:: bash

   poudriere bulk -j 11-2x64 -p HEAD -f /usr/local/etc/poudriere.d/port-list

Dies startet eine Reihe von Arbeitern (abhängig von Ihrer
poudrière.conf-Datei oder der Anzahl der verfügbaren CPUs) und beginnt
mit dem Aufbau der Ports.

Während des Erstellungsprozesses können Sie jederzeit Informationen über
den Fortschritt erhalten, indem Sie die STRG-Taste gedrückt halten und t
drücken.

Bestimmte Teile des Prozesses produzieren mehr Output als andere.

Einrichten von Nginx für das Bereitstellen des Front-Ends und des Repositorys
-----------------------------------------------------------------------------

Für diesen Schritt muss der `NGINX <NGINX>`__ mit Virtualhosts
eingerichtet sein.

In der /etc/hosts wird der folgende Hostname eingetragen: 127.0.0.1
bsd.«domain»

Damit Poudriere unter NGINX läuft wird unter
/usr/local/etc/nginx/vhosts/ folgende Datei unter dem Namen
poudriere.conf mit diesem Inhalt angelegt:

.. code:: bash

   server {
     listen 80 default;
     server_name bsd.><domain>>;
     root /usr/local/share/poudriere/html;

     location /data {
       alias /usr/local/poudriere/data/logs/bulk;
       autoindex on;
     }

     location /packages {
       root /usr/local/poudriere/data;
       autoindex on;
     }
   }

Anschließend den NGINX neu starten:

.. code:: bash

   service nginx restart

Als nächstes werden wir eine kleine Änderung an unserer mime.types-Datei
vornehmen. Wenn Sie bei den aktuellen Einstellungen auf ein Protokoll im
Webbrowser klicken, wird die Datei heruntergeladen und nicht als
normaler Text angezeigt. Wir können dieses Verhalten ändern, indem Sie
Dateien mit der Endung .log als Nur-Text-Dateien markieren.

Öffnen Sie die Datei Nginx mime.types in Ihrem Texteditor:

.. code:: bash

   nano /usr/local/etc/nginx/mime.types

Suchen Sie den Eintrag, der den text / plain-Inhaltstyp angibt, und
hängen Sie das Protokoll an das Ende der aktuellen Liste der Dateitypen
an, getrennt durch ein Leerzeichen:

.. code:: bash

   text/mathml                         mml;
   text/plain                          txt log;
   text/vnd.sun.j2me.app-descriptor    jad;

Jetzt können Sie das Poudriere-Webinterface anzeigen, indem Sie in Ihrem
Webbrowser zum Domain-Namen oder zur IP-Adresse Ihres Servers gehen
bsd.<<domain>>

Paket-Clients konfigurieren
---------------------------

Nachdem Sie nun Pakete erstellt und ein Repository für die
Bereitstellung Ihrer Pakete konfiguriert haben, können Sie Ihre Clients
so konfigurieren, dass sie Ihren Server als Quelle für ihre Pakete
verwenden.

Konfigurieren des Build-Servers für die Verwendung des eigenen Paket-Repos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Wir können damit beginnen, den Build-Server so zu konfigurieren, dass er
die Pakete verwendet, die er erstellt hat.

Zuerst müssen wir ein Verzeichnis für unsere
Repository-Konfigurationsdateien erstellen:

.. code:: bash

   mkdir -p /usr/local/etc/pkg/repos

In diesem Verzeichnis können wir unsere Repository-Konfigurationsdatei
erstellen. Es muss in .conf enden, daher werden wir es poudriere.conf
nennen, um seinen Zweck zu verdeutlichen:

.. code:: bash

   nano /usr/local/etc/pkg/repos/poudriere.conf =>

   poudriere: {
       url: "file:///usr/local/poudriere/data/packages/11-2x64-HEAD",
       mirror_type: "srv",
       signature_type: "pubkey",
       pubkey: "/usr/local/etc/ssl/certs/poudriere.cert",
       enabled: yes,
       priority: 100
   }

Wenn Sie nur Pakete auswählen, die Sie selbst erstellt haben (die
sicherere Route), können Sie die Prioritätseinstellung weglassen, aber
Sie sollten die Standard-Repositorys deaktivieren. Sie können dies tun,
indem Sie eine andere Repo-Datei erstellen, die die
Standard-Repository-Datei überschreibt und sie deaktiviert:

.. code:: bash

   nano /usr/local/etc/pkg/repos/freebsd.conf => 

   FreeBSD: {
       enabled: no
   }

Unabhängig von Ihrer Konfigurationsauswahl sollten Sie jetzt bereit
sein, Ihr Repository zu verwenden. Aktualisieren Sie Ihre Paketliste,
indem Sie Folgendes eingeben:

.. code:: bash

   pkg update

Jetzt kann Ihr Server den Befehl pkg verwenden, um Pakete von Ihrem
lokalen Repository zu installieren.

Konfigurieren von Remote-Clients für die Verwendung des Repository Ihres Build-Rechners
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Einer der überzeugendsten Gründe, Poudriere auf einer Build-Maschine
einzurichten, besteht darin, diesen Host als Repository für viele andere
Maschinen zu verwenden. Alles, was wir tun müssen, um dies zum Laufen zu
bringen, ist das Herunterladen des öffentlichen SSL-Zertifikats von
unserer Build-Maschine und das Einrichten einer ähnlichen
Repository-Definition.

Um von unseren Client-Computern aus eine Verbindung mit unserem
Build-Host herzustellen, sollten Sie einen SSH-Agenten auf **Ihrem
lokalen Computer starten**, um Ihre SSH-Schlüsselanmeldeinformationen zu
speichern.

Sie müssen Ihren SSH-Schlüssel hinzufügen, indem Sie Folgendes eingeben:

.. code:: bash

   ssh-add

Anschließend können Sie Ihre lokalen SSH-Anmeldeinformationen an Ihre
Clientcomputer weiterleiten, wenn Sie die Verbindung mit dem -A-Flag
herstellen. Auf diese Weise können Sie von Ihrem Clientcomputer aus auf
einen beliebigen Computer zugreifen, als ob Sie von Ihrem Heimcomputer
darauf zugreifen würden:

.. code:: bash

   ssh -A freebsd@client_domain_or_IP

Sobald Sie sich auf Ihrem Remote-Client-Rechner befinden, müssen Sie
zuerst die Verzeichnisstruktur erstellen (falls diese nicht existiert),
damit Sie das Zertifikat speichern können. Wir werden weitermachen und
ein Verzeichnis für Schlüssel erstellen, damit wir es für zukünftige
Aufgaben verwenden können:

.. code:: bash

   mkdir -p /usr/local/etc/ssl/{keys,certs}

Jetzt können wir uns mit SSH mit unserer Build-Maschine verbinden und
die Zertifikatsdatei an unseren Client-Rechner zurückleiten. Da wir
unsere SSH-Zugangsdaten weitergeleitet haben, sollten wir dies tun
können, ohne ein Passwort anzufordern:

.. code:: bash

   ssh user@server_domain_or_IP 'cat /usr/local/etc/ssl/certs/poudriere.cert' | tee /usr/local/etc/ssl/certs/poudriere.cert

Dieser Befehl stellt mithilfe Ihrer lokalen SSH-Anmeldeinformationen
eine Verbindung zur Build-Maschine von Ihrem Clientcomputer her. Sobald
die Verbindung hergestellt ist, zeigt sie den Inhalt Ihrer
Zertifikatsdatei an und leitet sie durch den SSH-Tunnel zurück zu Ihrem
Remote-Client-Rechner. Von dort verwenden wir die tee Kombination, um
das Zertifikat in unser Verzeichnis zu schreiben.

Sobald dies abgeschlossen ist, können wir unsere
Repository-Verzeichnisstruktur wie auf dem Build-Rechner selbst
erstellen:

.. code:: bash

   mkdir -p /usr/local/etc/pkg/repos

Jetzt können wir eine Repository-Datei erstellen, die der auf der
Build-Maschine verwendeten ähnlich ist:

.. code:: bash

   nano /usr/local/etc/pkg/repos/poudriere.conf =>

   poudriere: {
       url: "http://server_domain_or_IP/packages/11-2x64-HEAD/",
       mirror_type: "http",
       signature_type: "pubkey",
       pubkey: "/usr/local/etc/ssl/certs/poudriere.cert",
       enabled: yes,
       priority: 100
   }

Wenn Sie nur Ihre kompilierten Pakete verwenden möchten, sollte Ihre
Datei in etwa so aussehen:

.. code:: bash

   poudriere: {
       url: "http://server_domain_or_IP/packages/11-2x64-HEAD/",
       mirror_type: "http",
       signature_type: "pubkey",
       pubkey: "/usr/local/etc/ssl/certs/poudriere.cert",
       enabled: yes
   }

Wenn Sie nur Ihre eigenen Pakete verwenden, denken Sie daran, eine
andere Repository-Konfigurationsdatei zu erstellen, um die
standardmäßige FreeBSD-Repository-Konfiguration zu überschreiben:

.. code:: bash

   nano /usr/local/etc/pkg/repos/freebsd.conf => 

   FreeBSD: {
       enabled: no
   }

Nachdem Sie fertig sind, aktualisieren Sie Ihre pkg-Datenbank, um mit
der Verwendung Ihrer benutzerdefinierten kompilierten Pakete zu
beginnen:

.. code:: bash

   pkg update

Cronjob
-------

Damit Poudriere automatisch die Jails aktualisiert und die Pakete baut
werden Sie einen Cronjob einrichten.

Erstellen Sie folgende Datei unter
``/usr/local/etc/poudriere.d/scripts``:

.. code:: bash

   mkdir /usr/local/etc/poudriere.d/scripts

   nano /usr/local/etc/poudriere.d/scripts/poudriere-cron.sh =>

   #!/bin/sh

   SCRIPTNAME=`basename "$0"`
   PORTSTREE="HEAD"
   POUDRIERE="/usr/local/bin/poudriere"
   PORTLIST="/usr/local/etc/poudriere.d/port-list"
   JAIL="11-2x64"
   REPOS="HEAD"
   URL="https://bsd.<<domain>>/"

   poudriere_build() {
           for REPO in $REPOS; do
                   echo "Started $REPO ("`/bin/date | /usr/bin/tr -d '\n'`")"
                   "$POUDRIERE" bulk -j "$JAIL" -p "$PORTSTREE"  -f "$PORTLIST" > /dev/null
                   echo "    Cleaning $REPO ("`/bin/date | /usr/bin/tr -d '\n'`")"
                   "$POUDRIERE" pkgclean -j "$JAIL" -p "$PORTSTREE" -f "$PORTLIST" -y > /dev/null
                   echo "    Finished $REPO ("`/bin/date | /usr/bin/tr -d '\n'`")"
           done
   }

   echo "This is a log of poudriere. Details: $URL"
   echo ""
   echo "[$SCRIPTNAME] Updating ports tree..."
   "$POUDRIERE" ports -p "$PORTSTREE" -u > /dev/null
   if [ $? -ne 0 ]; then
           echo "    Error updating ports tree."
           exit 1
   fi
   echo "    Ports tree has been updated."

   poudriere_build

   echo "[$SCRIPTNAME] Cleaning distfiles..."
   "$POUDRIERE" distclean -p "$REPOS" -f "$PORTLIST" -y > /dev/null

   echo "[$SCRIPTNAME] Finished. ("`/bin/date | /usr/bin/tr -d '\n'`")"

   exit 0 

Anschließend ausführbar machen:

.. code:: bash

   chmod +x /usr/local/etc/poudriere.d/scripts/poudriere-cron.sh

Dannach sollten Sie das Script einmal ausführen, um zu testen ob alles
korrekt läuft.

Wenn alles passt können Sie in der ``/etc/crontab`` wie folgt einbinden:

.. code:: bash

   nano /etc/crontab => 

   10      0       *       *       *       root    /usr/local/etc/poudriere.d/scripts/poudriere-cron.sh

Der Build-Prozess wird dann jede Nacht ausgeführt. Wenn Sie Pakete
haben, wie zum Beispiel Chromium, wo der Build länger braucht als 24
Stunden, kann es hier angepasst werden.

Befehlsliste
------------

**Poudriere make.conf**

.. code:: bash

   nano /usr/local/etc/poudriere.d/make.conf

**Poudriere conf**

.. code:: bash

   nano /usr/local/etc/poudriere.conf

**Portlist**

.. code:: bash

   nano  /usr/local/etc/poudriere.d/port-list

**Eine neue Jail erstellen**

.. code:: bash

   poudriere jail -c -j 11-2x64 -v 11.2-RELEASE

**Eine Jail aktualisieren**

.. code:: bash

   poudriere jail -u -j 11-2x64

**Ports aktualisieren**

.. code:: bash

   poudriere ports -u -p HEAD

**Unbenutze Pakete entfernen**

.. code:: bash

   poudriere pkgclean -j 11-2x64 -p HEAD -f /usr/local/etc/poudriere.d/port-list

**Optionen für die Portliste setzen**

.. code:: bash

   poudriere options -c -j 11-2x64 -p HEAD -f /usr/local/etc/poudriere.d/port-list

\*\* Optionen für ein Paket setzen*\*

.. code:: bash

   poudriere options -c -j 11-2x64 -p HEAD  databases/mariadb103-client

**Pakete ohne cronjob bauen**

.. code:: bash

   poudriere bulk -j 11-2x64 -p HEAD -f /usr/local/etc/poudriere.d/port-list

**Bei einem Systemupgrade die Options auf die neue Version kopieren**

.. code:: bash

   cp -R /usr/local/etc/poudriere.d/11-2x64-HEAD-options/**.* /usr/local/etc/poudriere.d/12-0x64-HEAD-options/

**Anlegen des Quarterly Ports-Trees**

.. code:: bash

   poudriere ports -c -B branches/2018Q3 -m svn+https -p 2018Q3

* :ref:`genindex`

Zuletzt geändert: |date|

