---
layout: post
title:  u-blox ZED-X20P initialization and GPSD
date:   2026-05-31 16:56:00 CET
categories: u-blox GNSS 
---

I use the gpsd tool set since a while. In Debian it is available as package or you can find it here: [gpsd](https://gitlab.com/gpsd/gpsd){:target="_blank"} <br> Several programs from the gpsd tool set worked as expected when I use it with a ZED-F9P. Not so with the newer ZED-X20P. The reason is simple. When starting the gpsd process it initialize the F9P and others u-blox devices with several messages, but not so with the X20P. The X20P stays in the factory default setup. 
<br>So I saw that `gpscsv -n 1 -c SAT` didn't work. With help of several resources I could initialize the X20P in way so that gpscsv is working. These are the parameters

<pre>
  ubxtool -z CFG-MSGOUT-UBX_NAV_PVT_UART1,1 
  ubxtool -z CFG-MSGOUT-UBX_NAV_SAT_UART1,1 
  ubxtool -z CFG-MSGOUT-UBX_NAV_SIG_UART1,1 
  ubxtool -z CFG-UART1OUTPROT-NMEA,0 
  ubxtool -z CFG-UART1OUTPROT-UBX,1 
  ubxtool -z CFG-MSGOUT-UBX_NAV_TIMEGPS_UART1,1 
</pre>

Of course I cannot say if these are all parameters which are set for F9P and others but at least for my gpscsv issue with ZED-X20P is it working. Currently I am using gpsd version 3.27.6. Maybe in later releases it will work. 

See also: [ZED-X20P](/2026/03/09/u-blox_ZED-X20P.html){:target="_blank"}