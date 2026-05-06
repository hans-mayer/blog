---
layout: post
title:  RTK - base/rover with U-blox GNSS receivers
date:   2026-05-03 16:47:00 CET
categories: GNSS
---


After some tests with Precise Point Positioning ( see below 1-5 ) and in detailed with (4) I describe this time a setup with a base station sending RTCM (Radio Technical Commission for Maritime Services) data to a rover station. For both ( base and rover ) I use Raspberry Pi's with a GNSS pi-hat with the latest Debian OS trixie Version 13 and the latest Version of [https://gitlab.com/gpsd/gpsd](https://gitlab.com/gpsd/gpsd){:target="_blank"}. I also use [github.com/rtklibexplorer/RTKLIB](https://github.com/rtklibexplorer/RTKLIB){:target="_blank"} written by Jens Reimann. It's not necessary to have this tool on these servers. Most of the time I use this from a third server as well as the gpsd package where I use "ubxtool" remotely. 

Base station is a Raspberry Pi5 with a [ZED-X20P](/2026/03/09/u-blox_ZED-X20P.html){:target="_blank"} from U-blox. The pi-hat is from sparkfun. <br>
Rover is a Raspberry Pi4 with a [ZED-F9P](/2022/07/29/ublox-ZED-F9P.html){:target="_blank"} from U-blox. The pi-hat is from uputronics. <BR>
OS is in both cases Debian 13 (trixie)
 
In both cases I use the second interface UART2 to communicate between base and rover for the RTCM traffic. How to setup I described in [second interface for u-blox receiver](/2026/03/09/second-interface-for-u-blox-receiver-on-pi4-and-pi5.html){:target="_blank"}

In advance I want to say that this combination with ZED-X20P and ZED-F9P is not perfect but possible. The reasons are multiple: ZED-F9P can handle only the L1 and L2 band. ZED-X20P is designed for L1/L2/L5/E6/B3/L. Another reason is that ZED-X20P cannot handle GLONASS (Globalnaja nawigazionnaja sputnikowaja sistema) at the moment. (And maybe never) And the Navigation Indian Constellation (NavIC) can only be used by ZED-X20P. Independant of that I don't see any Indian satellite here in Vienna ( 48N 16E ). Therefore there are left 3 GNSS: GPS, Galileo and BeiDou as least common multiple and common source. 

Below you can find 2 scripts. The first one is to setup the base station which is a little bit more laborious. The second one is for the rover. These scripts require certain prerequisites. For example there is a server with hostname "base" and "rover" or at least an DNS CNAME for it. SSH should be possible without password. 

functubxtool_ksh defines a function "ubxtool" like this 

<pre>
    export UBXOPTS='-P 27.50'
    /usr/local/bin/ubxtool $@ rover:gpsd:/dev/serial0 
</pre>

This is to avoid to add each time "rover:gpsd:/dev/serial0" as aditional argument 

## setup_base_sh 

<pre>
#!/usr/bin/env bash 

# ident setup_base_sh 
# Wed May  6 05:26:22 PM CEST 2026 - mayer 

. functubxtool_ksh base

ntrip(){

  logger -p user.debug "setup_base_sh ntrip with argument $1  " 
  
  case "$1" in 
    stop ) 
        # kill a possible running str2str 
        ssh base pkill str2str 
        ;; 
    start ) 
        # start a new one - this is the communication to the rover for RTCM traffic 
        ssh base "str2str -in serial://ttyAMA3:921600:8:n:1:off -out tcpsvr://:42101 --deamon"
        ;;
    status ) 
        ssh base 'pgrep -a -f "str2str -in serial://ttyAMA3:921600:8:n:1:off -out tcpsvr://:42101 --deamon"' 
        ;;
    "" ) 
        ntrip stop ; ntrip start 
        ;;
  esac 
}


