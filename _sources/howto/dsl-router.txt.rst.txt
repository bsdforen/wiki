DSL-Router
==========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

HOWTO - DSL Router mit OpenBSD 4.1

`Original <http://www.scatterbrained.de/openbsd/>`__ von Jürgen Graf
(openbsd@scatterbrained.de)

Dieser Artikel wird in freundlicher Genehmigung von Jürgen Graf hier
veröffentlicht. Das `Original <http://www.scatterbrained.de/openbsd/>`__
ist weiterhin verfügbar.

Der Artikel beschreibt die komplette Installation eines OpenBSD als
Routerplattform mit allen Goodies die ein "Heim-Router" zur Verfügung
stellen sollte.

Was sind die Ziele dieses HOWTOs?
---------------------------------

Dieses HOWTO beschreibt wie man OpenBSD 4.1 auf einem Rechner
installiert und diesen so einrichtet, dass er als DSL Router verwendet
werden kann. Bei allen Einstellungen die auf dem System vorgenommen
wurden, war vor allem die grösstmögliche Funktionalität wichtig. Bei
Entscheidungen zwischen grösserer Sicherheit und mehr Funktionalität
bzw. bequemerer Handhabung, musste die Sicherheit zurückstecken. Soweit
es ging habe ich trotzdem versucht das System so sicher wie möglich zu
konfigurieren. Auch werde ich an den Stellen bei denen ich mich für mehr
Funktionalität und gegen Sicherheit entschieden habe darauf hinweisen.
Alle Angaben und Vorgehensweisen beschreibe ich hier nach meinem besten
Wissen, was aber an einigen Stellen falsch oder etwa unvollständig sein
kann. Für Verbesserungsvorschläge bin ich deswegen immer dankbar.

Gleich am Anfang sollte man klar stellen, dass ein Router unter OpenBSD
momentan noch nicht in der Lage ist, wie es z.B. ein Linux System
könnte, einige Internetdienste (NetMeeting, Datei senden/empfangen bei
ICQ und AIM, etc.) allen Computern im internen Netzwerk zur Verfügung zu
stellen. Aber es existiert zumindest die Möglichkeit durch
Portforwarding diese Services einzelnen Computern zu ermöglichen.
Richtigstellungen bitte selbst durchführen, oder unter Diskussion
klarstellen.

Jetzt aber zur wichtigsten Frage:

**Was kann der Computer wenn ich alles wie beschrieben eingerichtet
habe?**

-  Auf ihm wird mit OpenBSD 4.1 das derzeit sicherste, frei verfügbare
   Betriebsystem laufen. Dieses Betriebsystem wird von vielen
   Sicherheitsexperten ständig aktiv auf Sicherheitslücken hin geprüft
   und entsprechend verbessert. Alle weiteren Informationen findet man
   auf `www.openbsd.org <http://www.openbsd.org/de/>`__.
-  Er kann eine DSL Internetverbindung aufbauen und wird sich bei einer
   Trennung automatisch wieder einwählen.
-  Die Internetverbindung wird per NAT (unter Linux als Masquerading
   bekannt) allen Computern im internen Netzwerk völlig transparent
   ermöglicht. Dabei können folgende Dienste genutzt werden:

   -  HTTP, HTTPS, SMTP, POP3, IRC, ... usw. - alle Internetdienste bei
      denen nur der Client eine Verbindung aufbauen muss.
   -  FTP im aktiv und passiv Modus über einen Proxy.
   -  EMule und DirectPlay können einem ausgewähltem Rechner im Netzwerk
      zur Verfügung gestellt werden.

-  Die Clients können sich per DHCP automatisch konfigurieren.
-  Es wird ein FTP Server auf dem Router laufen, der über das Internet
   und über das interne Netz zugänglich ist.
-  Dank eines Timeservers können sich alle im LAN befindlichen Computer
   die exakte aktuelle Zeit abholen.
-  Der Computer ist ständig über einen dynamischen Domainnamen im
   Internet erreichbar. Einen solchen Namen, wie z.B.:
   "**mein-router.no-ip.com**", kann man sich bei
   `www.no-ip.com <http://www.no-ip.com/>`__ kostenlos registrieren.
-  Er ist über SSH administrierbar. D.h. er kann bequem von jedem
   Computer, der an das LAN oder Internet angeschlossen ist, gewartet
   werden. Einen komfortabler SSH-Client für Win*:
   `Putty <http://www.chiark.greenend.org.uk/~sgtatham/putty/>`__
-  Den Computern im LAN wird ein DNS-Server zur Verfügung gestellt, was
   vor allem beim Einrichten dieser Rechner sehr hilfreich ist.
-  Bei manchen IRC-Netzen (z.B.: DALnet) muss ein Ident-Server laufen,
   der über Benutzernamen auskunft gibt, damit man Zugang zu ihnen
   erhält. Deswegen läuft ein "fake" Ident-Server, der erfundene
   Benutzernamen zurückgibt.

**Was brauche ich dafür?**

-  Einen alten Computer (ab ca. 486DX2/66 - 32MB RAM) mit:

   -  Festplatte ab ca. 500MB und Diskettenlaufwerk (die
      Grundinstallation benötigt ohne X Server <200MB)
   -  2 Netzwerkkarten (möglichst PCI - da diese automatisch erkannt
      werden)
   -  optional ein bootfähiges CD-Rom
   -  es ist auch möglich OpenBSD auf Rechnern mit sehr wenig RAM (<
      20MB) noch installiert zu bekommen. Dafür ist aber ein spezielles
      Vorgehen, wie es in der `Installations
      FAQ <http://www.openbsd.org/faq/de/faq4.html#SmallRAM>`__
      beschrieben wird, nötig.

-  Für den Zeitraum der Installation braucht man auch noch ein Keyboard
   und einen Monitor.
-  Einen schon bestehenden Internetzugang (für die Installation ohne
   CD-Rom). Am Besten nimmt man da seinen eigenen DSL-Anschluss, den
   später dieser Computer nutzen soll.
-  Sehr hilfreich sind schon vorhandene Kenntnisse über UNIX-artige
   Betriebssysteme, wie \*BSD oder Linux, und deren Programme. Auf
   OpenBSD gibt es standardmäsig nur `vi <http://www.bostic.com/vi/>`__
   als Editor. Ich werde aber auch erklären wie man einen anderen Editor
   installiert, der sich etwas leichter bedienen lässt. Grundkenntnisse
   in der Bedienung von vi können allerdings nie schaden.
-  Viel Geduld, gute Nerven und eine übermäsige Begeisterungsfähigkeit
   (abhängig vom vorhergehenden Punkt ;)).

Installation von OpenBSD 4.1
----------------------------

Ich werde hier nicht genau auf jede Einzelheit, die bei der Installation
von OpenBSD zu beachten ist eingehen. Für diesen Zweck gibt es schon
ausreichend viele und gute Informationsquellen (z.B. die `Installations
FAQ <http://www.openbsd.org/faq/de/faq4.html>`__ oder allgemein alle
anderen `FAQ's <http://www.openbsd.org/faq/de/index.html>`__ die sich
auch noch auf dieser Seite befinden und natürlich
`google <www.google.de>`__ ).

Für eine Installation von CD empfehle ich, dass man sich die original
OpenBSD CD bestellt (ebenfalls auf
`www.openbsd.org <http://www.openbsd.org/de/orders.html>`__) - schon
alleine wegen der wirklich coolen Aufkleber, die mitgeliefert werden.
Ausserdem kann man sich bei dieser Gelegenheit gleich mit passenden
Postern und T-Shirts eindecken ;). Das soll selbstverständlich die
Härteren unter uns (oder diejenigen ohne bootfähiges CD-Rom) nicht davon
abhalten eine Installation per FTP über das Internet durchzuführen. Das
hat unter anderem auch den Vorteil, dass man immer die absolut
aktuellste Version bekommt. Hierfür benötigt man nur einen bestehenden
Internetzugang, den man über das interne Netzwerk erreichen kann (sprich
einen schon vorhandenen Router...), und eine einzige formatierte
Diskette. Das Image dieser Diskette findet man z.B. auf einem der
deutschen OpenBSD `FTP Mirrors <http://www.openbsd.org/de/ftp.html>`__:
`floppy41.fs <ftp://openbsd.informatik.uni-erlangen.de/pub/OpenBSD/4.1/i386/floppy41.fs>`__.
Diese Datei muss dann nur noch per

::

   # dd if=floppy41.fs of=/dev/fd0 bs=32k

