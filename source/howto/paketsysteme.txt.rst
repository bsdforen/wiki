Paketsysteme
============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Grundsätzliches
---------------

Auf den meisten unixoiden Betriebsystemen gibt es Paketsysteme. Sie
regeln Installation, Upgrades etc. und lösen Abhängigkeiten auf. Bei den
BSDs gibt es drei:

-  die FreeBSD-Ports
-  die OpenBSD-Ports
-  PkgSrc (wird von NetBSD und DragonFlyBSD verwendet)

Die OpenBSD-Ports und PkgSrc sind ursprünglich Forks der FreeBSD-Ports,
deswegen ist die Bedienung in vielen Fällen ähnlich.

Dieser Artikel soll eine Einführung in die grundlegende Bedienung der
Paketsysteme mit den in dem jeweiligen BSD standardmäßig vorhandenen
Programmen erläutern. Man kann mit den Paketsystemen noch wesentlich
mehr machen als hier beschrieben wird, aber dafür gibt es dann spezielle
Artikel.

Grundsätzlich unterscheiden sich die BSD-Paketsysteme von anderen
bekannten (wie z.B. apt-get/dpkg oder RPM), dadurch dass sie sowohl
Installation durch Binärpakete, als auch durchs Kompilieren der Quellen
erlauben. Welches von beiden man nutzt oder ob man beides mischen
kann/soll/darf ist Geschmacksache und löst oft Diskussionen aus. Muss im
Endeffekt jeder selber wissen!

Im Gegensatz zu den meisten GNU/Linux-Distributionen, wird unter \*BSD
nicht das komplette System vom Paketsystem verwaltet. Der Kernteil des
Betriebsystems, die sog. Basis, zu der Kernel und Unixstandardprogramme
gehören (in OpenBSD gehört der X-Server auch dazu), werden über andere
Mechanismen auf dem neusten Stand gehalten. Unter FreeBSD siehe dazu
`Make World <Make World>`__.

Noch was: wenn im folgenden von dem Befehl **make** gesprochen wird, so
müssen DragonFlyBSD-Nutzer und andere Nicht-NetBSD-Nutzer von PkgSrc
stattdessen **bmake** nehmen!

Terminologie
------------

Die Terminologie ist nicht sehr eindeutig oder konsistent, aber
grundsätzlich kann man folgende Dinge festhalten:

-  bereits installierte "Software" und "Software" die
   vorkompiliert(binär) vorliegt wird als *Paket* bezeichnet.
-  Anweisungen zum Kompilieren und Installieren von "Software" durch
   Quellcode, bezeichnet man unter Free- und OpenBSD als *Port*, unter
   Net- und DragonFlyBSD als *"PackageSource"* bzw. *Paketquelle*.

| Diese "Anweisungen", also *Ports* bzw. *Paketquellen* liegen unter
  Free- und OpenBSD in /usr/ports und unter Net- und DragonFlyBSD in
  /usr/pkgsrc. Um Platz und Nerven zu sparen werde ich im weiteren
  Verlauf des Artikels von *Ports* sprechen und das Verzeichnis mit
  $BAUM bezeichnen, wenn es sowohl um /usr/ports, als auch um
  /usr/pkgsrc geht.
| Unterhalb von $BAUM sind die *Ports* in Kategorien
  unterteilt(\ *Ports* können mehreren Kategorien zugeordnet werden,
  sind aber stets nur in einer Kategorie unterhalb von $BAUM zu finden).
  Der *Port* des textbasierten Browsers **Links** befindet sich also
  unter $BAUM/www/links . Im Allgemeinen kann man also sagen, dass der
  *Port* eines Programms unter $BAUM\ */kategorie/programmname* liegt.
| In bestimmten Fällen ist dies nicht so, wenn es z.B. mehrere *Ports*
  für dasselbe Programm in unterschiedlichen Versionen gibt. Unter
  FreeBSD wird dann meistens die Version an *programmname* angehangen
  wie z.B. bei GCC, für den es dann /usr/ports/lang/gcc34,
  /usr/ports/lang/gcc40 usw. gibt (ACHTUNG: der Name des *Pakets* dass
  durch diese *Ports* gebaut wird ist trotzdem GCC, **nicht** GCC34 oder
  GCC40).
| Unter OpenBSD wurde für solche *Ports* eine dritte Ebene eingeführt,
  hier liegen die beiden GCC-Ports also unter /usr/ports/lang/gcc/34 und
  /usr/ports/lang/gcc/40... aber das sind Spezialfälle ;)
| Ein *Port* ist nur eindeutig wenn, der komplette Pfad nach $BAUM
  angegeben wird (der zweite Teil, also der Programmname, kann mehrmals
  vorkommen -in unterschiedlichen Kategorien- oder vom Namen des später
  installierten Pakets abweichen!!!). Dieser Pfad wird als ORIGIN
  bezeichnet, die ORIGIN von Links wäre also **www/links**.

Installieren eines Programms über die Ports/PkgSrc
--------------------------------------------------