setup_initial(){ 

  # Setup Script for u-blox (ZED-X20P) base station 
  logger -p user.debug "setup_base_sh setup_initial " 
  # this is the initial setup to prepare the base sation for it function 
  
  # make sure in advance that baudrate for uart1 is high enough 
  if test -z "`ubxtool -g  CFG-UART1-BAUDRATE | grep UART1-BAUDRATE | head -1 | grep 921600`" 
    then 
      echo $0: UART1-BAUDRATE,921600 failed 
      exit 1 
  fi 
  
  ubxtool -z  CFG-UART2-BAUDRATE,921600 | grep UBX-ACK-ACK: 
  
  if test $? -ne 0 
    then 
      echo $0: UART1-BAUDRATE,921600 failed 
      exit 1 
  fi 

  # make sure that no Survey-In is running 
  ubxtool -z CFG-TMODE-MODE,0 | grep UBX-ACK-ACK:
  
  # reference coordinates set to ECEF
  ubxtool -z CFG-TMODE-POS_TYPE,0 | grep UBX-ACK-ACK:
  
  # position of the base station , unit is cm 
  # 48.1493013022 16.2838442507 288.08 
  # make sure that there is the exact position of the base station 
  ubxtool -z CFG-TMODE-ECEF_X,409252331 | grep UBX-ACK-ACK:
  ubxtool -z CFG-TMODE-ECEF_Y,119548502 | grep UBX-ACK-ACK:
  ubxtool -z CFG-TMODE-ECEF_Z,472818312 | grep UBX-ACK-ACK:
  
  # set  High-Precision Register to zero 
  ubxtool -z CFG-TMODE-ECEF_X_HP,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-TMODE-ECEF_Y_HP,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-TMODE-ECEF_Z_HP,0 | grep UBX-ACK-ACK:
  
  # RTCM data (1 Hz Intervall, 1230 each 5 Sek)
  # activate RTCM3 output on UART2 (Port 2)
  # 1005: Station ID & Position, 1077-1207: MSM7 Nachrichten for GPS, GLO, GAL, BDS
  for msg in 1005 1077 1087 1097 1124 1127 ; do
    ubxtool -z CFG-MSGOUT-RTCM_3X_TYPE${msg}_UART2,1  | grep UBX-ACK-ACK:
  done
  ubxtool -z CFG-MSGOUT-RTCM_3X_TYPE1230_UART2,5 | grep UBX-ACK-ACK:
  
  # FIXED MODE schalten
  ubxtool -z CFG-TMODE-MODE,2 | grep UBX-ACK-ACK:

  # necessary as the rover (ZED-F9P) can only bands L1 and L2 
  ubxtool -z CFG-SIGNAL-PLAN,1 | grep UBX-ACK-ACK:
    
  ubxtool -z CFG-SIGNAL-GPS_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-SBAS_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-GAL_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-BDS_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-QZSS_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-GLO_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-NAVIC_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-BDS_B2A_ENA,1 | grep UBX-ACK-ACK:
  
  ubxtool -z CFG-SIGNAL-GPS_L1CA_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-GPS_L2C_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-GPS_L5_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-SBAS_L1CA_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-GAL_E1_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-GAL_E5A_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-GAL_E5B_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-GAL_E6_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-BDS_B1_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-BDS_B2_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-BDS_B1C_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-BDS_B3_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-QZSS_L1CA_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-QZSS_L1S_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-QZSS_L2C_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-QZSS_L5_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-GLO_L1_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-GLO_L2_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z CFG-SIGNAL-NAVIC_L5_ENA,0 | grep UBX-ACK-ACK:

  ntrip 

}  
  
help(){
  echo "usage: $0 help | setup_initial | ntrip " 
  echo "          help  ...  this help "
  echo "          setup_initial ... this will initialise the base station " 
  echo '          ntrip start | stop | status | "" '
  echo "                to manage the communication with the base station with a str2str process "
  echo "                without argument it will restart the str2str process " 
  exit 1 
}

usage(){ 
  echo "usage: $0 help | setup_initial | ntrip " 
  exit 1 
} 


case "$1" in 
  setup_initial ) setup_initial ;; 
  ntrip ) ntrip $2 ;; 
  help ) help ;; 
  * ) usage ;; 
esac 

</pre>

<br>

## setup_rover_sh

<pre>
#!/usr/bin/env bash 

# ident setup_rover_sh 
# Wed May  6 05:26:22 PM CEST 2026 - mayer 

. functubxtool_ksh rover


ntrip(){

  logger -p user.debug "setup_rover_sh ntrip with argument $1  " 
  case "$1" in 
    stop ) 
  	# kill a possible running str2str 
  	# ssh rover pkill str2str 
	ssh rover 'pkill -f "str2str -in tcpcli://base:42101 -out serial://ttyAMA5:921600:8:n:1:off --deamon"'
	;; 
    start ) 
  	# start a new one - this is the communication to the base for RTCM traffic 
  	ssh rover "str2str -in tcpcli://base:42101 -out serial://ttyAMA5:921600:8:n:1:off --deamon"
	;;
    status ) 
	ssh rover 'pgrep -a -f "str2str -in tcpcli://base:42101 -out serial://ttyAMA5:921600:8:n:1:off --deamon"'
	;;
    "" ) 
	ntrip stop ; ntrip start 
	;;
  esac 
}

