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

`export UBXOPTS="-P 27.50"`

This is the version I update end of 2025.

`ubxtool -p MON-VER  localhost:gpsd:/dev/serial0`

```
UBX-MON-VER:
  swVersion EXT CORE 1.00 (9e1716)
  hwVersion 00190000
  extension ROM BASE 0x118B2060
  extension FWVER=HPG 1.51
  extension PROTVER=27.50
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

The above command is deprecated. This should be preferred:

`ubxtool -g CFG-SIGNAL localhost:gpsd:/dev/serial0 | sed -n -e '/^UBX-CFG-VALGET:/,/^$/ p' | awk -v RS= 'NR==1'`

```
UBX-CFG-VALGET:
 version 1 layer 0 position 0
  layers (ram)
    item CFG-SIGNAL-GPS_L1CA_ENA/0x10310001 val 1
    item CFG-SIGNAL-GPS_L2C_ENA/0x10310003 val 1
    item CFG-SIGNAL-SBAS_L1CA_ENA/0x10310005 val 1
    item CFG-SIGNAL-GAL_E1_ENA/0x10310007 val 1
    item CFG-SIGNAL-GAL_E5B_ENA/0x1031000a val 1
    item CFG-SIGNAL-BDS_B1_ENA/0x1031000d val 1
    item CFG-SIGNAL-BDS_B2_ENA/0x1031000e val 1
    item CFG-SIGNAL-QZSS_L1CA_ENA/0x10310012 val 0
    item CFG-SIGNAL-QZSS_L1S_ENA/0x10310014 val 0
    item CFG-SIGNAL-QZSS_L2C_ENA/0x10310015 val 0
    item CFG-SIGNAL-GLO_L1_ENA/0x10310018 val 1
    item CFG-SIGNAL-GLO_L2_ENA/0x1031001a val 1
    item CFG-SIGNAL-GPS_ENA/0x1031001f val 1
    item CFG-SIGNAL-SBAS_ENA/0x10310020 val 1
    item CFG-SIGNAL-GAL_ENA/0x10310021 val 1
    item CFG-SIGNAL-BDS_ENA/0x10310022 val 1
    item CFG-SIGNAL-QZSS_ENA/0x10310024 val 0
    item CFG-SIGNAL-GLO_ENA/0x10310025 val 1
    item CFG-SIGNAL-39/0x10310027 val 1
```

Each individual satellite can be disabled or enabled. For example QZSS disabled in RAM layer:

`ubxtool -z CFG-SIGNAL-QZSS_ENA,0,1 localhost:gpsd:/dev/serial0` 


