MPI auf FreeBSD
===============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

Was ist MPI?
------------

MPI, das s.g. `Message Passing
Interface <http://de.wikipedia.org/wiki/Message_Passing_Interface>`__,
dient dazu um Nachrichten und Daten zwischen verteilten Rechnern und
parallelisierten Programmen auszutauschen. Es wird insbesonde für
numerische Berechnungen auf mehreren Rechnern und sogar auf großen
Clustern eingesetzt, um die Berechnungen erheblich zu beschleunigen.

Was bringt mit das?
-------------------

MPI lohnt sich zB schon, wenn man 2 Kerne hat, oder mehrere Rechner, die
man benutzen möchte. Aber es muss gesagt werden, das sich das nur auf
MPI basierte Programme beschränkt. Im folgenden wird es eine kleine
Anleitung zur Installation von OpenMPI, einer freien MPI
Implementierung, geben und eine Anleitung wie man es mit HPL voll
ausreizt. HPL, der High Performance Linpack ist ein Benchmark, um die
Leistungsfähigkeit von Clustern zu testen, mit dem übrigens auch die
Floating Point Operations per second (Flops) der Top500 Supercomputer
gemessen werden.

Installation
------------

OpenMPI
~~~~~~~

Zuerst sollte man sein System und insbesondere die Ports auf den
aktuellsten Stand bringen. Dann installiert man mit einem:

::

  $ cd /usr/ports/net/openmpi 
  $ make install clean

OpenMPI auf allen Rechnern, die man für seinen "Cluster" benutzen möchte
und damit ist die Installation auch schon abgeschlossen.

Einrichtung von SSH für OpenMPI
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Für den Fall das man mehr als einen Rechner für seinen "Cluster"
benutzen möchte, lohnt es sich den ssh-agent zu benutzen. So erspart man
sich das lästige eintippen des Passworts nachdem man sein Programm
gestartet hat. Zuerst erzeugt man sich ein Schlüsselpaar den man dann
auf seinen Zielrechner ablegt:

::

  $ ssh-keygen -t rsa Generating public/private rsa key pair. 
  Enter file in which to save the key (/home/michi/.ssh/id_rsa): <Enter> 
  Enter passphrase (empty for no passphrase): Enter same passphrase again:

Man kann die passphrase selbstverständlich auch leer lassen, dann muss
man keinen ssh-agent benutzen. Trotzdem wird das hier am Beispiel von
ssh-agent gezeigt, da es nur ein unwesentlicher Mehraufwand ist. Zuerst
muss aber noch der öffentliche Schlüssel auf den Zielrechner gebracht
werden.

::

  $ cat $HOME/.ssh/id_rsa.pub\| ssh [user@]zielrechner "cat >> .ssh/authorized_keys"

Beim Login auf den "zielrechner" sollte danach nach der Passphrase
gefragt werden. Um sich das zu erparen kann man ssh-agent wie folgt
benutzen:

::

  $ ssh-agent $SHELL $ ssh-add Enter passphrase for /home/michi/.ssh/id_rsa: 
  Identity added: /home/michi/.ssh/id_rsa (/home/michi/.ssh/id_rsa)

Mit ssh-agent $SHELL wird eine neue Shell in der jetzigen gestartet und
ssh-add fügt den Schlüssel hinzu. Jetzt muss man nur noch einmal die
Passphrase eingeben und solte sich einloggen können ohne Passphrase,
solange man mit "exit" diese Shell nicht verlässt.

HPL
~~~

HPL kann `hier <http://www.netlib.org/benchmark/hpl/>`__ geholt werden.
Es braucht aber noch zuerst eine s.g. BLAS Library, die algebraische
Routinen enthält. Es gibt verschiedene BLAS Libraries, wovon sich die
GotoBLAS Library als sehr innovativ und effizient (schnell)
herausgestellt hat. Sie ist
`hier <http://www.tacc.utexas.edu/resources/software/>`__ zu finden,
wofür man sich aber anmelden muss. Möchte man das nicht kann man auch zu
einer anderen BLAS Library greifen. zb der cBLAS, die auch in den Ports
vorhanden ist.