(Linux/*BSD) oder `rawwritewin <http://www.chrysocome.net/rawwrite>`__
(Win*) auf eine formatierte 1.44MB Diskette kopiert werden.

Bei einer Installation per FTP (Diskette) sollte nun eine Netzwerkkarte
des Computers an das interne Netzwerk angeschlossen werden, über das
dieser Zugang zum Internet erhalten kann. Dann Diskette oder CD ins
Laufwerk und los gehts!

Nach einem ganzen Haufen an blau-weissen Textzeilen erscheint die Frage

::

  (I)nstall,(U)pgrade or (S)hell? **i**

welche wir selbstverständlich mit "**i**" beantworten. Danach wird man
noch kurz nach dem Terminaltyp gefragt 

::

  Specify terminal type [vt220] **Enter**

wobei man einfach per Enter die Vorgabe übernehmen kann. Als nächstes
hat man die Möglichkeit die Tastatur auf deutsches Layout umzustellen.

::

  Do you wish to select a keyboard encoding table? [no] **yes**
  Select your keyboard type: (P)C-AT/XT, (U)SB or 'done' [P] **Enter** 
  The available keyboard encoding tables are:

    be de dk es fr it jp lt no pt ru sf sg sv ua uk us

  Table name? (or 'done') [us] **de.nodead** 
  keyboard mapping set to de.nodead 

Diese Einstellung wird auch sofort übernommen, so dass ab nun nicht mehr lange
nach den passenden Tasten gesucht werden muss. Jetzt wird es schon
interessanter: OpenBSD fragt uns auf welche Festplatte wir das System
installieren wollen.

::

  IS YOUR DATA BACKED UP? As with anything that modifies disk
  contens, this program can cause SIGNIFICANT data loss.
  
  It is often helpful to have the installation notes handy. For complex disk 
  configurations, relevant disk hardware manuals and a calculator are useful.
  
  Proceed with install? [no] **yes** 
  Cool! Let's get to it...
  
  You will now initialize the disk(s) that OpenBSD will use. To enable all
  avaliable security features you should configure the disk(s) to allow
  the creation of separate filesystems for /, /tmp, /var, /usr, and /home.
  
  Available disks are: wd0 
  Which one is the root disk? (or done) [wd0] **Enter**

Da ich davon ausgehe, dass dieser Computer ein reiner OpenBSD Rechner
wird, sollte wohl gleich die erste Platte (wd0) die richtige Wahl sein.
Die "**wdX**" Kürzel stehen dabei für folgende IDE Platten: wd0 -
primary master, wd1 - primary slave, wd2 - secondary master, wd3 -
secondary slave. Nächste Frage: 

::

  Do you want to use the \*all\* of wd0 for OpenBSD? [no] **yes**

Logisch wollen wir das, also "**yes**". Und schon befinden wir uns an
einem kritischen Punkt in der Installation - den Disklabels. Im
Gegensatz zu Betriebssystemen wie Windows oder Linux benutzt \*BSD
Disklabels zur Erstellung von "**Unterpartitionen**". Das soll uns nun
aber nicht weiter verwirren. Das Einzige was es praktisch bedeutet ist,
dass man auf der Platte nur eine Partition für OpenBSD erstellt (was
nach dem Bejahen der vorhergehenden Frage automatisch geschehen ist) und
diese dann mit Hilfe der Disklabels weiter unterteilt. Und das machen
wir jetzt auch. Beim Prompt angekommen kann man sich mit dem Kommando
"**p**" einige Daten über seine Platte anzeigen lassen: 

::

  Initial label editor (enter '?' for help at any prompt)
  > p
  device: /dev/rwd0c
  type: ESDI
  disk: ESDI/IDE disk
  label: VMware Virtual I
  bytes/sector: 512
  sectors/track: 63
  tracks/cylinder: 15
  sectors/cylinder: 945
  cylinders: 6502
  total sectors: 6144390
  free sectors: 6136641
  rpm: 3600
  
  16 partitions:
  #        size    offset    fstype    [fsize bsize   cpg]
  a:  6136641        63    unused         0     0
  c:  6144390         0    unused         0     0
  > Enter

Interessant ist hier vor allem der untere Teil der Ausgabe, auf dem wir
die momentan vorhandenen Disklabels (a: und c:) mit ihrer jeweiligen
Grösse angezeigt bekommen. Dabei sind vor allem zwei Dinge zu beachten:

#. Die Namen der Disklabels (a, b, ...) haben **nichts** mit den aus der
   M$-Welt bekannten Laufwerksbuchstaben zu tun.
#. Das Label "**c**" steht immer als Synonym für die gesamte Platte.

Nun löscht man zuerst einmal alle vorhandenen Labels (ausser "**c**"
natürlich, welches man ohnehin nicht löschen kann), damit wir den ganzen
Plattenplatz sinnvoll partitionieren können. In unserem Fall muss nur
"**a**" gelöscht werden ("**d a**"). Danach lege ich in diesem Beispiel
eine **50MB grosse Rootpartition** mit dem Label "a" an. Es folgen
**100MB swap** (ca. 2xRAM Grösse aber maximal 300MB reichen), **50MB für
/tmp**, **80MB für /var** und **220MB für /usr**.

Wenn man mit dem Plattenplatz nicht sparsam umgehen muss, dann sind
folgende Grössen von der offiziellen OpenBSD Seite empfohlen: **150MB
für /**, **300MB für swap**, **120MB für /tmp**, **80MB für /var**,
**2GB für /usr** und **4GB für /home** - Der Rest kann unverteilt
bleiben, um so fexibel zu bleiben und später an beliebiger Stelle Platz
hinzufügen zu können.

::

  > d a
  > a a
  offset: [63] Enter
  size: [6136641] 50m
  Rounding to nearest cylinder: 101997
  FS type: [4.2BSD] Enter
  mount point: [none] /
  > a b
  offset: [102060] Enter
  size: [6034644] 100m
  Rounding to nearest cylinder: 205065
  FS type: [swap] Enter
  > a d
  offset: [307125] Enter
  size: [5829579] 50m
  Rounding to nearest cylinder: 102060
  FS type: [4.2BSD] Enter
  mount point: [none] /tmp
  > a e
  offset: [409185] Enter
  size: [5727519] 80m
  Rounding to nearest cylinder: 163485
  FS type: [4.2BSD] Enter
  mount point: [none] /var
  > a f
  offset: [572670] Enter
  size: [5564034] 220m
  FS type: [4.2BSD] Enter
  mount point: [none] /usr
  >


Auf die Extrapartitionen für "**/tmp**" und "**/var**" sollte man
möglichst nicht verzichten, da auf ihnen temporäre Dateien und Logfiles
gespeichert werden. Wenn diese zuviel werden ist so nur die "**/var**"
oder "**/tmp**" Partition überfüllt und das Rootdateisystem bleibt
unbehelligt. Falls man über eine grössere Festplatte verfügt, ist es
sinnvoll noch etwas mehr Platz für die "**/var**" Partition zu
reservieren. Will man eine graphische Oberfläche installieren oder die
neusten Errungenschaften des OpenBSD-Teams selbst compilieren, dann muss
"**/usr**" grösser gewählt werden (eine vernünftige Obergrenze ist
2200M). Den Rest der Platte kann man dann als "**/home**" Partition
verwenden. Das ist auch die Partition, die unser FTP Server später
verwenden wird. Will man keinen FTP Server installieren, dann kann man
diese Partition getrost kleiner gestalten oder ganz weglassen und den
Platz auf die restlichen Partitionen verteilen.

::

  > a g
  offset: [1023435] Enter
  size: [5113269] Enter
  FS type: [4.2BSD] Enter
  mount point: [none] /home
  >

Das fertige Werk lässt sich danach mit "**p**" nochmals begutachten und
sollte in etwa so aussehen:

::

  > p
  device: /dev/rwd0c
  type: ESDI
  disk: ESDI/IDE disk
  label: VMware Virtual I
  bytes/sector: 512
  sectors/track: 63
  tracks/cylinder: 15
  sectors/cylinder: 945
  cylinders: 6502
  total sectors: 6144390
  free sectors: 0
  rpm: 3600
  
  16 partitions:
  #        size    offset    fstype    [fsize bsize   cpg]
    a:   101997        63    4.2BSD      1024  8192    16  # /
    b:   205065    102060      swap
    c:  6144390         0    unused         0     0
    d:   102060    307125    4.2BSD      1024  8192    16  # /tmp
    e:   163485    409185    4.2BSD      1024  8192    16  # /var
    f:   450765    572670    4.2BSD      1024  8192    16  # /usr
    g:  5113269   1023435    4.2BSD      1024  8192    16  # /home
  > w
  > q

Wenn alles stimmt, dann wird mit "**w**" das Ganze auf die Platte
gebannt und mit "**q**" der Labeleditor verlassen. Es wird nocheinmal
kurz angezeigt welche Partitionen erstellt wurden und wo sie im
Verzeichnissbaum eingehängt sind und dann gehts mit "**done**" weiter.
Sind noch mehr Festplatten im Rechner eingebaut, so hat man nun noch die
Möglichkeit diese analog zu konfigurieren. In diesem Beispiel hat der
Rechner nur eine Platte also gehts jetzt schon weiter mit dem
formatieren.

::

  No label changes.
  
  The root filesystem will be mounted on wd0a.
  wd0b will be used for swap space.
  Mount point for wd0d (size=51030k), none or done? [/tmp] Enter
  Mount point for wd0e (size=81742k), none or done? [/var] Enter
  Mount point for wd0f (size=225382k), none or done? [/usr] Enter
  Mount point for wd0g (size=2556634k), none or done? [/home] Enter
  Mount point for wd0d (size=51030k), none or done? [/tmp] done
  No more disks to initialize.
  
  You have configured the following partitions and mount points:
  
  wd0a /
  wd0d /tmp
  wd0e /var
  wd0f /usr
  wd0g /home
  
  The next step creates a filesystem on each partition, ERASING existing data.
  Are you really sure that you're ready to proceed? [no] yes

Danach ist die Platte formatiert und man wird gefragt, ob man das
Netzwerk konfigurieren will. (Als "**hostname**" kann man
selbstverständlich auch einen phantasievolleren Namen als "**router**"
verwenden. Er sollte allerdings nur aus Kleinbuchstaben bestehen.)

::

  System hostname? (short form, e.g. 'foo') **router** 
  Configure the network? [yes] **Enter**
  Availiable interfaces are: rl0 rl1. 
  Which one do you wish to initialize? (or done) [rl0] **Enter**

Die Devicenamen "**rl0**" und "**rl1**" beziehen sich auf den Hersteller
des Chipsatzes ("**rl**" steht für den bekannten "*allerwelts billigst*"
RTL8139 Chip - "**de**" würde z.B. bei einem D-Link Chip erscheinen,
usw.) und die Anzahl der mit diesem Chipsatz installierten Karten. In
diesem Beispiel sind es also zwei Netzwerkkarten mit dem RTL8139 Chip.
Wer sich jetzt fragt: "*Woher weiss ich welche die Karte Nr. 0 und
welche Nr. 1 ist, wenn ich zwei Karten mit gleichem Chipsatz verwende?*"
dem sei gesagt: "*Keine Ahnung, ich wusste es auch nicht.*". Hier geht
wohl Probieren über Studieren - aber so ein Netzwerkkabel ist ja schnell
umgesteckt ;) . Eine der beiden Netzwerkkarten muss auf jeden Fall für
das lokale Netzwerk konfiguriert werden, während die Zweite, an der
später das DSL-Modem angeschlossen wird, völlig unkonfiguriert bleiben
sollte. Bei einer Installation per FTP muss man natürlich eine
Netzwerkkarte so einstellen, dass man damit auf das Internet zugreifen
kann. Im folgenden Beispiel gehe ich von einem Netzwerk mit dem
Adressbereich 192.168.1.0/24 aus, wobei die Adresse 192.168.1.1 noch
nicht vergeben ist.

::

  Symbolic (host) name for rl0? [router] Enter
  The default media for rl0 is
          media: Ethernet autoselect (100baseTX full-duplex)
  Do you want to change the default media? [no] Enter
  IP address for rl0? (or 'dhcp') 192.168.1.1
  Netmask? [255.255.255.0] Enter
  Done - Avaliable interfaces are: rl1.
  Which one do you wish to initialize? (or done) [rl1] done


