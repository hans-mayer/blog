---
layout: post
title:  The second Efratom LPRO-101
date:   2016-07-25 16:59:00 CET
categories: 
---

I got a second Efratom. As you can see from a [previous posting](/2016/01/03/efratom-lpro-101-adjusting.html){:target="_blank"} I own already an Efratom LPRO-101. This second one is owned by the institute I am working for and it doesn't really belong to me. The intention is to build a NTP server and to use this Efratom LPRO-101 as stratum 0 source. I purchased it at Ebay and it's coming again from Hong Kong / China. The price for this electronic part is 155 EUR. But there were already periods of time where you didn't get it below 300 EUR.

After unpacking I run it for some days and I made my first measurements. The way how-to measure is identical as I described [earlier](/2016/01/03/efratom-lpro-101-adjusting.html){:target="_blank"}. I run a NTP server with two 1PPS sources. One of this sources is the 1PPS signal from a GPS receiver which is my reference. The other comes from the Efratom. The 10 MHz output is divided by 10^7. 

And here are the results. 

Mon Jul 25 21:38:29 CEST 2016

Below one can see the offset of the second 1PPS ( 127.127.22.1 ) calculated out of the peerstats file. 

> ntp_shps -o -F 1 -x -0.5:20 -f png 127.127.22.1 . 

![offset_from_peerstats](/images/plot_20805_efra_2nd.png)

This shows the difference between these two 1PPS signals. 

> ntp_shdiff -F 1 -t 20 -x -0.5:20 -f png 127.127.22.1 127.127.22.0 . 

![difference_from_peerstats](/images/plot_20892_efra_2nd.png)

<pre>
drift per second: .00000000008285582538 ppb: .08285582538000000000
drift per hour: .00000029828097138051 
drift per day: .00000715874331313224 
drift per month: .00021774510910777230 
drift per year: .00261294130929326760 
</pre>

Without any adjustments this system much closer to the "real" clock than my first device. It has only an offset of 0.08 PPB. My first device is 3 PPB away from real 10 MHz. 

Sun Jul 31 19:01:49 CEST 2016

And sometimes it's even better.

This is the GPS based stratum 0 reference. 

![gps_reference](/images/plot_0730_18051.png)

This is the offset messured from the peerstats file for the new Efratron. 

![new_efratron](/images/plot_0730_18077.png)

And this is the difference. 

![difference_from_peerstatsfile](/images/plot_0730_18131.png)

<pre>
drift per second: _.00000000000962494532 ppb: -.00962494532000000000
drift per hour: -.00000003464980315935 
drift per day: _.00000083159527582440 
drift per month: _.00002529435630632550 
drift per year: -.00030353227567590600 
</pre>

All without any adjustments. 