Vorraussetzungen
^^^^^^^^^^^^^^^^

Folgende Dinge sollten installiert sein:

-  gmake
-  gcc42

Bauen der GotoBLAS Library
^^^^^^^^^^^^^^^^^^^^^^^^^^

Als erstes würde ich empfehlen das man die GotoBLAS Library aus den
Ports baut. Dazu meldet man sich wie oben beschrieben auf der Seite an
und lädt das Targeknäuell runter und kopiert es in sein
/usr/ports/distfiles Ordner. Anschließend ist es einfach mit::

  $ cd /usr/ports/math/gotoblas/ 
  $ make install clean

zu installieren.

Wenn man wirklich das letzte Quäntchen aus seinem "Cluster" rausholen
möchte, dann sollte man GotoBLAS selber mit seinen Optimierungen für
seine Maschine bauen. Jedoch ist das keine leichte Aufgabe da einige
Änderungen nötig sind. Ich skizziere hier nur kurz wie man da vorgehen
könnte: Nachdem man den Tarballen runtergeladen hat packt man die Goto
aus:

::

  $ tar -zxvf GotoBLAS-1.22.tar.gz 
  $ cd GotoBLAS

Danach muss die Datei "Makefile.rule" geändert und F_COMPILER angepasst
werden. Dort und in einigen weiteren sind noch einige Anpassungen nötig
auf die ich aber jetzt erstmal verzichten möchte, weil das recht
umfangreich ist. Falls man sich durch die ganzen Makefiles gewühlt hat,
übersetzt man das Programm mit **G**\ make::

  $ gmake

Es hilft auf jeden Fall mal einen Blick in die Ports Patchfiles zu
werfen.

Bauen von HPL
^^^^^^^^^^^^^

Da der fleißige Leser bestimmt schon HPL heruntergeladen hat kann dieses
nun jetzt auch ausgepackt werden::

  $ tar -zxvf hpl.tar.gz $ cd hpl

Jetzt kopiert man sich am besten ein Makefile aus setup. Es bietet sich
zB: setup/Make.FreeBSD_PIV_CBLAS an. Und um Schreibarbeit zu sparen
nennt man es am besten gleich um::

  $ cp setup/Make.FreeBSD_PIV_CBLAS Make.i386 
  $ vi Make.i386

Jetzt müssen wir dort einige Informationen von Hand eintragen. Da sie
eigentlich gut beschrieben sind sollte das selbstklärend sein, aber ich
beschreibe kurz worauf man achten sollte und schreibe ein Beispiel:

Unbedingt muss man folgende Punkte anpassen:

::

   ARCH         = i386

Oder wie auch immer man sein Make.xyz file genannt hat. Danach muss man
die richtigen Pfade zu HPL, MPI, GotoBLAS und den MPI Linkern setzen:

::

   TOPdir       = $(HOME)/hpl
   [...]
   MPdir        = /usr/local/mpi/openmpi

   MPinc        = -I$(MPdir)/include
   MPlib        = $(MPdir)/lib/libmpi.so
   [...]
   LAdir        = /usr/local/lib/
   LAinc        =
   LAlib        = $(LAdir)/libgoto.so
   [...]
   CC           = $(MPdir)/bin/mpicc
   [...]
   LINKER       = $(MPdir)/bin/mpif77
   [...]

Hier ist ein Beispiel wie es aussehen könnte, wenn man GotoBLAS aus den
Ports gebaut hat. Aber man kann es nicht oft genug betonen, wie wichtig
es ist, es sich von vorne bis hinten genau durchzulesen und seine
passenden Pfade einzutragen!

