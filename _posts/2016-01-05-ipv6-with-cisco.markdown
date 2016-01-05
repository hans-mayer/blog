---
layout: post
title:  IPv6 with Cisco devices 
date:   2016-01-05 15:50:00 CET
categories: IP routing network
---

A typical interface configuration on a Cisco router

<pre>
interface Ethernet1/0
 ip address 10.125.117.1 255.255.255.0
 ipv6 address 2001:db8:21F0:117::1/64
 ipv6 address 2001:db8:21F0:117::/64 eui-64
 ipv6 enable
 ipv6 nd prefix 2001:db8:21F0:117::/64
 ipv6 nd managed-config-flag
 ipv6 nd other-config-flag
!
</pre>

If the DHCP server is on a different network segment than the client one has to add to following 2 lines for IPv4 and IPv6 

<pre>
 ip helper-address 10.125.116.2
 ipv6 dhcp relay destination 2001:db8:21F0:116::2 Ethernet1/7
</pre>

## See also 

> [IPv6 with GNU/Linux]({% post_url 2015-12-29-ipv6-with-gnulinux%})

> [DHCP with IPv6 and GNU/Linux]({% post_url 2015-12-31-dhcp-with-ipv6%})

