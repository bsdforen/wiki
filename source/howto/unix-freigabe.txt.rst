Unix-Freigabe
=============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Es gibt zwei Möglichkeiten unter Unix Ordner bzw. Dateien im Netzwerk
freizugeben. `NFS <NFS>`__ oder `Samba <Samba>`__. `NFS <NFS>`__ benutzt man
normalerweise beim Datenaustausch von Unix auf Unix und Samba zwischen Unix-
und Windows-Clients.

Samba
-----

Unter FreeBSD wird der Port
`net/samba3 <https://www.google.com/search?q=net/samba3&btnI=lucky>`__
benötigt. Unter Pkgsrc findet sich Samba unter
`net/samba <https://www.google.com/search?q=net/samba&btnI=lucky>`__ und
in den OpenBSD Ports unter
`net/samba <https://www.google.com/search?q=net/samba&btnI=lucky>`__.

Nach der Installation muss unter ``/usr/local/etc/`` oder
``/usr/pkg/etc/`` die ``smb.conf`` angepasst werden.

In diesem Config-File legt ihr fest welche Verzeichnisse freigegeben
werden und welche Rechte die User auf diese freigebenen Verzeichnisse
haben (Lese- oder Schreibrechte). Hier könnt ihr euch zum Testen kurz
diese ``smb.conf`` anpassen, danach solltet ihr eure eigene ``smb.conf``
erstellen, da diese ``smb.conf`` volle Schreib-, Ausführungs- und
Leserechte allen Usern bietet die auf das Verzeichniss zugreifen.
HowTo's und Hilfe findet ihr im `The Unofficial Samba
HOWTO <http://www.oregontechsupport.com/samba>`__.

``smb.conf``

::

   [global]
   workgroup = home
   netbios name = rechnername
   guest account = nobody
   keep alive = 30
   os level = 2
   kernel oplocks = false
   security = share
   encrypt passwords = yes
   socket options = TCP_NODELAY
   map to guest = Bad User

   wins support = no

   [download]
   comment = download
   path = <hier den vollstaendigen pfad den ihr freigeben wollt>
   browseable = yes
   public = yes
   read only = no
   create mode = 0777

Nun müsst ihr noch die Samba-Dienste starten

::

   # /usr/sbin/smbd -D
   # /usr/sbin/nmbd -D

oder (bei FreeBSD)

::

   # /usr/local/etc/rc.d/samba onestart

Es ist wichtig das ihr zuerst ``smbd`` und dann ``nmbd`` startet, das
Skript erledigt das automatisch. Damit unter FreeBSD Samba beim Booten
automatisch gestartet wird muss noch die Zeile

::

   samba_enable="YES"

in die Datei ``/etc/rc.conf`` eingetragen werden.

Die Unix-Maschine mit Samba erscheint dabei in der Netzwerkumgebung des
Windows-Clients, wenn man den Rechner in der Netzwerkumgebung durch
einen Doppelklick öffnet sieht man die Freigaben. Soll aber vorher eine
Authentifizierung durchgeführt werden, muss der Wert von ``security``
auf ``user`` geändert werden. Entsprechende Samba-Benutzer können dann
durch den Befehl ``smbpasswd`` verwaltet werden. Durch einen Rechtsklick
und ``Netzlaufwerk verbinden`` wird die Verbindung zum Server
hergestellt. Man muss ihm dabei nur noch einen Laufwerksbuchstaben
zuweisen und angeben ob bei jedem Start die Verbindung wiederhergestellt
werden soll.

NFS
---

Um einen NFS-Server aufzusetzen greift ihr am besten auf die jeweiligen
Dokumentationen der jeweiligen BSD-Version zurück da diese sehr gut
sind.

-  `OpenBSD <http://www.openbsd.org/faq/faq6.html#NFS>`__
-  `FreeBSD <http://www.freebsd.org/doc/de_DE.ISO8859-1/books/handbook/network-nfs.html>`__
-  `NetBSD <http://www.netbsd.org/Documentation/network/netboot/nfs.html#netbsd>`__

HFS und Apples MacOS
--------------------

HFS und HFS-Plus Freigaben für Apples MacOS unter FreeBSD.

Yar Tikhiy hat die HFS und HFS Plus Module von Darwin für FreeBSD
portiert. Sie sind in der aktuellen Version 5.1 von FreeBSD enthalten.
HFS und HFS Plus ist das Dateisystem von Apples MacOS Betriebsystem.
Diese Module sind Interessant wenn man in einem Netzwerk von FreeBSD aus
Netzwerkfreigaben für MacOS verwirklichen möchte. Der Port ist noch in
der Experimentierphase und es soll daher noch viel getestet werden. Mehr
Informationen gibt es auf der offiziellen Seite zu diesen
Kernel-Modulen.

Verweise
--------

-  `The Unofficial Samba
   HOWTO <http://www.oregontechsupport.com/samba>`__
-  `HFS and HFS Plus in FreeBSD <http://people.freebsd.org/~yar/hfs/>`__

* :ref:`genindex`

Zuletzt geändert: |date|

