DISCLAIMER: USE AT YOUR OWN RISK!!!!

The purpose of this howto is to show what my configuration looks like after I installed OpenBSD 7.3 on my APU-PC, so that it can act as a router.
Some of the steps are being done with the help of QEMU on my Desktop. Hence, the QEMU-Commandlines are specific to the particular Operating system on it. Which, concidentally, is also OpenBSD 7.3.
It is going to use unbound as DNS, and dhcpd, along with pf as firewall and PPPoE for Internet connectivity.
In addition to this, an Ethernet Connected Phone will also be added. When ready, the machine can be used as a replacement for any Fritz-Box.


0.) Bill of materials
My hardware setup looks like this:

    1 DSL Connection by M-Net
    1 DSL Modem (Not a DSL router)
    1 APU-PC (But any other PC would do)
    1 Printer
    1 TV
    1 Telephone (VoIP, Ethernet connection)
    1 WiFi Access Point
    1 Hub
    Several Notebooks/PCs/SGI Indigos/Sun Ultras...

(Phew! Quite the collection)

Since the APU-PC has 3 Ethernet connections, I decided to create 2 subnets:

    192.168.3.0/24 for wired Ethernet
    192.168.70.0/24 for WiFi Ethernet

Why those numbers, you might ask? WTF not? ;)

My subnet will be called dettus.privates


1.) Installation in QEMU
The APU-PC has an SD-Card slot, and it boots from there. Since it was more conveniant for me, I decided to prepare the Image for the SD Card in QEMU.
I used my trusty OpenBSD desktop for that.
The SD Card I had was 4 GByte in size, but 3 Gigabyte are enough for a DSL router. The closest OpenBSD mirror to my home is in Erlangen, so downloading the latest installation ISO, and starting the virtual machine can be done with the following commands:

Bash:

dumbuser@desktop> su -
root@desktop> dd if=/dev/zero of=router.img bs=1M count=3096
root@desktop> ftp https://ftp.fau.de/pub/OpenBSD/7.3/amd64/install73.iso
root@desktop> qemu-system-x86_64 -m 1024 -smp cpus=1,cores=1 \
    -drive format=raw,file=router.img \
    -cdrom install73.iso -boot d \
    -device virtio-net-pci,netdev=network0 \
    -netdev user,id=network0,hostfwd=tcp::2001-:22 \
    -serial telnet:localhost:4321,server,nowait     \
    -display vnc=:1


Pro Tip: You are going to copy and paste those lines anyways. Paste them in a shell script. WAY easier to correct typos this way.

Once QEMU is running, it can be accessed with the vnc viewer on console :1

Bash:

dumbuser@desktop> vncviewer :1


(side note: I used tigervnc.)


2.) Installing OpenBSD
If you have done it once, you can do this in your sleep. If not, do not hesitate to press CTRL+C and CTRL+D to start the installation over.

* I must apologize, I do not remember the order in which those questions came up. *
One thing you will have to decide on is the HOSTNAME, I have decided to use router.dettus.privates.

The (W)hole Disk can be used for OpenBSD, I wanted to create a (C)ustom Layout:
In the disklabel editor, I created three slices:

a: size 2G, mount point /
b: size 512M, type swap
d: size 512M (the rest), mount point /tmp

When the packages are to be selected, I deselected the Xenocara packages by typing in

-x*

As for the installation medium: Chose the cd0 option, it is the fastest. Or a network mirror if you prefer.


* VERY IMPORTANT *
Set up a dumbuser.
(Y)es, run sshd
(N)o, do not permit root login
(Y)es, change the console to COM0
Baud rate: 115200

The APU PC comes with a vintage UART on the front side, so Answering the last two questions will allow you to access the console in case of a Network meltdown.

(H)alt the machine when everything has been installed.


3.) Running the VM
In theory, you are ready to write the image to the SD Card and perform the next couple of steps directly on the APU-PC. TRUST ME: This is easier.

