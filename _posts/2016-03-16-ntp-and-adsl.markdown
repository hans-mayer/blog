---
layout: post
title:  NTP and ADSL
date:   2016-03-16 22:13:00 CET
categories: UNIX Linux ntp network
---

At home I am connected to the Internet via ADSL.
Up-link speed is 768 kbit/sec and download speed is 8 Mbit/sec. 
Therefore an NTP frame with 90 bytes or 720 bits needs 937.5 &mu;s going to the Internet and 90 &mu;s coming from the Internet.
Reading NTP documentation it says the protocol assumes a symmetric situation for both directions. 
I was always wondering how precisely NTP could be at home. To verify this I prepared a stratum 1 server. It consists of a Banana Pi with a GPS module from M0UPU. The 1PPS is feed to the GPIO and "ntpd" is using this signal directly. The time information is read via SHMEM over "gpsd". 

I installed this NTP server in a network which is connected with 1 Gbit/sec to the backbone of Vienna's Internet. In "ntp.conf" I configured a stratum 1 server as reference with keyword "noselect". The typical round-trip-time to this server is 4.5 ms. 

Below you see a graph with the offset over a day (20160313). The average offset is 22.57 &mu;s. Probably this could be adjusted but I don't care about this.  

![plot_adsl_20160313](/images/plot_adsl_20160313.png)

Now I took this server at home connecting to the LAN and via ADSL to the Internet. I run the server for more than a day (20160315) and the average offset is now 5.076 ms or 5076 &mu;s. If we take this 22 &mu;s from before it is still 5054&nbsp;&mu;s. 

![plot_adsl_20160315](/images/plot_adsl_20160315.png)

You can also see jitter is much more than in the Gigabit LAN. Also round-trip-time is more than 21 ms.
And of course my local server "believes" that the remote stratum 1 server is 5 milliseconds behind and we are 5 ms ahead. 

<pre>
# ntpq -pn
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
o127.127.22.0    .PPS.            0 l    5   16  377    0.000    0.004   0.010
*127.127.28.0    .GPS.            0 l    4   16  377    0.000   -1.191   1.096
 XXX.XXX.127.149 .ATOM.           1 u  161  256  377   21.342   -5.066  12.087
</pre>

And what is the practical significance ? 

If somebody is using NTP as a client in a network with ADSL connectivity it is always 5 ms behind the reference. Not important, but good to know. 

