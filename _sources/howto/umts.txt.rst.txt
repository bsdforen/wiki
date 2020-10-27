UMTS
====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

In diesem Artikel will ich euch zeigen, wie ihr mit einer Novatel Merlin 630
unter FreeBSD surfen könnt. Die Karte kann UMTS und GPRS und ich habe sie von
E-Plus. Ich habe die Karte unter FreeBSD 6.1 und 6.2 erfolgreich getestet. Mir
kommt es irgendwie vor das UMTS unter FreeBSD 6.2 schneller ist, aber das kann
auch an anderen Ursachen liegen.

Kernel patchen
--------------

Leider müssen wir eine kleine Veränderung am Kernel vornehmen. Wir
müssen die Datei **/usr/src/sys/dev/sio/sio.c** in der Zeile 1929 wie
folgt ändern:

::

   cp4ticks = speed / 10 / hz * 4;

in

::

   cp4ticks = speed / 10 / 100 * 40;

Nachdem wir das geändert haben müssen wir den Kernel neu compilieren und
booten. Wie das geht, steht
`hier <http://wiki.bsdforen.de/Kernel_erstellen_(FreeBSD)>`__ ganz gut.

Karte initialisieren
--------------------

Zunächst müssen wir auch schauen, ob die Karte richtig erkannt wurde.
Dazu stecken wir sie in den PCMCIA-Slot und schauen uns die Datei
**/var/log/messages** an. Dabei müsste sowas erscheinen:

::

   pccard0: Allocation failed for cfe 7
   sio4: at port 0×2f8-0×2ff irq 11 function 0 config 15 on pccard0
   sio4: type 16550A
   sio4: unable to activate interrupt in fast mode - using normal mode
   pccard0: Allocation failed for cfe 7
   pccard0: Allocation failed for cfe 15
   sio5: at port 0×2e8-0×2ef irq 11 function 1 config 23 on pccard0
   sio5: type 16550A
   sio5: unable to activate interrupt in fast mode - using normal mode

Dann wurde alles einwandfrei erkannt. Bitte versucht nicht das Device
sio4 oder ein anderes zu suchen, so wie ich, denn die heissen anders und
zwar cuadX. Jetzt dürfte eure Karte vor sich hin blinken und zwar ROT.

PIN übergeben
-------------

Als nächstes müssen wir der Karte den PIN übergeben und das machen wir
mit einem Perl script (es gibt auch noch eine andere möglichkeit, aber
die habe ich nicht erfolgreich getestet). Hier das Script:

:: 

   #!/usr/bin/perl
   use strict;
   use warnings;my $modem = “/dev/cuad4″;
   # Substitute xxxx with your PIN.
   # You should probably put your pin somewhere else, e.g. on an USB stick,
   # an encrypted file system or something else, and read it from there…
   # You have been warned!
   my $pin = “XXXX”;$SIG{ALRM} = sub {
       die(”timeout: no response from modem $modem\n”);
   };
   open(MODEM, "+<", $modem) or die("can't open modem $modem");
   alarm(10);
   print(MODEM "AT+CPIN=\"$pin\"\n\r");
   while () {
       if (m/OK/) {
           close(MODEM);
           print("PIN accepted\n");
           exit(0);
       }
       if (m/ERROR/) {
           close(MODEM);
           print("PIN rejected\n");
           exit(1);
       }
   }

Das Device an sich sollte bei euch auch das gleiche sein, falls nicht
einfach anpassen und euren PIN dort eingeben. Ist zwar nicht das tollste
da es in Klartext da steht, aber man kann nicht alles haben… Wenn ihr
das Perl script jetzt ausführ sollte ein “PIN accepted” kommen. Falls
nicht schaut dochmal ob die SIM-Karte Richtig herum in der Karte steckt
face-wink.png Jetzt sollte die Karte anfangen Blau zu pingen (falls ihr
in einem UMTS-Gebiet seid).

PPP-Konfigurieren
-----------------

Jetzt müssen wir ppp konfigurieren und zwar benutzen wir das auf
Userland-Ebene, also müssen wir die **/etc/ppp/ppp.conf** wie folgt
anpassen:

::

  3g:
  set ifaddr 10.253.253.253/0 10.253.253.252/0 255.255.255.255
  set log local chat error warning connect
  set dial “ABORT ERROR ABORT BUSY ABORT NO\sCARRIER TIMEOUT 8 \
  -AT+cops?;+csq;+creg?;+cgreg?-OK ATe0v1&d2&c1 OK AT+cgdcont=1,\”IP\”,\”internet.eplus.de\”,\”\”,1,1 OK \dATDT\T TIMEOUT 60 CONNECT”
  set cd 30!
  set crtscts on
  set timeout 0
  set reconnect 3 90
  set speed 115200
  set vj slotcomp off
  disable mppe shortseq ipv6cp vjcomp pred1 deflate24 acfcomp protocomp
  deny mppe deflate24 pred1 protocomp
  set login
  set device /dev/cuad4
  set authname eplus
  set authkey gprs
  set phone “*99*1#”
  add default HISADDR

Wenn ihr einen anderen Anbieter als Eplus benutzt müsst ihr natürlich
den Authnamen und die Nummer ändern.

UMTS Einwählen
--------------

Jetzt ist es auch schon soweit und wir können uns einwählen. Das erreichen wir durch einen simplen Befehl

::

  # ppp 3g

jetzt befinden wir uns in der “ppp console” und müssen noch “dial” einegeben, damit wir uns verbinden: 

::

  PPP ON zlaptop> dial

daraufhin sollte etwas ähnliches erscheinen wie folgendes:

::

  ppp ON zlaptop> dial
  Chat: Phone: *99*1#
  Chat: deflink: Dial attempt 1 of 1
  Chat: Expect(8):
  ppp ON zlaptop> Chat: Expect timeout
  Chat: Send: AT+cops?;+csq;+creg?;+cgreg?
  Chat: Expect(8): OK
  Chat: Received: AT+cops?;+csq;+creg?;+cgreg?
  Chat: Received: +COPS: 0,0,”E-Plus”,3
  Chat: Received:
  Chat: Received: +CSQ: 18,99
  Chat: Received:
  Chat: Received: +CREG: 0,1
  Chat: Received:
  Chat: Received: +CGREG: 0,0
  Chat: Received:
  Chat: Received: OK
  Chat: Send: ATe0v1&d2&c1
  Chat: Expect(8): OK
  Chat: Received: ATe0v1&d2&c1
  Chat: Received: OK
  Chat: Send: AT+cgdcont=1,”IP”,”internet.eplus.de”,”",1,1
  Chat: Expect(8): OK
  Chat: Received:
  Chat: Received: OK
  Chat: "Send: ATDT*99***1#"
  Chat: Expect(60): CONNECT
  Chat: Received:
  Chat: Received: CONNECT
  Ppp ON zlaptop>
  PPp ON zlaptop>
  PPP ON zlaptop>

Falls es keinen Fehler gab, können wir noch schauen, ob wir eine IP bekommen haben (kann so ~15 Sekunden dauern):

::

  [steffen@zlaptop umts]$ ifconfig tun0
  tun0: flags=8051 mtu 1500
  inet 10.161.xxx.xx –> 10.253.253.252 netmask 0xffffffff
  Opened by PID 656

jetzt könnt ihr schon anfangen und wild in der Gegend herumpingen. 

Tipp
----

Ich hatte das Problem, daß meine DNS-Server irgendwie über UMTS nicht so
ganz wollten und habe deshalb zwei DNS-Server direkt von eplus genommen
und jetzt funktioniert alles. Dazu tragen wir in der
**/etc/resolv.conf** folgendes ein:

::

   nameserver 212.23.97.2
   nameserver 212.23.97.3

Wenn das so geklappt hat, seid ihr mit UMTS online und ich wünsche euch
Viel Spaß…

Siehe auch
----------

-  `umts_mit_huawei </howto/umts_mit_huawei>`__
-  `umts_mit_sunrise_und_swisscom </howto/umts_mit_sunrise_und_swisscom>`__

* :ref:`genindex`

Zuletzt geändert: |date|

