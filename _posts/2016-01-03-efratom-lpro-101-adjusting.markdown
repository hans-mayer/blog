---
layout: post
title:  EFRATOM LPRO-101 adjusting
date:   2016-01-03 11:59:00 CET
categories:  rubidium frequency standard NTP 
---

This document describes how-to adjust an EFRATOM LPRO-101 using a NTP daemon 

I bought a refurbished EFRATOM LPRO-101 from the second hand market. As this LPRO is a secondary frequency standard you cannot expect an accuracy of theoretical 3 * 10 ^ -15 for rubidium <br />
The one I bought had an offset less than 0.1 HZ at 10 MHz which is about 10 ^ -8 <br />
In most cases this is good enough for private use. Not so if you are ham operator and playing around with micro wave equipment or if you try to use this frequency standard as stratum 0 source for a NTP server. As you can easy calculate this clock would go wrong about 3 seconds per year. This is not much better than a quartz watch. 

Adjusting the frequency of the LPRO-101 is easy as it has a potentiometer for the C-field. It is easy if you have a second 10 MHz reference which is much more precisely. I don't have one so I took the approach to look for the drift over long time period. I compare the EFRATOM with a GPS 1PPS signal. To do so I feed both signals to the GPIO chip of a Banana Pi. ( To use a second 1PPS you have to compile a second PPS GPIO module see [https://github.com/hans-mayer/pps2gpio](https://github.com/hans-mayer/pps2gpio){:target="_blank"} and to compile a module you have to compiler the kernel first [Banana Pi compile kernel and header]({% post_url 2016-01-01-bananapi-compile-kernel %}){:target="_blank"} ) On this system I run "ntpd" with two PPS references: 

<pre>
server 127.127.22.0 minpoll 5 maxpoll 6
server 127.127.22.1 minpoll 5 maxpoll 6 noselect
</pre>

And a lot of other configuration directives. Running "ntpq -pn" one can see: 

<pre>
     remote       refid      st t when poll reach   delay   offset  jitter
==========================================================================
o127.127.22.0    .PPS.        0 l    7   64  377    0.000   -0.005   0.003
*127.127.28.0    .GPS.        0 l    9   32  377    0.000   -0.674   0.682
 127.127.22.1    .PPS2.       0 l   28   32  377    0.000  294.516   0.005
</pre>

I wrote a small shell/gawk/gnu-plot script called **ntp_shdiff** ( see [https://github.com/hans-mayer/ntpgraph](https://github.com/hans-mayer/ntpgraph){:target="_blank"} ) to plot the difference of two NTP server. Running this after several hours I could see 

![plot_4372.png](/images/plot_4372.png) 

Sun Jan  3 00:13:20 CET 2016 I took 2 measure points and did the calculation:

<pre>
mp1 : x=0.0173913 y=-0.292421
mp2 : x=16.8261 y=-0.293467  ( x = time decimal hours ) 

drift .00000001728601290803 per second
1.7 ^-2 = 17 ^-3  ppm  = 1.7 ^-8
</pre>

So I produced a screw driver ( those one I have owned were to short ) and turned the potentiometer clockwise for 90 degrees. Actually not knowing in which direction I should turn. But I had luck. Next day a measurement: 

![plot_4738.png](/images/plot_4738.png) 

<pre>
mp1: x=0.368613 y=-0.29471
mp2: x=9.24453 y=-0.29495

drift .00000000751096102708 per second
7.5 ^-3 ppm = 7.5 ^-9
</pre>

Sun Jan  3 10:23:35 CET 2016
I turned again the potentiometer clockwise for 90 degrees.

Let's see. 

Mon Jan  4 21:58:38 CET 2016

![plot_10209.png](/images/plot_10209.png) 

Surprise surprise. 

Instead of a smaller drift it's much bigger.

<pre>
mp1: x=79.012 y=-0.339719843
mp2: x=74703.012 y=-0.342186132 ( x = time in seconds ) 

drift .00000003304954170240 per second
</pre>

So I will turn the potentiometer counter clockwise 360 degrees. 

Let's see.


## Attachment 

[EFRATOM LPRO-101 user's guide](/images/efratom_LPRO-101.pdf){:target="_blank"} 