Danach muss man die "**default route**" angeben, wobei bei der
Installation von CD einfach die Vorgabe "**none**" übernommen werden
kann. Will man per FTP installieren, dann muss dort die Adresse des
Gateways/Routers eingetragen werden. Als Nameserver geben wir gleich mal
den Standard T-Online Nameserver an.

::

  DNS domain name? (e.g. 'bar.com') [my.domain] Enter
  DNS nameserver? (IP adress or 'none') [none] 194.25.2.132
  Use the nameserver now? [yes] Enter
  Default route? (IP adress or 'none') none
  Edit hosts with ed? [no] Enter
  Do you want to do any manual network configuration? [no] Enter

Als nächstes muss man sich ein Passwort für den Administratoraccount des
Routers ausdenken (bei \*NIX Systemen hat dieser immer den Namen root).
Dabei gilt es möglichst zu beachten, dass das Passwort aus Gross- und
Kleinbuchstaben, sowie aus Ziffern bestehen sollte. Ein gutes Passwort
wäre z.B. "**eWr43sR6**", schlecht hingegen ist "**gott**" oder
"**0815**". Merken sollte man sich das Passwort natürlich schon noch
können ;).

::

  Password for root account? (will not echo) <root_passwort>
  Password for root account? (again): <root_passwort>
  .....

  Sets can be located on a (m)ounted filesystem; a (c)drom, (d)isk or (t)ape
  device; or a (f)tp, (n)fs or (h)ttp server.
  Where are the install sets? (or 'done')

Hier ist es klar, dass man abhängig von der Installationsart entweder
"**f**" für FTP oder "**c**" für CD wählt. Entscheidet man sich für FTP,
so kann man sich aus einer Liste von diversen OpenBSD FTP Mirrors einen
passenden aussuchen. Wählt man Installation von CD, dann wird man noch
nach dem CD-ROM Device gefragt, in welchen sich die CD befindet. Sollten
es mehrere sein, dann führt auch hier einmal mehr einfaches Probieren
zum Ziel. Hat man das richtige Installationsmedium gefunden, dann gehts
so weiter:

::

  The following sets are available. Enter a filename, 'all' to select
  all the sets, or 'done'. You may de-select a set by prepennding a '-'
  to its name.
  
          [X] bsd
          [ ] bsd.rd
          [X] base41.tgz
          [X] etc41.tgz
          [X] misc41.tgz
          [X] comp41.tgz
          [X] man41.tgz
          [X] game41.tgz
          [ ] xbase41.tgz
          [ ] xshare41.tgz
          [ ] xfont41.tgz
          [ ] xserv41.tgz
  
  File name? (or 'done') [bsd.rd] -misc41.tgz    # Wildcard * ist ebenfalls erlaubt z.B. -mi*
  
  .....
  
  File name? (or 'done') [bsd.rd] -comp41.tgz
  
  .....
  
  File name? (or 'done') [bsd.rd] -game41.tgz
  
  .....
  
  The following sets are available. Enter a filename, 'all' to select
  all the sets, or 'done'. You may de-select a set by prepennding a '-'
  to its name.
  
          [X] bsd
          [ ] bsd.rd
          [X] base41.tgz
          [X] etc41.tgz
          [ ] misc41.tgz
          [ ] comp41.tgz
          [X] man41.tgz
          [ ] game41.tgz
          [ ] xbase41.tgz
          [ ] xshare41.tgz
          [ ] xfont41.tgz
          [ ] xserv41.tgz
  File name? (or 'done') [bsd.rd] done
  Ready to install sets? [yes] Enter
  
  .....
  
  Location of sets? (cd disk ftp http or 'done') [done] Enter
  Do you wish sshd(8) to be started by default? [yes] Enter
  Start ntpd(8) by default? [no] y
  Do you expect to run the X Window System? [no] Enter
  Change the default console to com0? [no] Enter
  Saving configuration files......done.
  Generating initial host.random file ......done.
  What timezone are you in? ('?' for list) [Canada/Mountain] UTC


Als Zeitzone wählen wir deswegen UTC, da wir so die Einträge in den
Logdateien in eben dieser Zeit erhalten. Das kann in einem globalen
Netzwerk ganz hilfreich sein. Wahlweise kann auch "**Europe/Berlin**"
verwendet werden. Nun dauerts nochmal etwas und dann sollte die
Installation beendet sein. Also nix wie CD oder Diskette raus und
"**reboot**" eingetippt. Wenn alles passt sollte der Rechner bis zum
"**login:**" Prompt booten und der erste Schritt zum "**OpenBSD
DSL-Router**" ist getan. Gratulation! Jetzt ist es Zeit für eine kleine
Entspannungspause bis es dann mit der Konfiguration weitergeht.

Konfiguration "afterboot"
-------------------------

Nach dem ersten Neustart des fertig installierten Systems loggen wir uns
als "**root**" ein. Um sich das
`mounten <http://www.openbsd.org/cgi-bin/man.cgi?query=mount&apropos=0&sektion=8&manpath=OpenBSD+Current&arch=i386&format=html>`__
von Diskettenlaufwerk und eventuell vorhandenem CD-Rom zu erleichtern,
erstellt man erstmal zwei passende Verzeichnisse.

::

  # mkdir -p /media/cdrom 
  # mkdir -p /media/floppy

Danach muss die Datei "**/etc/fstab**" nur noch um diese zwei Zeilen
(bzw. eine Zeile) erweitert werden,
`/etc/fstab <http://www.openbsd.org/cgi-bin/man.cgi?query=fstab&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__

::

   /dev/cd0a /media/cdrom cd9660 ro,nodev,nosuid,noauto 0 0
   /dev/fd0a /media/floppy msdos rw,nodev,nosuid,noauto 0 0

und Disketten bzw. CD's können mit "**mount /media/floppy**" oder
"**mount /media/cdrom**" ins Dateisystem eingehängt werden.

Wer das Ganze mit einem etwas einfacher zu bedienendem Editor versuchen
will, dem möchte ich
`nano <ftp://ftp.openbsd.org/pub/OpenBSD/4.1/packages/i386/nano-2.0.3.tgz>`__
ans Herz legen. Dieser kann auch mit
"**pkg_add**\ ftp://ftp.openbsd.org/pub/OpenBSD/4.1/packages/i386/nano-2.0.3.tgz"
installiert werden. Wenn der Rechner momentan noch keinen Zugang zum
Internet hat, dann braucht man zusätzlich noch die Pakete
`libiconv <ftp://ftp.openbsd.org/pub/OpenBSD/4.1/packages/i386/libiconv-1.9.2p3.tgz>`__
und
`gettext <ftp://ftp.openbsd.org/pub/OpenBSD/4.1/packages/i386/gettext-0.14.6.tgz>`__
im selben Verzeichniss um "**pkg_add nano-2.0.3.tgz**" erfolgreich
ausführen zu können.

Da wir später die ksh-Shell als Standard einstellen werden, ist eine
kleine Änderung in "**/etc/skel/.profile**" sinnvoll. Mit 

::

  # echo ". /etc/ksh.kshrc" >> /etc/skel/.profile

ist dafür gesorgt, dass beim Login die Datei "**/etc/ksh.kshrc**"
eingelesen wird und somit der Prompt hübscher aussieht. Die Dateien aus
"**/etc/skel/**" werden beim Erstellen eines neuen Benutzers in dessen
Homeverzeichniss kopiert. Somit ist diese Änderung für alle Benutzer,
die ab diesem Zeitpunkt dem System hinzugefügt werden, wirksam. Sollen
diese Änderungen auch für den Super-User greifen, dann müssen die
entsprechenden Dateien manuell nach "**/root/**" kopiert werden.

Standardmäsig sind bei OpenBSD 4.1 einige Dienste gestartet, die wir
momentan nicht benötigen. Ergo verhindern wir, dass sie beim nächsten
Neustart wieder geladen werden, indem wir die Datei
"**/etc/rc.conf.local**" erstellen. 

::

  # echo "inetd=NO" >> /etc/rc.conf.local

Jetzt wird es langsam Zeit, dass man sich einen neuen Benutzer anlegt.
Schon allein um die lästige Warnmeldung loszuwerden, die bei jedem Login
als "**root**" erscheint. Als Beispiel lege ich mit Hilfe von
"**adduser**" den Benutzer "**sepp**" an.

::

  # adduser
  Couldn't find /etc/adduser.conf: creating a new adduser configuration file
  Reading /etc/shells
  Enter your default shell: csh ksh nologin sh [[sh]]: ksh
  Your default shell is: ksh -> /bin/ksh
  Reading /etc/login.conf
  Default login class: auth-defaults auth-ftp-defaults daemon default staff
  [default]: Enter
  Enter your default HOME partition: [/home]: Enter
  Copy dotfiles from: /etc/skel no [/etc/skel]: Enter
  Send message from file: /etc/adduser.message no [no]: Enter
  Do not send message
  Prompt for passwords by default (y/n) [[y]]: Enter
  Default encryption method for passwords: auto blowfish des md5 old
  [auto]: Enter
  
  .....
  
  Enter username [a-z0-9_-]: sepp
  Enter full name []: Seppl
  Enter shell csh ksh nologin sh [ksh]: Enter
  Uid [1000]: Enter
  Login group sepp [sepp]: Enter
  Login group is ``sepp. Invite sepp into other groups: guest no
  [no]: wheel
  Default login class: auth-defaults auth-ftp-defaults daemon default staff
  [default]: Enter
  Enter password []: <sepps_passwort>
  Enter password again []: <sepps_passwort>
  
  …..
  
  OK? (y/n) [y]: Enter
  Added user ``sepp
  Copy files from /etc/skel to /home/sepp
  Add another user? (y/n) [y]: n

