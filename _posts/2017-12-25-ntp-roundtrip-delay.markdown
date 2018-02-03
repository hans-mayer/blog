---
layout: post
title:  NTP round-trip delay
date:   2017-12-25 18:44:00 CET
categories: unix
---

We all know, that the delay between the NTP server and a NTP client has important influence for the precision of the time. There are a lot of information in the Internet which I did already know, but I was surprised how dramatically this could be.

Since I run my own rubidium disciplined NTP server since this summer I try to adjust my stratum-1 server very accurate. Beside the rubidium NTP server I have also a GPS based and a DCF77 based time source as reference. To adjust these servers I need a reference from the Internet. Located near Vienna in Austria I took stratum-1 server with low RTT. My decision was to take the server from Bundesamt für Eich- und Vermessungswesen (BEV) in Vienna. BEV is also the official time source from Austria.

With a rubidium disciplined NTP server it's actually possible to adjust this server with an offset to "real time" within a millisecond or so. I realized that the reference couldn't be adjusted so precisely. Offset was sometimes higher and sometimes lower. So I looked at the GPS based NTP server and I found the same behavior.

Below you can see the offset between November 19th and 24th. The graph show an average value of all 3 NTP server at BEV ( 178.189.127.147, 178.189.127.148 and 178.189.127.147 ). In the image it is listed as 178.189.127.14

![offset_6days.png](/images/plot_bev_offs_14522_6days.png)

We can clearly see that during the first 3 days offset is about zero and than it is much higher ( with a negative value ).

If we do an average calculation of the first 3 days we get -5.8 micro seconds.

![offset_before.png](/images/plot_bev_offs_14414_before.png)

Doing the same with the next 3 days we have -37.1 micro seconds. So the difference is more than 31 micro seconds.

![offset_after.png](/images/plot_bev_offs_14468_after.png)

As I have the same phenomena on the rubidium server I was sure it was not my hardware. But I couldn't believe it was a problem of BEV. Finally it's the official time for Austria. But between me and BEV there is a third component. It's the network.

Looking at other statistics I found very quick the issue. The round-trip delay changed marginal.

![roundtripdelay_6days.png](/images/plot_bev_rtd_20800_6days.png)

It was during the first 3 days of observation 4.226 milliseconds.

![roundtripdelay_before.png](/images/plot_bev_rtd_20746_before.png)

and changed to 4.143 milliseconds during the next days. Which is only 83 micro seconds less.

![roundtripdelay_after.png](/images/plot_bev_rtd_20692_after.png)

Very obviously it's not only the round-trip time. I assume that the delay difference between sending and receiving packets also changed. See also my experiments with ADSL connectivity [NTP and ADSL](/2016/03/16/ntp-and-adsl.html){:target="_blank"} and [NTP and ADSL p2](/2016/05/02/ntp-and-adsl-part2.html){:target="_blank"}.

I have to accept that I have some high precisely time server but I never know how close I am to the real time.