setup_initial(){ 
  
  logger -p user.debug "setup_rover_sh setup_initial " 
  # this is the initial setup to prepare the rover sation for it function 

  # make sure in advance that baudrate for uart1 is high enough 
  if test -z "`ubxtool -g  CFG-UART1-BAUDRATE | grep UART1-BAUDRATE | head -1 | grep 921600`" 
    then 
      echo $0: UART1-BAUDRATE,921600 failed 
      exit 1 
  fi 
  
  ubxtool -z  CFG-UART2-BAUDRATE,921600 | grep UBX-ACK-ACK: 
  
  if test $? -ne 0 
    then 
      echo $0: UART1-BAUDRATE,921600 failed 
      exit 1 
  fi 
  
  # is set per default layer 7 
  ubxtool -z CFG-UART2INPROT-RTCM3X,1  | grep UBX-ACK-ACK: 
  
  # disable not usable GNSS 
  ubxtool -z  CFG-SIGNAL-SBAS_ENA,0 | grep UBX-ACK-ACK: 
  ubxtool -z  CFG-SIGNAL-QZSS_ENA,0 | grep UBX-ACK-ACK: 
  ubxtool -z  CFG-SIGNAL-GLO_ENA,0 | grep UBX-ACK-ACK: 
  
  ubxtool -z  CFG-SIGNAL-SBAS_L1CA_ENA,0 | grep UBX-ACK-ACK: 
  ubxtool -z  CFG-SIGNAL-QZSS_L1CA_ENA,0 | grep UBX-ACK-ACK: 
  ubxtool -z  CFG-SIGNAL-QZSS_L1S_ENA,0 | grep UBX-ACK-ACK: 
  ubxtool -z  CFG-SIGNAL-QZSS_L2C_ENA,0 | grep UBX-ACK-ACK: 
  ubxtool -z  CFG-SIGNAL-GLO_L1_ENA,0 | grep UBX-ACK-ACK: 
  ubxtool -z  CFG-SIGNAL-GLO_L2_ENA,0 | grep UBX-ACK-ACK: 

  ntrip 
  
  ubxtool -z CFG-MSGOUT-UBX_RXM_RTCM_UART1,1 | grep UBX-ACK-ACK:
  
  # set high precision mode 
  # The accuracy is only as good as that of the base station. 
  ubxtool -z CFG-NMEA-HIGHPREC,1 | grep UBX-ACK-ACK: 
  ubxtool -z CFG-MSGOUT-UBX_NAV_HPPOSLLH_UART1,1 | grep UBX-ACK-ACK: 

  # don't restart gpsd after initialization 
  # this is one of the parameters changed at start or use option -p --passive for gpsd restart 
  ubxtool -z CFG-MSGOUT-NMEA_ID_GGA_UART1,1  | grep UBX-ACK-ACK:  

}

nmea_pipe(){
  	
  logger -p user.debug "setup_rover_sh nmea_pipe with argument $1  " 
  # this pipe is for monitoring with rtkplot_q 

  case "$1" in 
    stop ) 
  	# kill a possible running gpspipe 
  	ssh rover "pkill -f 'socat EXEC:gpspipe -r TCP-LISTEN:10001,reuseaddr,fork'"
	;; 
    start )  
  	# start a gpspipe for monitoring with rtkplot_qt / option -r is NMEA output 
  	ssh rover nohup "socat EXEC:'gpspipe -r' TCP-LISTEN:10001,reuseaddr,fork  > /dev/null 2>&1 & disown " 
	echo pipe ready for rtkplot_qt as TCP Client , server rover at port 10001 and solution format NMEA0183 
 	;;
    status )  
	ssh rover 'pgrep -a -f "socat EXEC:gpspipe -r TCP-LISTEN:10001,reuseaddr,fork"'
 	;;
    "" ) 
	nmea_pipe stop ; nmea_pipe start 
 	;;
  esac  
}

