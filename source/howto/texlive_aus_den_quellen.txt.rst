TeXlive aus dem Quellcode installieren
======================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Dieser Artikel beschreibt die Installation von `TeXlive
<http://www.tug.org/texlive>`__ auf FreeBSD 7.0 AMD64. Für diese Platform sind
leider keine Binärpakete von TeXlive verfügbar.

Vorbedingungen
--------------

Es sollten folgende ports installiert sein:

-  `devel/bison <https://www.google.com/search?q=devel/bison&btnI=lucky>`__
-  `textproc/flex <https://www.google.com/search?q=textproc/flex&btnI=lucky>`__
-  `x11-fonts/fontconfig <https://www.google.com/search?q=x11-fonts/fontconfig&btnI=lucky>`__
-  `devel/subversion <https://www.google.com/search?q=devel/subversion&btnI=lucky>`__

Schritt für Schritt Anleitung
-----------------------------

Die Code-Ausschnitte beziehen sich auf ein Beispielsystem und müssen
angepasst werden.

Sourcen beschaffen
~~~~~~~~~~~~~~~~~~

#. In ein Verzeichnis wechseln, das das Basisverzeichnis von texlive
   aufnehmen soll.
#. Den source code von TeXlive 2007 mittels
   `devel/subversion <https://www.google.com/search?q=devel/subversion&btnI=lucky>`__
   ziehen (kann dauern):

::

     [mickraus@gandalf texlive]\$ svn co svn://tug.org/texlive/branch2007

Sourcen übersetzen
~~~~~~~~~~~~~~~~~~

#. Vor dem Übersetzen muß der icu port von FreeBSD entfernt werden,
   damit TeXlive seine eigene Version verwendet. Sonst kommt es zu
   Problemen beim Linken.

::

     gandalf# pkg_info | grep icu
     eclipse-3.2.2_1     An open extensible IDE for anything and nothing in particul
     icu-3.8.1_1         International Components for Unicode (from IBM)
     gandalf# pkg_delete -f icu-3.8.1_1

- Wechseln Sie in das Unterverzeichnis ''branch2007/Build/source''
- TeXlive kann nur mit [[freshports>devel/gmake]] gebaut werden. Um anzuzeigen, wo GNUmake liegt, muß eine spezielle Umgebungsvariable gesetzt werden:
  <xterm>[mickraus@gandalf source]\$ export TL_MAKE=/usr/local/bin/gmake</xterm>
- Starten des eigentlichen Builds. Um fontconfig finden zu können, benötigt der Build-Lauf ein ''--with-fontconfig=/usr/local''. 
  <xterm>[mickraus@gandalf source]\$ ./Build --with-fontconfig=/usr/local</xterm>
- Jetzt kann das icu aus den ports wieder zurückgespielt werden.
  <xterm>gandalf# pkg_add -vr icu</xterm>

Betrieb vorbereiten
~~~~~~~~~~~~~~~~~~~

Die Programme von TeXlive liegen nach dem Build in
``./inst/bin/<stdplatform>``. Sie müssen nach ``branch2007/Master/bin``
kopiert werden: <xterm> [mickraus@gandalf bin]\$ cd
/home/mickraus/latex/texlive/branch2007/Build/source/inst/bin
[mickraus@gandalf bin]\$ ls x86_64-unknown-freebsd7.0 [mickraus@gandalf
bin]\$ cp -R x86_64-unknown-freebsd7.0
/home/mickraus/latex/texlive/branch2007/Master/bin </xterm> Zuletzt muß
das Verzeichnis mit den TeXlive-Programmen in die ``PATH``-Variable
aufgenommen werden. Im Fall der zsh hängt man dazu folgende Zeile an
``~/.zshrc`` an:

::

   ...
   export PATH=/home/mickraus/latex/texlive/branch2007/Master/bin/x86_64-unknown-freebsd7.0/:$PATH

Nach dem Neuanmelden des Benutzers oder dem erneuten Einlesen von
``.zshrc`` sollte TeXlive funktionieren.

* :ref:`genindex`

Zuletzt geändert: |date|

