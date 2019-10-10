ISO-Images mounten
==================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

Dieser Artikel soll erklären wie man unter den verschiedenen BSDs ISO-Images
mountet.

Einleitung
----------

In diesem Beispiel wird ein ISO-Image z.B. **/tmp/LiveCD.iso** unter
**/mnt/iso** eingebunden/gemountet werden. Im wesentlichen unterscheidet
man hierfür zwei Verfahren. Dies sind **vnd/vnconfig** und
**md/mdconfig**. md/mdconfig ersetzt ab FreeBSD 5 ersatzlos das in den
anderen BSD vorhanden vnd/vnconfig. FreeBSD-Versionen **vor** 5
verwenden
`vn/vnconfig <http://www.freebsd.org/cgi/man.cgi?query=vnconfig&apropos=0&sektion=0&manpath=FreeBSD+4.11-RELEASE&format=html>`__.

DragonFly BSD
-------------

Da DragonFly BSD aus dem 4er-Zweig von FreeBSD entstanden ist verwendet
es
`vn/vnconfig <http://leaf.dragonflybsd.org/cgi/web-man?command=vn&section=ANY>`__.
Zur Nutzung sei hier auf den unten stehenden NetBSD-Abschnitt verwiesen.

FreeBSD
-------

Die einfachste Methode unter **FreeBSD > 5.x** ein ISO-Image zu mounten
ist, es als Memory Disk Device einzurichten. Dafür gibt es seit FreeBSD
5.x das Programm
`mdconfig <http://www.freebsd.org/cgi/man.cgi?query=mdconfig>`__, das
Devices mit dem Namen /dev/md[n] erzeugt.

mount
~~~~~

Zuerst erzeugen wir also mit

::

   # mdconfig -a -t vnode -f /tmp/LiveCD.iso -u 0

ein Device angelegt. Der parameter **-a** steht für **attach**, **-t
vnode** bedeutet das das Device aus einer Datei erzeugt wird, **-f
filename** gibt die Datei an und **-u [n]** gibt die Nummer des Devices
an.

Jetzt kann man das Device mit

::

   # mount -r -t cd9660 /dev/md0 /mnt/iso

mounten.

umount
~~~~~~

Um das Device wieder zu entfernen muss man zuerst das Image unmounten.

::

   # umount /mnt/iso

Dann kann man mit

::

   # mdconfig -d -u 0

das Device wieder entfernen.

NetBSD
------

Angelehnt an FreeBSD bis 4.x funktioniert das mounten von ISO-Images
unter NetBSD mit dem Befehl
`vnconfig <http://netbsd.gw.com/cgi-bin/man-cgi?vnconfig++NetBSD-current>`__.

.. _mount-1:

mount
~~~~~

Das Device erzeugen:

::

   # vnconfig -c /dev/vnd0d /tmp/LiveCD.iso

Das Device mounten:

::

   # mount -t cd9660 /dev/vnd0d /mnt/iso

Der Befehl

::

   # ls -la /mnt/iso

sollte jetzt den Inhalt des Image anzeigen.

.. note::

  Das Disklabel **d** funktioniert beim Port i386. Bei anderen Ports kann es
  abweichend auch **c** oder ein anderer Buchstabe sein.

.. _umount-1:

umount
~~~~~~

::

   # umount /mnt/iso
   # vnconfig -u /dev/vnd0d

Mehr Infos mit "man vnconfig" unter EXAMPLES...

OpenBSD
-------

Verwendet ebenfals vnd/vnconfig. Es bietet als einziges BSD aber die
Erweiterung
`svnd/vnconfig <http://www.openbsd.org/cgi-bin/man.cgi?query=svnd&apropos=0&sektion=0&manpath=OpenBSD+Current&arch=i386&format=html>`__.
**svnd** erlaubt es, im Gegensatz zu vnd, ISO-Images zu mounten die
nicht lokal vorliegen. Die Nutzung von svnd ist entsprechend derer von
vnd. Es sei somit auf den obenliegenden NetBSD-Abschnitt verwiesen.

Verweise
--------

-  `ISO-Images von optischen Medien erzeugen </howto/ISO-Images von optischen Medien erzeugen>`__
-  `ISO-Dateien automatisch mounten </howto/ISO-Dateien automatisch mounten>`__

* :ref:`genindex`

Zuletzt geändert: |date|

