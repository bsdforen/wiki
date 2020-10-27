Windows-Freigabe
================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-netbsd.png

Dieser Artikel erklärt den Zugriff auf Windows-Freigaben.

mount_smbfs
-----------

.. note::

  Der Befehl ``mount_smbfs`` ist unter `FreeBSD </FreeBSD>`__ verfügbar, nicht
  aber unter `OpenBSD </OpenBSD>`__. Unter OpenBSD findet stattdessen
  Sharity-Light oder smbclient Anwendung, was dann so ähnlich wie ein
  FTP-Client wäre.  

Windows-Freigabe einbinden
~~~~~~~~~~~~~~~~~~~~~~~~~~

Wenn von einer BSD-Maschine auf eine Windows-Freigabe zugegriffen werden
soll, kann man dies tun mit ``mount_smbfs``. Hier dazu ein Beispiel:

::

   # mount_smbfs -I 192.168.1.22 -W <Workgroup> //user@rechnername/data /mnt/windows

-  Mit -I gibt man die IP-Adresse des Windows-Rechners an, auf dem sich
   die Freigabe befindet. Die IP kann auch durch einen gültigen Hostname
   ersetzt werden.
-  Mit -W gibt man die `Workgroup </kompendium/Workgroup>`__
   (Arbeitsgruppe) an, in der sich der Windows-Rechner befindet
   (*optional*).
-  Anschliessend folgt der Pfad zur Freigabe, die eingebunden werden
   soll. Hierbei ist ``user`` ein auf dem Windows-Rechner gültiger und
   mit den nötigen Rechten ausgestatteter Benutzer, unter dessen
   Verwendung die Freigabe eingebunden werden soll. Nach dem 'at' (@)
   folgt der NetBIOS-Name des Windows-Rechners (hier ``rechnername``).
   Herauszufinden ist dieser über die Eigenschaften des Arbeitsplatzes
   und der Lasche 'Computername'. Zu guter letzt folgt der Name der
   Freigabe, hier ``data`` genannt.
-  Zum Schluss muss der Pfad angegeben werden, in welchen die
   Windows-Freigabe eingebunden werden soll. Hier werden die Freigaben
   unter ``/mnt/windows`` verfügbar gemacht.

Windows-Freigabe ausbinden
~~~~~~~~~~~~~~~~~~~~~~~~~~

Wenn die Windows-Freigabe wieder ausgebunden werden soll, tut man dies
wie gewohnt mit **umount(8)**.

::

   # umount //USER@RECHNERNAME/DATA

oder einfacher

::

   # umount /mnt/windows

Sharity-Light
-------------

<note important> In der ``/etc/hosts`` müssen die Hosts, auf die man
zugreifen möchte, eingetragen sein. </note>

Eine andere Möglichkeit Windows-Freigaben unter \*BSD (und allgemein
unter Unix) zu mounten, ist Sharity-Light zu verwenden. Es ist im
Endeffekt das gleiche wie `Samba <Samba>`__. Wenn nur auf
Windows-Verzeichnisse zugegriffen werden soll, ist Sharity-Light wohl
die beste Lösung. Wenn man Verzeichnisse von der BSD-Maschine (für
Windows) freigegeben will, muss man `Samba <Samba>`__ installieren (man
kann auch `NFS <NFS>`__ nehmen, aber es gibt soweit ich weiß keinen
freien NFS-Client für Windows, zumindest habe ich noch keinen gefunden).
Unter `Unix </kompendium/Unix>`__ selber braucht man kein
`Samba <Samba>`__, da genügt `NFS <NFS>`__; der Client dazu ist unter
`Unix </kompendium/Unix>`__ natürlich umsonst. Bei Sharity-Light werden
die Passwörter unverschlüsselt über das Netz gesendet.

Sharity-Light ist in den FreeBSD Ports unter
`net/sharity-light <https://www.google.com/search?q=net/sharity-light&btnI=lucky>`__
und in Pkgsrc unter
`net/sharity-light <https://www.google.com/search?q=net/sharity-light&btnI=lucky>`__
zu finden.

Freigabe einbinden
~~~~~~~~~~~~~~~~~~

::

   # shlight //192.168.1.22/data /mnt/windows -U user

-  Zuerst die IP des Windows-Rechners + Freigabe angeben
-  Zielverzeichnis auf eurem Rechner
-  den User muss mit -U angeben werdeb, ansonsten will er sich mit
   `root </kompendium/root>`__ anmelden
-  als letztes folgt die Abfrage des Passwortes

Freigabe ausbinden
~~~~~~~~~~~~~~~~~~

Einfach das Verzeichnis angeben, in welches das Share gemountet wurde.

::

   # unshlight /mnt/windows/

Weblinks
--------

-  `Sharity-Light <http://www.obdev.at/products/sharity-light/>`__
-  `Samba <http://www.samba.org>`__
-  `net/sharity-light <https://www.google.com/search?q=net/sharity-light&btnI=lucky>`__
   in den FreeBSD Ports
-  `net/sharity-light <https://www.google.com/search?q=net/sharity-light&btnI=lucky>`__
   in Pkgsrc

* :ref:`genindex`

Zuletzt geändert: |date|

