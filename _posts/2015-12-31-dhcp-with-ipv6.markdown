---
layout: post
title:  "DHCP with IPv6 and GNU/Linux "
date:   2015-12-31 23:30:00 CET
categories: IP routing network
---

Most sites will run their DHCP and DNS configuration for all Windows clients on the Windows server. 
But this is also easily done with Linux. 

Generally it is not necessary to run DHCP for IPv6. See my previously blog [ipv6-with-gnulinux]({% post_url 2015-12-29-ipv6-with-gnulinux%})

But DHCP offers some additional possibilities, so called DHCP options. In this case we speak about stateful configuration. With DHCP you can

* assign additional services for example NTP server, WINS server, ...
* define a network 
* assign a fixed IP address for a client 

If you think about to run a DHCP server for IPv6 maybe you run already one for IPv4. I am using the one from isc.org but not saying that others do their jobs worse. For IPv6 I run a second instance. The following ports are used 

* IPv4 67/UDP bootps
* IPv6 547/UDP dhcpv6-server 

"dhcpd" is started with option -4 for IPv4 and the second instance with -6 for IPv6. 

Here an example of the configuration file **/etc/dhcp/dhcpd6.conf**

<pre>
ddns-update-style interim;
ddns-updates on;
ddns-domainname "mydomain.com" ;
ddns-rev-domainname "ip6.arpa";
allow client-updates;
update-conflict-detection false;
update-optimization false;

authoritative ;
option dhcp6.name-servers 2001:db8::3 ;
default-lease-time 43200;
preferred-lifetime 40000;
allow leasequery;
option dhcp6.name-servers 2001:db8::3 ;
option dhcp6.domain-search "subdomain.mydomain.com","mydomain.com";

include "/etc/rndc.key";
option dhcp6.preference 255;

log-facility local7;

zone 0.0.0.0.0.0.0.0.8.b.d.0.1.0.0.2.ip6.arpa. {
        primary 127.0.0.1 ;
        key rndc-key ;
}

zone mydomain.com. {
        primary 127.0.0.1 ;
        key rndc-key;
}

subnet6 2001:db8::64 {
        # Range for clients
        range6 2001:db8:0:0:1::129 2001:db8:0:0:1::254 ;

        # Range for clients requesting a temporary address
        range6 2001:db8:0:0:2::/80 temporary;

}

# fixed host address
host mywinpc {
        host-identifier option dhcp6.client-id 00:01:00:01:1d:db:2d:09:c0:20:03:10:a6:4f ;
        fixed-address6 2001:db8:0:0:3::43 ;
        option dhcp6.domain-search "mydomain.com" ;
}

</pre> 

Make sure that your DNS is configured well for zone "0.0.0.0.0.0.0.0.8.b.d.0.1.0.0.2.ip6.arpa" ( as example ) 


