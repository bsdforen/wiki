UEFI & OpenBSD 6.9 & Ubuntu 21.04 & Windows 10 Triple boot on 2 hard drives (engl)
==================================================================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-openbsd.png

Als Ausnahme für unser Wiki dieser englische Artikel von Dettus, damit er nicht im Forum verloren geht.

Usally this wiki is only in German, but we thought this might be intresting for everyone.

So, I am posting this in English, since I googled A LOT to find it. I am sure there are other people out there with the same problem.

I have a special setup! My computer has 2 hard drives, and I wanted to install 3 operating systems. So why Dual Boot when you can TRIPLE BOOT?
I was also forced to use UEFI (Damn you Ubuntu 21.04), luckily this is no longer a problem for my favorite OS, OpenBSD 6.9. And because I like playing StarCraft 2, I needed a Windows 10 as well.

Here it goes...

.. hint::

  First of all: NO BACKUP, NO MERCY!
  Secondly: The order of installing the Operating systems is important. Mine was Windows, OpenBSD, Ubuntu.

  Windows got the first hard drive, (actually nvme).
  OpenBSD got the first 50% of the second drive, Ubuntu the second 50%.

For the impatient ones: This is what it looks like from the Ubuntu point of view in the end.

::

  % fdisk /dev/nvme0n1
 
  ..
  Command (m for help): p
  ...
  Device              Start        End    Sectors   Size Type
  /dev/nvme0n1p1       2048     206847     204800   100M EFI System
  /dev/nvme0n1p2     206848     239615      32768    16M Microsoft reserved
  /dev/nvme0n1p3     239616 1999364205 1999124590 953,3G Microsoft basic data
  /dev/nvme0n1p4 1999366144 2000406527    1040384   508M Windows recovery environm
  ...
  Command (m for help): q
  ...
  % fdisk /dev/nvme1n1
  ...
  Command (m for help): p
  ...
  Device              Start        End    Sectors   Size Type
  /dev/nvme1n1p2         64       1023        960   480K EFI System
  /dev/nvme1n1p3 1000101124 2000409200 1000308077   477G Linux filesystem
 /dev/nvme1n1p4       1024 1000101123 1000100100 476,9G OpenBSD data
  ...

  Command (m for help): q
  ...

  % cat /etc/grub.d/40_custom
  #!/bin/sh
  exec tail -n +3 $0
  # This file provides an easy way to add custom menu entries.  Simply type the
  # menu entries you want to add after this comment.  Be careful not to change
  # the 'exec tail' line above.
  menuentry 'OpenBSD 6.9/amd64' {
    set root='(hd0,gpt4)'
    chainloader (hd0,gpt2)/efi/boot/bootx64.efi
  }
  % update-grub


STEP1: I started by doing a backup. (NO BACKUP, NO MERCY).

STEP 2: I used a different machine to create 3 USB drives for my 3 operating systems.
One for Windows: https://www.microsoft.com/en-us/software-download/
One for OpenBSD: https://ftp.openbsd.org/pub/OpenBSD/6.9/amd64/install69.img
One for Ubuntu: https://ubuntu.com/tutorials/create-a-usb-stick-on-windows#1-overview

Creating the USB drive for the OpenBSD install works best on a Unix system:

::

 % wget https://ftp.openbsd.org/pub/OpenBSD/6.9/amd64/install69.img
 % dd if=install69.img of=/dev/XXXX bs=1M

For the other two, I used Windows.

STEP 3: I erased the first sector from my drives, to clear the partition table

::

  % dd if=/dev/zero of=/dev/nvme0XXXX bs=1M count=16
  % dd if=/dev/zero of=/dev/nvme1XXXX bs=1M count=16

Now there was no way back. Luckily, I had a backup ;)

STEP 4: I installed Windows. My BIOS required me to press F12 to see the boot options, I chose UEFI boot from the USB drive. I installed it on the first hard disk.

STEP 5: I booted the OpenBSD stick. At the prompt, I chose (S)hell.

::

  # cd /dev/
  # sh MAKEDEV sd1
  # fdisk -e /dev/rsd1c

set up the partions. In the following order: 0, 1, 3, 2

::

  0: unused
  1: type EF. From 64-960
  3: type A6. From 1024-1000100100
  2: type 83. From 1000100124-end

The exit and (I)nstall. I used the OpenBSD partition.
To be honest: I am writing this one down from memory. If it does now work, I guess that starting the installation, choosing (WHOLE) when prompted, and Interrupting during the disklabel setup, and then doing an fdisk might also be an option.

The next time I booted using the boot options, I had a new UEFI entry which was giving me OpenBSD. NICE!

STEP 6: I installed Ubuntu. When asked about the partitioning, I chose "Something else" and was LUCKY THAT Ubuntu used the Windows-UEFI-Partition for its bootloader.
The GRUB bootloader was able to find Windows on its own. I rebooted twice: Once to see if Linux was booting. Once to see if windows was booting, and if I could select which one using GRUB menu.

STEP 7: I had to figure out which harddrive the OpenBSD efi bootloader was on.
So when the grub screen showed up, I pressed (c) for command line options.

::

  grub> ls

Which showed me some partitions. Since I installed the OpenBSD efi loader on partition 1 and OpenBSD on partition 3, I was able to find what i needed on (hd0,gpt2) and (hd0,gpt4).

::

  grub> ls (hd0,gpt2)/efi/boot
  bootx64.efi ...
  grub> ls (hd0,gpt4)/
 bsd bsd.rd ...

I tried it out, using

::

  grub> set root='(hd0,gpt4)'
  grub> chainloader (hd0,gpt2)/efi/boot/bootx64.efi
  grub> boot

And it booted!

STEP 8: I rebooted into Linux, and updated grub. More precisely /etc/grub.d/40_custom. This is what it looked like afterwards:

::

  #!/bin/sh
  exec tail -n +3 $0
  # This file provides an easy way to add custom menu entries.  Simply type the
  # menu entries you want to add after this comment.  Be careful not to change
  # the 'exec tail' line above.
  menuentry 'OpenBSD 6.9/amd64' {
      set root='(hd0,gpt4)'
      chainloader (hd0,gpt2)/efi/boot/bootx64.efi
  }


All that was left now was running

::

  % update-grub

  (I ignored the warnings about those extra partitions. )
  So now, at boot time, I can choose any of my three Operating systems.

Keywords: Tutorial. Tripe Boot. Dual Boot. Grub. UEFI. OpenBSD. Linux. Ubuntu. Windows.

This howto was originally created by dettus  `dettus <https://www.bsdforen.de/members/dettus.1918/>`_ and postet in our `forum <https://www.bsdforen.de/threads/uefi-openbsd-6-9-ubuntu-21-04-windows-10-triple-boot-on-2-hard-drives.36218/>`_ .

* :ref:`genindex`

Zuletzt geändert: |date|