---
layout: post
title:  PPP with NTRIP source for u-blox GNSS receiver over gpsd
date:   2026-02-28 18:32:00 CET
categories: 
---

This is now my fourth attempt to get an exact position (Precise Point Positioning) of my fixed mounted GNSS antenna at the roof of my house. 
You can find the methods I used previously in my blogs here: <br> 
(1) [PPP - Precise Point Positioning](/2023/06/03/PPP-Precise-Point-Positioning.html){:target="_blank"} <br>
(2) [PPP with gpsrinex, CSRS-PPP and ECTT](/2026/01/21/PPP-with-gpsrinex.html){:target="_blank"} <br>
(3) [PPP with RTKlib and local correction](/2026/02/21/PPP-with-RTKLIB.html){:target="_blank"} <br>

As GNSS receiver I used again my [u-blox ZED-F9P](/2022/07/29/ublox-ZED-F9P.html){:target="_blank"} <br>
to manage this device I use the [gpsd](https://gitlab.com/gpsd/gpsd){:target="_blank"} package. 

The method is quite simple. Feed RTCM date as a NTRIP ( Networked Transport of RTCM via Internet Protocol ) stream to the GNSS receiver. 
To do so, one must use any NTRIP caster. There are several available for free and of course also some commercial. In any case you have to register as you need username and password. 

As I am using the gpsd package I use the daemon gpsd itself to do this job. <br>
This can be achieved by 2 different methods <br>
1. start gpsd with an additional argument like ntrip://my.user:my.passwd@157.90.249.44:2101/MOUNTPOINT <br>
2. run "gpsdctl add ntrip-URL". This needs that gpsd was started with option "-F /run/gpsd.sock"

So what we need is username, password, the DNS name or IP address of the caster, the port number which is in almost all cases 2101 and the mountpoint. All mountpoints for a caster can be found at the caster itself and you should use one which is very close to you. 

To be sure that my ZED-F9P is well configured I run some checks.

<pre>
  ubxtool -g CFG-UART1INPROT-RTCM3X | grep CFG-UART1INPROT-RTCM3X 
  
  echo check if skipped frames count up 
  ubxtool -p MON-COMMS  | grep -A 7 UBX-MON-COMMS: | grep skipped
  sleep 10 
  ubxtool -p MON-COMMS  | grep -A 7 UBX-MON-COMMS: | grep skipped
  
  ubxtool  -g CFG-NAVSPG-DYNMODEL | grep CFG-NAVSPG-DYNMODEL | head -1 
  ubxtool  -z CFG-NAVSPG-DYNMODEL,2 | grep UBX-ACK-ACK: 
  
  if test -z "`ubxtool -p NAV-PVT -v 2 | grep -i carrSoln | grep Fixed`"
    then 
      echo state not fixed 
      ubxtool -p NAV-PVT -v 2 | grep -i carrSoln
      exit 1 
  fi 
 
  echo number of satellites with cno greater 40 
  ubxtool -p NAV-SIG -v 2 | grep cno | awk '{ if ( $12 > 40 ) print ( $2, $4 ) }' | sort -u | wc -l 
 
  echo accuracy in 0.1 mm 
  ubxtool -p NAV-RELPOSNED -v 2 | grep -i accN

  ubxtool -z CFG-MSGOUT-UBX_RXM_RTCM_UART1,1 | grep UBX-ACK-ACK:
  
  RESULT=`ubxtool -v 2 -w 10 | grep -i RTCM`
  
  if test -z "$RESULT" 
    then 
      echo we dont get RTCM data 
      exit 1 
  fi
    
  ubxtool -z CFG-NMEA-HIGHPREC,1  | grep UBX-ACK-ACK: # ist default 0 
  ubxtool -z CFG-MSGOUT-NMEA_ID_GGA_UART1,1 | grep UBX-ACK-ACK:  
  
  # for standard deviation 
  ubxtool -z CFG-MSGOUT-NMEA_ID_GST_UART1,1 | grep UBX-ACK-ACK: 
</pre>

I start gpspipe 

<pre>nohup socat EXEC:'gpspipe -rRB gpsdhost\:2947\:/dev/serial0' 'TCP-LISTEN:10001,reuseaddr,fork' & </pre>

and I collect the data with 

<pre>
SEC=3600
DAT=`date '+%j%H%M00'`
timeout $SEC nc 127.0.0.1 10001 | grep --line-buffered -aE "GGA|GST" > $MP/messung_$DAT.nmea </pre>

$SEC is the time in seconds how long I want to collect the data. One hour is normally good enough. $MP is the mountpoint selected in the ntrip-URL. If all is fine I extract the position data from the file with this script

`nmea2pos.bash $MP/messung_$DAT`

This will create a file `$MP/messung_$DAT.pos` which can be viewed with `rtkplot-qt`. 
