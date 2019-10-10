Ezjail
======

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel beschreibt, wie man sehr leicht und schnell `Jails
</howto/Jails>`__ mit `sysutils/ezjail
<https://www.google.com/search?q=sysutils/ezjail&btnI=lucky>`__ erstellen kann.

Einleitung
----------

Der Vorteil von ezjail ist die äußerst einfache Erstellung und das
Updaten der Jails. Außerdem sind die frisch erstellten Jails gerade mal
~1.5 MB groß, da alle Verzeichnisse nur aus einer Basis-Jail verlinkt
werden.

Nach der Installation mit ``portinstall``, ``pkg_add –r`` oder einer
anderen Installationmethode der Wahl und einem ``rehash``, stehen
``ezjail-admin``, ``ezjail.sh`` und die manpages zur Verfügung.

Basisjail generieren
--------------------

Mit

::

  # ezjail-admin update -bp

wird erst einmal die Basis-Jail erstellt. (Dies kann je nach
Maschinenleistung ein wenig dauern (~1-3h), da als erstes ``make world``
aufgerufen wird – wer sein Hostsystem schon upgedated hat, kann das mit
der Option –i statt -b übergehen und den Vorgang deutlich
beschleunigen).

Jails erstellen
---------------

Es gibt mehrere Möglichkeiten seine Jails zu erstellen. Die üblichere
und einfachere ist es für jedes Jail eine eigene, extern zugreifbare, IP
zu verwenden. Die zweite Methode vergibt den Jails "nicht route-bare
Adressen" und leitet nur bestimmte Ports an die Jails weiter.

Routebare IPs
~~~~~~~~~~~~~

Nachdem die Basisjail generiert wurde, lassen sich mit

::

  # ezjail-admin create <Jailname> <JailIP>

die Jails erstellen z.B.:

::

  # ezjail-admin create web1 192.168.2.101 
  # ezjail-admin create web2 192.168.2.102

Die IPs sollten schon vorher den Interface(s) zugewiesen werden. Die
Netzwerkkarte fxp0 ist hier als Beispiel gewählt. Mit der Eingabe von

::

  # ifconfig

läßt sich die benutzte Netzwerkkarte ermitteln (z.B. xl0, rl0, sk0 und
Andere):

::

  # ifconfig fxp0 alias 192.168.2.101 netmask 255.255.255.255 
  # ifconfig fxp0 alias 192.168.2.102 netmask 255.255.255.255

Um beim späteren Hochfahren des Systems die IPs verfügbar zu haben,
erfolgt in der ``/etc/rc.conf`` noch der folgende Eintrag:

::

   ifconfig_fxp0_alias0="inet 192.168.2.101 netmask 255.255.255.255"
   ifconfig_fxp0_alias1="inet 192.168.2.102 netmask 255.255.255.255"

Nach dem Kopieren der Dateien (ezjail-admin create ...) wird man
netterweise gewarnt, wenn irgendwelche Dienste laufen, die sich an alle
IPs des Systems binden, wie z.B. bind oder sendmail. Mehr dazu auch in
`ASGs Jail-howto <http://www.grunix.de/dokumentationen/jails.html>`__
unter „Problematische Dienste“.

nur interne IPs
~~~~~~~~~~~~~~~

Bei dieser Methode werden IPs aus dem lokalen 127.0.0.1/8 Netz
verwendet.

Auch hier werden es die Jails mittels::

  # ezjail-admin create web1 127.0.0.10

erstellt.

Statt fxp0 nimmt man hier lo0::

  # ifconfig lo0 alias 127.0.0.10 netmask 255.255.255.255

und trägt auch das Alias auf dem Loopback-Interface in die /etc/rc.conf
ein:

::

   ifconfig_lo0_alias0="inet 127.0.0.10 netmask 255.255.255.255"

Anders als bei der ersten Methode müssen noch die Ports per pf an die
entsprechenden Jails weiter geleitet werden. In der pf.conf sind dafür
folgende Zeilen nötig:

::

   nat on $ext_if proto {tcp udp icmp} from lo0 to any -> ($ext_if)

Hier wird dafür gesorgt, dass wir in den Jails Zugang zu externen
Netzwerken haben.

::

   rdr on $ext_if proto tcp from any to any port 80 -> 127.0.0.10
   pass in on $ext_if inet proto tcp from any to 127.0.0.10 port 80

Für andere Dienste muss der Port angepasst werden.

Jails starten
-------------

Mit

::

  # /usr/local/etc/rc.d/ezjail.sh start 

oder 

::

  # ezjail-admin start

lassen sich die so erstellten Jails nun Starten, Stoppen und Neustarten.
Als Argument nimmt ``ezjail.sh`` den Namen entweder einer oder mehrerer
Jails. Wenn keine Argumente angegeben werden, sind alle Jails betroffen.

