FreeBSD Boot Manager
====================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

.. warning::

  Achtung! Die hier beschriebenen Anleitungen betreffen einen sehr wichtigen
  Teil des Bootvorgangs. Wenn Ihr nicht im entferntesten wisst wovon hier die
  Rede ist, dann probiert es auf jeden Fall nicht blind aus. Im schlimmsten
  Fall droht Datenverlust! Es besteht keine Haftung für Schäden!

Basisinformationen
------------------

Der von FreeBSD mitgelieferte Bootmanager hat einen Vorteil, aber auch
einen Nachteil.

**Der Vorteil ist:** Er ist so klein, daß im Gegensatz zu anderen
Bootmanagern, keine Extra-Partition nur für den Bootmanager angelegt
werden muß.

**Der Nachteil ist:** Er kann nur Betriebssysteme von der aktuell als
Bootlaufwerk eingestellten Festplatte, von einer beliebigen Partition
hochfahren. Das heisst konkret: Will man FreeBSD auf einer Slave-Platte
(bei IDE) oder einer Nicht-Bootplatte installieren, muss man den
FreeBSD-Bootmanager auch auf die andere Platte installieren (siehe dazu:
*boot0cfg*). Beim Booten kann man dann die zweite Platte (oder genauer
gesagt den zweiten FreeBSD-Bootmanager) vom ersten FreeBSD-Bootmanager
mit <F5> auswählen. Dies kann man auch mehrfach machen. Es ergibt sich
dann eine Kette von FreeBSD-Bootmanagern. Man kann sogar dafür sorgen,
dass ein Betriebssystem von der zweiten oder noch weiteren Platte
automatisch bootet. Man wählt dann in den entsprechenden
FreeBSD-Bootmanagern 5 als Standardeinstellung und am Ende der Kette
dann die Slice, die gebootet werden soll.

Optional ist es möglich, im Rechner-BIOS die entsprechende Festplatte
bei jedem Start als Bootplatte anzugeben.

Erneute Installation des Boot Managers im MBR
---------------------------------------------

Manchmal ist es sinnvoll den FreeBSD Boot Manager noch einmal neu zu
installieren. Ich mag zum Beispiel nicht, dass dieser sich merkt, was
ich als letztes gestartet habe und dies als nächste Standardeinstellung
nimmt. Zweitens mag ich auch nicht, dass ich so lange warten muss.

Abhilfe gibt es mit diesem Befehl (bitte vorsichtig und nochmal
nachschauen ob Ihr das wirklich so wollt!):

::

   # boot0cfg -B -v -f /boot/oldmbr.bak -o packet,noupdate -s <slicenr> -t 50 /dev/<bootdev>

-  Bitte ersetzt <slicenr> durch die Slice, die Ihr standardmäßig zum
   Booten ausgewählt haben wollt.
-  Bei "-t" steht eine 50. Das bedeutet *50 Ticks*, wobei 18 Ticks ca. 1
   Sekunde bedeuten.
-  Als <bootdev> müsst Ihr natürlich die Platte wählen, die gebootet
   wird und wo der FreeBSD Boot Manager drauf soll.

Kooperation mit MS Windows Vista
--------------------------------

Hier ein kurzes HowTo, mit welchem man Microsoft Windows Vista mit dem
FreeBSD Boot Manager zum Laufen kriegen kann.

Ausgangssituation
~~~~~~~~~~~~~~~~~

Wenn man FreeBSD installiert, dann installiert man oft blauäugig den
FreeBSD Boot Manager mit, weil man das System ja nutzen will und es ja
früher immer gut geklappt hat, mit dem MS-Windows booten. Microsoft hat
aber den Bootvorgang bei Windows Vista modifiziert. Ich will hier nicht
erörtern ob das gut oder schlecht ist.

Fakt ist, dass man bei der Auswahl von "DOS" unter dem FreeBSD Boot
Manager kurz danach einen blauen Bildschirm erhält, wo man sehen kann
dass WINLOAD.EXE nicht gefunden worden ist ("WINLOAD.EXE is missing").

Keine Panik! Das sieht schlimmer aus als es ist.

Lösung
~~~~~~

#. Von Microsoft-Windows-Vista-DVD starten (also von der DVD booten).
#. Auf die Reparaturkonsole wechseln.
#. Folgendes eintippen:

::

   > c:
   > bcdedit /store \boot\bcd /set {bootmgr} device boot
   > bcdedit /store \boot\bcd /set {default} device boot
   > bcdedit /store \boot\bcd /set {default} osdevice boot

Fertig! Beim nächsten Booten geht wieder beides: FreeBSD und Microsoft
Windows Vista.

* :ref:`genindex`

Zuletzt geändert: |date|