raw_pipe(){

  logger -p user.debug "setup_rover_sh raw_pipe  with argument $1  " 

  case "$1" in 
    stop ) 
  	# kill a possible running gpspipe
  	ssh rover "pkill -f 'socat EXEC:gpspipe -RB TCP-LISTEN:10002,reuseaddr,fork'"
	;;
    start ) 
  	# start a gpspipe for logging with strsvr_qt or str2str 
  	ssh rover nohup "socat EXEC:'gpspipe -RB' TCP-LISTEN:10002,reuseaddr,fork  > /dev/null 2>&1 & disown " 
  	echo for example its possible to start now: str2str -in tcpcli://rover:10002 -out file://log_%Y%m%d%h%M.ubx
	;;
    status ) 
  	ssh rover "pgrep -a -f 'socat EXEC:gpspipe -RB TCP-LISTEN:10002,reuseaddr,fork'"
	;;
    "" )
	raw_pipe stop ; raw_pipe start 
	;;
  esac 
}


navpvt(){
  logger -p user.debug "setup_rover_sh navpvt " 
  ubxtool -p NAV-PVT -v 2 | sed -n -e '/^UBX-NAV-PVT:/,/^$/ p' | awk -v RS= 'NR==2' 
}


sat_used(){

  logger -p user.debug "setup_rover_sh sat_used " 

  # NAVSAT=`ubxtool -p NAV-SAT -v 2 | sed -n -e '/^UBX-NAV-SAT:/,/^$/ p' | awk -v RS= 'NR==2'  | grep -B 2 -A 2  -i rtcm | egrep 'flags'`

  echo only GPS, Galileo and BeiDou are counted 
  echo -e -n Status: ; navpvt | grep carrSoln
  NAVSAT=`ubxtool -p NAV-SAT -v 2 | sed -n -e '/^UBX-NAV-SAT:/,/^$/ p' | awk -v RS= 'NR==2' ` 

  echo "                    satellites total seen : " `echo "$NAVSAT" | grep -c gnssId `

  SYST=`echo "$NAVSAT" | grep gnssId  | awk '{ print ( $2 ) }' | uniq -c`
  echo $SYST | awk '{ print ( "                                      GPS :  "  $1 "  Galileo: " $3  "  BeiDou: " $5 ) }' 

  echo "                          satellites used : " `echo "$NAVSAT" | grep 'flags(' | grep -c svUsed `

  SYST=`echo "$NAVSAT" | grep  -B 2 svUsed | grep gnssId  | awk '{ print ( $2 ) }' | uniq -c`
  echo $SYST | awk '{ print ( "                                      GPS :  "  $1 "  Galileo: " $3  "  BeiDou: " $5 ) }' 

  echo "     satellites used with rtcm coorection : " `echo "$NAVSAT" | grep -c rtcm `
  echo "  satellites with pseudorange corrections : " `echo "$NAVSAT" | grep -c prCorrUsed `

  echo "satellites with carrier range corrections : " `echo "$NAVSAT" | grep -c crCorrUsed `
  SYST=`echo "$NAVSAT" | grep  -B 2 crCorrUsed | grep gnssId  | awk '{ print ( $2 ) }' | uniq -c`
  echo $SYST | awk '{ print ( "                                      GPS :  "  $1 "  Galileo: " $3  "  BeiDou: " $5 ) }' 
}

help(){ 
  echo "usage: $0 help | setup_initial | nmea_pipe | raw_pipe | sat_used | navpvt | ntrip " 
  echo "          help  ... this help " 
  echo "          setup_initial ... this will initialise the base station " 
  echo "          nmea_pipe ... this will create a gpspipe with NMEA protocol listen on port 10001 "
  echo "          raw_pipe ... this will create a gpspipe with raw data listen on port 10002 "
  echo "          sat_used ... will show the used satellites based on ubxtool -p NAV-SAT command "
  echo "          navpvt ... will show the status based on ubxtool -p NAV-PVT command "
  echo '          ntrip start | stop | status | "" '
  echo "                to manage the communication with the base station with a str2str process "
  echo "                without argument it will restart the str2str process " 
  exit 1 
} 

usage(){ 
  echo "usage: $0 help | setup_initial | nmea_pipe | raw_pipe | sat_used | navpvt | ntrip " 
  exit 1 
} 