Um die Jails automatisch bei Systemstart zu Starten genügt der Eintrag:

::

   ezjail_enable="YES“

in der ``/etc/rc.conf`` des Hostsystems.

Jails updaten
-------------

Mit 

::

  # ezjail-admin update -i

lassen sich dann nach dem nächsten Update des Hostsystems alle Jails bequem auf
einmal aktualisieren.

Jails löschen
-------------

Mit 

::

  # ezjail-admin delete web2

wird die in unserem Beispiel die 2.Jail gelöscht. Die Dateien bleiben
allerdings nach wie vor auf der Platte, sodaß mit ein **ezjail-admin
create** mit der Option **-x**

::

  # ezjail-admin create -x web2 192.168.2.102 

die Jail wieder hergestellt.

Mit der Option **-w**

::

  # ezjail-admin delete -w web2

wird die Jail web2 endgültig gelöscht.

Weiterführende Themen
---------------------

Anwendungsbeispiel - Apache
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Um nun zum Beispiel einen Webserver in einer Jail zu starten, sollten
noch einige Dateien der Jail angepasst werden:

in der ``/usr/jails/web1/etc/resolv.conf``

::

   nameserver <IP-des-DNS-Servers>

und damit der apache automatisch startet in
``/usr/jails/web1/etc/rc.conf``

::

   apache_enable=“YES“

danach sollte sich vom Hostsystem mit ::

  # jexec JID pkg_add –r apache

der Webserver installieren lassen. Die JID lässst sich mit ``jls``
herausfinden.

Beim nächsten Neustart läuft der apache nun in einer Jail.

Portstree des Basejails aktualisieren
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

  # ezjail-admin update -P

Portstree in einer Jail
~~~~~~~~~~~~~~~~~~~~~~~

Bei der oben genannten Beschreibung zur Einrichtung einer oder mehrerer
Jails wird kein Portstree in den Jails angelegt. Es gibt zwei
Möglichkeiten die Ports auch in die Jails zu bringen:

1. Eine Kopie des vorhandenen Portstrees wird in die Jails kopiert
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bei der Anlage der Jail wird die Option **-P** angegeben::

  # ezjail-admin update -P web1 192.168.2.101

Hiermit wird eine 1:1 Kopie des Portstrees der Hostsystems in dem Jail
angelegt. Diese Variante benötigt naturgemäß mehr Speicher auf der
Festplatte als die 2. Lösung.

2. Mittels ``nullfs`` kann auf den original-Portstree zugegriffen werden
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In der Datei **/etc/fstab.<Name der Jail>** des Hostsystems muß
folgender Eintrag hinzugefügt werden:

::

   /usr/ports           /usr/jails/<Name der Jail>/usr/ports     nullfs ro 0 0

In unserem Beispiel würde in der Datei **/etc/fstab.web1** dann
folgendes stehen:

::

   /usr/jails/basejail  /usr/jails/web1/basejail    nullfs ro 0 0
   /usr/ports           /usr/jails/mail/web1/ports  nullfs ro 0 0

In der web1-Jail muß nun im Pfad **/usr** ein leerer Ordner names
**ports** vorhanden sein. Ist dies nicht der Fall muß er per Hand
angelegt werden. Es kann auch sein, daß ein symbolischer Link auf
**basejail/usr/ports** vorhanden ist. Dieser muß mittels ``rm ports``
gelöscht werden. Dann ist ebenfalls ein Ordner **ports** anzulegen.

Sollte die web1-Jail laufen, muß sie nun gestoppt und erneut gestartet
werden::

  # /usr/local/etc/rc.d/ezjail.sh restart

Nun sollte noch überprüft werden, ob in der web1-Jail im
**/etc/make.conf** die Angabe WRKDIRPREFIX gesetzt ist. Dies sollte
normalerweise automatisch durch den ``ezjail-admin`` passiert sein. Dies
ist notwendig, weil der Ordner oben als ``read only`` gemountet wurde
und somit kein Schreibzugriff erlaubt ist.

Problembeseitigung
------------------

mount_nullfs: Operation not supported by device
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Problem:** Das Kernelmodul nullfs fehlt bzw. wurde nicht geladen. Beim
generischen Kernel sollte diese Meldung nicht auftreten, da dieses Modul
im Kernel standardgemäß eingebunden ist. Dies kann passieren, wenn man
einen "recht schlanken" Custom-Kernel benutzt.

**Lösung:** In /etc/make.conf unter MODULES_OVERRIDE= **nullfs**
eintragen und erneut den Kernel bauen.

Siehe auch
----------

-  `Jails </howto/Jails>`__
-  http://erdgeist.org/arts/software/ezjail/
-  http://www.grunix.de/dokumentationen/jails.html

Quellen
-------

Dieser Artikel basiert auf Mangas Forumseintrag in
http://www.bsdforen.de.

* :ref:`genindex`

Zuletzt geändert: |date|

