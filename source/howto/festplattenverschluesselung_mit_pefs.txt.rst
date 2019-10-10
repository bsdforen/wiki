Festplattenverschlüsselung mit pefs
===================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png


``pefs`` stellt eine stapelbare Dateisystemverschlüsselung zur
Verfügung. Damit ist möglich, ohne weiteres mit Hilfe eines Passworts,
eine kryptografische Transformation von Daten und Dateinamen zu
schaffen. ``pefs`` ist also sozusagen eine ``encfs``-Alternative im
BSD-Stil (inkompatibel mit ``encfs``, das von Linux her bekannt ist).

Ergänzt wird ``pefs`` durch das Modul ``pam_pefs``, welches erlaubt, das
Passwort an PAM beim Login zu übergeben und so den ``pefs``-Stapel zu
initialisieren.

Vorteile gegenüber anderen Verschlüsselungsmethoden
---------------------------------------------------

#. Der Rechner kann bis zum Login gestartet werden und kann hiermit ohne
   Hindernisse für andere Nutzer zur Verfügung stehen, die kein Passwort
   für die persönlichen verschlüsselten Daten haben.
#. Man ist in der Lage eine Sicherung der verschlüsselten Daten
   durchzuführen (zum Beispiel mit ``dump``, ohne das Dateisystem selbst
   zu entschlüsseln.
#. Wird man aufgefordert, dass Passwort für das Notebook auszugeben,
   kann man seine Daten schützen, indem man einen Pseudonutzer anlegt,
   den man herausgibt und nicht den echten.

Vorgehen
--------

Ich erkläre hier wie man es mit ``pefs`` und ``pam_pefs`` schafft, sein
``$HOME`` für sich persönlich zu verschlüsseln. Man kann diese Anleitung
natürlich beliebig variieren, je nach Geschmack.

Diese Anleitung geht davon aus, dass Euer komplettes ``$HOME`` beim
Einloggen zur Verfügung gestellt werden soll. ``pam_pefs`` funktioniert
auch nur in diesem Fall. Man kann nicht automatisch ein anderes
Verzeichnis als sein vordefiniertes ``$HOME`` damit behandeln.

Man hat hier also die Möglichkeit ``zwei Passwörter`` zu definieren.
Bitte denkt daran, dass PEFS das Passwort, dass bei ``addchain``
angegeben worden ist braucht, um das Dateisystem zu entschlüsseln. Euer
Login-Passwort tut das nicht. Ihr könnt Euch dennoch mit dem
Login-Passwort einloggen, aber nur dann, wenn mindestens ein Mal das
PEFS-Passwort (beim Login für PAM) eingebeben worden ist.

Die Schritte muss man natürlich aus Administrator durchführen.

Vorsichtsmaßnahmen
------------------

Bitte beachtet, dass es um Daten und Dateien geht. Macht unbedingt eine
Sicherung. Verlust von wichtigen Daten schmerzt immer. Ihr seid hiermit
gewarnt, dass man nicht einfach blind auf meine Anleitung verlassen
sollte und dass man leicht Fehler machen kann, die zum Datenverlust
führen können.

Installation
------------

``pefs`` installiert man mit dem Port
`sysutils/pefs-kmod <https://www.google.com/search?q=sysutils/pefs-kmod&btnI=lucky>`__.

::

   portmaster -d sysutils/pefs-kmod

Optional kann man AESNI beim Kernel dazuladen. Es nützt aber nur, wenn
man eine CPU hat, die AESNI unterstützt.

In ``/boot/loader.conf``:

::

   aesni_load="YES"

$HOME vorbereiten
-----------------

Bitte hier genau aufpassen. Schaut bitte auch was tatsächlich Euer
``$HOME`` ist. Bei einigen FreeBSD-Installationen ist es oft
``/usr/home/myuser``. Ich benutze hier ``/home/myuser``. PEFS geht hier
ziemlich exakt vor und wird sich weigern zu funktionieren, wenn Euer
Benutzerverzeichnis ``/home/myuser`` ist und das mit Softlinks
realisiert wurde, sodass eigentlich Euer $HOME physikalisch auf
``/usr/home/myuser`` liegt. Am Besten mit "vipw" das $HOME auf
``/usr/home/myuser`` ändern.

::

   cd /
   mv /home/myuser /home/myuser.bak
   mkdir /home/myuser

Nun wird die verschlüsselte Schicht mit einem leeren Keychain erstellt
und ein Passwort angegeben.

::

   pefs addchain -fZ -a aes256-xts /home/myuser
   pefs mount /home/myuser /home/myuser

(Ihr könnt natürlich einen anderen Algorithmus mit ``-a`` bei addchain
angeben.)

Jetzt testet man, ob alles korrekt funktioniert:

::

   pefs mount /home/myuser /home/myuser
   pefs addkey -c /home/myuser

Sollte alles OK sein, dann muss der Schlüssel für PAM zugreifbar gemacht
werden. Dazu setzt man nur die Permissions korrekt:

::

   chown myuser:myuser /home/myuser/.pefs.db

Daten auf das verschlüsselte $HOME kopieren
-------------------------------------------

Bitte beachtet den letzten Slash am Quellpfad!!

::

   cp -Rp /home/myuser.bak/ /home/myuser

Das Quellverzeichnis wird hier noch nicht gelöscht. Es erspart Ärger,
wenn man es nicht jetzt schon aus der Sicherung fischen muss, falls
etwas schief gehen sollte. Denkt aber bitte daran, es zu löschen, wenn
alles glatt gegangen ist. Es sind schließlich private Daten, die
verschlüsselungswürdig sind.

PAM vorbereiten
---------------

PAM wird so eingestellt, dass mit dem oben angegebenen ``pefs``-Passwort
ein entschlüsselnder Mount durchgeführt wird und beim normalen
``passwd``-Passwort einfach ein Login ohne weiteres Entschlüsseln
gemacht wird.

In ``/etc/pam.d/system``:

::

   auth            sufficient      pam_opie.so             no_warn no_fake_prompts
   auth            requisite       pam_opieaccess.so       no_warn allow_local
   auth            sufficient      /usr/local/lib/pam_pefs.so      try_first_pass
   auth            required        pam_unix.so             no_warn try_first_pass nullok

   .
   .
   .
    
   session         optional        /usr/local/lib/pam_pefs.so
   session         required        pam_lastlog.so          no_fail

Solltet Ihr KDM benutzen oder andere durch PAM authentifizierte Dienste,
muss man diese Zeilen auch dort einfügen.

Des Weiteren ist es wichtig, dass ``pefs`` mit einem ``mount`` beim
Start vorinitialisiert wird. Dieser ``mount`` entschlüsselt nicht,
sondern hält fest, dass das Verzeichnis ein ``pefs``-Dateisystem
darstellt.

Deswegen fügt man diese Zeile in die ``/etc/rc.local`` ein:

::

   /usr/local/sbin/pefs mount /home/myuser /home/myuser

Testen des Logins
-----------------

Erstmal alle Schlüssel entfernen. Der ``pefs``-Mountpoint bleibt dabei
erhalten.

::

   pefs flushkeys /home/myuser

Schaut bitte nach wie Euer ``$HOME`` aussieht:

::

   ls -la /home/myuser

Es sollte aus vielen seltsam aussehenden Punktdateien bestehen. Auf
einer anderen Konsole versucht nun Euren Benutzernamen einzugeben und
das Passwort, das für die Entschlüsselung angegeben wurde. Gebt nicht
Euer ``passwd``-Passwort ein. Das führt nur dazu, dass Euer ``$HOME``
nicht richtig erkannt wird.

Schaut nach, ob die Dateien entschlüsselt sind:

::

   ls -l ~

Optional sollte man das auch einzeln für einen KDM-Login ausprobieren.
Natürlich muss man sich zunächst wieder ausloggen und die
``pefs``-Schicht auch wieder mit

::

   pefs flushkeys /home/myuser

sperren bevor man es in KDM ausprobiert. KDM loggt sich bei Problemen
gar nicht ein, weil es direkt Dateien anlegen möchte, was auf einem
``pefs``-Dateisystem ohne Schlüssel wegen fehlender Schreibrechte nicht
gehen wird.

* :ref:`genindex`

Zuletzt geändert: |date|

