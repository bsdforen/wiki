DNS-Cache
=========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Wenn man FreeBSD als Gateway für mehrere andere Rechner in einem Netz mit
statischer Konfiguration verwendet, dann bietet es sich an, einen DNS-Cache
einzurichten, der Namensserveranfragen der Client-Rechner im Netzwerk
stellvertretend für einen DNS-Server des Internetanbieters beantworten kann.
Das hat den Vorteil, dass einerseits Anfragen nach draußen reduziert werden,
was die Internetverbindung minimal entlastet und ggf. Anfragen beschleunigt,
andererseits kann man rasch den DNS-Server wechseln, ohne die
Netzwerkeinstellungen jedes Rechners im Netz anpassen zu müssen.

Diese kleine Anleitung zeigt die entsprechende Einrichtung des Namensservers
``bind``, der bei FreeBSD standardmäßig installiert ist bzw. im Basissystem
steckt. Als gern verwendete und schlanke Alternative gilt z.B. dns/djbdns. Ein
weiterer kleiner DNS-Server mit MySQL-Schnittstelle ist dns/powerdns. Auf deren
Konfiguration wird hier nicht eingegangen.

Zunächst sollte der Rechner einen Hostnamen besitzen. Wenn man das bei der
Konfiguration der Netzwerkadapter nicht bereits erledigt hat, dann lässt sich
das schnell nachholen: 

::

  # echo 'hostname="rechnername.heimnetz"' >> /etc/rc.conf # hostname rechnername.heimnetz

Neben dem Rechnernamen "rechnername" gibt es das ebenfalls frei wählbare Suffix
"heimnetz", das einen Domänennamen darstellt, der u.a. die Zusammengehörigkeit
mehrerer Hosts in einem Netz anzeigt. Hier wird auch gerne "local" gewählt.

Vorbereitend wird nun eine Zonendatei im Verzeichnis /etc/named/master für die
Domäne angelegt, die zwei Einträge für "rechnername" und "localhost" enthält:

::

  # cd /etc/namedb 
  # sh make-localhost

Falls man zuvor beim Rechnernamen keinen Domänenamen vergeben hat, dann fragt
das Skript explizit danach.

In den nächsten beiden Schritten passen wir die Konfigurationsdatei
``/etc/namedb/named.conf`` unseren Bedürfnissen an indem wir sie mit
einem Editor bearbeiten.

Man lässt den Namensserver auf der IP der internen Netzwerkkarte laufen,
die von allen Rechnern im Netz als Gateway-IP-Adresse verwendet wird. Im
Beispiel benutzen wir dafür 192.168.0.1. Dazu ändert man die Zeile

::

   listen-on       { 127.0.0.1; };

zu beispielsweise

::

   listen-on       { 127.0.0.1; 192.168.0.1; };

Als weiteren Schritt trägt man die Namensserver ein, die dieser
DNS-Cache zur Anfrage selbst benutzen soll, falls er keine direkte
Antwort an den Client liefern kann. Dazu entfernt man die
Kommentarzeichen um den Eintrag ``forwarders`` und fügt die IP-Adressen
der DNS-Server von seinem Internetanbiet ein:

::

     /* Namensserver von QSC für Q-DSL-Kunden */
     forwarders {
         213.148.129.10; 213.148.130.10;
     }

Abschließend muß noch eine entsprechende Zeile in die ``rc.conf``
eingefügt werden, damit der Namensserver u.a. auch beim Systemstart
gestartet wird und wir sagen, dass der Namensserver sich nur an
IPv4-Adressen binden soll: 

::

  # echo 'named_enable="YES"' >> /etc/rc.conf # echo 'named_flags="${named_flags} -4"' >> /etc/rc.conf

Damit nun auch der DNS-Cache auch von lokalen Programmen benutzt wird, muss die
Datei /etc/resolv.conf entsprechend angepasst werden:

::

   domain heimnetz
   nameserver 127.0.0.1

Jetzt können wir ``named`` manuell starten: 

::

  # /etc/rc.d/named start 
  wrote key file "/etc/namedb/rndc.key" 
  wrote key file "/var/named//etc/namedb/rndc.key" 
  Starting named.

Das Systemlog sollte dann etwa folgende Meldungen anzeigen 

::

  # tail /var/log/messages 
  Jan 25 23:41:20 rechnername named[14043]: starting BIND 9.3.0 -u bind -4 -t /var/named 
  Jan 25 23:41:20 rechnername named[14043]: command channel listening on 127.0.0.1#953

und wir können erste Namensauflösungen ausprobieren 

::

  # host bsdforen.de 
  bsdforen.de has address 212.204.60.79 bsdforen.de mail is handled by 20 bsdforen.de.

  # host localhost localhost.heimnetz has address 127.0.0.1

Es funktioniert. Bei jedem Rechner im Heimnetzwerk kann nun die IP
192.168.0.1 als DNS-Server bei den Netzwerkeinstellungen angegeben werden.

* :ref:`genindex`

Zuletzt geändert: |date|