The installation required a graphic console, and VNC was used to do the job. But since the console was changed to COM0, it is now possible to have the serial output in the same terminal window in which QEMU is running:

Bash:

root@desktop> qemu-system-x86_64 -m 1024 -smp cpus=1,cores=1 \
    -drive format=raw,file=router.img \
    -cdrom install73.iso -boot c \
    -device virtio-net-pci,netdev=network0 \
    -netdev user,id=network0,hostfwd=tcp::2001-:22 \
    -nographic


With this, it is now possible to connect from a second terminal window with ssh . Do this, become root:

Bash:

dumbuser@desktop> ssh -p 2001 dumbuser@localhost
dumbuser@router> su -
Password:
root@router> whoami


4.) Base configuration
In the /etc/rc.conf, I changed the following lines:

Code:

library_aslr=NO        # NOT reordering the libraries at boot time is faster
dhcpd_flags=NO        # This is correct
unbound_flags=""    # Not sure if this is the proper way to write, it worked
resolvd_flags=NO    # Otherwise, it screws up the resolv.conf



I also created an /etc/sysctl.conf:
Bash:

echo "net.inet.ip.forwarding=1" >/etc/sysctl.conf


And changed the installation source:
Bash:

echo "https://ftp.fau.de/pub/OpenBSD" >/etc/installurl



At this point, it is a good idea to reboot

Bash:

root@router> reboot



5.) Network configuration
The APU-PC has three Ethernet connectors: em0, em1 and em2.
The connection to the internet is realized through PPPoE, and my Internet provider needed vlan40 for some reason.

I wanted to have two networks: 192.168.3.0/24 and 192.168.70.0/24. I decided to use em0 for the 3.x subnet, em1 for connecting the DSL modem and em2 for the 70.x subnet.

So, in the end, I had to create 5 hostname files:
Bash:

root@router> echo "inet 192.168.3.1 255.255.255.0 NONE" >/etc/hostname.em0
root@router> echo "up" >/etc/hostname.em1
root@router> echo "inet 192.168.70.1 255.255.255.0 NONE" >/etc/hostname.em2
root@router> echo "vlan 40 vlandev em1 up" >/etc/hostname.vlan40
root@router> echo "inet 0.0.0.0 0.0.0.1 pppoedev vlan40 autoproto chap authname NOTGONNAWRITEHERE@mdsl.mnet-online.de authkey NOTTHISONEEITHER mtu 1452 up" >/etc/hostname.pppoe0


Spoiler alert: hostname.pppoe0 does not work for you. ;)


6.) DNS configuration: unbound
Again, I had two subnets. 192.168.3.0/24 and 192.168.70.0/24. In the previous chapter, I set my IP addresses for this machine to the IP addresses 192.168.3.1 and 192.168.70.1
I had a TV and a printer, and I wanted those to have the same IP addresses
with the DHCP later, as well as the access point.

So, in the end, my /var/unbound/unbound.conf looked like this

Code:

server:
    access-control: 0.0.0.0/0 refuse
    access-control: 127.0.0.0/8 allow
    access-control: ::0/0 refuse
    access-control: ::1 allow
    access-control: 192.168.3.0/24 allow
    access-control: 192.168.70.0/24 allow

    verbosity: 0
    interface: 127.0.0.1
    interface: 192.168.3.1
    interface: 192.168.70.1
    port:    53
    do-ip4:    yes
    do-ip6:    no
    do-udp: yes
    do-tcp:    yes

    prefer-ip6: no
    harden-glue: yes
    harden-dnssec-stripped: yes

    use-caps-for-id: no

    edns-buffer-size: 1232
    prefetch: yes
    num-threads: 1

    so-rcvbuf: 1m

    private-address: 192.168.0.0/16


