---
layout: post
title:  NTP - driver 22 versus 28
date:   2017-06-14 14:12:00 CET
categories:
---

# Here I want to share some information about different drivers for NTP daemon

To run a GPS disciplined NTP server one has to us a so called driver. There are several different possibilities to achieve this goal. I decided to use driver 22 and driver 28. But it's also possible to use only one or the other. (Or to go with a different solution)

## Driver 22 - PPS Clock Discipline

Driver 22 needs at least one preferred source. In my case it's driver 28 but it could be as well any other "remote" NTP server.

As you can see offset is typically within a range of +/- 10 us.

![plot_20592_driver_22.png](/images/plot_20592_driver_22.png)

See also: [https://www.eecis.udel.edu/~mills/ntp/html/drivers/driver22.html](https://www.eecis.udel.edu/~mills/ntp/html/drivers/driver22.html){:target="_blank"}

Driver 22 does the ticks.

## Driver 28 - Shared Memory Driver

Driver 28 is used to do the "second numbering". The information is coming over a serial interface. Therefore it's more or less a random situation when the time information is delivered. And we can see driver 28 is far away to be so accurate as driver 22.

![plot_20624_driver_28.png](/images/plot_20624_driver_28.png)

See also: [https://www.eecis.udel.edu/~mills/ntp/html/drivers/driver28.html](https://www.eecis.udel.edu/~mills/ntp/html/drivers/driver28.html){:target="_blank"}

## Disadvantage

If connectivity to the satellites is lost and finally available the NTP daemon starts to synchronize with the preferred server (28). This pulls the server away from the real time even if the free running server wouldn't be so bad.