Als "**login class**" ist für den normalen Standardbenutzer
"**default**" ganz gut geeignet, was ja der Name auch schon vermuten
lässt. Interessant wird es später nochmal, wenn es darum geht einen
Benutzer ausschliesslich für den FTP Server zu erstellen. Generelle
Infos zu den Login-Klassen erhält man mit `man
login.conf <http://www.openbsd.org/cgi-bin/man.cgi?query=login.conf&sektion=5&arch=i386&apropos=0&manpath=OpenBSD+Current>`__.
Wie vielleicht schon bemerkt, wird "**sepp**" auch in die Gruppe
"**wheel**" eingetragen, damit er das Recht erhält per
`sudo <http://www.openbsd.org/cgi-bin/man.cgi?query=sudo&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
Befehle mit root Rechten auszuführen. Soll einem Benutzer dies nicht
erlaubt sein, dann reicht bei der entsprechenden Frage ein "**no**" als
Antwort. Um Benutzern der Gruppe wheel die uneingeschänkte Nutzung von
sudo zu ermöglichen ist nur eine kleine Änderung in der Datei
"**/etc/sudoers**" nötig, welche man aber nur mit Hilfe des speziellen
Befehls

::

   # visudo

editieren sollte. Dort kommentiert man dann nur folgende, mit fetter
Schrift markierte, Zeile aus.
``[[http://www.openbsd.org/cgi-bin/man.cgi?query=sudoers&sektion=5&arch=i386&apropos=0&manpath=OpenBSD+Current|/etc/sudoers]]``

::

  ...
  # User privilege specification
  root    ALL=(ALL) ALL

  # Uncomment to allow people in group wheel to run all commands
  **%wheel   ALL=(ALL)   ALL**

  # Same thing without a password
  # %wheel    ALL=(ALL)   NOPASSWD: ALL
  ...

Da man nun einen Benutzer angelegt hat, der sich mit "**su**" und
"**sudo**" Rootrechte verschaffen kann, macht es Sinn das Einloggen von
Root über den
`sshd <http://www.openbsd.org/cgi-bin/man.cgi?query=sshd&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
zu unterbinden. Dafür ist in der Datei
"`/etc/ssh/sshd_config <http://www.openbsd.org/cgi-bin/man.cgi?query=sshd_config&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__"
der Eintrag "*PermitRootLogin*" auf "*no*" zu setzen. Nach einem
Neustarten des sshd kann man sich nun nicht mehr per ssh als Root
einloggen. Das macht Sinn, da man somit viele Bruteforce Angriffe aus
dem Internet gleich im Keim erstickt.

Hat man die Installation per FTP durchgeführt, dann musste
wahrscheinlich ein Gateway bzw. eine "**default route**" angegeben
werden. Diese Einstellung wird aber jetzt nicht mehr gebraucht und muss
sogar entfernt werden, da dieser Rechner selbst als Gateway arbeiten
soll. In diesem Fall existiert eine Datei namens "**/etc/mygate**" in
der die IP Adresse des Gateways eingetragen ist, welche man bedenkenlos
löschen kann. Damit nicht extra neu gestartet werden muss, löschen wir
die "**default route**" auch noch manuell.

::

  # rm /etc/mygate 
  # route delete default

Will man nachträglich die IP Adresse des Computers ändern, dann tut man
das am Besten indem man die zum Device (Netzwerkkarte) passende Datei
"**/etc/hostname.<device>**" (z.B. /etc/hostname.rl0) und
"**/etc/hosts**" entsprechend abändert. Siehe hierzu `man
hosts <http://www.openbsd.org/cgi-bin/man.cgi?query=hosts&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
und `man
hostname.if <http://www.openbsd.org/cgi-bin/man.cgi?query=hostname.if&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__.
Ein Neustart ist nicht notwendig, es reicht wenn danach "**sh
/etc/netstart**" ausgeführt wird.

Alle weiteren Informationen über Dinge, die man noch so nach dem ersten
Neustart einstellen kann/soll, erfährt der ambitionierte Leser per `man
afterboot <http://www.openbsd.org/cgi-bin/man.cgi?query=afterboot&sektion=&apropos=0&manpath=OpenBSD+Current&title=New%20System%20afterboot>`__.
Jetzt ist es eigentlich schon soweit, dass man den Computer ohne Monitor
und Tastatur in die Ecke stellen könnte. Nur die Leute, die gerne an
Firewallregeln herumspielen, seien gewarnt: Schnell ist mal das Falsche
geblockt und man ist wieder mit Monitor und Tastatur unterm Arm
unterwegs zur Abstellkammer...

DSL - mit pppoe ab ins Netz
---------------------------

**Ganz wichtig: Bitte denkt daran mindestens ein "Return" also eine neue
Zeile/Newline in dem Editor eurer Wahl am ENDE jeder Datei mit
abzuspeichern. Sonst kann es arge Probleme geben.**