Möchte man ein noch nicht installiertes Programm aus den Quellen
installieren, so geht man nach $BAUM/$ORIGIN und gibt **make install
clean** ein. Also z.B.: <xterm> # cd /usr/ports/net-im/pidgin # make
install clean </xterm> Wenn man die ORIGIN nicht kennt kann man durch
ein **whereis name** oder durch <xterm> # cd /usr/ports # make
quicksearch key=pidgin Port: finch-2.0.0 Path: /usr/ports/net-im/finch
Info: Finch multi-protocol messaging client (Console UI)

Port: gaim-hotkeys-0.2.2_2 Path: /usr/ports/net-im/gaim-hotkeys Info: A
gaim plugin that allows user to assign global hotkeys

Port: libpurple-2.0.0 Path: /usr/ports/net-im/libpurple Info: Backend
library for the Pidgin multi-protocol messaging client

Port: pidgin-2.0.0 Path: /usr/ports/net-im/pidgin Info: Pidgin
multi-protocol messaging client (GTK+ UI)

</xterm> danach suchen.

Durch <xterm> # cd net-im/pidgin # make install clean </xterm> wechselt
man in das entsprechende Verzeichnis. Danach werden dann die Quellen für
das Programm und alle seine Abhängigkeiten heruntergeladen, wenn nötig
gepatcht, kompiliert und installiert. Zum Schluß werden die nicht mehr
benötigten Sourcen und Objektdateien wieder gelöscht.

Installieren eines Programms über die Pakete
--------------------------------------------

Die BSDs bieten für viele (nicht alle) der *Ports* vorkompilierte Pakete
an. Diese sind nicht immer aktuell oder vollständig (auch abhängig von
Rechnerarchitektur), aber wenn es für einen *Port* ein *Paket* gibt,
dann sollte es auch für alle Abhängigkeiten Pakete geben. Zum Hinzufügen
eines noch nicht installierten *Pakets* und seinen Abhängigkeiten nutzt
man den Befehl **pkg_add**, bzw. bei FreeBSD **pkg**. Also für unser
Beispiel Links: (OpenBSD-Ports und PkgSrc) <xterm> # pkg_add links
</xterm> bzw. unter FreeBSD-Ports <xterm> # pkg install links </xterm>
Standardmäßig sollten (automatisch) unter allen BSDs die, für die
verwandte Betriebsystemsversion passenden Pakete verwendet werden.
Besonders bei FreeBSD kann es aber schlauer sein auch auf einem Release
die Pakete von -STABLE zu verwenden, siehe dazu
`FreeBSD-Zweige <FreeBSD-Zweige>`__.

Löschen eines installierten Pakets
----------------------------------

| Mit **pkg_info** erhält man eine Liste der installierten Pakete.
  Möchte man ein Paket löschen, das von keinem anderen benötigt wird,
  reicht ein **pkg_delete**, also z.B.: <xterm> # pkg_delete
  links-2.1.p28,1 </xterm> Bei FreeBSD ist es wichtig die Version mit
  anzugeben, möchte man dies nicht, kann man auch **-x** als Option
  hinzufügen.
| Wird ein Paket noch von anderen benötigt, gibt es zwei Möglichkeit:

-  Man löscht die Pakete, die es benötigen gleich mit. Bei FreeBSD-Ports
   und PkgSrc: **pkg_delete -r**, OpenBSD-Port: **pkg_delete -F
   dependencies**; oder
-  man erzwingt das Löschen mit der **-f**-Option (FreeBSD-Ports und
   PkgSrc) bzw. **-F (ohne Parameter)** (OpenBSD-Ports). Dies sollte nur
   gemacht werden wenn das Programm danach neuinstalliert wird (z.B.
   beim Upgrade, siehe unten)!

| Bei PkgSrc gibt es noch die Option **-R** (großes R), die nach einem
  Löschvorgang überflüssig gewordene Abhängigkeiten mitlöscht.

Upgrade eines installierten Pakets
----------------------------------

| Grundsätzlich kann man so vorgehen, dass man ein Paket erst löscht
  (wenn nötig mit **-f** bzw. **-F**) und dann neuinstalliert (über
  *Ports* oder *Pakete*). Dabei kann es manchmal nötig sein, ein von dem
  Paket benötigtes Paket vorher genauso neuzuinstallieren.
| Bei PkgSrc und den OpenBSD-Ports, kann man Upgrades durch Pakete auch
  einfacher machen mit der **-u** Option für **pkg_add**: <xterm> #
  pkg_add -u links </xterm> Dies würde die neuste in den Paketen
  verfügbare Version von links herunterladen und die bisherige ersetzen.
  Unter PkgSrc kann man mit **-uu** ein Paket inklusive seiner
  Abhängigkeiten auf den neusten Stand bringen.
| Unter FreeBSD wird meist
  `Portupgrade </anwendungen/portupgrade>`__\  [1]_ oder
  `Portmaster </anwendungen/portmaster>`__\  [2]_ benutzt, um Programme
  auf den neusten Stand zu bringen. Mit
  `bsdadminscripts#pkg_upgrade </anwendungen/bsdadminscripts#pkg_upgrade>`__\  [3]_
  ist es jedoch auch Möglich mit Paketen zu aktualisieren.

