TeX Live
========

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

TeX Live ist die umfangreichste der TeX-Distributionen und wird v. a.
für die Nutzung von LaTeX eingesetzt. Seit Juni 2011 gibt es TeX Live
2011 nativ für FreeBSD.

Installation
------------

TeX Live 2011 ist nicht als Port oder Package für FreeBSD erhältlich.
Die Installation erfolgt mittels Perl-Installer. Als erstes muss der
Net-Installer von der `TeX-Live-Webseite heruntergeladen
werden <http://www.tug.org/texlive/acquire-netinstall.html>`__. Die
Installation `mittels
DVD-Image <http://www.tug.org/texlive/acquire.html>`__ ist ebenfalls
möglich.

Anschließend das Archiv
`install-tl-unx.tar.gz <http://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz>`__
entpacken und in den ``install-tl-*``-Unterordner wechseln. Der
Installer kann mittels

::

     ./install-tl

gestartet werden. Die weitere Vorgehensweise ist in der
`TeX-Live-2011-Anleitung <http://www.tug.org/texlive/doc/texlive-de/texlive-de.html>`__
beschrieben. Umgebungsvariablen werden durch den Installer nicht
gesetzt. Daher ist, z. B. für die amd64-Version, das Verzeichnis

::

    /usr/local/texlive/2011/bin/amd64-freebsd

noch in die ``$HOME/.profile`` oder ``$HOME/.cshrc`` einzutragen
(alternativ auch global unter ``/etc``).

Update
------

Eine vorhandene TeXLive-Installation wird mittels

::

    tlmgr update --all

aktualisiert.

* :ref:`genindex`

Zuletzt geändert: |date|