local-zone:    "dettus.privates." static
    # wired network
    local-data:    "router.dettus.privates. IN A 192.168.3.1"
    local-data:    "printer.dettus.privates. IN A 192.168.3.2"
    local-data:    "television.dettus.privates. IN A 192.168.3.3"
    local-data:    "telephone.dettus.privates. IN A 192.168.3.4"
    local-data-ptr:    "192.168.3.1 router.dettus.privates"
    local-data-ptr: "192.168.3.2 printer.dettus.privates"
    local-data-ptr:    "192.168.3.3 television.dettus.privates"
    local-data-ptr:    "192.168.3.4 telephone.dettus.privates"
    # WiFi network
    local-data:    "router2.dettus.privates. IN A 192.168.70.1"
    local-data:    "access-point.dettus.privates IN A 192.168.70.2"
    local-data-ptr:    "192.168.70.1 router2.dettus.privates"
    local-data-ptr:    "192.168.70.2 access-point.dettus.privates"



Afterwards, it should be possible to enable unbound.

Bash:

root@router> unbound-checkconf /var/unbound/etc/unbound.conf
root@router> rcctl enable unbound
root@router> rcctl disable resolvd  # this program screws up the resolv.conf
root@router> echo "nameserver 127.0.0.1" >/etc/resolv.conf


* IF YOU ARE STILL INSIDE THE VM, IT WILL FAIL TO START THE UNBOUND SERVER! *



7.) DHCP configuration: dhcpd

The way I did it was by creating two configuration files, and to start the
dhcpd twice, one for each interface em0 and em2.

I wanted my appliances to have the same IP address. So after I found out their MAC-Adress, I was able to write the configurations.


Since the IP addresses where already configured in the nameserver, it was possible to use hostnames in the /etc/dhcpd.em0: (Note the special nameserver for the printer)


Bash:

shared-network LOCAL-NET
{
    option  domain-name "dettus.privates";
    option  domain-name-servers 192.168.3.1;

    default-lease-time 5000;
    max-lease-time 7200;

    subnet 192.168.3.0 netmask 255.255.255.0
    {
        range 192.168.3.111 192.168.3.150;
        option routers 192.168.3.1;
        option domain-name-servers 192.168.3.1;
    }

    host printer
    {
        hardware ethernet ab:cd:ef:00:01:23;
        fixed-address printer.dettus.privates;
        option routers 192.168.3.1;
        option domain-name-servers 127.0.0.1;
    }
    host television
    {
        hardware ethernet ab:cd:ef:00:11:42;
        fixed-address television.dettus.privates;
        option routers 192.168.3.1;
        option domain-name-servers 192.168.23.1;
    }
    host telephone
    {
        hardware ethernet be:af:87:81:12:65;
        fixed-address telephone.dettus.privates;
        option routers 192.168.3.1;
        option domain-name-servers 192.168.23.1;
    }
}



The /etc/dhcpd.em2 looked similar, albeit for different subnet:

Bash:

shared-network LOCAL-NET
{
    option  domain-name "dettus.privates";
    option  domain-name-servers 192.168.70.1;

    default-lease-time 5000;
    max-lease-time 7200;

    subnet 192.168.70.0 netmask 255.255.255.0
    {
        range 192.168.70.211 192.168.3.250;
        option routers 192.168.70.1;
        option domain-name-servers 192.168.70.1;
    }

    host access-point
    {
        hardware ethernet 80:86:23:de:ad:42;
        fixed-address access-point.dettus.privates;
        option routers 192.168.70.1;
        option domain-name-servers 192.168.70.1;
    }
}


8.) The pf.conf

Apparently, port 5060 is needed for the VoIP telephone. In the end, my /etc/pf.conf looked like this:


Bash:

