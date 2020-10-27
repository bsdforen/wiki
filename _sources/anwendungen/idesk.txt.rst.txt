IDesk
=====

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

iDesk gibt Benutzern von Windowmanagern wie \*box, Windowmaker usw. die
Möglichkeit, Icons auf ihrem Desktop darzustellen. Die Icons können
entweder im png- oder im svg-Format vorliegen und unterstützen
Transparenz. Jedes Icon kann so konfiguriert werden, daß es ein oder
mehrere Kommandos ausführt, wenn darauf geklickt wird. Wenn man also
Icons möchte, dann muss man nicht KDE oder GNOME benutzen. |Idesk.jpg|

Features
--------

-  Ausführen mehrerer Shell-Kommandos
-  PNG Unterstützung
-  Skalierte Icons
-  Anti-Aliased Schriften
-  Transparenz
-  Schriftschatten
-  Gitter zum Verschieben der Icons
-  Tooltips

Konfiguration
-------------

Um iDesk zu konfigurieren, muß man sich zwei Bereiche ansehen. Der Erste
ist die allgemeine Konfiguration des Erscheinungsbildes und Verhaltens
der Icons. Die Informationen über die Icons wie der Ort, das Kommando
und selbstverständlich das Iconbild werden in einer extra Datei für
jedes Icon gespeichert.

Also zuerste sehen wir uns die allgemeine Konfigurationsdatei
**``Datei|~/.ideskrc``** an:

::

   $ cd ~
   $ vi .ideskrc

::

      table Config
        FontName: tahoma          
        FontSize: 8
        FontColor: #ffffff        
        Locked: false
        Transparency: 150
        Shadow: false
        ShadowColor: #000000
        ShadowX: 1
        ShadowY: 1
        Bold: false
        ClickDelay: 300
        IconSnap: true
        SnapWidth: 48
        SnapHeight: 48
        SnapOrigin: BottomRight
        SnapShadow: true
        SnapShadowTrans: 200
        CaptionOnHover: false
      end
      
      table Actions
        Lock: control right doubleClk
        Reload: middle doubleClk
        Drag: left hold
        EndDrag: left singleClk
        Execute[0]: left doubleClk
        Execute[1]: right doubleClk
      end

::

   :wq!

Als nächstes eine Beispielkonfiguration für ein Icon. Dazu muss man aber
erst das Verzeichnis dazu anlegen:

::

   $ cd ~
   $ mkdir .idesktop
   $ cd .idesktop
   $ vi xterm.lnk

::

      table Icon
        Caption: xterm
        Command: xterm
        Icon: /home/user/icons/xterm.png
        X: 1184
        Y: 880
      end

::

   :wq!

Weblinks
--------

-  `x11/idesk <https://www.google.com/search?q=x11/idesk&btnI=lucky>`__
-  `idesk.sourceforge.net <http://idesk.sourceforge.net>`__

.. |Idesk.jpg| image:: images/idesk.jpg
   :class: align-right
   :width: 256px

* :ref:`genindex`

Zuletzt geändert: |date|