::

   #  
   #  -- High Performance Computing Linpack Benchmark (HPL)                
   #     HPL - 1.0a - January 20, 2004                          
   #     Antoine P. Petitet                                                
   #     University of Tennessee, Knoxville                                
   #     Innovative Computing Laboratories                                 
   #     (C) Copyright 2000-2004 All Rights Reserved                       
   #                                                                       
   #  -- Copyright notice and Licensing terms:                             
   #                                                                       
   #  Redistribution  and  use in  source and binary forms, with or without
   #  modification, are  permitted provided  that the following  conditions
   #  are met:                                                             
   #                                                                       
   #  1. Redistributions  of  source  code  must retain the above copyright
   #  notice, this list of conditions and the following disclaimer.        
   #                                                                       
   #  2. Redistributions in binary form must reproduce  the above copyright
   #  notice, this list of conditions,  and the following disclaimer in the
   #  documentation and/or other materials provided with the distribution. 
   #                                                                       
   #  3. All  advertising  materials  mentioning  features  or  use of this
   #  software must display the following acknowledgement:                 
   #  This  product  includes  software  developed  at  the  University  of
   #  Tennessee, Knoxville, Innovative Computing Laboratories.             
   #                                                                       
   #  4. The name of the  University,  the name of the  Laboratory,  or the
   #  names  of  its  contributors  may  not  be used to endorse or promote
   #  products  derived   from   this  software  without  specific  written
   #  permission.                                                          
   #                                                                       
   #  -- Disclaimer:                                                       
   #                                                                       
   #  THIS  SOFTWARE  IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
   #  ``AS IS'' AND ANY EXPRESS OR IMPLIED WARRANTIES,  INCLUDING,  BUT NOT
   #  LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
   #  A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE UNIVERSITY
   #  OR  CONTRIBUTORS  BE  LIABLE FOR ANY  DIRECT,  INDIRECT,  INCIDENTAL,
   #  SPECIAL,  EXEMPLARY,  OR  CONSEQUENTIAL DAMAGES  (INCLUDING,  BUT NOT
   #  LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
   #  DATA OR PROFITS; OR BUSINESS INTERRUPTION)  HOWEVER CAUSED AND ON ANY
   #  THEORY OF LIABILITY, WHETHER IN CONTRACT,  STRICT LIABILITY,  OR TORT
   #  (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
   #  OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE. 
   # ######################################################################
   #  
   # ----------------------------------------------------------------------
   # - shell --------------------------------------------------------------
   # ----------------------------------------------------------------------
   #
   SHELL        = /bin/sh
   #
   CD           = cd
   CP           = cp
   LN_S         = ln -s
   MKDIR        = mkdir
   RM           = /bin/rm -f
   TOUCH        = touch
   #
   # ----------------------------------------------------------------------
   # - Platform identifier ------------------------------------------------
   # ----------------------------------------------------------------------
   #
   ARCH         = i386
   #
   # ----------------------------------------------------------------------
   # - HPL Directory Structure / HPL library ------------------------------
   # ----------------------------------------------------------------------
   #
   TOPdir       = $(HOME)/hpl
   INCdir       = $(TOPdir)/include
   BINdir       = $(TOPdir)/bin/$(ARCH)
   LIBdir       = $(TOPdir)/lib/$(ARCH)
   #
   HPLlib       = $(LIBdir)/libhpl.a
   #
   # ----------------------------------------------------------------------
   # - Message Passing library (MPI) --------------------------------------
   # ----------------------------------------------------------------------
   # MPinc tells the  C  compiler where to find the Message Passing library
   # header files,  MPlib  is defined  to be the name of  the library to be
   # used. The variable MPdir is only used for defining MPinc and MPlib.
   #
   MPdir        = /usr/local/mpi/openmpi

   MPinc        = -I$(MPdir)/include
   MPlib        = $(MPdir)/lib/libmpi.so
   #
   # ----------------------------------------------------------------------
   # - Linear Algebra library (BLAS or VSIPL) -----------------------------
   # ----------------------------------------------------------------------
   # LAinc tells the  C  compiler where to find the Linear Algebra  library
   # header files,  LAlib  is defined  to be the name of  the library to be
   # used. The variable LAdir is only used for defining LAinc and LAlib.
   #
   hi/prog/GotoBLAS
   LAdir       = /usr/local/lib/
   LAinc        =
   LAlib        = $(LAdir)/libgoto.so
   #
   # ----------------------------------------------------------------------
   # - F77 / C interface --------------------------------------------------
   # ----------------------------------------------------------------------
   # You can skip this section  if and only if  you are not planning to use
   # a  BLAS  library featuring a Fortran 77 interface.  Otherwise,  it  is
   # necessary  to  fill out the  F2CDEFS  variable  with  the  appropriate
   # options.  **One and only one**  option should be chosen in **each** of
   # the 3 following categories:
   #
   # 1) name space (How C calls a Fortran 77 routine)
   #
   # -DAdd_              : all lower case and a suffixed underscore  (Suns,
   #                       Intel, ...),                           [default]
   # -DNoChange          : all lower case (IBM RS6000),
   # -DUpCase            : all upper case (Cray),
   # -DAdd__             : the FORTRAN compiler in use is f2c.
   #
   # 2) C and Fortran 77 integer mapping
   #
   # -DF77_INTEGER=int   : Fortran 77 INTEGER is a C int,         [default]
   # -DF77_INTEGER=long  : Fortran 77 INTEGER is a C long,
   # -DF77_INTEGER=short : Fortran 77 INTEGER is a C short.
   #
   # 3) Fortran 77 string handling
   #
   # -DStringSunStyle    : The string address is passed at the string loca-
   #                       tion on the stack, and the string length is then
   #                       passed as  an  F77_INTEGER  after  all  explicit
   #                       stack arguments,                       [default]
   # -DStringStructPtr   : The address  of  a  structure  is  passed  by  a
   #                       Fortran 77  string,  and the structure is of the
   #                       form: struct {char *cp; F77_INTEGER len;},
   # -DStringStructVal   : A structure is passed by value for each  Fortran
   #                       77 string,  and  the  structure is  of the form:
   #                       struct {char *cp; F77_INTEGER len;},
   # -DStringCrayStyle   : Special option for  Cray  machines,  which  uses
   #                       Cray  fcd  (fortran  character  descriptor)  for
   #                       interoperation.
   #
   F2CDEFS      = -DAdd_ -DF77_INTEGER=int -DStringSunStyle
   #
   # ----------------------------------------------------------------------
   # - HPL includes / libraries / specifics -------------------------------
   # ----------------------------------------------------------------------
   #
   HPL_INCLUDES = -I$(INCdir) -I$(INCdir)/$(ARCH) $(LAinc) $(MPinc)
   HPL_LIBS     = $(HPLlib) $(LAlib) $(MPlib)
   #
   # - Compile time options -----------------------------------------------
   #
   # -DHPL_COPY_L           force the copy of the panel L before bcast;
   # -DHPL_CALL_CBLAS       call the cblas interface;
   # -DHPL_CALL_VSIPL       call the vsip  library;
   # -DHPL_DETAILED_TIMING  enable detailed timers;
   #
   # By default HPL will:
   #    *) not copy L before broadcast,
   #    *) call the BLAS Fortran 77 interface,
   #    *) not display detailed timing information.
   #
   HPL_OPTS     =
   #
   # ----------------------------------------------------------------------
   #
   HPL_DEFS     = $(F2CDEFS) $(HPL_OPTS) $(HPL_INCLUDES)
   #
   # ----------------------------------------------------------------------
   # - Compilers / linkers - Optimization flags ---------------------------
   # ----------------------------------------------------------------------
   #
   CC           = $(MPdir)/bin/mpicc
   CCNOOPT      = $(HPL_DEFS)
   CCFLAGS      = $(HPL_DEFS) -fomit-frame-pointer -O3 -funroll-loops
   #
   # On some platforms,  it is necessary  to use the Fortran linker to find
   # the Fortran internals used in the BLAS library.
   #
   #LINKER       = /usr/bin/f77
   #LINKER       = /usr/local/mpi/openmpi/bin/mpif77
   LINKER      = $(MPdir)/bin/mpif77
   CCNOOPT      = $(HPL_DEFS)
   LINKFLAGS    = $(CCFLAGS)
   #
   ARCHIVER     = ar
   ARFLAGS      = r
   RANLIB       = /usr/bin/ranlib
   #
   # ----------------------------------------------------------------------

