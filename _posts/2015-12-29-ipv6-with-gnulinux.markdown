---
layout: post
title:	"IPv6 with GNU/Linux "
date:   2015-12-29 22:00:00 CET
categories: IP routing network 
---

IP address assignment in IP version 4 was easy:
IP address, subnetmask and gateway was configured either <br />

- directly within the network device <br />
- or assigned via DHCP to the network client <br />

IPv6 is slightly different:

The management of the IP address is split up in a network part and in a host part. 

- Only the network part is advertised typically by a router <br />
- the host part is managed by the client itself or via DHCP <br />

This document describes how-to setup IPv6 address assignment with GNU/Linux. 
In my case I am using a Banana Pi with BananianOS which is actually Debian. 

Install the router advertisement daemon for IPv6 called **"radvd"**

<pre>
apt-get install radvd 
</pre> 

Define an IPv6 address for the interface. 

<pre>
iface eth0 inet6 static
        address 2001:db8:0:0::1
        netmask 64
</pre>

In the example above it's interface "eth0". The network part has 64 bit. Therefore the host part has also 64 bit. ( Total 128 bit ) The host part should never be smaller than 64 bit even  if you don't need 18446744073709551616 addresses in this single network. Network 2001:db8:0:0::/64 is the documentation example. You should ask your provider for a network block. 

Now configure **/etc/radvd.conf**

<pre>
interface eth0
{

        AdvManagedFlag on;
        AdvSendAdvert on;
        AdvLinkMTU 1480;
        AdvOtherConfigFlag on;
        MinRtrAdvInterval 3;
        MaxRtrAdvInterval 60;

        prefix 2001:db8::/64
        {
                AdvOnLink on;
                AdvRouterAddr on;
                AdvAutonomous on;
        }; # End of prefix definition

        DNSSL mydomain.com
        {
                AdvDNSSLLifetime 70;
        } ;

}; # End of interface definition
</pre>

In this case we are speaking about a **stateless** configuration. Some of the parameter above are not used for stateless. But it does not hurt. It will also work for stateful setup. 

Now you are IPv6 ready. The client must be enabled for IPv6. But this is typically the case. Test the setup for example with a Windows client. You should see lines like this if you run **"ipconfig /all"** for one of your interfaces:

<pre>
   IPv6-Address. . . . . . . . . . . : 2001:db8::cc2b:5a5f:52a:1007(Preferred)
   Temporary IPv6-Address. . . . . . : 2001:db8::8920:a137:8689:3703(Preferred)
   Link-local IPv6-Address . . . . . : fe80::cc2b:5a5f:52a:1007%10(Preferred)
</pre>

Maybe you are not happy with this long addresses. Especially if you want connect remotely to this IPv6 client. In this case you need a DHCP server which updates DNS records. See [dhcp-with-ipv6]({% post_url 2015-12-31-dhcp-with-ipv6%})

For IPv6 access to the Internet we are missing 2 aspects:

* IPv6 routing to the ISP 
* a working firewall configuration 

If all is done you can check your configuration for example with 

> [http://test-ipv6.com/](http://test-ipv6.com/)

For a well working network a well configured DNS ( domain name system ) service is necessary. You have to make sure that your clients can update your DNS database. Using BIND from isc.org it is done within the zone definition. 

<pre>
   allow-update {
       127.0.0.1 ; ::1/128 ; 192.168.1.0/24 ; 2001:db8::/64 ; 
   };
</pre>

Typically you need only your local IPv4 address range - for example 192.168.1.0

I have seen that most clients are using their IPv4 address to update their IPv6 entries within DNS. 