case "$1" in 
  setup_initial ) setup_initial ;; 
  nmea_pipe ) nmea_pipe "$2" ;; 
  raw_pipe ) raw_pipe "$2" ;; 
  sat_used ) sat_used ;; 
  navpvt ) navpvt ;; 
  ntrip ) ntrip "$2" ;; 
  help ) help ;; 
  * ) usage ;; 
esac 

</pre>

## Usage 

### setup_initial 

### nmea_pipe

After setting up base and rover it will take some time to get a precision position with status `Fixed`. Worst case is one hour in my situation. But typically it takes 10 minutes or a little bit more. Running command `setup_rover_sh nmea_pipe` will create a gpspipe with `socat EXEC:gpspipe -r TCP-LISTEN:10001,reuseaddr,fork`. Running `rtkplot_qt &` and connecting to this port 10001 will show you the current possition at the rover. 

![rover for a short period](/images/rover_short_2026.png)

The graph above shows the measurement for a short period of time. Each dot symbols a second. As we can see almost all dots are within a circle of 5 mm radius. If I move the rover antenna for example 3 cm away from the current possition then a new cloud of dots will be create in a distance of 3 cm from the old one. If the antenna is moved further away - for example one meter - then the status "Fixed" is lost and falls back to "Floating". 
 

![rover for a longer period](/images/rover_long_2026.png)

The example above shows a longer period of time and we can see that all dots are in a square of 3 x 3 cm. <br> <br>

### sat_used 

With option `sat_used` will give you the information how many satellites are used. A typical output could be this: 

```
only GPS, Galileo and BeiDou are counted
Status:    carrSoln (Fixed)
                    satellites total seen :  38
                                      GPS :  11  Galileo: 11  BeiDou: 16
                          satellites used :  26
                                      GPS :  10  Galileo: 6  BeiDou: 10
     satellites used with rtcm coorection :  26
  satellites with pseudorange corrections :  26
satellites with carrier range corrections :  21
                                      GPS :  7  Galileo: 6  BeiDou: 8
```

In the example above we see a status fixed. I have never seen more than 25 satellites with carrier range corrections. Maybe this is a limitation from the U-blox receiver or there are never more than 25 satellites available with the needed requirements. <br>
There are 3 states possible 

```
    carrSoln (None)
    carrSoln (Floating)
    carrSoln (Fixed)
```
Status "None" is only visible short time after power on. Fixed is of course our goal. <BR>

### raw_pipe 

Using option `raw_pipe` will create a second gpspipe. This will allow to use `str2str -in tcpcli://rover:10002 -out file://log_%Y%m%d%h%M.ubx` which logs the data to file. Than it's possible to exctract data in a future process. 

### navpvt 

navpvt show the output of command "ubxtool -p NAV-PVT"

```
UBX-NAV-PVT:
  iTOW 576850000 time 2026/05/02 16:13:52 valid x37
  tAcc 24 nano 341196 fixType 3 flags x83 flags2 xea
  numSV 29 lon 162837865 lat 481492049 height 277494
  hMSL 235357 hAcc 15 vAcc 24
  velNED 2 2 16 gSpeed 3 headMot 24410926
  sAcc 131 headAcc 18000000 pDOP 119 flags3 x4 reserved0 x334c2e2c
  headVeh 0 magDec 0 magAcc 0
    valid (validDate ValidTime fullyResolved)
    fixType (3D)
    flags (gnssFixOK, diffSoln, Carrier Phase fixed,)
    flags2 (confirmedAvai confirmedDate confirmedTime)
    psmState (Not Active)
    carrSoln (Fixed)
    flags3 () lastCorrectionAge 2
```

## some internal links 

These are some possibilities to look for a precise point position. Definitelly one needs to have one exact position for the base station in a rover/base setup. 

(1) [PPP - Precise Point Positioning with averaging](/2023/06/03/PPP-Precise-Point-Positioning.html){:target="_blank"} <br>
(2) [PPP with gpsrinex, CSRS-PPP and ECTT](/2026/01/21/PPP-with-gpsrinex.html){:target="_blank"} <br>
(3) [PPP with RTKlib and local correction](/2026/02/21/PPP-with-RTKLIB.html){:target="_blank"} <br>
(4) [PPP with NTRIP source for u-blox GNSS receiver over gpsd](/2026/02/28/PPP-with-NTRIP-source.html){:target="_blank"} <br>
(5) [PPP with NTRIP source and rtknavi_qt](/2026/03/15/PPP-with-NTRIP-source-and-rtknavi_qt.html){:target="_blank"} <br>

