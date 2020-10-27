FreeBSD CD-ISO in USB Stick Image umwandeln
===========================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Der folgende Artikel enthält hauptsächlich ein Skript, das FreeBSD-CD-ISO
Images in Images für USB-Sticks umwandelt. Damit ist die Erstellung eines
bootbaren FreeBSD-Installations-Sticks sehr komfortabel möglich. Da FreeBSD
mittlerweile USB-Stick Images anbietet die direkt auf USB-Sticks kopiert werden
können (mittels ``dd``) ist es nicht mehr notwendig ein USB-Stick-Image aus der
CD zu erzeugen.

Voraussetzungen
---------------

<note warning>Die Daten auf dem USB Stick können nach dem Dump des
Images auf den USB Stick nicht mehr zugegriffen werden!</note>

-  USB-Stick
-  FreeBSD-System
-  Gewünschtes CD-ISO Image

Vorgehen
--------

Das Skript fbsd2img.sh wird folgendermaßen aufgerufen:

::

   fbsd2img.sh SOURCE DESTINATION

Dabei steht SOURCE für das heruntergeladene CD-ISO, beispielsweise
`7.2-RELEASE-i386-bootonly.iso <ftp://ftp.freebsd.org/pub/FreeBSD/ISO-IMAGES-i386/7.2/7.2-RELEASE-i386-bootonly.iso>`__.
DEST steht für das zu erstellende USB-Image. Der entsprechende Befehl
könnte dementsprechend so aussehen:

::

   fbsd2img.sh 7.2-RELEASE-i386-bootonly.iso 7.2-RELEASE-i386-bootonly.usb

Das so erstellte USB-Image kann per dd auf den USB-Stick übertragen
werden: <note warning>Auf jeden Fall sicherstellen, dass das richtige
Device ausgewählt wird, da sonst Daten und die komplette
Partitionstabelle zerstört werden können!</note>

::

   dd if=7.2-RELEASE-i386-bootonly.usb of=/dev/REPLACE_WITH_TARGET_USB_DEVICE

Nach diesen beiden Befehlen ist der USB-Stick ein voll einsatzfähiges
FreeBSD-Installationsmedium :) Getestet mit:

-  7.0-Release Bootonly
-  7.2-Release Bootonly
-  8.0-Beta2 Bootonly

Anhang
------

Das folgende Skript ist das oben referenzierte fbsd2img.sh. Hinweise:

-  Vor dem Ausführen vorsichtshalber ein dos2unix durchführen
-  Das Skript muss ausführbar gemacht werden (chmod 0755 fbsd2img.sh)
   oder einer Shell als Parameter übergeben werden (sh fbsd2img.sh)
-  Das Skript speichert temporäre Dateien in /tmp. Soll also ein großes
   FreeBSD-Image in ein USB-Image umgewandelt werden, muss dort genug
   Platz zur Verfügung stehen. Andernfalls kann das Skript problemlos
   auf die Verwendung eines alternativen tmp-Verzeichnisses angepasst
   werden (Im Code markiert).

::

   #!/bin/sh
   #Dario Freni
   #FreeSBIE developer (http://www.freesbie.org)
   #GPG Public key at http://www.saturnero.net/saturnero.asc
   #
   # fbsd-install-iso2img.sh iso-path img-path

   # You can set some variables here. Edit them to fit your needs.

   # Set serial variable to 0 if you don't want serial console at all,
   # 1 if you want comconsole and 2 if you want comconsole and vidconsole
   serial=0

   set -u

   if [ $# -lt 2 ]; then
       echo "Usage: $0 source-iso-path output-img-path"
       exit 1
   fi

   isoimage=$1; shift
   imgoutfile=$1; shift

   export tmpdir=$(mktemp -d -t fbsdmount)
   # ADJUST TMPDIR HERE!
   # Temp file and directory to be used later
   export tmpfile=$(mktemp -t bsdmount)

   export isodev=$(mdconfig -a -t vnode -f ${isoimage})

   echo "#### Building bootable UFS image ####"

   ISOSIZE=$(du -k ${isoimage} | awk '{print $1}')
   SECTS=$((($ISOSIZE + ($ISOSIZE/5))*2))

   # Root partition size

   echo "Initializing image..."
   dd if=/dev/zero of=${imgoutfile} count=${SECTS}
   ls -l ${imgoutfile}
   export imgdev=$(mdconfig -a -t vnode -f ${imgoutfile})

   bsdlabel -w -B ${imgdev}
   newfs -O1 /dev/${imgdev}a

   mkdir -p ${tmpdir}/iso ${tmpdir}/img

   mount -t cd9660 /dev/${isodev} ${tmpdir}/iso
   mount /dev/${imgdev}a ${tmpdir}/img

   echo "Copying files to the image..."
   ( cd ${tmpdir}/iso && find . -print -depth | cpio -dump ${tmpdir}/img )

   if [ ${serial} -eq 2 ]; 
   then
           echo "-D" > ${tmpdir}/img/boot.config
           echo 'console="comconsole, vidconsole"' >> ${tmpdir}/img/boot/loader.conf
   elif [ ${serial} -eq 1 ]; 
   then
           echo "-h" > ${tmpdir}/img/boot.config
           echo 'console="comconsole"' >> ${tmpdir}/img/boot/loader.conf
   fi

   cleanup() {
       umount ${tmpdir}/iso
       mdconfig -d -u ${isodev}
       umount ${tmpdir}/img
       mdconfig -d -u ${imgdev}
       rm -rf ${tmpdir} ${tmpfile}
   }

   cleanup

   ls -lh ${imgoutfile}

Verweise
--------

-  Das Skript geht ähnlich wie diese Anleitung vor:
   http://miwi.bsdcrew.de/2009/06/freebsd-80-install-with-a-usb-stick/

* :ref:`genindex`

Zuletzt geändert: |date|

