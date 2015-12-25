---
layout: post
title:  "banana pi r1 with switch configuration and wireless "
date:   2015-12-25 23:30:00
categories: jekyll update
---

This document describes how to configure a BananaPi R1 with a switch configuration and an additional bridge interface for wireless connectivity. 

### OS version

<pre>
Linux 3.4.108-bananian armv7l GNU/Linux
</pre>

### Additional packages to install 

<pre>
||/ Name                Version      Architecture      Description
+++-===================-============-=================-==========================================================
ii  bridge-utils        1.5-9        armhf             Utilities for configuring the Linux Ethernet bridge
ii  hostapd-rtl         2.4-4        armhf             IEEE 802.11 AP, IEEE 802.1X/WPA/WPA2/EAP/RADIUS Authenticator
</pre>

### Switch config 

In my case I used: <br>
eth0.101 = WAN (single port) <br>
eth0.102 = LAN (4 port switch) <br>

#### /etc/network/if-pre-up.d/swconfig

<pre>
ifconfig eth0 up
swconfig dev eth0 set reset 1
swconfig dev eth0 set enable_vlan 1
swconfig dev eth0 vlan 101 set ports '3 8t'
swconfig dev eth0 vlan 102 set ports '4 0 1 2 8t'
swconfig dev eth0 set apply 1
</pre>

### Network config 

#### /etc/network/interfaces

<pre>
auto lo
iface lo inet loopback

# static ip configuration
auto eth0.101
iface eth0.101 inet static
        address 10.0.0.1
        netmask 255.255.255.252
        gateway 10.0.0.2

# eth0.102 will be managed by bridge interface
allow-hotplug eth0.102

allow-hotplug wlan1

auto br0
iface br0 inet static
      address 192.168.241.93
      netmask 255.255.255.0
      gateway 192.168.241.10
      bridge_ports eth0.102 wlan1
      bridge_waitport 0

</pre> 

### Wireless config 

#### /etc/hostapd/hostapd.conf

The following 5 lines has to be adapted. All others can be used from the original installed file. 

<pre>
wpa_passphrase=MySecretPassword
bridge=br0 		# same name as defined in file /etc/network/interfaces
channel=6		# look for a free channel 
ssid=MySSID		# find a useful name 
country_code=XY 	# where you are living 
</pre>

### Troubleshooting tools

<pre>

# ping 

# tcpdump -i wlan1 

# brctl show br0
bridge name     bridge id               STP enabled     interfaces
br0             8000.024c05c2f439       no              eth0.102
                                                        wlan1

</pre> 


