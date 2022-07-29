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

Details:

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
