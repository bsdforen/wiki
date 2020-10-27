Crossbuilding FreeBSD
=====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel zeigt auf wie sowohl Kernel und Welt für verschiedene Rechner
und Architekturen von einem Rechner aus gebaut werden können. Wer den Artikel
lesen will sollte mit `Kapitel 8 des FreeBSD Handbuchs
<http://www.freebsd.org/doc/de/books/handbook/kernelconfig.html>`__ vertraut
sein.

Verschiedene Kernel mit ihren Welten verwalten
----------------------------------------------

Dem Vorschlag des FreeBSD Handbuchs folgend sollten Kernel
Konfigurationen im Verzeichnis ``/root/kernels`` gesammelt werden. Dort
können auch **make(1)** Einstellungen gesammelt werden, die auf einem
gewöhnlichen System in der Datei ``/etc/make.conf`` wären. Wenn ein
Kernel Konfiguration ``MYKERNEL`` genannt ist sollte die make
Konfiguration entsprechend ``MYKERNEL.mk`` genannt werden. In der Datei
``/root/kernels/MYKERNEL.mk`` können dann diverse Variablen wie
**CPUTYPE**, **CFLAGS**, **TARGET** oder **PORTS_MODULES** gesetzt
werden. Eine Auswahl und Beschreibung der Variablen findet sich in der
Manpage **make.conf(5)**. Die Beschreibung für **TARGET** befindet sich
in der Datei ``/usr/src/Makefile``.

In einem Netz mit mehreren Rechnern kann sich der Einsatz von
`Distcc </anwendungen/Distcc>`__ lohnen.

make Konfiguration einbinden
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Damit die passende make Konfiguration automatisch geladen wird, müssen
folgende Zeilen der ``make.conf`` hinzugefügt werden:

::

   .if exists(/root/kernels/${KERNCONF}.mk)
   .include "/root/kernels/${KERNCONF}.mk"
   .endif

.. warning::

  Die Verwendung von ``CPUTYPE?=`` in der Datei ``/etc/make.conf`` darf erst
  hinter diesem Block stattfinden, damit ein Kernel-spezifischer Aufruf von
  ``CPUTYPE?=`` Vorrang hat.

Hier ist ein Beispiel ``THINKPAD.mk`` für das Thinkpad des Autors, um
die Funktion der Datei zu verdeutlichen:

::

   CPUTYPE?=       pentium-m
   TARGET=         i386
   PORTS_MODULES=      net/iwi-firmware-kmod
   NO_PROFILE=     yes

Die Option **PORTS_MODULES** bereitet hier übrigens keine Probleme, da
die angegebenen Ports erst beim Aufruf von **installkernel** neu gebaut
werden. Hier ist allerdings vorsicht angesagt. Wenn einer der
angegebenen Ports scheitert, bricht die Installation ab und das
Kernelverzeichnis **/boot/kernels** ist unvollständig. Deshalb vorher
immer ein Backup machen.

OBJDIR nach Kernel trennen
~~~~~~~~~~~~~~~~~~~~~~~~~~

Beim Kompilieren von verschiedenen Kerneln und Welten entsteht das
Problem, dass alle in **/usr/obj** landen, was es verhindert
verschiedene Systeme vorrätig zu halten. Dem kann abgeholfen werden
indem der Bauvorgang folgendermaßen gestartet wird:

::

   # env MAKEOBJDIRPREFIX=/usr/obj/MYKERNEL make buildworld buildkernel KERNCONF=MYKERNEL

Diese Lösung ist allerdings umständlich. Aus dem Autor nicht
erfindlichen Gründen darf **MAKEOBJDIRPREFIX** nur auf diese Weise
gesetzt werden. Mit einem Trick kann das aber trotzdem in der
**make.conf** getan werden. Dazu wird einfach folgendes eingetragen:

::

   .if !make(dummy)
   .if defined(TARGET) && ${TARGET} != ${MACHINE}
   MAKEOBJDIRPREFIX?=  /usr/obj/${KERNCONF}
   .else
   MAKEOBJDIRPREFIX?=  /usr/obj/${KERNCONF}/${MACHINE}
   .endif
   .endif

So reicht bereits der Aufruf

::

   # make buildworld buildkernel KERNCONF=MYKERNEL

aus.

Gebaute Systeme verteilen
-------------------------

Die einfachste Möglichkeit solche fertigen Systeme zu verteilen ist über
NFS.

.. warning::

  Im folgenden gibt es mehrmals die Anweisung die Zeile
  ``nfslocking_enable="YES"`` in die Datei ``/etc/rc.conf`` aufzunehmen.  Ab
  FreeBSD 7 **entfällt** diese genau so wie das Kommando ``/etc/rc.d/nfslocking
  start``.

NFS Server einrichten
~~~~~~~~~~~~~~~~~~~~~

