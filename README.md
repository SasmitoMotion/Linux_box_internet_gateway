# Linux_box_internet_gateway
Control over Internet access

# Routing Internet Traffic to the Linux Box

The DHCP server its requires the configuration usually located at /etc/dhcp/dhcpd.conf it is able to assign fixed IP numbers to known hardware MAC addresses, and this is a crucial part of this system, here sample of dhcp.conf file
note : maybe you must to check file dhcp.conf and to edit the file your configuration wireless, check your ip address ifconfig

default-lease-time 1000000;
max-lease-time 1000000;
option subnet-mask 255.255.255.0;
option broadcast-address 192.168.1.255;
option routers 192.168.1.2;
option domain-name-servers 208.67.222.222, 208.67.220.220;
option domain-name "homenetwork";

subnet 192.168.1.0 netmask 255.255.255.0 {
   range 192.168.1.100 192.168.1.150;
}

#Static IP definitions

host fred-laptop {
  hardware ethernet C4:88:E5:FE:2A:48;
  fixed-address 192.168.1.60;
}

host mary-phone {
  hardware ethernet BC:47:60:9C:74:BB;
  fixed-address 192.168.1.61;
}

#More IP definitions here...

Each host entry assigns a (different) IP number to each device on the network. For this to work, you'll need the hardware MAC address. On a Windows PC, run the command ipconfig /all in a DOS box, 

# Arrange for Internet Traffic to be routed to the real internet router or modem

So what we need to do is to assign another IP to this Linux machine — I've assumed above that it is 192.168.1.2 — and arrange for packets sent to 192.168.1.2 to be forwarded to 192.168.1.1

To enable forwarding of packets at all, we need to configure it at the kernel level. In most systems, the appropriate setting is in /etc/sysctl.conf:

net.ipv4.ip_forward = 1

Alternatively, arrange for the following command to be executed at boot time:

echo 1 > /proc/sys/net/ipv4/ip_forward

You'll also need to provide the additional IP number 192.168.1.2; in an industrial-grade set-up you'd probably want to install a separate network adapter but, for home use, a virtual IP should be sufficient. If the main network adapter is eth0, then you need:

ifconfig eth0:1 192.168.1.2 up

To enable forwarding of (all) packets, with NAT, we need to set up the FORWARD and POSTROUTING chains as follows:

# Remove all rules from the FORWARD chain
iptables -F FORWARD
# Enable NAT for IP 192.168.1.2
iptables -t nat -A POSTROUTING -o eth0:1 -j MASQUERADE
# Enable forwarding between 192.168.1.1 and 192.168.1.2
iptables -A FORWARD -i eth0:1 -o eth0 -j ACCEPT

Again, it's worth testing this configuration before applying any restrictions. With these settings, all traffic sent to 192.168.1.2 should be forwarded with NAT applied.

# Create Firewall Rules

To apply restrictions to the flow of traffic through the Linux box, we can create additional firewall rules. 

iptables -F FORWARD
iptables -t nat -A POSTROUTING -o eth0:1 -j MASQUERADE

# Retain established connections
/sbin/iptables -A FORWARD -i p255p1 -o p255p1:1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# Drop all packets from disallowed devices, other than established connections
/sbin/iptables -A FORWARD -s 192.168.1.60 -j DROP
/sbin/iptables -A FORWARD -s 192.168.1.61 -j DROP
# And more...

# Accept everything else
iptables -A FORWARD -i eth0:1 -o eth0 -j ACCEPT

