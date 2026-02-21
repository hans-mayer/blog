---
layout: post
title:  PPP with RTKLIB
date:   2026-02-21 13:43:00 CET
categories: 
---

This is now my third attempt to get an exact position of my fixed mounted GNSS antenna at the roof of my house. 
You can find the methods I used previously in my blogs here: <br> 
(1) [PPP - Precise Point Positioning](/2023/06/03/PPP-Precise-Point-Positioning.html){:target="_blank"} <br>
(2) [PPP with gpsrinex](/2026/01/21/PPP-with-gpsrinex.html){:target="_blank"}

As GNSS receiver I used again my [u-blox ZED-F9P](/2022/07/29/ublox-ZED-F9P.html){:target="_blank"} <br>
to manage this device I use the [gpsd](https://gitlab.com/gpsd/gpsd){:target="_blank"} package. 

This time I used RTKlib to make the post-processing by myself. There are different versions available, I used this: [github.com/rtklibexplorer/RTKLIB](https://github.com/rtklibexplorer/RTKLIB){:target="_blank"}

The method is about this: <br>
1) create an UBX file over several hours. <br>
2) apply the correction with a RINEX file offered from an external service provider.

Within a shell script I prepared my ZED-F9P with the following settings: 

check if baud-rate is high enough 
<pre>ubxtool -g CFG-UART1-BAUDRATE | grep CFG-UART1-BAUDRATE </pre> 
I use 921600 bd. This is the highest value I can use. 38400 would be the default. <br>
disable NMEA protocol 
<pre>ubxtool -d NMEA | grep UBX-ACK-ACK:
ubxtool -z CFG-UART1OUTPROT-NMEA,0 | grep UBX-ACK-ACK:
ubxtool -g CFG-UART1OUTPROT-NMEA | grep CFG-UART1OUTPROT-NMEA | head -1 </pre>
enable or disable different satellites 
<pre>ubxtool -z CFG-SIGNAL-SBAS_L1CA_ENA,1 | grep UBX-ACK-ACK:
ubxtool -z CFG-SIGNAL-SBAS_ENA,1 | grep UBX-ACK-ACK:
ubxtool -z CFG-SIGNAL-QZSS_L1CA_ENA,0 | grep UBX-ACK-ACK:
ubxtool -z CFG-SIGNAL-QZSS_L1S_ENA,0 | grep UBX-ACK-ACK:
ubxtool -z CFG-SIGNAL-QZSS_L2C_ENA,0 | grep UBX-ACK-ACK:
ubxtool -z CFG-SIGNAL-QZSS_ENA,0 | grep UBX-ACK-ACK:
ubxtool -g CFG-SIGNAL | sed -n -e '/^UBX-CFG-VALGET:/,/^$/ p' | awk -v RS= 'NR==1' </pre>
enable raw data 
<pre>ubxtool -e RAWX | grep UBX-ACK-ACK:
ubxtool | grep RAWX </pre>
to get satellite track data
<pre>ubxtool -e SFRBX | grep UBX-ACK-ACK: 
ubxtool -z CFG-MSGOUT-UBX_RXM_SFRBX_UART1,1 | grep UBX-ACK-ACK: </pre>

<pre>SECS=50400
DAT=`date '+%Y%j%H%M00'`
FILE=`echo result$DAT`
gpspipe -x $SECS -R gpsdhost:gpsd:/dev/serial0 > $FILE.ubx </pre>

gpspipe will run 14 hours. <br>
to generate $FILE.obs , $FILE.nav and $FILE.sbs run convbin 

<pre>convbin $FILE.ubx </pre>

Instead of convbin one can run the GUI `rtkconv_qt &` interactive. 

If files  $FILE.obs and $FILE.nav are available one can view some graphs with <br>
`rtkplot_qt -r $FILE.obs $FILE.nav &`

For example the skyplot for Galileo 

![skyplot obs](/images/skyplot_20260221.png) 

Now comes the part of preparing the correction. 

