Festplattenverschlüsselung mit NetBSD
=====================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-netbsd.png

Über
----

NetBSD ist ein vielseitiges Betriebssystem. Es ist in weiten Teilen
hervorragend dokumentiert, nur der Teil zur Festplattenverschlüsselung
reduziert sich leider auf das aller Wesentlichste. Für Leute, die nicht
so viel Erfahrung mit NetBSD haben, aber ihr System physikalisch
absichern wollen gestaltet sich diese Aufgabe ohne genau Anleitung meist
sehr schwierig, oft zu schwierig. Dem will ich mit diesem Artikel
Abhilfe schaffen.

Dieser Artikel basiert auf einem Englischen HowTo von Johnny C. Lam
(siehe Weblinks). Der Festplattenverschlüsselungsteil ist fast fertig.
Wer will kann ihn Testen, verbessern und erweitern

Der Weg
-------

Anforderungen
~~~~~~~~~~~~~

-  Ein NetBSD Installationsmedium (CD, Diskette, ...)
-  USB-Stick, auf dem der Schlüssel gespeichert wird - kann praktisch
   beliebiger Größe sein
-  Selbstverständlich brauchen wir auch ein System, auf welches NetBSD
   installiert werden. Es gibt dabei kaum ein System, das nicht in Frage
   kommt. Man sollte allerdings aufpassen, dass sich das System auch
   wirklich für verschlüsselte Partitionen eignet und nicht so schon zu
   langsam ist

Backup
~~~~~~

Wie vor jeder Installation, sollte man darauf achten, dass es ein Backup
aller wichtigen Daten existiert. Man sollte dieses Backup
selbstverständlich nicht auf dem dem System haben, auf dem die
Installation stattfinden soll.

NetBSD installieren
~~~~~~~~~~~~~~~~~~~

Es ist von Vorteil, aber nicht zwingend notwendig, mit der Installation
vertraut zu sein. Die Installation verläuft mit ein paar Ausnahmen, wie
immer.

Der unveränderte Teil
^^^^^^^^^^^^^^^^^^^^^

NetBSD wird, wie gewohnt gebootet und fragt zuerst nach der gewünschten
Sprache und dem Keyboardlayout. Da NetBSD installiert werden soll, wird
nächsten Menü den entsprechende Punkt ausgewählt. Der Installer macht
nochmals darauf aufmerksam, dass wir NetBSD installieren und unbedingt
ein Backup gemacht werden sollte. Im nächsten Schritt wird nach der
Festplatte gefragt auf welche NetBSD installiert werden soll. Nach der
Auswahl schlägt der Installer einige Möglichkeiten der Installation vor.
Für NetBSD-Neulinge empfiehlt sich hier die komplette Installation. Alle
Anderen sollten selbst wissen, welchen Punkt sie wählen.

Was beachtet werden muss
^^^^^^^^^^^^^^^^^^^^^^^^

Der nächste Schritt betrifft die Partitionierung, diese soll verändert
werden. Drei Partitionen werden angelegt.

-  Root (/) (unverschlüsselt!): diese sollte mindestens 256MB haben,
   aber muss nicht größer sein. Auf sie wird NetBSD installiert.
-  Swap (verschlüsselt): Wie gewohnt. Als Faustregel für die Größe gilt
   2*RAM
-  Crypt (/crypt) (verschlüsselt): Hier landen alle Daten, die
   verschlüsselt werden sollen/können. Diese Partition sollte den
   restlichen verfügbaren Platz bekommen. Dazu wird einfach ein "+"
   statt der Menge in Megabyte verwendet.

Nachdem man die Größen gewählt und akzeptiert hat wird nach den Optionen
für die erstellen Partitionen gewählt. Die Einzige Partition, die eine
Anpassung benötigt ist /crypt. Dazu wählen wir sie aus und ändern den
Dateisystemtyp auf ccd, welcher unter "andere Dateisystemtypen" zu
finden ist.

Installation fertig stellen
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Der Rest der Installation verläuft wie gewohnt.

