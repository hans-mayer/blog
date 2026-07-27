---
layout: post
title:  Update ZED-X20P with Debian on piHAT
date:   2026-07-23 12:32:00 CET
categories: u-blox GNSS
---

Here I want to show you how to update a ZED-X20P without u-center2. This scenario is typically used if the X20P is mounted on a piHAT sitting on a Raspberry Pi running Linux like Debian. <BR>

In my case, I'm use the pHAT from sparkfun. It is mounted on a Raspberry Pi5 with Debian 13 trixie. And of course there is no easy way to run u-center2 directly. I found information to connect the Linux box and the Windows box with "ser2net" on Debian and HW VSP (Virtual Serial Port) on Windows part. But this didn't work for me. 

Luckily, I found a thread at https://portal.u-blox.com/ where someone posted a binary file "ubxfwupdate" which runs perfect on 64-bit ARM platform. 

So I had to download the latest image which is available from the official U-blox website: UBX_20_HPG_210_ZED_X20P-01B.512369040097ce18fd3475e71e7c627f.bin

Normally I run "gpsd" from the gpsd package. It's important to stop this as we need the serial interface for the update. 

<pre>
systemctl stop gpsd.service 
systemctl stop gpsd.socket 
</pre>

The next step is also important. Set the baud rate for UART1 to the same value as the first parameter of the "-b" option. In my case it is 460800 bd. Since I also have the [second interface](/2026/03/09/second-interface-for-u-blox-receiver-on-pi4-and-pi5.html){:target="_blank"} available I could do this easily over the second interface for UART1. 

```ubxtool3 -z CFG-UART1-BAUDRATE,460800``` 

Now check the communication between X20P and tty of the Raspberry.

```/usr/local/bin/ubxtool -f /dev/ttyAMA0 -s 460800 -p MON-VER``` 

This should show the current (old) version. 

Now comes the magic moment. Run the update:

<pre>ubxfwupdate -p /dev/ttyAMA0  -v 1 --no-fis 1 -C 1 -s 1 -b 460800:9600:460800 UBX_20_HPG_210_ZED_X20P-01B.512369040097ce18fd3475e71e7c627f.bin</pre>

<pre>
----------CMD line arguments-----------
Image file:        UBX_20_HPG_210_ZED_X20P-01B.512369040097ce18fd3475e71e7c627f.bin
Flash:             <compiled-in>
Fis:               flash.xml
Port:              /dev/ttyAMA0
Baudrates:         460800/9600/460800
Safeboot:          1
Reset:             1
AutoBaud:          0
Verbose:           1
Erase all:         1
Erase only:        0
Training sequence: 1
Chip erase:        1
Merging FIS:       1
Update RAM:        0
Use USB alt:       0
---------------------------------------
  0.0 u-blox Firmware Update Tool version 24.11
  0.0 Updating Firmware 'UBX_20_HPG_210_ZED_X20P-01B.512369040097ce18fd3475e71e7c627f.bin' of receiver over '/dev/ttyAMA0'
  0.0   - Opening and buffering image file
  0.0   - Verifying image
  0.0   - Got an encrypted image with footer info
  0.0   - CRC Value         	:    BC410967
  0.0   - Footer Version    	:    1
  0.0   - Number of Images  	:    2
  0.0   - Footer Size       	:    28
  0.0   - Image Config Size 	:    40
  0.0   - Image 0 Size     	:    716648
  0.0   - Image 1 Size     	:    444856
  0.0   - Trying to open port /dev/ttyAMA0
  0.0   - Setting baudrate to 460800
  0.2   - Received Version information
  0.2   - Receiver currently running SW 'EXT HPG 2.02 (43e74c)'
  0.2   - Receiver HW '000B0000', Generation 20.0
  0.2   - Sending ROM CRC Poll
  0.2 ROM CRC: 0x00A9D329
  0.2 u-blox20 ROM1.10 hardware detected (0x00A9D329)
  0.2  Getting Port connection to receiver
  1.6   - Connected port is: UART1
  1.6  Commanding Safeboot
  5.1   - Setting baudrate to 9600
  5.3   - Sending training sequence
  5.5  Detecting Flash manufacturer and device IDs
  5.5   - Flash ManId: 0x009D DevId: 0x6016
  5.7   - Not merging anything
  5.7   - Flash size:   4194304
  5.7   -  Flash block:  1024 x 4096
  5.7 Stable clock enabled successfully
  5.8   - Setting baudrate to 460800
  6.0  Start Flash retention
  6.0   - flash retention success
  6.0 Chip erase started
  6.0  Receiver info collected, downloading to flash...
 11.5 Chip erase complete
 37.0 FW download complete
 37.0  Verifying Image on hardware
 39.4  Firmware Verification and Flash update complete
 39.4  Verify Flash retention
 39.4  flash retention success
 39.4  Rebooting receiver
 39.4 Firmware Update SUCCESS
</pre>

If you check the MON_VER again, you should see:

<pre>
UBX-MON-VER:
  swVersion EXT HPG 2.10 (b0eda3)
  hwVersion 000B0000
  extension ROM BASE 0x00A9D329
  extension FWVER=HPG 2.10
  extension PROTVER=50.11
  extension MOD=ZED-X20P
  extension GPS;GLO;GAL;BDS
  extension SBAS;QZSS
  extension NAVIC;LBAND
</pre>

And what you can see here GLONASS is also available, at least in my geographic region. 

My [ZED-X20P](/2026/03/09/u-blox_ZED-X20P.html){:target="_blank"} from sparkfun in action. 

