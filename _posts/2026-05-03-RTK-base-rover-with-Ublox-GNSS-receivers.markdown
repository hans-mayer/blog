---
layout: post
title:  RTK - base/rover with U-blox GNSS receivers
date:   2026-05-03 16:47:00 CET
categories: 
---


After some tests with Precise Point Positioning ( see below 1-5 ) and in detailed with (4) I describe this time a setup with a base station sending RTCM (Radio Technical Commission for Maritime Services) data to a rover station. For both ( base and rover ) I use Raspberry Pi's with a GNSS pi-hat with the latest Debian OS trixie Version 13 and the latest Version of [https://gitlab.com/gpsd/gpsd](https://gitlab.com/gpsd/gpsd){:target="_blank"}. I also use [github.com/rtklibexplorer/RTKLIB](https://github.com/rtklibexplorer/RTKLIB){:target="_blank"} written by Jens Reimann. It's not necessary to have this tool on these servers. Most of the time I use this from a third server as well as the gpsd package where I use "ubxtool" remotely. 

Base station is a Raspberry Pi5 with a ZED-X20P from U-blox. The pi-hat is from sparkfun. <br>
Rover is a Raspberry Pi4 with a ZED-F9P from U-blox. The pi-hat is from uputronics. 

In both cases I use the second interface UART2 to communicate between base and rover for the RTCM traffic. How to setup I described in [second interface for u-blox receiver](/2026/03/09/second-interface-for-u-blox-receiver-on-pi4-and-pi5.html){:target="_blank"}

In advance I want to say that this combination with ZED-X20P and ZED-F9P is not perfect but possible. The reasons are multiple: ZED-F9P can handle only the L1 and L2 band. ZED-X20P is designed for L1/L2/L5/E6/B3/L. Another reason is that ZED-X20P cannot handle GLONASS (Globalnaja nawigazionnaja sputnikowaja sistema) at the moment. (And maybe never) And the Navigation Indian Constellation (NavIC) can only be used by ZED-X20P. Independant of that I don't see any Indian satellite here in Vienna ( 48N 16E ). Therefore there are left 3 GNSS: GPS, Galileo and BeiDou as least common multiple and common source. 

Below you can find 2 scripts. The first one is to setup the base station which is a little bit more laborious. The second one is for the rover. These scripts require certain prerequisites. For example there is a server with hostname "base" and "rover" or at least an DNS CNAME for it. SSH should be possible with password. 

functubxtool_ksh defines a function "ubxtool" like this 

<pre>
    export UBXOPTS='-P 27.50'
    /usr/local/bin/ubxtool $@ rover:gpsd:/dev/serial0 
</pre>

This is to avoid to add each time "rover:gpsd:/dev/serial0" as aditional argument 

## setup_base_sh 

<pre>
#!/usr/bin/env bash 

. functubxtool_ksh base

# ident setup_base_sh 

setup_initial(){ 

  # Setup Script for u-blox Base (X20P)
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
  
  # reference cooridnates set to ECEF
  ubxtool -z CFG-TMODE-POS_TYPE,0 | grep UBX-ACK-ACK:
  
  # position of the base station , unit is cm 
  # 48.1493013022 16.2838442507 288.08 
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
  ubxtool -z CFG-MSGOUT-RTCM_3X_TYPE1230_USB,5 | grep UBX-ACK-ACK:
  
  # FIXED MODE schalten
  ubxtool -z CFG-TMODE-MODE,2 | grep UBX-ACK-ACK:

  ubxtool -z  CFG-SIGNAL-PLAN,1 | grep UBX-ACK-ACK:
    
  ubxtool -z  CFG-SIGNAL-GPS_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-SBAS_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-GAL_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-BDS_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-QZSS_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-GLO_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-NAVIC_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-BDS_B2A_ENA,1 | grep UBX-ACK-ACK:
  
  ubxtool -z  CFG-SIGNAL-GPS_L1CA_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-GPS_L2C_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-GPS_L5_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-SBAS_L1CA_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-GAL_E1_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-GAL_E5A_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-GAL_E5B_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-GAL_E6_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-BDS_B1_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-BDS_B2_ENA,1 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-BDS_B1C_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-BDS_B3_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-QZSS_L1CA_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-QZSS_L1S_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-QZSS_L2C_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-QZSS_L5_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-GLO_L1_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-GLO_L2_ENA,0 | grep UBX-ACK-ACK:
  ubxtool -z  CFG-SIGNAL-NAVIC_L5_ENA,0 | grep UBX-ACK-ACK:

  # kill a possible running process 
  ssh base pkill str2str 
  # send the RTCM date to the rover 
  ssh base "str2str -in serial://ttyAMA3:921600:8:n:1:off -out tcpsvr://:42101 --deamon"

}  
  
