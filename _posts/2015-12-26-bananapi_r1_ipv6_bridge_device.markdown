---
layout: post
title:  "banana pi r1 with a bridge device and IPv6 "
date:   2015-12-26 22:40:00
categories: jekyll update
---


Adding an IPv6 address is simple as for a normal interface too. Modify file /etc/network/interfaces and add the following 

<pre>
iface br0 inet6 static
        address 2001:db8:0:0::1
        netmask 64
</pre>

All bridged interfaces will get this new IPv6 address. 

For details about bridge configuration see my yesterdays posting: [bananapi_r1_wireless](/2015/12/26/bananapi_r1_wireless.html)



