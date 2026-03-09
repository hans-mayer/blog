---
layout: post
title:  second interface for u-blox GNSS receiver on Pi4 and Pi5
date:   2026-03-09 18:35:00 CET
categories: 
---

This post will just describe how to setup a second interface (UART2) on a Raspberry Pi4 and Pi5 with Debian 13 (trixie). <br>
A detailed description how to prepare a Raspberry Pi4 and Pi5 can be found here: <br>
[GPSD on Raspberry Pi5 with Debian Trixie](/2025/12/20/GPSD-NTP-on-Raspberry-PI5-with-Trixie.html){:target="_blank"} <br>
[GPSD on Raspberry Pi4 with Debian Bullseye](/2022/06/19/GPSD-NTP-on-Raspberry-PI4-with-Bullseye.html){:target="_blank"}

I own a pHAT from uputronics with a ZED-F9P on Pi4 and a pHAT from sparkfun with a ZED-X20P on Pi5. <br>
Both GNSS receivers have a second interface called UART2 and both systems are wired to GPIO pins of the Raspberry Pi.

And sometimes it's useful to have a second connection to the receiver. For example than if the interface speed of the primary interface is intended changed. But there are also other usecases where it's useful to have access over a second way. 

## Pi 4 ##

In my case the uputronics board UART2 is connected to GPIO12 for TXD5 and GPIO13 for RXD5. Don't mixup the GPIO name with the pin number. For example GPIO12 is pin 32 and GPIO13 is pin 33 on the 40 pin connector. 

For the Pi 4 the modification is easy. Add a line in `/boot/firmware/config.txt` in the global section

`dtoverlay=uart5`

and reboot. That's it. Now you will find a new device `/dev/ttyAMA5`. One can access this interface for example with `/usr/local/bin/ubxtool -f /dev/ttyAMA5 -s 38400` and the command you want to submit.

## Pi 5 ## 

On the Raspberry Pi5 it was a little bit more tricky as there is a complete redesign. The pin usage was unchanged but the hardware below changed. <br>
sparkfun did connect UART2 of ZED-X20P to GPIO8 and GPIO9 which is UART3. I created a section `[all]` already for the first interface at the end of the file `/boot/firmware/config.txt` and did add a new line with `dtoverlay=uart3-pi5`. The complete change looks like this 

<pre>
[all]
# iface 1: Standard-UART on GPIO 14 (TX) and 15 (RX)
enable_uart=1

# GPIO 8/9 (UART3)
dtoverlay=uart3-pi5
</pre >

Reboot. That's it. Now you will find a new device `/dev/ttyAMA3`. If you create a function like this 

<pre>
ubxtool3(){
  /usr/local/bin/ubxtool -f /dev/ttyAMA3 -s 38400 $@ 
}
</pre>

you can easily access the u-blox device without adding each time device and speed. Just use `ubxtool3` instead of `ubxtool`. 

Just another hint. I use this function for normal operation.

<pre>
ubxtool(){
  /usr/local/bin/ubxtool $@ localhost:gpsd:/dev/serial0 
}
</pre>

