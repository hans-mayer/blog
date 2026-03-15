---
layout: post
title:  PPP with NTRIP source and rtknavi_qt to compute
date:   2026-03-15 11:13:00 CET
categories: 
---


This is now my fifth attempt and probably the last one to get an exact position (Precise Point Positioning) of my fixed mounted GNSS antenna at the roof of my house. 
You can find the methods I used previously in my blogs here: <br> 
(1) [PPP - Precise Point Positioning with averaging](/2023/06/03/PPP-Precise-Point-Positioning.html){:target="_blank"} <br>
(2) [PPP with gpsrinex, CSRS-PPP and ECTT](/2026/01/21/PPP-with-gpsrinex.html){:target="_blank"} <br>
(3) [PPP with RTKlib and local correction](/2026/02/21/PPP-with-RTKLIB.html){:target="_blank"} <br>
(4) [PPP with NTRIP source for u-blox GNSS receiver over gpsd](/2026/02/28/PPP-with-NTRIP-source.html){:target="_blank"} <br>


As GNSS receiver I used my brand new [u-blox ZED-X20P](/2026/03/09/u-blox_ZED-X20P.html){:target="_blank"} <br>
to manage this device I use the [gpsd](https://gitlab.com/gpsd/gpsd){:target="_blank"} package. 

The method is quite simple. Use `rtknavi_qt` to configure a `rover / base station` setup. Rover is the own GNSS receiver with a fix mounted antenna. The base station is an external RTCM stream. So feed RTCM date as a NTRIP ( Networked Transport of RTCM via Internet Protocol ) stream to rtknavi_qt. rtknavi_qt is part of the package RTKlib which can be found here [github.com/rtklibexplorer/RTKLIB](https://github.com/rtklibexplorer/RTKLIB){:target="_blank"}. I run it on Debian Linux. 

To do so, one must use any NTRIP caster. There are several available for free and of course also some commercial. In any case you have to register as you need username and password. 

This methode doesn't differ very much to method (4). The only difference: now the processing is done by rtknavi_qt, in (4) it was done by the u-blox receiver itself. 

To prepare my u-blox for this job I run the following commands: 

<pre>
ubxtool -z CFG-MSGOUT-UBX_RXM_SFRBX_UART1,1 | grep ACK-ACK
ubxtool -g CFG-MSGOUT-UBX_RXM_SFRBX_UART1 | grep UART
ubxtool  | grep SFRBX

ubxtool -z CFG-MSGOUT-UBX_RXM_RAWX_UART1,1 | grep ACK-ACK
ubxtool -g CFG-MSGOUT-UBX_RXM_RAWX_UART1 | grep UART
ubxtool  | grep RAWX

# DYNMODEL stationary 
ubxtool -z CFG-NAVSPG-DYNMODEL,2 | grep ACK-ACK
ubxtool -g CFG-NAVSPG-DYNMODEL | grep CFG-NAVSPG-DYNMODEL
</pre>
 
Now configure rtknavi_qt 

Open `rtknavi_qt &` and new window will appear. <br>
Click on `Options...` on the bottom left site there is a `Load...` button. Load the file `f9p_ppk.conf` which comes with the source tree. Some of the settings I changed. `Positioning Mode` I set to `Static` as my antenna is fix mounted. As `Navigation Systems` I selected GPS, Galileo and BDS. This should fit what the base station is delivering. In the `Positions` tab for `Base Station` I selected `RTCM/Raw Antenna Position`. It is also possible to set Lat/Lon/height or X/Y/Z but as long as the base station is propagating its own position "RTCM/Raw Antenna Position" is the easy way. <br>
Back in the main menu press `I` for input streams. Select `Rover` with `TCP Client` as Stream Type. As Stream Options enter IP address and port number where you start <br>

`socat EXEC:'gpspipe -RB' TCP-LISTEN:10001,reuseaddr,fork  &` 

and Format `u-blox UBX` 

`gpspipe` must be able to connect to the own `gpsd` process and `socat` offers this data on port 10001 <br>

The base station is `NTRIP Client`, in Stream Options enter Caster Address, port, mountpoint, user name and password what you want to use. Format is `RtCM 3`. 

![rtknavi input streams](/images/rtknavi_input.png)

Press the "OK" button. Ideally save the new configuration with a new name in the options tab. <br>

Now it's time to press the `Start` button.  Very soon I get as solution "float". But it took almost every time up to 30 minutes to get status `FIX`. So take a coffee or do something useful in the meantime. 

![rtknavi main screen](/images/rtknavi_main.png)

Even I can see 40 or more satellites normally only 14 of them are used for calculation. If not enough satellites are available a "FIX" could fail completely. 

Press the `O` button to define an output stream if you want to document the results. 


