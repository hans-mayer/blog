---
layout: post
title:  u-blox ZED-F9P
date:   2022-07-29 18:21:00 CET
categories:
---


These days my new u-blox ZED-F9P arrived.
Here you can see it in action.


<video width="320" height="240" controls>
  <source src="/images/20220729_114006.mp4" type="video/mp4">
</video>

The board is from uputronics. It is mounted on a Raspberry PI4. Not to loose the possibility to connect a small fan I mounted first an expansion module with 90 degrees pins. There you can see the black and red wire for the fan.

Very important for the module from uputronics. The side with the electronic parts must be face up. The delivered 40-pin connector must be plugged on the bottom side (face down).

As antenna I use the recommended multi-band active gnss antenna HAB-ANN-MB-00-00 with SMA Connector.

Some details:

`export UBXOPTS="-P 27.12"`

`ubxtool -p MON-VER  localhost:gpsd:/dev/serial0`

```
UBX-MON-VER:
  swVersion EXT CORE 1.00 (f10c36)
  hwVersion 00190000
  extension ROM BASE 0x118B2060
  extension FWVER=HPG 1.13
  extension PROTVER=27.12
  extension MOD=ZED-F9P
  extension GPS;GLO;GAL;BDS
  extension SBAS;QZSS
```

`ubxtool -p MON-GNSS -v 2  localhost:gpsd:/dev/serial0`

```
UBX-MON-GNSS:
   version 0 supported 0xf defaultGnss 0xf enabled 0xf
   simultaneous 4 reserved1 0 0 0
     supported (GPS Glonass Beidou Galileo)
     defaultGnss (GPS Glonass Beidou Galileo)
     enabled (GPS Glonass Beidou Galileo)
```

`ubxtool -p CFG-GNSS  -v 2  localhost:gpsd:/dev/serial0`

```
UBX-CFG-GNSS:
 msgVer 0  numTrkChHw 60 numTrkChUse 60 numConfigBlocks 6
  gnssId 0 TrkCh  8 maxTrCh 16 reserved 0 Flags x11110001
   GPS L1C/A L2C enabled
  gnssId 1 TrkCh  3 maxTrCh  3 reserved 0 Flags x01010001
   SBAS L1C/A enabled
  gnssId 2 TrkCh 10 maxTrCh 18 reserved 0 Flags x21210001
   Galileo E1 E5b enabled
  gnssId 3 TrkCh  2 maxTrCh  5 reserved 0 Flags x11110001
   BeiDou B1I B2I enabled
  gnssId 5 TrkCh  0 maxTrCh  4 reserved 0 Flags x15110001
   QZSS L1C/A L2C enabled
  gnssId 6 TrkCh  8 maxTrCh 12 reserved 0 Flags x11110001
   GLONASS L1 L2 enabled
```
