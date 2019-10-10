M0n0wall & SIP
==============

.. |date| date::

.. sidebar:: Info

  .. image:: ../images/logo-freebsd.png

`M0n0wall <http://m0n0.ch/wall>`__ ist eine umfangreiche und einfach zu
konfigurierende Firewall-Distribution von FreeBSD.

Das Protokoll VoIP verträgt sich sehr schlecht mit NAT-Firewalls, welche
jedoch stark verbreitet sind und z.B. auch eine der Hauptfunktionen von
M0n0wall darstellt.

In den folgenden Varianten wird gezeigt, wie man ein VoIP-Device hinter
einer NAT-Firewall wie M0n0wall zum Laufen bekommen kann.

Variante: Port-Forward
----------------------

Mit folgenden statischen Portforwards auf direkt auf die Fritzbox ist
VoIP zum Laufen zu bekommen. Die Konfiguration ist getestet mit GMX
(bzw. United-Internet) und iptel.org .

NAT-Regeln
~~~~~~~~~~

Entsprechend den folgenden NAT-Regeln muss auch zusätzlich eine
Firewall-Regel angelegt werden, was bei m0n0wall aber automatisch
gemacht werden kann.

fritzbox = IP der Fritzbox

=== ===== ============== ======== ==============
If  Proto Ext.port range NAT IP   Int.port range
=== ===== ============== ======== ==============
WAN UDP   3478           fritzbox 3478
WAN UDP   3479           fritzbox 3479
WAN UDP   5060           fritzbox 5060
WAN UDP   5062           fritzbox 5062
WAN UDP   5070-5079      fritzbox 5070-5079
WAN UDP   7077-7081      fritzbox 7077-7081
WAN UDP   30000-30019    fritzbox 30000-30019
=== ===== ============== ======== ==============

Nachteile
~~~~~~~~~

Der deutliche Nachteil hierbei ist, dass die Registrierung beim
Provider-Registrar z.B. nach der 24-Stunden DSL-Trennung nicht
zuverlässig auf die neue IP-Adresse korrigiert wird. Es ist nicht
unüblich, dass man für Stunden nicht erreichbar ist. Außerdem ist diese
Konfiguration aufwändig und eben sehr statisch, so dass dies nicht
praktisch ist, wenn man mehrere VoIP-Telefone oder Softphonse wie
Twinkle, KPhone oder Linphone im Einsatz hat.

Variante: SIP-Proxy
-------------------

Seit der Version 1.3b7 enthält M0n0wall einen funktionierenden SIP-proxy
(siproxd), welcher die Problematik deutlich vereinfachen kann.

aktivieren des SIP-Proxys
~~~~~~~~~~~~~~~~~~~~~~~~~

In dem Webinterface wird dieser unter dem Menü **Services**-->**SIP
Proxy** aktiviert. Die folgenden Standardeinstellungen sollten für die
meisten Fälle korrekt sein:

================== =============
Enable SIP Proxy   [x]
================== =============
Interface          LAN
SID UDP port       5060
RTP UDP port range 7010 bis 7019
================== =============

Firewall-Regeln anlegen
~~~~~~~~~~~~~~~~~~~~~~~

Damit der SIP-Proxy von außen erreichbar ist, müssen für das
**WAN-Interface** zwei Regeln angelegt werden. Dies geschieht anders als
in dem Hilfetext beschrieben nicht automatisch:

===== ====== ==== =========== =========== ==================
Proto Source Port Destination Port        Description
===== ====== ==== =========== =========== ==================
UDP   \*     \*   WAN address 5060        SIProxd UDP-Port
UDP   \*     \*   WAN address 7010 - 7019 RTP UDP port Range
===== ====== ==== =========== =========== ==================

NAT-Regeln sind nicht weiter notwendig, da der siproxd dies auf
Applikationsebene als Gateway regelt.

Telefon konfigurieren
~~~~~~~~~~~~~~~~~~~~~

============== ============================================================
Benutzername   Anmeldename/Telefonummer des VoIP-Providers
============== ============================================================
Passwort       dito
Domain         die Domäne des VoIP-Providers (z.B. gmx.de oder iptel.org)
Registrar      ip oder hostname der m0n0wall, nicht registrar des Providers
Outbound-Proxy ip oder hostname der m0n0wall
============== ============================================================

**NAT-Durchtunnelung** oder STUN bzw. STUN-Server **abschalten**.

Einschränkungen
~~~~~~~~~~~~~~~

Diese Konfiguration funktioniert mit Softphones wie Twinkle - jedoch
bisher nicht mit einer Fritzbox, da man dort die Domain und den
Registrar nicht getrennt angeben kann. FIXME

* :ref:`genindex`

Zuletzt geändert: |date|