Jetzt da das schlimmste geschafft ist kann ein Versuch gestartet werden,
HPL zu bauen:

::

  $ gmake arch=i386

Falls es Probleme gibt und man etwas an seinem Makefile korrigiert hat
lohnt sich ein::

  $ gmake arch=i386 clean

bevor man es versucht erneut zu bauen.

Danach sollte in "bin/i386/" eine "HPL.dat" und eine "xhpl" liegen.

Bedienung von OpenMPI am Beispiel von HPL
-----------------------------------------

Jetzt wo wir HPL installiert haben, müssen wir zuerst die HPL.dat
anpassen. Doch zuerst sollte man wissen, was HPL eigentlich macht: HPL
erzeugt eine Matrix und führt anschließend LR-Zerlegung durch und das
falls mehrere Prozesse gestartet werden, parallel. Die Rechenschritte
die es dazu benötigt pro Zeit geben dann die "Flops" an. Was jetzt in
der HPL.dat steht sind Informationen über die Matrix und das Verfahren
mit welchem HPL sie zerlegen soll. Hier kann man bei HPL am meisten
tunen. Die wichtigsten Sachen nun am folgenden Beispiel:

::

   HPLinpack benchmark input file
   Innovative Computing Laboratory, University of Tennessee
   HPL.out      output file name (if any)
   6            device out (6=stdout,7=stderr,file)
   1            # of problems sizes (N)
   5000  Ns
   1            # of NBs
   200       NBs
   0            PMAP process mapping (0=Row-,1=Column-major)
   3            # of process grids (P x Q)
   1 2 1        Ps
   2 1 2        Qs
   16.0         threshold
   3            # of panel fact
   0 1 2        PFACTs (0=left, 1=Crout, 2=Right)
   2            # of recursive stopping criterium
   2 4          NBMINs (>= 1)
   1            # of panels in recursion
   2            NDIVs
   3            # of recursive panel fact.
   0 1 2        RFACTs (0=left, 1=Crout, 2=Right)
   1            # of broadcast
   0            BCASTs (0=1rg,1=1rM,2=2rg,3=2rM,4=Lng,5=LnM)
   1            # of lookahead depth
   0            DEPTHs (>=0)
   2            SWAP (0=bin-exch,1=long,2=mix)
   64           swapping threshold
   0            L1 in (0=transposed,1=no-transposed) form
   0            U  in (0=transposed,1=no-transposed) form
   1            Equilibration (0=no,1=yes)
   8            memory alignment in double (> 0)

