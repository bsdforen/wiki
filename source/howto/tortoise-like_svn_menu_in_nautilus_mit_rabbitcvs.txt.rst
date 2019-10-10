Tortoise like SVN Menu in Nautilus
==================================

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png
  .. image:: ../images/logo-openbsd.png
  .. image:: ../images/logo-netbsd.png
  .. image:: ../images/logo-dragonflybsd.png

In den Gnome Explorer Nautilus (es geht auch mit Thunar) ein Tortoise
aehnliches Menu integrieren. Das Programm dafuer ist RabbitVCS:
http://www.rabbitvcs.org/

Vorschau
--------

|image0|

Benoetigte Pakete
-----------------

::

   devel/subversion
   x11-fm/py-nautilus
   devel/py-subversion
   devel/apr0
   devel/apr1
   devel/pysvn
   devel/py-configobj

Ich bin mir leider nicht ganz sicher welche apr\* Version genau
benoetigt wird, ich hatte bei mir vorsichtshalber

Den Rest der Dependencies kann man nochmal hier nachschlagen:
http://wiki.rabbitvcs.org/wiki/install/manual

Installation
------------

Download
~~~~~~~~

http://rabbitvcs.googlecode.com/files/rabbitvcs-0.14.2.1.tar.gz

.. _installation-1:

Installation
~~~~~~~~~~~~

::

   user> tar xvfz rabbitvcs-0.14.2.1.tar.gz
   user> cd rabbitvcs-0.14.2.1
   user> python setup.py build
   root> python setup.py install
   user> mkdir ~/.nautilus/python-extensions
   user> cp clients/nautilus/RabbitVCS.py ~/.nautilus/python-extensions

Fehler
~~~~~~

libpythonpython2.7.so.1.0
^^^^^^^^^^^^^^^^^^^^^^^^^

Bei mir kam der folgende Fehler nachdem ich dann nautilus ueber die
Konsole neugestartet habe.

::

   user> nautilus
   (nautilus:88497): Nautilus-Python-WARNING **: g_module_open libpython failed:
   Cannot open "/usr/local/lib/libpythonpython2.7.so.1.0"
   Traceback (most recent call last):
     File "/usr/local/lib/python2.7/site-packages/gtk-2.0/gobject/__init__.py", line 26, in <module>
       from glib import spawn_async, idle_add, timeout_add, timeout_add_seconds, \
     File "/usr/local/lib/python2.7/site-packages/gtk-2.0/glib/__init__.py", line 22, in <module>
       from glib._glib import *
   ImportError: /usr/local/lib/python2.7/site-packages/gtk-2.0/glib/_glib.so: Undefined symbol "_Py_ZeroStruct"''

Siehe dazu auch:
http://www.bsdforen.de/showthread.php?p=226160#post226160

Diesen konnte ich dann wie folgt beheben:

::

   root> ln -s /usr/local/lib/libpython2.7.so.1 /usr/local/lib/libpythonpython2.7.so.1.0

Keine Icons im Nautilus Menu
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

   gconftool-2 –type bool –set /desktop/gnome/interface/menus_have_icons true

Keine RabbitVCS Icons im Nautilus Menu
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Im rabbitVCS Ordner unter rabbitvcs-0.14.2.1/data/icons befinden sich
die Icons, diese wurden bei mir nicht mit in meine Gnome Theme
uebernommen, darum musste ich sie manuell in
~/.icons/<Icon_Theme_Folder> kopieren

.. |image0| image:: images/rabbitcvs_menu.png

* :ref:`genindex`

Zuletzt geändert: |date|