set skip on lo
block return
pass
block return in on ! lo0 proto tcp to port 6000:6010
ext_if="pppoe0"
int0_if="em0"
int2_if="em2"
match on $ext_if scrub (max-mss 1340)
match out on pppoe0 inet from $int0_if:network to any nat-to ($ext_if)
match out on pppoe0 inet from $int2_if:network to any nat-to ($ext_if)
#match in on $ext_if proto { tcp, udp } from any to any port 80 rdr-to 192.168.3.14 port 8080  # In case I want to run a HTTP server one day
# For the telephone
pass in quick on $ext_if proto { tcp, udp } from $ext_if to any port 5060 keep state






9.) My patches (aka spit and duct-tape)
The way I configured everything was somehow incompatible with the way OpenBSD works. For some reason, the PPPoE network was not coming up at the right point in time, so I had to hack my /etc/rc.

Usually, the /etc/rc ends like this:

Code:

# Re-link the kernel, placing the objects in a random order.
# Replace current with relinked kernel and inform root about it.
/usr/libexec/reorder_kernel &

date
exit 0


I took the liberty of adding a few lines:

Code:

# Re-link the kernel, placing the objects in a random order.
# Replace current with relinked kernel and inform root about it.
/usr/libexec/reorder_kernel &


# go online
ifconfig pppoe0 `cat /etc/hostname.pppoe0`
sleep 5
# set the proper default route
route add default `ifconfig pppoe0 | grep "inet " | awk -F" " '{ print $4; }' -`
# restart the firewall
pfctl -d
pfctl -e -f /etc/pf.conf
# start the dhcp daemons
dhcpd -c /etc/dhcpd.em0 em0
dhcpd -c /etc/dhcpd.em2 em2

date
exit 0



THERE IS MAYBE A PROPER WAY TO DO THIS, AND THOSE CHANGES WILL BE LOST DURING A SYSUPGRADE, but it worked for me! :)


10.) Finally: Prepare the SD-Card for the APU-PC

Once the changes are done, shut down the VM:

Bash:

root@router> halt



Plug in the SD Card to your desktop. Find out which the proper device for the SD card is.

Bash:

root@desktop> dmesg


Lets assume it is sd9. (Even if you are doing this on Linux, you should find it out the same way)
Bash:

root@desktop> dd if=router.img of=/dev/rsd9c bs=1M



11.) First boot

Once the SD card has been prepared, put it inside your APU-PC, connect it via serial cable to your desktop and connect to it. Personally, I recommend minicom.
Bash:

root@desktop> minicom -s    # to set it up


The device on my machine was /dev/ttyU0. I had to disable the hardware flow control, but once I have saved the configuration, I was able to use

Bash:

root@desktop> minicom


I suppose, cu works as well, but I have not used it:

Bash:

root@desktop> cu -l /dev/ttyU0 -s 115200


Make sure your APU-PC boots correctly. Have a look at the /etc/resolv.conf, or
if resolvd screwed it up before:

Bash:

root@router> cat /etc/resolv.conf
nameserver 127.0.0.1


If it did, make sure resolvd is disabled, reboot, rewrite /etc/resolv.conf.

Test your DSL connection
Bash:

root@router> ifconfig pppoe0
root@router> ping 131.188.12.211



Test unbound by using

Bash:

root@router> nameserver
> server 127.0.0.1
> ftp.fau.de
Server:        127.0.0.1
Address:    127.0.0.1#53

Non-authoritative answer:
Name:    ftp.fau.de
Address: 131.188.12.211
> exit


Make sure the dhcpds are working
Bash:

root@router> ps auxww | grep -i dhcp
_dhcp    12458  0.0  0.0   832  1532 ??  Ipc     7:00AM    0:00.01 dhcpd -c /etc/dhcpd.em0 em0
_dhcp     2055  0.0  0.0   824  1492 ??  Ipc     7:00AM    0:00.01 dhcpd -c /etc/dhcpd.em2 em2


And your new router is ready.
Enjoy!


dettus@dettus.net
P.S.: Check out my retro gameserver on https://magneticscrolls.net



Keywords: OpenBSD, DSL, pppoe, unbound, dhcpd, pf.conf forwarding, Internet router, DSL router