Neustart in den Single User Mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nach der Installation erfolgt der Neustart des Sytems. Da die
Partitionen sind allerdings noch nicht verschlüsselt. Um das zu tun
wählt man im Bootloader den Single User Mode aus. Dort wird die
Standardshell verwendet, also einfach Eingabe drücken. <note tip>Das
Standard Keyboard Layout ist englisch. Um es auf deutsch zu stellen
nutzt man **wsconsctl -k -w encoding=de**\ </note> Im Single User Mode
mountet man die Root-Partition mit einem

::

   # mount -uw /

Da das in das Basissystem aus Sicherheitsgründen read-only gemountet
werden soll müssen einige Mountpoints erstellt werden.

::

   # mkdir /emul /crypt /usr/pkg

Wie erwähnt soll das System read-only sein. Das führt mit der
Standard-rc zu ein paar Problemen. Deshalb muss /etc/rc leicht
abgeändert werden. Dazu wird einfach die Zeile **[ "$rc_hooked" = "YES"
] \|\| . /etc/rc.hook** hinzugefügt:

::

   . /etc/rc.subr
   . /etc/rc.conf
   [ "$rc_hooked" = "YES" ] || . /etc/rc.hook
   _rc_conf_loaded=true

Damit der USB-Stick mit dem Schlüssel gefunden wird, wird außerdem ein
**usbkeydev=sd0** in /etc/rc.conf benötigt.

::

   # Add local overrides below
   #
   usbkeydev=sd0

Als nächstes wird der USB-Stick mit dem Key bestückt. Dazu muss der (mit
FAT formatierter) Stick zuerst gemountet werden:

::

   # mount -t msdos /dev/sd0e /mnt

Nun kann verschlüsselt werden (<crypt> ist durch die /crypt-Partition zu
ersetzen) :

::

   # cgdconfig -g -V disklabel -k storedkey -k pkcs5_pbkdf2/sha1 -o /mnt/key aes-cbc 256
   # cgdconfig -V re-enter cgd0 /dev/<crypt> /mnt/key
   /dev/<crypt>'s passphrase: **********
   re-enter device's passphrase: **********

**cgd0** ist die verschlüsselte Partition. Sie kann wie jede Andere
verwendet werden. Zuerst wird sie formatiert und mit allem, was
verschlüsselt werden darf/soll bestückt:

::

   # newfs /dev/cgd0a
   # mount -t ffs /dev/cgd0a /crypt
   # cd / && pax -rwpe etc root tmp var /crypt

Um auch die verschlüsselten Daten zu verwenden muss /etc/fstab angepasst
werden:

::

   /dev/root / ffs ro 1 1
   /dev/swap none swap sw 0 0
   /dev/crypt /local ffs rw,noatime,nodevmtime 0 0
   /crypt/etc /etc null rw,hidden
   /crypt/tmp /tmp null rw,hidden
   /crypt/var /var null rw,hidden
   /crypt/root /root null rw,hidden
   kernfs /kern kernfs rw
   procfs /proc procfs rw,noauto

Da all diese Partitionen möglichst früh gemountet werden sollen muss
**/crypt/etc/rc.conf** angepasst werden.

::

   rc_hooked=YES
   critical_filesystems_local="/crypt /etc /tmp /var"
   securelevel=2

Jetzt muss nur noch das **/etc/rc.hook** script mit folgendem Inhalt
platziert werden:

