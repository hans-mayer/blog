---
layout: post
title:  NTP offset - Rubidium vs GPS
date:   2018-03-11 17:18:00 CET
categories: time
---

In this previous blog [Stratum-1 NTP Server with rubidium source](/2017/06/11/stratum-0-server.html){:target="_blank"} I described how to build a rubidium controlled NTP server.
Here you can see the difference for the offset for a rubidium based NTP server versus a GPS based NTP server.
Both server are running on the same hardware, the same operating system and the same (latest) version of NTP classic daemon software.

This graph below shows the offset for the GPS based NTP server. As you can see offset is between +/- 10 &mu;s. And this values is some time bigger and some time smaller.

![GPS](/images/ntp_gps_plot_10820.png){:target="_blank"}

And this is the graph for the rubidium controlled NTP server. Offset is most of the time between +/- 5 &mu;s.

![rubidium](/images/ntp_rubi_plot_32072.png){:target="_blank"}

The rubidium server doesn't give you an absolute timestamp to UTC. As well as the GPS server has also to be adjusted. This has to be done with a comparison to another stratum-1 server over long time, for example one week. And of course the round-trip time for this external source has to be very small. See my previous blog about [round-trip](/2017/12/25/ntp-roundtrip-delay.html){:target="_blank"} time. But more important is to synchronize the network and not to be accurate for a micro second in relation to UTC. Once both are in sync the rubidium NTP server is the better one.