usage(){ 
  echo "usage: $0 setup_initial " 
  exit 1 
} 


case "$1" in 
  setup_initial ) setup_initial ;; 
  * ) usage ;; 
esac 

</pre>

<br>

## setup_rover_sh

<pre>
#!/usr/bin/env bash 

# ident setup_rover_sh 

. functubxtool_ksh rover


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
  ubxtool -z  CFG-SIGNAL-SBAS_ENA,0
  ubxtool -z  CFG-SIGNAL-QZSS_ENA,0
  ubxtool -z  CFG-SIGNAL-GLO_ENA,0
  
  ubxtool -z  CFG-SIGNAL-SBAS_L1CA_ENA,0
  ubxtool -z  CFG-SIGNAL-QZSS_L1CA_ENA,0
  ubxtool -z  CFG-SIGNAL-QZSS_L1S_ENA,0
  ubxtool -z  CFG-SIGNAL-QZSS_L2C_ENA,0
  ubxtool -z  CFG-SIGNAL-GLO_L1_ENA,0
  ubxtool -z  CFG-SIGNAL-GLO_L2_ENA,0

  # kill a possible running str2str 
  ssh rover pkill str2str 
  
  # start a new one - this is the communication to the base for RTCM traffic 
  ssh rover "str2str -in tcpcli://192.168.241.93:42101 -out serial://ttyAMA5:921600:8:n:1:off --deamon"
  
  ubxtool -z CFG-MSGOUT-UBX_RXM_RTCM_UART1,1 | grep UBX-ACK-ACK:
  
  # set high precision mode 
  ubxtool -z CFG-NMEA-HIGHPREC,1 | grep UBX-ACK-ACK: 

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
	echo pipe ready at port 10001 
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
    "" )
	raw_pipe stop ; raw_pipe start 
	;;
  esac 
}

sat_used(){

  logger -p user.debug "setup_rover_sh sat_used " 

  # NAVSAT=`ubxtool -p NAV-SAT -v 2 | sed -n -e '/^UBX-NAV-SAT:/,/^$/ p' | awk -v RS= 'NR==2'  | grep -B 2 -A 2  -i rtcm | egrep 'flags'`
  NAVSAT=`ubxtool -p NAV-SAT -v 2 | sed -n -e '/^UBX-NAV-SAT:/,/^$/ p' | awk -v RS= 'NR==2' ` 
  echo "                    satellites total seen :  " `echo "$NAVSAT" | grep -c gnssId `
  echo "                          satellites used :  " `echo "$NAVSAT" | grep 'flags(' | grep -c svUsed `
  echo "     satellites used with rtcm coorection :  " `echo "$NAVSAT" | grep -c rtcm `
  echo "  satellites with pseudorange corrections :  " `echo "$NAVSAT" | grep -c prCorrUsed `
  echo "satellites with carrier range corrections :  " `echo "$NAVSAT" | grep -c crCorrUsed `
  SYST=`echo "$NAVSAT" | grep  -B 2 crCorrUsed | grep gnssId  | awk '{ print ( $2 ) }' | uniq -c`
  echo $SYST | awk '{ print ( "GPS: " $1 "  Galileo: " $3  "  BeiDou: " $5 ) }' 
}

navpvt(){
  logger -p user.debug "setup_rover_sh navpvt " 
  ubxtool -p NAV-PVT -v 2 | sed -n -e '/^UBX-NAV-PVT:/,/^$/ p' | awk -v RS= 'NR==2' 
}

usage(){ 
  echo "usage: $0 setup_initial | nmea_pipe | raw_pipe | sat_used | navpvt " 
  exit 1 
} 


case "$1" in 
  setup_initial ) setup_initial ;; 
  nmea_pipe ) nmea_pipe "$2" ;; 
  raw_pipe ) raw_pipe "$2" ;; 
  sat_used ) sat_used ;; 
  navpvt ) navpvt ;; 
  * ) usage ;; 
esac 

</pre>


(1) [PPP - Precise Point Positioning with averaging](/2023/06/03/PPP-Precise-Point-Positioning.html){:target="_blank"} <br>
(2) [PPP with gpsrinex, CSRS-PPP and ECTT](/2026/01/21/PPP-with-gpsrinex.html){:target="_blank"} <br>
(3) [PPP with RTKlib and local correction](/2026/02/21/PPP-with-RTKLIB.html){:target="_blank"} <br>
(4) [PPP with NTRIP source for u-blox GNSS receiver over gpsd](/2026/02/28/PPP-with-NTRIP-source.html){:target="_blank"} <br>
(5) [PPP with NTRIP source and rtknavi_qt](/2026/03/15/PPP-with-NTRIP-source-and-rtknavi_qt.html){:target="_blank"} <br>

