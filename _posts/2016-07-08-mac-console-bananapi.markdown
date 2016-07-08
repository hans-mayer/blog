---
layout: post
title:  MAC as serial console for BananaPi
date:   2016-07-08 06:22:00 CET
categories: 
---


How to use a MAC with OS X as a serial console for a Banana Pi M3 running Linux 

Buy an USB serial TTL adapater like this 

![usb_ttl_adapter](/images/usb_ttl_adapter.png) 

For the chipset PL-2303HX one needs a driver. Try to find this 

PL2303_MacOSX_1.6.0_20151022.zip

or a newer one. And install the package.

<img src="/images/usb_ttl_driver_mac.png"  width="600">

After a reboot you can connect the adapter. You should find a USB device with "ls -ld /dev/\*usb*" like this 

<pre>
crw-rw-rw-  1 root  wheel   18,   3 Jul  8 09:39 /dev/cu.usbserial
crw-rw-rw-  1 root  wheel   18,   2 Jul  8 09:48 /dev/tty.usbserial
</pre>

Now connect the cables. Ground ( black ) to ground. Red cable ( +5V ) stays unconnected. Connect the white cable RxD from the adapter to TX of the BananaPi and the green cable TxD from the adapter to RX of the BananaPi. 

Open a "terminal" session and enter

> screen /dev/tty.usbserial 115200 8N1 

After booting the Pi you should see the boot process.