Alle Programme und Treiber die zum Aufbauen einer DSL-Verbindung
notwendig sind, sind schon in OpenBSD integriert. Deshalb gestaltet sich
das Einrichten relativ problemlos. Die zweite, noch unkonfigurierte,
Netzwerkkarte sollte nun an das DSL-Modem angeschlossen werden. Danach
müssen 3 Dateien in /etc/ppp/ erstellt werden: ppp.conf, ppp.linkup und
ppp.linkdown. Die restlichen Dateien, die sich in diesem Verzeichniss
befinden können unverändert bleiben. Beginnen wir mit ppp.conf:
``[[http://www.scatterbrained.de/openbsd/3.5/files/ppp.conf|/etc/ppp/ppp.conf]]``

::

       default:
        set log Phase Chat IPCP CCP tun command
        set redial 15 0
        set reconnect 15 10000
        
       pppoe:
        set device "!/usr/sbin/pppoe -i **<interface>**"
        disable acfcomp protocomp
        deny acfcomp
        set mtu max 1454
        set crtscts off
        set speed sync
        enable lqr
        set lqrperiod 5
        set dial
        set login
        set timeout 0
        set authname "**<benutzername>**"
        set authkey **<passwort>**
        add! default HISADDR
        enable dns
        enable mssfixup

Bei dieser und den zwei folgenden Dateien ist so ziemlich jedes Zeichen
von Bedeutung. Schon das Hinzufügen oder Weglassen eines Leerzeichens
kann dazu führen, dass die Einwahl unter einem Wulst von Fehlermeldungen
abbricht. Die Schlüsselwörter "**default:**" und "**pppoe:**" müssen
z.B. unbedingt am Anfang einer Zeile stehen. Deswegen ist es in diesem
Fall ratsam, die Beispieldateien direkt auf den Router zu kopieren und
nur die notwendigen Veränderungen selbst daran vorzunehmen: Als
**<interface>** ist der Devicename der Netzwerkkarte einzutragen, an der
das DSL-Modem angeschlossen ist. In diesem Beispiel wäre es das Device
"**rl1**" (die ganze Zeile würde so aussehen: **set device
"!/usr/sbin/pppoe -i rl1"**). Bei **<benutzername>** ist der T-Online
typische Benutzername einzutragen, der sich aus Anschlusskennung und
Mitbenutzernummer zusammensetzt (sieht in etwa so aus:
**012345678901234567890123#0001@t-online.de** - bei einer T-Online
Nummer die länger als 11 Zeichen ist taucht die Raute nicht mehr im
Benutzernamen auf). Dabei ist vor allem wichtig, dass dieser Name in
Anführungzeichen steht, da es sonst zu Problemen wegen des Zeichens '#'
im Benutzernamen kommt. Das T-Online Passwort ist (meines Wissens immer)
eine 8-stellige Zahl (etwa: 01234567) und muss anstelle von
**<passwort>** eingetragen werden.

Seit neuestem habe ich die MTU von 1492 auf 1454 herabgesetzt, da mir
der Tipp von Gernot Pörner mit dem Verweis auf die Seite
http://www.mynetwatchman.com/kb/adsl/pppoemtu.htm als durchaus sinnvoll
erscheint und hoffentlich die gelegentlichen Verbindungsprobleme noch
weiter reduziert.
``[[http://www.scatterbrained.de/openbsd/3.5/files/ppp.linkup.1|/etc/ppp/ppp.linkup]]``

::

       MYADDR:
        ! sh -c "/sbin/ifconfig pflog0 up"
        ! sh -c "/sbin/pflogd"
        ! sh -c "/sbin/pfctl -e -F all -f /etc/pf.conf"

Die Befehle in der Datei ppp.linkup werden nach dem erfolgreichen Aufbau
der PPP/DSL-Verbindung ausgeführt. In dieser Beispieldatei wird zuerst
das "**Loginterface**" initialisiert, der "**Logdaemon**" gestartet und
zuletzt die Firewall eingeschaltet. Da wir bis jetzt aber noch keine
Firewall Regeln definiert haben, ist das natürlich etwas sinnlos. Bei
dem Format dieser Datei (und auch der Nachfolgenden) ist unbedingt zu
beachten, dass vor jedem '!' ein Leerzeichen zu stehen hat, sonst wird
diese Zeile nicht ausgeführt.
``[[http://www.scatterbrained.de/openbsd/3.5/files/ppp.linkdown|/etc/ppp/ppp.linkdown]]``

::

       MYADDR:
        ! sh -c "/sbin/pfctl -d -F all"
        ! sh -c "kill `cat /var/run/pflogd.pid`"
        ! sh -c "/sbin/ifconfig pflog0 down"
        ! sh -c "/sbin/route delete default"

Analog zu ppp.linkup gibt es die Datei ppp.linkdown, deren Befehle
ausgeführt werden nachdem die Verbindung getrennt wurde. Zuerst wird die
Firewall ausgeschaltet, dann der "**Logdaemon**" gestoppt, das
"**Loginterface**" deaktiviert und zuletzt die "**Defaultroute**"
gelöscht. Damit ist wieder der Zustand vor der Einwahl exakt
hergestellt.

Zum Schluss sollten noch die Rechte der Konfigurationsdateien angepasst
werden.

::

   # chmod 600 /etc/ppp/ppp.conf /etc/ppp/ppp.linkup /etc/ppp/ppp.linkdown

Die Verbindung sollte man nun mit folgenden Befehlen aufbauen können:

::

   # ifconfig <interface> up media 10baseT
   # ppp -ddial pppoe

Da die meisten Netzwerkarten heutzutage die Auto-Negotiation beherschen
und die DSL Anschlüsse gerne auch mal mehr als 10Mbit liefern reicht ein

::

   # ifconfig <interface> up
   # ppp -ddial pppoe

Als **<interface>** ist auch hier wieder das Device anzugeben, an dem
das DSL-Modem angeschlossen ist. Mit dem Parameter "**-ddial**" wird ppp
angewiesen, sich bei Trennung automatisch wieder Einzuwählen. Eine
einmalige Einwahl ist durch den Parameter "**-background**" möglich.
Damit die Verbindung schon automatisch nach dem Einschalten des
Computers aufgebaut wird, erstellen wir "**/etc/hostname.<interface>**"

::

   # echo "up media 10baseT" > /etc/hostname.<interface>

oder

::

   # echo "up" > /etc/hostname.<interface>

und fügen folgendes am Ende von "**/etc/netstart**" hinzu:
``[[http://www.scatterbrained.de/openbsd/3.5/files/netstart|/etc/netstart]]``

::

       /usr/bin/touch /etc/ppp/firsttime
       /usr/sbin/ppp -ddial pppoe

Vom Router aus sollte man nun problemlos ins Internet kommen (z.B:
"**ping www.openbsd.org**"). Was nun genau passiert sieht man in der
**/var/log/daemon** Logdatei. Näheres über ppp und auch Hilfe zur
Beseitigung von eventuell auftretenden Fehlern, findet man wie gewohnt
auf der entprechenden Manpage: `man 8
ppp <http://www.openbsd.org/cgi-bin/man.cgi?query=ppp&apropos=0&sektion=8&manpath=OpenBSD+Current&arch=i386&format=html>`__

Als Alternative bietet sich auch das erstellen eines Interfaces fuer
pppoe an. Wenn dies gemacht wird sind die obengenannten
Konfigurationsschritte nicht auszuführen, da es zu nicht vorhersehbaren
Zuständen führen könnte.

Es muss die Datei /etc/hostname.pppoe erstellt werden. Diese hat
ebenfalls alle benötigten Angaben für den Aufbau der DSL Verbindung
vorhanden.
http://www.openbsd.org/cgi-bin/man.cgi?query=pppoe&apropos=0&sektion=4&manpath=OpenBSD+4.0&arch=i386&format=html

::

  inet 0.0.0.0 255.255.255.255 0.0.0.1 pppoedev **<interface>** \
    authproto pap authname **<benutzername>** authkey **<passwort>** up
  !/sbin/route add default 0.0.0.1
  ! sh -c "/sbin/pfctl -e -F all -f /etc/pf.conf"

Anschliessend muss dem Interface an dem das DSL Modem angeschlossen ist
noch der Zustand "up" mitgeteilt werden

::

   # echo "up media 10baseT" > /etc/hostname.<interface>

Wenn das Interface Auto-Negotiation beherscht reicht ein

::

   # echo "up" > /etc/hostname.<interface>

Das Interface entscheidet mit dem angeschlossenen Modem zusammen welche
Geschwindigkeit zwischen beiden gefahren wird.

[STRIKEOUT:Es funktionieren beide Methoden]. Die Alternative mit dem
pppoe0 Interface wurde aber erst in der Version 3.9 implementiert,
weswegen es im der Original How-To nicht vorhanden war.

Vorsicht! Die „neuere“ Methode (genannt PPPOE(4) im Gegensatz dazu heißt
die „alte“ Version PPPOE(8)), hat gravierende Nachteile die bis zur
heutigen Version (OpenBSD 4.4) nicht behoben sind. Die Config-Datei
heißt **hostname.pppoe**, auch **hostname.pppoe0** genannt.

Es gibt erhebliche Probleme mit der sogenannten MTU Size. Diese lässt
sich meines Wissens nach nicht in der **hostname.pppoe0** einstellen.
Der Workaround in der pf.conf habe ich nicht zum laufen bekommen.

::

   # scrub out on pppoe0 max-mss 1440

Am gemeinsten sind die völlig willkürlichen “lustigen” Fehler die man
sich einfängt. Falls euer Router also ins Internet kommt die Clients aus
dem lokalem Netzwerk auch nach draußen pingen können aber ihr irgendwie
außer Google keine einzige Webseite öffnen könnt seid ihr gradewegs in
diese MTU Falle gelaufen. Natürlich läuft dann das Skript für den
Timeserver (siehe 8) nicht. Weitere Fehlermeldungen:

::

   # pppoe0: pap failure

und

::

   # pfctl: SIOCGIFMTU: Device not configured.

Nähere Erläuterungen in der Manpage zu PPPOE(4) und in den BSD
Foren/OpenBSD/Netzwerk. Des weiteren hat der Workaround laut manpage
noch den Nachteil dass die OS Erkennung des pf nicht richtig
funktioniert.

Falls ihr dennoch PPPOE(4) ausprobieren wollt, bitte in der **pf.conf**

::

   # Ext = "tun0" 

zu

::

   # Ext = „pppoe=“ ändern.

Nach dem Erstellen der **hostname.pppoe0** müsst ihr noch ein

::

   # ifconfig create pppoe0

machen.

Ein

::

   # ifconfig pppoe0 up

sollte dann die Verbindung nach draußen öffnen.

Sämtliche anderen Rechner im lokalen Netzwerk haben momentan noch keine
Chance ins Internet zu kommen. Damit einem die Konfiguration dieser
sogenannten "**Clients**" leichter fällt, sollte man als nächstes einen
Nameserver auf dem Router einrichten.

Caching Nameserver
------------------

Der Nameserver, den wir einrichten wollen, wird kein vollwertiger
Nameserver werden, sondern DNS-Anfragen einfach nur an die "**echten**"
T-Online-Nameserver weiterleiten. Deshalb wird er auch als "**Caching
Nameserver**" bezeichnet. Bei der Konfiguration der Clients im lokalen
Netzwerk muss so nur die IP des Routers als Nameserver angegeben werden.

Genau für diesem Zweck existert ein kleiner und leicht konfigurierbarer
Nameserver. Man findet ihn unter
`http:\ thekelleys.org.uk/dnsmasq/doc.html]]. Damit man sich nicht mit
dem compilieren der Sourcen rumschlagen muss, habe ich extra ein
[[http:\ www.scatterbrained.de/openbsd/3.5/files/dnsmasq-2.11.tgz|Paket <http://thekelleys.org.uk/dnsmasq/doc.html>`__
erstellt. So ist auch eine saubere Deinstallation jederzeit möglich. In
der Version 4.4 ist dnsmasq in den Packages in der Version 2.45
enthalten, der sich genau so konfigurieren läßt. `Version 2.45 für
OpenBSD
4.4 <ftp://ftp.openbsd.org/pub/OpenBSD/4.4/packages/i386/dnsmasq-2.45.tgz>`__

Ist das Paket lokal auf dem Rechner, dann kann es mit dem Befehl

::

   # pkg_add dnsmasq-2.11.tgz

installiert werden. Es ist auch möglich die HTTP Adresse des Pakets als
Parameter anzugeben. Der Nameserver kann einfach per

::

   # /usr/local/sbin/dnsmasq -i <interface> -z

aufgerufen werden, wobei für "**<interface>**" der Name des internen
Netzwerkinterfaces eingesetzt werden muss (hier rl0). Damit er auch nach
dem nächsten Reboot gestartet wird, fügen wir den Aufruf in
"**/etc/rc.local**" ein:

::

   # echo "/usr/local/sbin/dnsmasq -i <interface> -z" >> /etc/rc.local

Fertig ist der "**Caching Nameserver**", der ürigens auch die Einträge
in "**/etc/hosts**" mit berücksichtigt.

Automatische Konfiguration per DHCP
-----------------------------------

Mit Hilfe des Dynamic Host Configuration Protocol (näheres siehe
`RFC1497 <http://www.ietf.org/rfc/rfc1497.txt?number=1497>`__) ist es
ganz einfach möglich, dass sich die Computer im internen Netzwerk
automatisch konfigurieren. In unserem Fall bekommen sie vom DHCP Server
eine IP Adresse, den Nameserver und die "**default route**" zugewiesen.
Dazu müssen zuerst die Einstellungen in "**/etc/dhcpd.conf**" angepasst
werden.

``[[http://www.scatterbrained.de/openbsd/3.5/files/dhcpd.conf|/etc/dhcpd.conf]]``

::

       shared-network LOCAL-NET {
           option  domain-name "my.domain";
           option  domain-name-servers 192.168.1.1;
    
           subnet 192.168.1.0 netmask 255.255.255.0 {
               option routers 192.168.1.1;
    
               range 192.168.1.127 192.168.1.254;
    
               default-lease-time 43200;
               max-lease-time 43200;
           }
       }

Wie vielleicht schon aufgefallen gebe ich als "**range**" die IP
Adressen zwischen 192.168.1.127 und 192.168.1.254 an. Was für alle
Clients, die sich per DHCP konfigurieren, bedeutet, dass sie eine IP
Adresse in diesem Bereich erhalten werden. Somit hat man den Bereich
192.168.1.1-126 frei um diese Adressen für Rechner zu verwenden, die
kein DHCP benutzen sollen. Es ist im übrigen auch möglich einem
bestimmten Computer anhand der eindeutigen Ethernetadresse seiner
Netzwerkkarte eine vorher ausgewählte Adresse zu geben (siehe "`man
dhcpd.conf <http://www.openbsd.org/cgi-bin/man.cgi?query=dhcpd.conf&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__"
- eine kleine Beispieldatei gibts von mir ;)
`dhcpd.conf.fixed <http://www.scatterbrained.de/openbsd/3.5/files/dhcpd.conf.fixed>`__).
Die "**default-lease-time**" und die "**max-lease-time**" geben an wie
lange (in Sekunden) der DHCP Server eine IP, im Falle einer
unterbrochenen Verbindung zum Client, reserviert. Bricht also die
Verbindung ab, so wird in unserem Fall 12h gewartet bis die vergebene IP
endgültig verworfen wird. Mehr zu diesem Thema findet man in der `DHCP
FAQ <http://www.dhcp-handbook.com/dhcp_faq.html#hlsal>`__. Zum Schluss
ist nun dem DHCP Server nur noch zu sagen auf welcher der Netzwerkkarten
des Routers er seinen Dienst verrichten soll. Das geht ganz einfach mit

::

   # echo "<internes_device>" >> /etc/dhcpd.interfaces

Der Platzhalter **<internes_device>** muss durch den Devicenamen der
Netzwerkkarte, an dem das interne Netz angeschlossen ist, ersetzt
werden. In diesem Beispiel wäre das "**rl0**". Der DHCP Server ist nun
fertig konfiguriert und kann per

::

   # touch /var/db/dhcpd.leases
   # /usr/sbin/dhcpd <internes_device>

gestartet werden. Damit der DHCP Server nach einem Neustart automatisch
loslegt, ist dies noch in "**/etc/rc.conf.local**" einzustellen.
``[[http://www.scatterbrained.de/openbsd/3.5/files/rc.conf.local.2|/etc/rc.conf.local]]``

::

       inetd=NO
       dhcpd_flags="<internes_device>"

Dynamic DNS - immer erreichbar
------------------------------

Zuerst muss man sich bei `www.no-ip.com <http://www.no-ip.com/>`__ einen
Namen, wie z.B. "**mein-router.no-ip.com**", kostenlos registrieren.
Wenn das erfolgreich geschehen ist, dann sollte man über einen
Loginnamen, ein Passwort und natürlich den "**hostname**" selbst
verfügen. Um dem Dynamic DNS Service bescheid zu geben auf welche IP der
registrierte Name zeigen soll, findet sich das Programm
`no-ip-2.1.3.tgz <ftp://ftp.openbsd.org/pub/OpenBSD/4.1/packages/i386/no-ip-2.1.3.tgz>`__
bei den vorcompilierten Paketen.

::

   # pkg_add ftp://ftp.openbsd.org/pub/OpenBSD/4.1/packages/i386/no-ip-2.1.3.tgz

Eine passende Konfigurationsdatei erzeugt man am besten mit dem Programm
selbst.

::

   # /usr/local/sbin/noip2 -C

::

    Auto configuration for Linux client of no-ip.com.
    
    Multiple network devices have been detected.
    
    Please select the Internet interface from this list.
    
    By typing the number associated with it.
    0       lo0
    1       rl0
    2       rl1
    3       tun0
    3
    Please enter the login/email string for no-ip.com  **<loginname>**
    Please enter the password for user '<loginname>'  **<passwort>**
    
    Only one host [[<hostname>]] is registered to this account.
    It will be used.
    Do you wish to run something at successful update?[[N]] (y/N)  **Enter**
    
    New configuration file '/etc/no-ip2.conf' created.

Als **<loginname>** wird die bei der Registrierung verwendete
Emailadresse eingesetzt, **<passwort>** ist das entsprechende Passwort,
**<hostname>** ist nur der Name bei no-ip.com registrierte Name des
Hosts. Wenn Du mehrere Namen registriert haben solltest, dann wird an
dieser Stelle wahrscheindlich eine Auswahl möglich sein.

Um zu sehen ob alles glatt läuft starten wir noip manuell.

::

   # /usr/local/sbin/noip2

Danach sollte man etwas (ca. 2-5 min) warten und man kann per "**ping
<hostname>**" testen ob noip erfolgreich war. Den aktuellen Status aller
mometan laufenden noip2 Instanzen kann man sich praktischerweise mit
"**/usr/local/sbin/noip2 -S**" anzeigen lassen. Wenn noip2 aus
irgendeinem Grund nicht funktionieren sollte, findet man die
Fehlermeldungen des Programms in der Logdatei "**/var/log/daemon**". Hat
es geklappt, dann fügen wir es nur noch "**/etc/rc.local**" hinzu, damit
der dynamic DNS-Name Daemon schon beim Systemstart aktiviert ist.

::

   # echo "/usr/local/sbin/noip2" >> /etc/rc.local

Die nächsten 3 Punkte (Timeserver, FTP Server und Identd) sind optional
und können übersprungen werden. Ich finde diese Dienste allerdings
relativ praktisch und würde sie empfehlen.

Timeserver
----------

Damit alle Computer im lokalen Netzwerk immer die exakte Zeit anzeigen
ist ein Timeserver geradezu ideal. Diesen Daemon (ntpd), der die lokale
Zeit ständig mit einem Timeserver im Internet abgleicht, können die
lokalen Computer per "**ntpdate**" nach der aktuellen Uhrzeit befragen.
Der NTPd befindet sich seit OpenBSD 3.6 wieder im Basis System. Diesem
kann durch ändern der Konfigurationsdatei "**/etc/ntp.conf**" gesagt
werden, mit welchem Rechner im Internet er die lokale Uhrzeit abgleichen
soll.

``[[http://www.scatterbrained.de/openbsd/3.5/files/ntp.conf|/etc/ntp.conf]]``

::

   server de.pool.ntp.org
   driftfile /etc/ntp.drift

Damit vor dem Start des nptd die Systemuhr richtig tickt, sorgen wir
dafür, dass ntpdate zuvor gestartet wird. Das erreicht man am
einfachsten mit einem Eintrag in "**/etc/rc.conf.local**".

``[[http://www.scatterbrained.de/openbsd/3.5/files/rc.conf.local.3|/etc/rc.conf.local]]``

::

   inetd=NO
   ntpdate_flags="de.pool.ntp.org"
   dhcpd_flags="<internes_device>"

Weil es zu Problemen mit dem ntp-Daemon kommt wenn die DSL Verbindung
abbricht, bastelt man sich ein Skript "/etc/ppp/reset_ntp" das ihn in
diesem Falle neu startet.

``[[http://www.scatterbrained.de/openbsd/3.5/files/reset_ntp|/etc/ppp/reset_ntp]]``

::

   #!/bin/sh
   if [[|-f /etc/ppp/firsttime ]]; then
     rm -f /etc/ppp/firsttime
     exit 0
   fi
    
   if [[|-f /var/run/ntpd.pid ]]; then
      kill `cat /var/run/ntpd.pid`
      rm -f /var/run/ntpd.pid
   fi
      /usr/local/sbin/ntpd -p /var/run/ntpd.pid

Mit "**chmod 500 /etc/ppp/reset_ntp**" machen wir es noch für root
ausführbar und lesbar. Gestartet wird dieses Skript dann durch einen
Eintrag in "**/etc/ppp/ppp.linkup**".

``[[http://www.scatterbrained.de/openbsd/3.5/files/ppp.linkup.3|/etc/ppp/ppp.linkup]]``

::

   MYADDR:
     ! sh -c "/sbin/ifconfig pflog0 up"
     ! sh -c "/sbin/pflogd"
     ! sh -c "/sbin/pfctl -e -F all -f /etc/pf.conf"
     ! sh -c "/etc/ppp/reset_ntp"

Die Uhr des Routers sollte jetzt immer einwandfrei eingestellt sein.
Damit hat man die Konfiguration des Servers abgeschlossen.

Bleibt nur die Frage: "*Wie bringt man die anderen Rechner im Netzwerk
dazu sich die aktuelle Zeit vom Router abzuholen?*". Unter Linux/*BSD
gibt es zu diesem Zweck das Programm "**ntpdate**". Ein einfacher
Eintrag in der "**crontab**" bringt den jeweiligen Computer dann dazu
sich z.B. alle 30min die neue Uhrzeit vom Router zu holen. Als root kann
man per

::

   # crontab -e

die Cronjob-Tabelle editieren und um die Zeile

::

   */30    *       *       *       *       /usr/sbin/ntpdate -u -b -s **<ip_des_routers>**

erweitern. Anstatt von **<ip_des_routers>** sollte logischerweise die
lokale IP Adresse des Routers (in diesem Beispiel wäre das die
192.168.1.1) eingetragen werden. Leider ist es nicht ganz sicher, dass
sich "**ntpdate**" immer in "**/usr/sbin/**" befindet. Am Besten
überprüft man das zuerst per "**type ntpdate**" oder "**find / -name
"ntpdate"**" und ändert den Eintrag entsprechend ab. Auch für Win\*
Rechner gibt es entsprechende NTP-Client Software, die man zur
Synchronisation mit dem Timeserver verwenden kann. Als Beispiel seien
hier `Automachron <http://www.oneguycoding.com/automachron/>`__ und
`AboutTime <http://www.arachnoid.com/abouttime/>`__ genannt. Unter
Windows XP und 2000 ist ein solcher Client ürigens schon integriert. Bei
XP findet man irgendwo bei den Einstellungen der Uhr einen Hinweis auf
die Zeitsynchronisation. Der standardmäsig dort eingestellte Timeserver
(irgendein Microsoft eigener Server) kann bedenkenlos durch die interne
IP unseres Routers ersetzt werden. Unter Windows 2000 kann man den
Timeserver per Kommandozeile setzen:

::

   C:\> net time /setsntp:<ip_des_routers>

Diese Änderung wird erst nach einem Neustart des "**Windows Zeitgeber**"
Dienstes oder des ganzen Systems wirksam.

FTP-Server
----------

Das "**File Tranfer Protocol**" hat leider den Nachteil, dass es alle
Daten (auch Loginname und Passwort) unverschlüsselt verschickt. Deswegen
sollte man für den FTP Server einen extra Benutzer anlegen, der sich per
ssh nicht einloggen darf/kann. Den anderen Benutzern sollte der Zugriff
auf den FTP Server auch explizit untersagt werden, indem man sie in die
Datei "**/etc/ftpusers**" einträgt. In diesem Beispiel haben wir bis
jetzt nur einen neuen Benutzer, mit dem Namen "**sepp**", angelegt. Also
fügen wir seinen Loginnamen hinzu.

::

   # echo "sepp" >> /etc/ftpusers

Danach erstellen wir mit "adduser" einen Benutzer, der sich nicht
einloggen darf.

::

    # adduser -silent
    Enter Username [[a-z0-9_-]]: **ftpguy**
    Enter full name [[]]: **FTP Guy**
    Enter shell csh ksh nologin sh [[ksh]]: **nologin**
    Uid [[1001]]: **Enter**
    Login group ftpguy [[ftpguy]]: **Enter**
    Login group is ``ftpguy//. Invite ftpguy into other groups: guest no
    [[no]]: **Enter**
    Login class auth-defaults auth-ftp-defaults daemon default staff
    [[default]]: **auth-ftp-defaults**
    Enter password [[]]: **<ftpguy_passwort>**
    Enter password again [[]]: **<ftpguy_passwort>**
    
    .....
    
    OK? (y/n) [[y]]: **Enter**
    Added user ftpguy
    Add another user? (y/n) [[y]]: **n**

Wie gewohnt ersetzt man **<ftpguy_passwort>** durch das gewünschte
Passwort, welches als Loginpasswort für den FTP Server dient. Um diesen
Benutzer in seinem Homeverzeichniss einzusperren (sehr ratsam) muss man
nur seinen Namen in "**/etc/ftpchroot**" hinzufügen.

::

   # echo "ftpguy" >> /etc/ftpchroot

Da sich der Benutzer "**ftpguy**" auf diesem Rechner nicht einloggen
darf, macht es auch wenig Sinn die von "**/etc/skel**" automatisch
kopierten Dateien zu behalten.

::

   # rm /home/ftpguy/.[[a-z]]*

Sind alle Einstellungen getroffen, beginnt man mit dem Editieren von
"**/etc/inetd.conf**" und kommentiert dort alle Zeilen, die noch nicht
mit einem '#' beginnen, per '#' aus und fügt zum Schluss nur noch
folgende Zeile hinzu (siehe `man
ftpd <http://www.openbsd.org/cgi-bin/man.cgi?query=ftpd&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__).
``[[http://www.scatterbrained.de/openbsd/3.5/files/inetd.conf.1|/etc/inetd.conf]]``

::

   ftp             stream  tcp     nowait  root    /usr/libexec/tcpd       ftpd -US -n -A -l -T 120 -t 60

Anstelle von "**/usr/libexec/ftpd**" wird "**/usr/libexec/tcpd**"
verwendet um die Möglichkeit einer zusätzlichen Zugangskontrolle zu
schaffen. Wie in `Hardening OpenBSD Internet
Servers <http://geodsoft.com/howto/harden/OpenBSD/services.htm#tcpd>`__
genauer beschrieben wird, können dadurch die "**/etc/hosts.allow**" und
"**/etc/hosts.deny**" Dateien zur Einschränkung der erlaubten FTP
Verbindungen verwendet werden.

Manuell kann man inetd einfach per
"`inetd <http://www.openbsd.org/cgi-bin/man.cgi?query=inetd&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__"
starten und schon kann sich "**ftpguy**" auf dem FTP Server einloggen.
Eine kleine Änderung in "**/etc/rc.conf.local**" sorgt dann noch dafür,
dass inetd beim nächsten Reboot gestartet wird.
``[[http://www.scatterbrained.de/openbsd/3.5/files/rc.conf.local.4|/etc/rc.conf.local]]``

::

       inetd=YES
       ntpdate_flags="128.100.102.201"
       dhcpd_flags="<internes_device>"

Identd
------

Manche IRC-Netze (z.B. das DALnet) verlangen, dass ein identd-Server auf
ihre Anfragen antwortet. Dieser sollte im Normalfall den richtigen
Benutzernamen des Chatters zurückliefern. Da wir das aber erstens nicht
wollen und es zweitens etwas umständlich zu realisieren wäre (der
Chatter wird meistens wohl nicht auf dem Router eingeloggt sein, sondern
von irgendeinem Computer im LAN seinen IRC-Client verwenden), bedient
man sich eines kleinen Tricks. Im Internet findet man viele
"**Schummel-identd's**", die einfach erfundene Benutzernamen
zurückgeben. So sind beide zufrieden, wir und das DALnet. Einen
vielversprechenden "**fake identd**" habe ich auf dieser Seite gefunden:
`http:\ www.clock.org/~fair/opinion/identd.html]]. Damit auf dem Router
kein Compiler installiert werden muss habe ich nun ein kleines Paket
erstellt
([[http:\ www.scatterbrained.de/openbsd/3.5/files/fakeidentd-1.00.tgz\|
fakeidentd-1.00.tgz <http://www.clock.org/%7Efair/opinion/identd.html>`__)
welches sich mit "**pkg_add fakeidentd-1.00.tgz**" installieren lässt.
Ein Eintrag in "**/etc/inetd.conf**" sorgt dafür, dass der
"**fake_identd**" auch gestartet wird.

``[[http://www.scatterbrained.de/~grafj/openbsd/3.5/files/inetd.conf.2|/etc/inetd.conf]]``

::

   ident           stream  tcp     nowait  _identd  /usr/local/libexec/fake_identd  fake_identd

Wie immer ist jetzt ein Neustart von inetd fällig, damit die Änderungen
übernommen werden. Der Eintrag "**inetd=YES**" sollte in
"**/etc/rc.conf.local**" auch nicht fehlen (siehe 9. `FTP
Server <http://www.scatterbrained.de/openbsd/3.5/index.html#9._FTP_Server>`__).

NAT und Firewall
----------------

Unser Router macht nun inzwischen so ziemlich alles was wir von ihm
erwarten, ausser dem eigentlichen Routen selbst. Es wird langsam Zeit,
dass sich das ändert. Dazu schalten wir mit

::

   # sysctl -w net.inet.ip.forwarding=1

das Weiterleiten von IP Paketen ein und sorgen mit einem Eintrag in
`/etc/sysctl.conf <http://www.openbsd.org/cgi-bin/man.cgi?query=sysctl.conf&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
dafür, dass es auch nach dem nächsten Neustart so bleibt.

``[[http://www.scatterbrained.de/openbsd/3.5/files/sysctl.conf|/etc/sysctl.conf]]``

::

       net.inet.ip.forwarding=1

Im Folgenden sollte man nun kurz überdenken, ob man auf FTP Server im
`active mode <http://slacksite.com/other/ftp.html>`__ zugreifen können
will. Das ist bei manchen FTP Servern, die sich hinter einer Firwall
befinden, zwingend notwendig, nur stellt es bei der Konfiguration
unseres Routers ein kleines Sicherheitsproblem dar. Es dürfen dafür
nämlich keine Anfragen an die Ports >49151 geblockt werden. Meiner
Meinung nach kann man mit dieser "**Sicherheitslücke**" gut leben, zumal
sie ja erhebliche Vorteile mit sich bringt. Will man sich also zu FTP
Server, die nur im `active mode <http://slacksite.com/other/ftp.html>`__
laufen, verbinden können, muss man "**/etc/inetd.conf**" noch um den
Eintrag des
`FTP-Proxy's <http://www.openbsd.org/cgi-bin/man.cgi?query=ftp-proxy&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__
erweitern.

``[[http://www.scatterbrained.de/openbsd/3.5/files/inetd.conf.3|/etc/inetd.conf]]``

::

   127.0.0.1:8021            stream  tcp     nowait  root    /usr/libexec/ftp-proxy  ftp-proxy

Danach ist der inetd neu zu starten und, falls noch nicht geschehen,
"**inetd=YES" in "/etc/rc.conf.local**" einzufügen (siehe 9. `FTP
Server <http://www.scatterbrained.de/openbsd/3.5/index.html#9._FTP_Server>`__).

Die Konfiguration der Firewall befindet sich in der Datei
`/etc/pf.conf <http://www.openbsd.org/cgi-bin/man.cgi?query=pf.conf&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__.

``[[http://www.scatterbrained.de/openbsd/3.5/files/pf.conf|/etc/pf.conf]]``

::

  ### VARIABLEN ###

  Ext = "tun0"            # Device an dem das Internet angeschlossen ist 
  Int = "<internes_device>"      # Device an dem das interne Netz haengt
  IntNet = "192.168.1.0/24"      # Adressraum des internen Netzes
  RouterIP = "192.168.1.1"       # IP Adresse des Routers
  Loop = "lo0"                   # Loopback Device

  # Adressen die auf dem externen Device nicht geroutet werden
  # (Adressbereich des internen Netzes muss man wegen der Weiterleitungen zulassen)
  table <NoRoute> { 127.0.0.1/8, 172.16.0.0/12, 192.168.0.0/16, !$IntNet, 10.0.0.0/8, 255.255.255.255/32 }

  # Ports die geoeffnet werden sollen
  InServicesTCP = "{ ssh, ftp, auth }"


  ### OPTIONS ###

  # Macht Statistiken fuer die DSL-Verbindung (pfctl -s info)
  set loginterface $Ext

  # Beendet inaktive Verbindungen schneller - geringerer Speicherverbrauch.
  set optimization aggressive

  # Fragmentierte Pakete saeubern
  scrub on $Ext all fragment reassemble random-id

  # Queueing 
  # !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
  # !! Achtung: Der unten stehende Wert von 100Kb (Kilobit) macht natuerlich
  # !! nur fuer den Standard DSL Anschluss mit 128kb upstream Sinn. Hat man
  # !! eine Verbindung mit groesserer Bandbreite beim upload, dann muss 
  # !! dieser Wert entsprechend angepasst werden. 
  # !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
  altq on $Ext priq bandwidth 100Kb queue { q_pri, q_def }
  queue q_pri priority 7
  queue q_def priority 1 priq(default)


  ### NAT & FORWARD ###

  # NAT aktivieren (unter Linux als Masquerading bekannt)
  nat on $Ext from $IntNet to any -> $Ext static-port

  # Active FTP - Umleitung zu unserem ftp-proxy
  rdr on $Int proto tcp from !$RouterIP to !$IntNet port 21 -> 127.0.0.1 port 8021

  rdr-anchor "redirect/*"


  ### FILTER ###

  # Zum Debuggen....
  #pass quick all             # Alles durchlassen

  # Generelle Block Regel
  block on $Ext

  # Freiwillig machen wir keinen mucks ;)
  block return log on $Ext

  # Wir wollen kein IPv6.0
  block quick inet6

  # Loopback Device darf alles
  pass quick on $Loop

  # IP Spoofing verhindern
  block in log quick on $Ext inet from <NoRoute> to any
  block in log quick on $Ext inet from any to <NoRoute>

  # Active FTP erlauben
  pass in quick on $Ext inet proto tcp from any to any port > 49151 user proxy flags S/SAFR keep state

  # Ping akzeptieren (ablehnen ist uebrigends wenig sinnvoll)
  pass in quick on $Ext inet proto icmp all icmp-type 8 code 0 keep state

  # Ports nach aussen oeffnen
  pass in quick on $Ext inet proto tcp from any to any port $InServicesTCP flags S/SAFR keep state label ServicesTCP

  anchor "passin/*"

  # Raus darf (fast) alles
  pass out quick on $Ext keep state queue (q_def,q_pri)

Diese Datei kann man 1 zu 1 übernehmen und auf dem eigenen Router
verwenden. Es muss lediglich **<internes_device>** durch den Devicenamen
der Netzwerkkarte, die an das LAN angeschlossen ist, ersetzt werden (in
diesem Beispiel ist das "**rl0**"). Eventuell muss auch noch der
Adressraum des internen Netzwerks (siehe Variable **$IntNet**), die nach
aussen geöffneten Ports (siehe Variable **$InServicesTCP**) oder die
interne IP des Routers (siehe Variable **$RouterIP**) angepasst werden.
Wer kein "active" FTP will, braucht nur die zwei entsprechenden Zeilen
in "**pf.conf**" auszukommentieren. Die so erstellten NAT/Firewall
Regeln können mit

::

   # /sbin/pfctl -e -F all -f /etc/pf.conf

neu geladen werden. Dank des "**ppp.linkup**" Skripts werden sie
automatisch nach jeder Neueinwahl reinitialisiert.

Gratulation! Dein OpenBSD 3.5 Router sollte nun vollständig
einsatzbereit sein. Bei der Konfiguration der Clients im LAN genügt es
auf die automatische IP-Vergabe per DHCP zu vertrauen. Wahlweise können
die Daten aber auch manuell angegeben werden. Dazu gehören die IP
Adresse des Routers als Gateway und Nameserver, sowie eine noch nicht
vergebene Adresse im Bereich von 192.168.1.2-126 als IP des Clients.

Mit Hilfe der seit OpenBSD 3.3 eingeführten "**anchors**" kann man im
laufenden Betrieb Regelsätze zur Firewall hinzufügen. Ein anchor dient
gewissermasen als Platzhalter für Regeln, die später dort eingefügt
werden. In obiger pf.conf gibt es zwei Anchors, einen mit dem Namen
"**redirect**" und den Anderen mit Namen "**passin**". Damit ist es nun
möglich beliebige Redirect und Pass/Block Regeln hinzuzufügen, ohne die
Firewall resetten zu müssen. Wie man dieses Feature sinnvoll verwenden
kann, sollen die zwei Beispiele für die Weiterleitung von eMule und
DirectPlay zeigen. Eventuell werden später noch weitere folgen - wenn du
eine gute Idee hast, dann her damit.

Seit OBSD 3.4 funktionert nun endlich auch das Zuweisen von
unterschiedlichen Prioritäten für Verbindungen auf dem tun0 Interface.
Die von Volker Kindermann vorgeschlagene Methode zur Verbesserung der
Verbindungsqualität bei einem laufenden Download, ist in der obigen
Konfiguration schon integriert. **Was allerdings beachtet werden
sollte:** Obige Konfiguration ist für eine Standard DSL Verbindung
ausgelegt bei der man über lediglich 128Kb Upstream verfügt. Bereinigt
um etwas Protokoll Overhead kommt man so auf die 100Kb. Wer mehr (oder
auch weniger) Upstream hat, der sollte diesen Wert tunlichst an die
Gegebenheiten anpassen. Dabei bleibt man einfach im Verhältniss zwischen
gegebener Upload Rate und Begrenzung von ca. 5:4 - bei 256Kb Upload ist
also ein Wert von etwa 200-210Kb denkbar... Wer nochmal nachlesen will
um was es dabei genau geht, der kann hier vorbeischauen: `ACKPRI für
OBSD 3.3 <http://www.scatterbrained.de/openbsd/ackpri/index.html>`__
oder `Prioritizing empty TCP ACKs with pf and
ALTQ <http://www.benzedrine.cx/ackpri.html>`__

Wenn man sich intensiver mit den beeindruckenden Fähigkeiten von pf
beschäftigen will, dann ist das offizielle
`pf-FAQ <http://www.openbsd.org/faq/pf/>`__ eine sehr gute Adresse.

Erweiterung für eMule
~~~~~~~~~~~~~~~~~~~~~

Die Regeln für die Weiterleitung von eMule an einen bestimmten Computer
im internen Netzwerk sind in zwei Dateien aufgespaltet. Zum einen in die
Regeln, die der eigentlichen Umleitung dienen ("**emule.redirect**") und
zum anderen Regeln, die dafür sorgen, dass die benötigten Ports nicht
mehr geblockt werden ("**emule.passin**"). Diese Dateien sollten wie
folgt angegeben erstellt werden.

``[[http://www.scatterbrained.de/openbsd/3.5/files/emule.redirect|/etc/emule.redirect]]``

::

       Ext = "tun0"               # Device an dem das Internet angeschlossen ist 
       MuleIP = "<emule_client>"  # IP Adresse des Emule Clients 
       IntNet = "192.168.1.0/24"  # Adressraum des internen Netzes
    
       rdr on $Ext proto tcp from !$IntNet to any port 4661:4662 -> $MuleIP port 4661:*
       rdr on $Ext proto udp from !$IntNet to any port 4665 -> $MuleIP port 4665
       rdr on $Ext proto udp from !$IntNet to any port 4672 -> $MuleIP port 4672

``[[http://www.scatterbrained.de/openbsd/3.5/files/emule.passin|/etc/emule.passin]]``

::

       Ext = "tun0"            # Device an dem das Internet angeschlossen ist 
       InMuleTCP = "{ 4661, 4662 }"
       InMuleUDP = "{ 4665, 4672 }"
    
       pass in quick on $Ext inet proto tcp from any to any port $InMuleTCP flags S/SAFR keep state label eMuleTCP
       pass in quick on $Ext inet proto udp from any to any port $InMuleUDP keep state label eMuleUDP

Ist der Platzhalter **<emule_client>** durch die IP des Rechners , dem
der Zugriff auf das eMule Netzwerk ermöglicht werden soll, ersetzt, dann
können die Regeln hinzugeladen werden.

::

   # pfctl -a "redirect/emule" -f /etc/emule.redirect
   # pfctl -a "passin/emule" -f /etc/emule.passin

Falls die Umleitung irgendwann nicht mehr benötigt wird, dann lässt sie
sich auch ganz einfach mit

::

   # pfctl -a "redirect/emule" -F nat
   # pfctl -a "passin/emule" -F rules

abschalten.

Erweiterung für DirectPlay
~~~~~~~~~~~~~~~~~~~~~~~~~~

Genau wie beim vorhergehenden Beispiel mit eMule wird auch mit der
Weiterleitung von DirectPlay verfahren. Die Regeln sind wieder in zwei
entsprechende Dateien aufgeteilt.
``[[http://www.scatterbrained.de/openbsd/3.5/files/directplay.redirect|/etc/directplay.redirect]]``

::

       Ext = "tun0"                # Device an dem das Internet angeschlossen ist 
       GameIP = "<game_client>"    # IP Adresse des DirectPlay Clients 
       IntNet = "192.168.1.0/24"   # Adressraum des internen Netzes
    
       rdr on $Ext proto tcp from !$IntNet to any port 2300:2400 -> $GameIP port 2300:*
       rdr on $Ext proto tcp from !$IntNet to any port 47624 -> $GameIP port 47624
       rdr on $Ext proto tcp from !$IntNet to any port 6073 -> $GameIP port 6073
       rdr on $Ext proto udp from !$IntNet to any port 2300:2400 -> $GameIP port 2300:*
       rdr on $Ext proto udp from !$IntNet to any port 9110 -> $GameIP port 9110

``[[http://www.scatterbrained.de/openbsd/3.5/files/directplay.passin|/etc/directplay.passin]]``

::

       Ext = "tun0"            # Device an dem das Internet angeschlossen ist 
       InDirectPlayTCP = "{ 2299><2401, 6073, 47624 }"
       InDirectPlayUDP = "{ 2299><2401, 9110 }"
    
       pass in quick on $Ext inet proto tcp from any to any port $InDirectPlayTCP flags S/SAFR keep state label DirectPlayTCP
       pass in quick on $Ext inet proto udp from any to any port $InDirectPlayUDP keep state label DirectPlayUDP

Wobei **<game_client>** durch die IP des Computers, der DirectPlay
nutzen soll, ersetzt werden sollte. Die Regeln werden wie folgt geladen

::

   # pfctl -a "redirect/directplay" -f /etc/directplay.redirect
   # pfctl -a "passin/directplay" -f /etc/directplay.passin

und können auch völlig analog zum vorhergehenden Beispiel wieder
gelöscht werden.

::

   # pfctl -a "redirect/directplay" -F nat
   # pfctl -a "passin/directplay" -F rules

Erweiterung für BitTorrent
~~~~~~~~~~~~~~~~~~~~~~~~~~

Auch für den Bit Torrent Service verläuft das Freischalten und
Weiterleiten des Dienstes analog zu den vorhergehenden 2 Beispielen.

**/etc/bittorrent.redirect**

http://www.scatterbrained.de/openbsd/3.5/files/bittorrent.redirect

::

       Ext = "tun0"                        # Device an dem das Internet angeschlossen ist
       BitIP = "<bittorrent_client>"       # IP Adresse des BitTorrent-Clients
       IntNet = "192.168.1.0/24"           # Adressraum des internen Netzes
    
       rdr on $Ext proto tcp from !$IntNet to any port 6969 -> $BitIP port 6969
       rdr on $Ext proto tcp from !$IntNet to any port 6881:6889 -> $BitIP port 6881:*

**/etc/bittorrent.passin**

http://www.scatterbrained.de/openbsd/3.5/files/bittorrent.passin

::

       Ext = "tun0"                        # Device an dem das Internet angeschlossen ist
       InBitTCP = "{ 6969, 6881:6889 }"    # Von Bit Torrent benoetigte Ports
    
       pass in quick on $Ext inet proto tcp from any to any port $InBitTCP flags S/SAFR keep state label BitTCP

Wie gehabt wird <bittorrent_client> durch die IP des Computers, der Bit
Torrent nutzen soll, ersetzt. Die Regeln werden wie folgt geladen

::

   # pfctl -a "redirect/bittorrent" -f /etc/bittorrent.redirect
   # pfctl -a "passin/bittorrent" -f /etc/bittorrent.passin

und können auch wie gewohnt wieder gelöscht werden.

::

   # pfctl -a "redirect/bittorrent" -F nat
   # pfctl -a "passin/bittorrent" -F rules

Quellen
-------

- http://www.openbsd.org/de/
- http://www.openbsd.org/faq/faq4.html
- http://www.openbsd.org/faq/pf/
- http://www.realo.ca/BSDinstall.html
- http://www.unixscout.de/
- http://geodsoft.com/howto/harden/bsdhardn.htm

Danksagung vom Autor des Originals
----------------------------------

Inzwischen sind bei mir schon einige Emails mit guten
Verbesserungsvorschlägen angekommen. Ich möchte mich auf diesem Weg
nochmals bei Allen bedanken, die mir dabei geholfen haben dieses HOWTO
möglichst verständlich, aktuell und fehlerfrei werden zu lassen. Danke
an: "*Crash Override*" vom `gEb <gEb>`__, Ingolf Schuchardt, Hendrik
Volkmer, Philipp Buehler aka "*fips*", Markus Pischulti, Danny Wagener,
Frank Postleb, Grigori Goronzy, Jan Riedel, Waldemar Brodkorb, Thorsten
Janssen, Tobias Wigand, Volker Kindermann, Lukas Neumann, Fabian aka
"*Morphium*", Oliver Arend, Christian Schmaler, Jan Tietjen, Dennis
Gross, Tobias Kerkering, Martin Weichselbaum, Marc Wirth, Fabian
Paganotto, Detlef Peeters, Christian Dellwo, Patrick Wettmann und Gernot
Pörner.

* :ref:`genindex`

Zuletzt geändert: |date|

