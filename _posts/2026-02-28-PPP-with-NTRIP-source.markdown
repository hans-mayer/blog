---
layout: post
title:  PPP with NTRIP source for u-blox GNSS receiver over gpsd
date:   2026-02-28 18:32:00 CET
categories: 
---

This is now my fourth attempt to get an exact position (Precise Point Positioning) of my fixed mounted GNSS antenna at the roof of my house. 
You can find the methods I used previously in my blogs here: <br> 
(1) [PPP - Precise Point Positioning with averaging](/2023/06/03/PPP-Precise-Point-Positioning.html){:target="_blank"} <br>
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

I run this scenario over several times. And this is the result

<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        padding: 2px;
        }
</style>
| &nbsp;&nbsp;date&nbsp;&nbsp;              | &nbsp;&nbsp;latitude&nbsp;&nbsp;       | &nbsp;&nbsp;longitude&nbsp;&nbsp;      | &nbsp;&nbsp;altitude&nbsp;&nbsp;        |
|-------------------------------------------|----------------------------------------|----------------------------------------|-----------------------------------------|
| &nbsp;&nbsp;messung_049220424&nbsp;&nbsp; | &nbsp;&nbsp;48.14928668858&nbsp;&nbsp; | &nbsp;&nbsp;16.28383489981&nbsp;&nbsp; | &nbsp;&nbsp;286.34149647059&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_049220558&nbsp;&nbsp; | &nbsp;&nbsp;48.14928651058&nbsp;&nbsp; | &nbsp;&nbsp;16.28383491739&nbsp;&nbsp; | &nbsp;&nbsp;286.34408541973&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_049224529&nbsp;&nbsp; | &nbsp;&nbsp;48.14928646948&nbsp;&nbsp; | &nbsp;&nbsp;16.28383476017&nbsp;&nbsp; | &nbsp;&nbsp;286.32093080357&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_051190411&nbsp;&nbsp; | &nbsp;&nbsp;48.14928674470&nbsp;&nbsp; | &nbsp;&nbsp;16.28383504772&nbsp;&nbsp; | &nbsp;&nbsp;286.35412230216&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_053164800&nbsp;&nbsp; | &nbsp;&nbsp;48.14928685024&nbsp;&nbsp; | &nbsp;&nbsp;16.28383472580&nbsp;&nbsp; | &nbsp;&nbsp;286.29313600000&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_053164900&nbsp;&nbsp; | &nbsp;&nbsp;48.14928721850&nbsp;&nbsp; | &nbsp;&nbsp;16.28383476294&nbsp;&nbsp; | &nbsp;&nbsp;286.34042372881&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_053170600&nbsp;&nbsp; | &nbsp;&nbsp;48.14928712056&nbsp;&nbsp; | &nbsp;&nbsp;16.28383469769&nbsp;&nbsp; | &nbsp;&nbsp;286.35376422764&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_053171700&nbsp;&nbsp; | &nbsp;&nbsp;48.14928646645&nbsp;&nbsp; | &nbsp;&nbsp;16.28383503276&nbsp;&nbsp; | &nbsp;&nbsp;286.33033740831&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_053182000&nbsp;&nbsp; | &nbsp;&nbsp;48.14928761124&nbsp;&nbsp; | &nbsp;&nbsp;16.28383548462&nbsp;&nbsp; | &nbsp;&nbsp;286.31665768194&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_053184700&nbsp;&nbsp; | &nbsp;&nbsp;48.14928675778&nbsp;&nbsp; | &nbsp;&nbsp;16.28383470345&nbsp;&nbsp; | &nbsp;&nbsp;286.25878328474&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_056172800&nbsp;&nbsp; | &nbsp;&nbsp;48.14928653393&nbsp;&nbsp; | &nbsp;&nbsp;16.28383432340&nbsp;&nbsp; | &nbsp;&nbsp;286.44596396396&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_056173000&nbsp;&nbsp; | &nbsp;&nbsp;48.14928641814&nbsp;&nbsp; | &nbsp;&nbsp;16.28383453253&nbsp;&nbsp; | &nbsp;&nbsp;286.46528272251&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_056174600&nbsp;&nbsp; | &nbsp;&nbsp;48.14928641787&nbsp;&nbsp; | &nbsp;&nbsp;16.28383439221&nbsp;&nbsp; | &nbsp;&nbsp;286.44075187970&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_056174700&nbsp;&nbsp; | &nbsp;&nbsp;48.14928635164&nbsp;&nbsp; | &nbsp;&nbsp;16.28383422094&nbsp;&nbsp; | &nbsp;&nbsp;286.38687192983&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_056203500&nbsp;&nbsp; | &nbsp;&nbsp;48.14928663041&nbsp;&nbsp; | &nbsp;&nbsp;16.28383471765&nbsp;&nbsp; | &nbsp;&nbsp;286.38973867596&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_057102300&nbsp;&nbsp; | &nbsp;&nbsp;48.14928749432&nbsp;&nbsp; | &nbsp;&nbsp;16.28383523683&nbsp;&nbsp; | &nbsp;&nbsp;286.41764218009&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_057113100&nbsp;&nbsp; | &nbsp;&nbsp;48.14928616931&nbsp;&nbsp; | &nbsp;&nbsp;16.28383569321&nbsp;&nbsp; | &nbsp;&nbsp;286.21828049137&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_057172000&nbsp;&nbsp; | &nbsp;&nbsp;48.14928643937&nbsp;&nbsp; | &nbsp;&nbsp;16.28383441440&nbsp;&nbsp; | &nbsp;&nbsp;286.37522851677&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_064192300&nbsp;&nbsp; | &nbsp;&nbsp;48.14928611437&nbsp;&nbsp; | &nbsp;&nbsp;16.28383541068&nbsp;&nbsp; | &nbsp;&nbsp;286.39431147541&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_064193000&nbsp;&nbsp; | &nbsp;&nbsp;48.14928626123&nbsp;&nbsp; | &nbsp;&nbsp;16.28383493320&nbsp;&nbsp; | &nbsp;&nbsp;286.45333041958&nbsp;&nbsp; |
| &nbsp;&nbsp;messung_065185800&nbsp;&nbsp; | &nbsp;&nbsp;48.14928580792&nbsp;&nbsp; | &nbsp;&nbsp;16.28383442479&nbsp;&nbsp; | &nbsp;&nbsp;286.43519302326&nbsp;&nbsp; |
| &nbsp;&nbsp;average&nbsp;&nbsp;           | &nbsp;&nbsp;48.14928662270&nbsp;&nbsp; | &nbsp;&nbsp;16.28383482534&nbsp;&nbsp; | &nbsp;&nbsp;286.36553964790&nbsp;&nbsp; |
{: .tablelines}

<pre>
4092523.5748 1195484.3667 4728180.7527
48.14928662270 16.28383482534 286.3655
48 8 57.43184172000  16 17 1.80537122400
</pre>

The maximum distance to the average point is 13.5 cm. The maximum distance between 2 measuring points is 21.5 cm. 

Takeing the average value and calculating the distance to method (2) we get an offset of 6.8 cm. 
Distance to method (3) is 3.9 cm. 

Tools at github: <br>
A commandline tool to [transform ecef wgs84](https://github.com/hans-mayer/transform_ecef_wgs84){:target="_blank"} data. <br>
A commandline tool which [converts NMEA to high-precision .pos position logs](https://github.com/hans-mayer/nmea2pos){:target="_blank"} data. <br>