::

   # Copyright (c) 2007 Johnny C. Lam.  All rights reserved.
   #
   # Redistribution and use in source and binary forms, with or without
   # modification, are permitted provided that the following conditions
   # are met:
   # 1. Redistributions of source code must retain the above copyright
   #    notice, this list of conditions and the following disclaimer.
   # 2. Redistributions in binary form must reproduce the above copyright
   #    notice, this list of conditions and the following disclaimer in the
   #    documentation and/or other materials provided with the distribution.
   # 3. The name of the author may not be used to endorse or promote
   #    products derived from this software without specific prior written
   #    permission.
   #
   # THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS
   # OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
   # WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
   # ARE DISCLAIMED.  IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY
   # DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
   # DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE
   # GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
   # INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
   # WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
   # NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
   # SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
   #

   #############################################################################
   #
   # usbkeydev disk device that stores the cgd(4) params file; if unset,
   #       in /etc/rc.conf, then it's assumed the params file lives
   #       in /etc/cgd.
   #
   # hostname  if usbkeydev is defined, then $hostname is the name of
   #       the params file on $usbkeydev if it is set.
   #
   : ${usbkeydev=}
   #
   #############################################################################

   #
   # num2alpha n
   #   Prints the (n-1)th letter of the alphabet to standard output.
   #
   num2alpha()
   {
       local _n
       _n=$1; shift
       set -- a b c d e f g h i j k l m n o p q r s t u v w x y z
       shift $_n
       echo $1
   }

   #
   # find_free_disk dev var [...]
   #   Look for free disk devices named by $1 and set them as values
   #   for the named variables.
   #
   #   Returns 0 if all named variables were set, and nonzero otherwise.
   #
   #   For example, the following will set vndlocal and vndswap to the
   #   first free vnd devices on the system:
   #
   #       find_free_disk vnd vndlocal vndswap
   #
   find_free_disk()
   {
       local _arg _d _dev _devices _devstatus _k _rawpart _value _var
       _dev="$1"; shift    # Remaining args are variables to set.

       _rawpart=`sysctl -n kern.rawpartition`
       _rawpart=`num2alpha $_rawpart`

       # Set _dev1, _dev2, etc. variables to "free" or "used" based on
       # whether the device is already named in "sysctl -n hw.disknames".
       #
       _devices=
       _k=0
       while [ 1 = 1 ]; do
           [ -b /dev/${_dev}${_k}${_rawpart} ] || break
           _devices="$_devices ${_dev}${_k}"
           _k=$((_k + 1))
       done
       [ $_k != 0 ] || return 1

       for _d in $_devices; do
           _var=_${_d}
           eval $_var=free
       done
       for _d in `sysctl -n hw.disknames`; do
           case $_d in
           ${_dev}[0-9]*)
               _var=_${_d}
               eval $_var=used
               ;;
           esac
       done

       # Set named variables to first free devices.
       for _d in $_devices; do
           _var=_${_d}
           eval _devstatus=\$${_var}
           if [ "$_devstatus" = "used" ]; then
               unset $_var
               continue
           fi
           for _arg in "$@"; do
               eval _value=\$${_arg}
               if [ -z "$_value" ]; then
                   eval ${_arg}=${_d}
                   break
               fi
           done
           unset $_var
       done

       for _arg in "$@"; do
           eval _value=\$${_arg}
           [ -n "$_value" ] || return 1
       done
       return 0
   }

   #
   # mount_usbkey dev mountpoint
   #   Mounts the USB key specified in $1 at the mount point specified
   #   in $2.
   #
   #   If the device is specified as a path of the form "/dev/*", then
   #   it is used as the MSDOS partition to mount.  Otherwise, the
   #   device is taken to be a simple device name, e.g. "sd0", and the
   #   disklabel of the device is searched for the first MSDOS partition
   #   to mount.
   #
   #   Returns 0 on a successful mount, and nonzero otherwise.
   #
   #   For example, the following will mount the first MSDOS partition
   #   on sd0 onto /mnt:
   #
   #       mount_usbkey sd0 /mnt
   #
   mount_usbkey()
   {
       local _dev _mntpt
       _dev=$1; _mntpt=$2

       case "$_dev" in
       /dev/*)
           mount -r -t msdos $_dev $_mntpt || return 1
           ;;
       *)
           # Mount the first MSDOS partition found on $_dev.
           _dev=${_dev##*/}
           disklabel $_dev 2>/dev/null |
           while read _line; do
               case "$_line" in
               [a-z]:*MSDOS*)
                   _dev=/dev/${_dev}${_line%%:*}
                   mount -r -t msdos $_dev $_mntpt || return 1
               esac
           done
           case $? in
           0)  ;;
           *)  return $? ;;
           esac
           ;;
       esac
       return 0
   }

   #
   # crypto_swap cgd swapdev
   #   Creates the cgd(4) device named in $1 in the partition named in
   #   $2 and rewrites the disklabel on the device so that the whole
   #   device is a swap partition.
   #
   #   If the device is specified as a path of the form "/dev/*", then
   #   it is used as the swap partition to configure.  Otherwise, the
   #   device is taken to be a simple device name, e.g. "wd0", and the
   #   disklabel of the device is searched for the first swap partition
   #   to configure.
   #
   #   Returns 0 if the encrypted swap partition was successfully
   #   set up, and nonzero otherwise.
   #
   #   For example, the following configures cgd1 onto the first swap
   #   partition on wd0:
   #
   #       crypto_swap cgd1 wd0
   #
   crypto_swap()
   {
       local _cgd _dev _rawpart
       _cgd=$1; _dev=$2

       case "$_dev" in
       /dev/*)
           cgdconfig -s $_cgd $_dev aes-cbc < /dev/urandom || return 1
           ;;
       *)
           # Configure the first swap partition found on $_dev.
           _dev=${_dev##*/}
           disklabel $_dev 2>/dev/null |
           while read _line; do
               case "$_line" in
               [a-z]:*swap*)
                   _dev=/dev/${_dev}${_line%%:*}
                   cgdconfig -s $_cgd $_dev aes-cbc 256 < /dev/urandom || return 1
               esac
           done
           case $? in
           0)  ;;
           *)  return $? ;;
           esac
           ;;
       esac

       _rawpart=`sysctl -n kern.rawpartition`
       _rawpart=`num2alpha $_rawpart`

       # Rewrite the disklabel on the newly-configured cgd(4) device
       # so that the whole disk is used for swap.
       #
       disklabel $_cgd 2>/dev/null |
       while read _line; do
           case "$_line" in
           ${_rawpart}:*)
               _bline=" b:${_line#${_rawpart}:}"
               _bline="${_bline%%unused*}  swap${_bline##*unused}"
               echo "$_bline"
               echo "$_line"
               ;;
           [a-z]:*) ;;
           *)  echo "$_line" ;;
           esac
       done | disklabel -R -r $_cgd /dev/stdin 2>/dev/null

       case $? in
       0)  ;;
       *)  cgdconfig -u $_cgd
           return 2
           ;;
       esac

       return 0
   }

   #
   # crypto_partition cgd dev [usbkeydev [paramsfile]]
   #   Creates the cgd(4) device named in $1 in the partition named in
   #   $2.  If the $3 argument is specified, then it is the partition
   #   from which the cgd(4) params file should be loaded.  If the $4
   #   argument is specified, then it is the name of the cgd(4) params
   #   file to use.
   #
   #   The cgd(4) params file is searched in the following order:
   #
   #       /mnt/$paramsfile    (if paramsfile is specified)
   #       /mnt/cgd/$paramsfile    (if paramsfile is specified)
   #       /mnt/$dev       e.g., /mnt/wd0e
   #       /mnt/cgd/$dev       e.g., /mnt/cgd/wd0e
   #       /etc/cgd/$dev       e.g., /etc/cgd/wd0e
   #
   #   Returns 0 if the encrypted partition was successfully
   #   configured, and nonzero otherwise.
   #
   #   For example, the following configures cgd0 on the first "ccd"
   #   partition on wd0, and uses the params file named "/myhostname"
   #   or "/cgd/myhostname" from sd0:
   #
   #       crypto_partition cgd0 wd0 sd0 myhostname
   #
   crypto_partition()
   {
       local _cgd _cgdparams _dev _keydev _paramsfile
       _cgd=$1; _dev=$2; _keydev=$3; _paramsfile=$4

       # Mount the sd(4) device on the USB key drive onto /mnt to locate
       # the cgd(4) params file.
       #
       if [ -n "$_keydev" ]; then
           mount_usbkey $_keydev /mnt || return 1
       fi

       # Look for the cgd(4) params file.
       _paramlist="/mnt/${_dev##*/} /mnt/cgd/${_dev##*/} /etc/cgd/${_dev##*/}"
       if [ -n "$_paramsfile" ]; then
           _paramlist="/mnt/$_paramsfile /mnt/cgd/$_paramsfile $_paramlist"
       fi
       for _file in $_paramlist; do
           if [ -f "$_file" ]; then
               _cgdparams="$_file"
               break
           fi
       done

       case "$_dev" in
       /dev/*)
           if ! cgdconfig $_cgd $_dev $_cgdparams; then
               [ -z "$_keydev" ] || umount /mnt
               return 2
           fi
           ;;
       *)
           # Configure the first ccd partition found on $_dev.
           _dev=${_dev##*/}
           disklabel $_dev 2>/dev/null |
           while read _line; do
               case "$_line" in
               [a-z]:*ccd*)
                   _dev=/dev/${_dev}${_line%%:*}
                   if ! cgdconfig $_cgd $_dev $_cgdparams; then
                       [ -z "$_keydev" ] || umount /mnt
                       return 2
                   fi
               esac
           done
           case $? in
           0)  ;;
           *)  return $? ;;
           esac
           ;;
       esac
       [ -z "$_keydev" ] || umount /mnt
       return 0
   }

   _rootdev=`sysctl -n kern.root_device`
   _rootpart=`sysctl -n kern.root_partition`
   _rootpart=`num2alpha $_rootpart`

   # Find 2 free cgd(4) devices and set them as values for _cgdlocal and
   # _cgdswap.
   #
   if ! find_free_disk cgd _cgdlocal _cgdswap; then
       echo "Not enough free cgd(4) devices found.  Multiuser boot aborted."
       exit 1
   fi

   echo "Configuring encrypted swap partition"
   if ! crypto_swap $_cgdswap $_rootdev; then
       echo "Unable to configure encrypted swap partition. " \
            "Multiuser boot aborted."
       exit 1
   fi

   echo "Configuring encrypted local device"
   if ! crypto_partition $_cgdlocal $_rootdev $usbkeydev $hostname; then
       echo "Unable to configure encrypted local device. " \
            "Multiuser boot aborted."
       exit 1
   fi

   # Mount /local to get the local configuration data.  We must fsck(8) the
   # partition beforehand to ensure that it's clean because we won't be able
   # to fix any problems later on after other filesystems are mounted.
   #
   echo "Starting bootstrap file system check:"
   fsck -p /dev/r${_rootdev}${_rootpart} /dev/r${_cgdlocal}a
   case $? in
   0)  ;;
   *)  echo "Automatic bootstrap file system check failed; help!"
       exit 1
       ;;
   esac

   echo "Mounting /local for read-write local data"
   if ! mount -t ffs -o noatime,nodevmtime /dev/${_cgdlocal}a /local; then
       echo "Unable to mount /local.  Multiuser boot aborted."
       exit 1
   fi

   echo "Mounting /etc for local configuration"
   if ! mount -t null -o hidden /local/etc /etc; then
       echo "Unable to mount /etc.  Multiuser boot aborted."
       exit 1
   fi

   echo "Fixing /etc/fstab"
   if [ ! -f /etc/fstab ]; then
       echo "Could not find /etc/fstab.  Multiuser boot aborted."
       exit 1
   fi
   while read _device _mntpt _fstype _rest; do
       case "$_device" in
       "#"*)   ;;
       *)  case "$_mntpt" in
           /)  _device="/dev/${_rootdev}${_rootpart}" ;;
           /local) _device="/dev/${_cgdlocal}a" ;;
           *)  [ "$_fstype" != "swap" ] || _device="/dev/${_cgdswap}b" ;;
           esac
           ;;
       esac
       if [ -z "$_rest" ]; then
           echo "$_device $_mntpt $_fstype"
       else
           echo "$_device $_mntpt $_fstype $_rest"
       fi
   done < /etc/fstab > /etc/fstab.hooked
   mv -f /etc/fstab.hooked /etc/fstab

   # Re-exec /etc/rc so that we use the correct configuration information,
   # e.g., /etc/rc.conf settings, etc.
   #
   echo "Re-executing boot process with local configuration"
   exec /bin/sh $0

Fertig! Finger kreuzen und neu starten:

::

   # umount /local /mnt
   # cgdconfig -u cgd0
   # reboot

Weblinks
--------

`Guide, auf dem dieser Artikel basiert (leicht
veraltet) <http://www.netbsd.org/~jlam/cgd_laptop.html>`__

::

   TODO:
   Probelesen
   Fertig machen (swap)
   Genauer erklären, warum was getan wird
   Wiki-Syntax Überarbeitung
   Sprachliche Überarbeitung
   nochmal Probelesen

* :ref:`genindex`

Zuletzt geändert: |date|