5000 ist die Größe unserer Matrix (also eine 5000x5000 Matrix), sie hat
damit eine Anzahl von 5000*5000 Elemente. Hier unten wird das Prozessor
Gitter gewählt::

  3            # of process grids (P x Q)
  1 2 1        Ps
  2 1 2        Qs

In diesem Fall würden wir 2 Prozesse (2*1) brauchen. Die 3 bedeutet das wir 3
verschiedene Gitter angegeben haben die HPL alle probieren soll, wobei 2 davon
gleich sind.

MPI Starten
~~~~~~~~~~~

Wenn man einen Dual-Core Rechner hat, kann man das sogar auf einem
Rechner ausprobieren, ohne einen anderen Computer über ein LAN, oder IB
zu haben. Das sähe dann wie folgt aus::

  $ mpirun -H localhost -np 2 ./xhpl

-H teilt MPI mit auf welchen Hosts er arbeiten soll.  Hier nur auf "locahost".
-np sind die maximal auszuführende Anzahl an Prozessen, also 2 in diesem Fall.
Das wird auch min. benötigt weil in der HPL.dat auch 2 Prozesse angegeben
haben.

Anschließend sollte jetzt so etwas wie folgt kommen:

::

  $ mpirun -H localhost -np 2 ./xhpl
  ============================================================================
  HPLinpack 1.0a  --  High-Performance Linpack benchmark  --   January 20, 2004
  Written by A. Petitet and R. Clint Whaley,  Innovative Computing Labs.,  UTK
  ============================================================================
  
  An explanation of the input/output parameters follows:
  T/V    : Wall time / encoded variant.
  N      : The order of the coefficient matrix A.
  NB     : The partitioning blocking factor.
  P      : The number of process rows.
  Q      : The number of process columns.
  Time   : Time in seconds to solve the linear system.
  Gflops : Rate of execution for solving the linear system.
  
  The following parameter values will be used:
  
  N      :    5000
  NB     :     200
  PMAP   : Row-major process mapping
  P      :       1        2        1
  Q      :       2        1        2
  PFACT  :    Left    Crout    Right
  NBMIN  :       2        4
  NDIV   :       2
  RFACT  :    Left    Crout    Right
  BCAST  :   1ring
  DEPTH  :       0
  SWAP   : Mix (threshold = 64)
  L1     : transposed form
  U      : transposed form
  EQUIL  : yes
  ALIGN  : 8 double precision words
  
  ----------------------------------------------------------------------------
  
  - The matrix A is randomly generated for each test.
  - The following scaled residual checks will be computed:
     1) ||Ax-b||_oo / ( eps * ||A||_1  * N        )
     2) ||Ax-b||_oo / ( eps * ||A||_1  * ||x||_1  )
     3) ||Ax-b||_oo / ( eps * ||A||_oo * ||x||_oo )
  - The relative machine precision (eps) is taken to be          1.110223e-16
  - Computational tests pass if scaled residuals are less than           16.0
  
  ============================================================================
  T/V                N    NB     P     Q               Time             Gflops
  ----------------------------------------------------------------------------
  WR00L2L2        5000   200     1     2               6.87          1.213e+01
  ----------------------------------------------------------------------------
  ||Ax-b||_oo / ( eps * ||A||_1  * N        ) =        0.0125140 ...... PASSED
  ||Ax-b||_oo / ( eps * ||A||_1  * ||x||_1  ) =        0.0082611 ...... PASSED
  ||Ax-b||_oo / ( eps * ||A||_oo * ||x||_oo ) =        0.0016126 ...... PASSED
  ============================================================================
  T/V                N    NB     P     Q               Time             Gflops
  ----------------------------------------------------------------------------
  WR00L2L4        5000   200     1     2               6.78          1.230e+01
  ----------------------------------------------------------------------------
  ||Ax-b||_oo / ( eps * ||A||_1  * N        ) =        0.0125120 ...... PASSED
  ||Ax-b||_oo / ( eps * ||A||_1  * ||x||_1  ) =        0.0082598 ...... PASSED
  ||Ax-b||_oo / ( eps * ||A||_oo * ||x||_oo ) =        0.0016123 ...... PASSED

D.h. also, das auf diesem Rechner(Dual-Core) HPL mit 2 Prozessen, wo
jeder Prozess auf einen eigenen Kern gelegt wird, 12,3 GFlops erreicht.

Für den Fall das man die Gesamtleistung mehrerer Rechner testen möchte:

::

  $ mpirun -H localhost,host1,host2[...] -np <# processes> ./befehl

Das Leben kann man sich auch erleichtern in dem man einfach eine Datei
der Namen seiner Rechner anlegt und sie dann mit: -machinefile
<dateinamen> aufruft, wobei jeder Rechner in eine eigene Zeile kommt.

Fragen?
-------

Bei Anregungen, Fragen, oder Problemen bitte im Forum melden.

Links
-----

-  `MPI <http://www-unix.mcs.anl.gov/mpi/>`__
-  `OpenMPI <http://www.open-mpi.org/>`__
-  `MPICH2 <http://www.mcs.anl.gov/research/projects/mpich2/>`__ Eine
   andere MPI Implementierung

-  `Referenz <http://www.tu-chemnitz.de/informatik/RA/projects/mpihelp/>`__
   Sehr sehr nützliche Referenz, wenn man selber MPI programmieren
   möchte.

* :ref:`genindex`

Zuletzt geändert: |date|