For this we need the RINEX (Receiver Independent Exchange Format) data from a reference station. I use the APOS-PP from BEV. BEV is the Federal Office of Metrology and Surveying ( Bundesamt für Eich- und Vermessungswesen ) in Austria and APOS-PP is a free service they offer. APOS-PP is the Austrian Position Service Post Processing. See [APOS](https://www.bev.gv.at/Services/Produkte/Grundlagenvermessung/APOS.html){:target="_blank"}

The free available RINEX files are accessible short time after 00:00 UTC, so I fetched them next day. As there are about 50 reference stations one has to select a station which is near to the own position. In my case it's `WIEN00AUT`. As each file has only 1 hour of data multiple files are needed. The files have type `.crx.gz`. To get a `.rnx` run first `gunzip` and than `CRX2RNX`.  CRX2RNX is also free available and should have at least `version : ver.4.2.0`. Older Versions are buggy. Finally all these files have to be combined to one file. This is done with `gfzrnx` which is also free available. Current available `VERSION: gfzrnx-2.2.0` at [GFZ.de](https://gnss.gfz.de/services/gfzrnx){:target="_blank"}

Now comes the real part of correction. There are 2 possibilities: <br>
- a GUI `rtkpost_qt` <br>
- a command line tool: `rnx2rtkp` 

As first step I run the GUI. The reason is that some settings can be saved in a configuration file which can be used with the command line tool as argument. I configured under "Setting1" position mode "Static" and selected Navigation Systems: GPS, Galileo, Glonass and BDS. In selector "Positions" I entered below "Base Station" the position in X/Y/Z-ECEF format. The values can be found in the final .rnx file in line marked with `APPROX POSITION XYZ`. Don't worry, it's not approximately, it is exact. In case of WIEN00AUT it is `4085097.7110 1200224.1682 4733306.7362`. It's important to enter the exact values, otherwise the calculated result is wrong. Now press the "Save" button and give the configuration file a name, e.g. "myconfig.conf". Back to the main window enter the necessary file names. It is the .obs file for Rover, the .rnx file for the basestation and also needed .nav and .sbs file. In the solutions sections you get a proposal for the final position file name. Press the "Execute" button. After a while you get the result. Press the "Plot" button. It will start `rtkplot_qt` with the .pos file. If "Ground Track" is selected we can see that the position goes around only few centimeter during the last hours. Better to see using the "Position" tab. 

![position](/images/position_20260221.png)

As we can see the curve swings in. As value I don't use the last line of the result file, I use the average of the last 3 hours. Especially don't use the first 2 lines for calculating the average. 

Instead of using the GUI with `rtkpost_qt` it's also possible to use the command line tool `rnx2rtkp`. As we have now a config file we can apply this as argument with option `-k` calling rnx2rtkp. A typical call looks like this:

`rnx2rtkp -k myconfig.conf -o $FILE.pos $FILE.obs $FILE.rnx $FILE.nav $FILE.sbs`

I run this scenario over several days. And this is the result

| &nbsp;&nbsp;date&nbsp;&nbsp;                    | &nbsp;&nbsp;longitude&nbsp;&nbsp;       | &nbsp;&nbsp;latitude&nbsp;&nbsp;        |
|-------------------------------------------------|-----------------------------------------|-----------------------------------------|
| &nbsp;&nbsp;result2026031081521.pos&nbsp;&nbsp; | &nbsp;&nbsp;48.149286929965&nbsp;&nbsp; | &nbsp;&nbsp;16.283835338478&nbsp;&nbsp; |
| &nbsp;&nbsp;result2026035020026.pos&nbsp;&nbsp; | &nbsp;&nbsp;48.149286770863&nbsp;&nbsp; | &nbsp;&nbsp;16.283834924148&nbsp;&nbsp; |
| &nbsp;&nbsp;result2026036020000.pos&nbsp;&nbsp; | &nbsp;&nbsp;48.149286798081&nbsp;&nbsp; | &nbsp;&nbsp;16.283835460302&nbsp;&nbsp; |
| &nbsp;&nbsp;result2026037020000.pos&nbsp;&nbsp; | &nbsp;&nbsp;48.149286919252&nbsp;&nbsp; | &nbsp;&nbsp;16.283835496834&nbsp;&nbsp; |
| &nbsp;&nbsp;result2026038020000.pos&nbsp;&nbsp; | &nbsp;&nbsp;48.149286670760&nbsp;&nbsp; | &nbsp;&nbsp;16.283835066090&nbsp;&nbsp; |
| &nbsp;&nbsp;result2026039020000.pos&nbsp;&nbsp; | &nbsp;&nbsp;48.149286846305&nbsp;&nbsp; | &nbsp;&nbsp;16.283835422324&nbsp;&nbsp; |
| &nbsp;&nbsp;result2026040020000.pos&nbsp;&nbsp; | &nbsp;&nbsp;48.149286713159&nbsp;&nbsp; | &nbsp;&nbsp;16.283835438211&nbsp;&nbsp; |
| &nbsp;&nbsp;result2026041020000.pos&nbsp;&nbsp; | &nbsp;&nbsp;48.149286842099&nbsp;&nbsp; | &nbsp;&nbsp;16.283835050953&nbsp;&nbsp; |
| &nbsp;&nbsp;average&nbsp;&nbsp;                 | &nbsp;&nbsp;48.149286811309&nbsp;&nbsp; | &nbsp;&nbsp;16.283835274668&nbsp;&nbsp; |

All results are within a circle with a radius of 26 mm. The maximum distance between 2 points is 46 mm. <br>
But when I compare this result with the result I got using gpsrinex (Method 2) then there is an offset of about 77 cm. 
Comparing this with the averaging method 1 I see an offset of 1.09 meters. 