Der Rechner auf dem die Systeme gebaut werden muss die Verzeichnisse
**/usr/src** und **/usr/obj** freigeben. Dazu wird der Datei
**/etc/exports** folgende Zeile hinzugefügt:

::

   /usr/src /usr/obj -ro -maproot=root CLIENT

**CLIENT** muss mit der IP oder dem Hostnamen des Rechners der zugreifen
soll ersetzt werden. Wer volles vertrauen in die Clients hat kann den
Parameter **-ro** weglassen. Dann ist allerdings voller Schreibzugriff
möglich. Wenn PORTS_MODULES definiert ist, ist Schreibzugriff auf
/usr/obj obligatorisch.

In der Datei **/etc/rc.conf** müssen folgende Eintragungen vorgenommen
werden:

::

   nfs_server_enable="YES"
   nfslocking_enable="YES"
   mountd_flags="-r"
   rpc_statd_enable="YES"
   rpc_lockd_enable="YES"

Nun kann der NFS Server mit den Kommandos

::

   # /etc/rc.d/nfsd start
   # /etc/rc.d/nfslocking start

gestartet werden. Nach Änderungen an der **/etc/exports** sollte der
Befehl

::

   # /etc/rc.d/mountd onerestart

aufgerufen werden.

NFS Client einrichten
~~~~~~~~~~~~~~~~~~~~~

Beim Client Rechner müssen in der **/etc/rc.conf** folgende Einträge
gemacht werden:

::

   nfs_client_enable="YES"
   nfslocking_enable="YES"
   rpc_statd_enable="YES"
   rpc_lockd_enable="YES"

Zum Starten müssen nun die folgenden Kommandos ausgeführt werden:

::

   # /etc/rc.d/nfsclient start
   # /etc/rc.d/nfslocking start

Als nächstes sollten an der Datei **/etc/make.conf** die gleichen
Änderungen wie am Rechner der das System gebaut hat vorgenommen werden.
Außerdem sollte das Verzeichnis **/root/kernels** angelegt werden und
die Dateien für das zu installierende System enthalten.

Nun kann mit dem Befehl

::

   # showmount -e SERVER

ausprobiert werden ob die NFS Freigabe des Servers funktioniert hat.
SERVER ist dabei natürlich mit dem Hostnamen oder der IP des Servers zu
ersetzen.

Installation durchführen
~~~~~~~~~~~~~~~~~~~~~~~~

Zur Installation bleibt nur noch die Freigaben zu mounten. Der
Einfachheit halber sollten sie in die **/etc/fstab** eingetragen werden:

::

   SERVER:/usr/src /usr/src    nfs ro,-b,-T,-R=5   0   0
   SERVER:/usr/obj /usr/obj    nfs rw,-b,-T,-R=5   0   0

Nach vollendeter Installation sollte die Option noauto hinzugefügt
werden. Mit den Befehlen

::

   # mount /usr/src
   # mount /usr/obj

können die Freigaben sofort gemountet werden. Es ist wichtig, dass bei
jedem Aufruf von make auch **KERNCONF** angegeben wird. Alternativ kann
**KERNCONF** natürlich auch in der **/etc/make.conf** eingetragen
werden.

::

   # cd /usr/src
   # make installkernel KERNCONF=MYKERNEL
   # reboot

Hier im Loader den Single User Mode auswählen. Im Single User Mode muss
ersteinmal einiges gemountet und gestartet werden, damit über NFS
gearbeitet werden kann:

::

   # mount -a
   # /etc/rc.d/netif start INTERFACE
   # /etc/rc.d/nfsclient start
   # /etc/rc.d/nfslocking start
   # mount -a

**INTERFACE**' muss natürlich durch die Inteface Bezeichnung der
Netzwerkkarte ersetzt werden (zum Beispiel fxp0). Mit dem Befehl
**mount** kann geprüft werden ob alle benötigten Verzeichnisse
tatsächlich gemountet sind.

::

   # cd /usr/src
   # mergemaster -p
   # make installworld KERNCONF=MYKERNEL
   # mergemaster
   # reboot

Verweise
--------

-  Ein eigener Kernel im `FreeBSD
   Handbuch <http://www.freebsd.org/doc/de/books/handbook/kernelconfig.html>`__.
-  Die Manpage **make(1)**.
-  Die Manpage **make.conf(5)** und **src.conf(5)** für **MYKERNEL.mk**.
-  `Distcc </anwendungen/Distcc>`__ für verteiltes Kompilieren.
-  Die Manpage **exports(5)** für die Konfiguration des NFS Servers.
-  Die Manpage **rc.conf(5)**.
-  Die Manpage **build(7)** für das Bauen von Welt und Kernel.
-  Auch Lesenswert sind die ersten 69 Zeilen von **/usr/src/Makefile**.

* :ref:`genindex`

Zuletzt geändert: |date|

