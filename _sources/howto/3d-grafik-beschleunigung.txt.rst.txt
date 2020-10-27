3D-Grafik-Beschleunigung
========================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

.. note:: 

  Die Inhalte sind arg veraltet!

Unter 3D-Grafik-Beschleunigung wird die Möglichkeit verstanden, 3D-Funktionen
(OpenGL) hardwarebeschleunigt (Grafikkarte) innerhalb eines X11-Fensters
anzuzeigen.

Technische Übersicht
--------------------

Diese Übersicht mag aus dem technischen Standpunkt nicht ganz korrekt
sein, zeigt aber verständlich auf, wie 3D-Funktionen
hardwarebeschleunigt werden:

::

   3D-Anwendung => X11-Server => DRI => DRM => Grafikkarte

Die 3D-Funktionen werden von der 3D-Anwendung über den X11-Server an DRI
weitergereicht. `DRI <http://dri.freedesktop.org/wiki/>`__ (Direct
Rendering Infrastructure) ist die Schnittstelle zwischen X11-Software
und Kernel für den direkten Zugriff auf die Grafikkarten-Hardware.

Von DRI geht es weiter zu DRM (Direct Rendering Module). DRM ist im
Kernel verankert und greift somit direkt auf die Grafikkarten-Hardware
zu.

Einrichtung von 3D-Grafik-Beschleunigung
----------------------------------------

Eine Schritt für Schritt-Anleitung für den X.Org-Server, wie sie zu
hardwarebeschleunigtem OpenGL kommen. Falls Sie den XFree86-Server
einsetzen, müssen Sie gegebenerfalls die Schritte anpassen (z.B. andere
Konfigurationsdatei)!

DRM
~~~

Falls der Kernel DRM für Ihre Grafikkarte unterstützt, sollte die
DRM-Unterstützung automatisch geladen werden. Überprüfen Sie mit:

::

  dmesg|grep drm

ob DRM vom Kernel unterstützt wird. Die Ausgabe bei FreeBSD 5.3 für eine
ATI Mobility 7500 sieht wie folgt aus:

::

  drm0: <ATI Radeon LW Mobility 7500 M7> port 0x3000-0x30ff mem
  und und.. info: [drm] AGP at 0xd0000000 256MB info: [drm] Initialized
  radeon 1.11.0 20020828 on minor 0

Bei Problemen sehen Sie bitte unter `DRI für
FreeBSD <http://people.freebsd.org/~anholt/dri/>`__ nach! Bei Problemen
mit DRM bei i915 Chipset gibt es
`hier <http://lists.freebsd.org/pipermail/freebsd-current/2006-March/061458.html>`__
abhilfe.

**DRM muß explizit in den Kernel eingebaut werden**, was bei den
verschiedenen Grafikkarten unterschiedlich aussieht.
/usr/src/sys/i386/conf/NOTES sagt dazu:

::

   # Direct Rendering modules for 3D acceleration.
   device      drm     # DRM core module required by DRM drivers
   device      mach64drm   # ATI Rage Pro, Rage Mobility P/M, Rage XL
   device      mgadrm      # AGP Matrox G200, G400, G450, G550
   device      r128drm     # ATI Rage 128
   device      radeondrm   # ATI Radeon
   device      sisdrm      # SiS 300/305, 540, 630
   device      tdfxdrm     # 3dfx Voodoo 3/4/5 and Banshee
   options     DRM_DEBUG   # Include debug printfs (slow)

DRI
~~~

Vergessen Sie nicht, DRI (graphics/dri oder bei Verwendung von
XFree86-4.5 /graphics/xfree86-dri) zu installieren::

  # pkg_add -r dri

oder::

  # pkg_add -r xfree86-dri

X11-Server
~~~~~~~~~~

Folgende Einträge sind in der X11-Server-Konfigurationsdatei
(/etc/X11/xorg.conf bzw. /etc/X11/XF86Config) notwendig:

::

   Section "Module"
     Load  "dri"
     Load  "glx"
   EndSection

   Section "DRI"
     Mode         0666
   EndSection

Abschlusstest
~~~~~~~~~~~~~

Kontrollieren Sie mit::

  # glxinfo | grep rendering direct rendering: Yes

ob hardwarebeschleunigtes OpenGL unterstützt wird. Falls es nicht
funktioniert, könnte es sein, dass Ihre Grafikkarte AGP nicht benutzt.
Sie müssen dann `AGP
aktivieren <http://www.freebsd.ch/doc/de_DE.ISO8859-1/books/handbook/x-config.html>`__.
``glxinfo`` ist in
`graphics/mesa-demos <https://www.google.com/search?q=graphics/mesa-demos&btnI=lucky>`__
enthalten.

Weiterführende Literatur
------------------------

-  `X.Org 7.0 -
   Dokumentation <http://ftp.x.org/pub/X11R7.0/doc/html/>`__
-  `X.Org 6.9 -
   Dokumentation <http://ftp.x.org/pub/X11R6.9.0/doc/html/>`__