Aktualisieren der Ports / PkgSrc
--------------------------------

| *Pakete* werden auf den Mirrors automatisch aktualisiert, bei seinen
  Ports muss man das selber machen. Unter PkgSrc und den FreeBSD-Ports
  geht dies mit einem **make update** in $BAUM, also z.B.: <xterm> # cd
  /usr/ports # make update </xterm> FreeBSD benutzt dafür im Hintergrund
  `Portsnap </anwendungen/portsnap>`__, dazu muss der Portstree aber
  auch ursprünglich von `Portsnap </anwendungen/portsnap>`__ erzeugt
  worden sein.
| Unter OpenBSD muss man dafür per Hand cvs bemühen: <xterm> # cd
  /usr/ports # cvs -q -d anoncvs@some.anon.server:/cvs up -r OPENBSD_4_1
  -Pd </xterm> Hier muss darauf geachtet werden auf jeden Fall den von
  einem selbst verwendeten OpenBSD-Zweig zu nehmen (in diesem Fall
  OpenBSD4.1). Bei PkgSrc sollte **make update** automatisch auf den
  richtigen Zweig aktualisieren; FreeBSD hat nur einen Zweig (siehe auch
  FreeBSD-Zweige).

Grafische Oberflächen zur Paketverwaltung
-----------------------------------------

-  `bpm </anwendungen/bpm>`__\  [4]_ - Ein GTK+ basiertes Ports-Frontend
   (Nur FreeBSD-Ports)
-  `desktopbsd-tools </anwendungen/desktopbsd-tools>`__ - Eine KDE
   Programmsammlung für DesktopBSD, die auch ein Ports-Frontend
   enthält.(Nur FreeBSD-Ports)
-  `kports </anwendungen/kports>`__\  [5]_ - Ein umfangreiches, KDE
   basiertes Frontend für die Ports. (Nur FreeBSD-Ports, in Zukunft auch
   PkgSrc und OpenBSD-Ports)
-  `portbrowser </anwendungen/portbrowser>`__\  [6]_ - Ein auf
   Gtk2-basiertes Frontend. (Nur FreeBSD-Ports)

Verweise
--------

-  FreeBSD Man-Seiten:
   `ports(7) <http://www.freebsd.org/cgi/man.cgi?query=ports>`__,
   `pkg_info(1) <http://www.freebsd.org/cgi/man.cgi?query=pkg_info>`__,
   `pkg_add(1) <http://www.freebsd.org/cgi/man.cgi?query=pkg_add>`__,
   `pkg_delete(1) <http://www.freebsd.org/cgi/man.cgi?query=pkg_delete>`__
-  FreeBSD-Hanbuch: `Kapitel
   4 <http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports.html>`__
-  OpenBSD Man-Seiten:
   `ports(7) <http://www.openbsd.org/cgi-bin/man.cgi?query=ports>`__,
   `packages(7) <http://www.openbsd.org/cgi-bin/man.cgi?query=packages>`__,
   `pkg_info(1) <http://www.openbsd.org/cgi-bin/man.cgi?query=pkg_info>`__,
   `pkg_add(1) <http://www.openbsd.org/cgi-bin/man.cgi?query=pkg_add>`__,
   `pkg_delete(1) <http://www.openbsd.org/cgi-bin/man.cgi?query=pkg_delete>`__
-  OpenBSD FAQ: `Kapitel 15 <http://www.openbsd.org/faq/faq15.html>`__
-  NetBSD Man-Seiten:
   `packages(7) <http://netbsd.gw.com/cgi-bin/man-cgi?packages>`__,
   `pkg_info(1) <http://netbsd.gw.com/cgi-bin/man-cgi?pkg_info>`__,
   `pkg_add(1) <http://netbsd.gw.com/cgi-bin/man-cgi?pkg_add>`__,
   `pkg_delete(1) <http://netbsd.gw.com/cgi-bin/man-cgi?pkg_delete>`__
-  Der `PkgSrc-Guide <http://www.netbsd.org/docs/pkgsrc/index.html>`__

.. [1]
   `ports-mgmt/portupgrade <https://www.google.com/search?q=ports-mgmt/portupgrade&btnI=lucky>`__

.. [2]
   `ports-mgmt/portmaster <https://www.google.com/search?q=ports-mgmt/portmaster&btnI=lucky>`__

.. [3]
   `sysutils/bsdadminscripts <https://www.google.com/search?q=sysutils/bsdadminscripts&btnI=lucky>`__

.. [4]
   `ports-mgmt/bpm <https://www.google.com/search?q=ports-mgmt/bpm&btnI=lucky>`__

.. [5]
   `ports-mgmt/kports <https://www.google.com/search?q=ports-mgmt/kports&btnI=lucky>`__

.. [6]
   `ports-mgmt/portbrowser <https://www.google.com/search?q=ports-mgmt/portbrowser&btnI=lucky>`__

* :ref:`genindex`

Zuletzt geändert: |date|

