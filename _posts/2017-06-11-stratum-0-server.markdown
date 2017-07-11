---
layout: post
title:  Stratum-1 NTP Server with rubidium source
date:   2017-06-11 17:06:00 CET
categories:
---


# Stratum-1 NTP Server with a rubidium frequency standard

To build a stratum-1 NTP server with a stratum 0 source is maybe a hobby. I am sure in most cases it's not necessary to build a high precisely time server in the own network. For most IT infrastructure it's good enough to get the time via NTP protocol from Internet. But sometime we are ambitious and we build our own NTP server.

This project results out of an idea to be independent. I am already running stratum 1 servers in the own network. These are a very low frequency receiver based server and a GPS disciplined NTP server. With this I am dependent of others. So could it be the political decision to switch off the system. This possibility is maybe small. But there could also be physical situations which prevents normal operation, for example during a solar storm. An other reason could be someone is using a jammer for a dedicated disruption of this service.

## Components to use

There is always the question about the costs. And therefore which time source can be used so that it is not too expensive. A rubidium frequency standard you can buy nowadays (2017) for about 300 EUR in the second-hand market at Ebay. My decision was to buy an EFRATOM LPRO-101 as time source. The server should also be  cheap. Therefore I decided to use a Banana Pi M1. At that time when I started this project (it's about 2 years ago) the M1 was the only with a 1GB interface. Of course a short latency is always good for NTP. And the operating system has to be a Linux tpye.

## How-to build such a server

With a hardware like the Banana Pi or a Raspberry Pi one has already an interface called GPIO. Source code of NTP classic or NTPsec has a driver which can handle an 1PPS (one pulse per second) signal. The rubidium oscillator delivers a 10 MHz sinus wave. To get a 1PPS you need a divider by 10^7. All together 3 components.

* Banana Pi M1 micro computer with GNU/Linux
* 10 MHz oscillator EFRATOM LPRO-101
* divider by 10^7

And of course power supplies for these devices.

The Banana Pi M1 and the EFRATOM I could buy. The divider I had to develop by myself. I put this product at GitHub. You can find the source for this PC-board here

[GitHub: divider by 10^7](https://github.com/hans-mayer/teiler10e7){:target="_blank"}

## OS with 1PPS support

As operating system I am using Bananian OS which is a Debian Linux for ARM hardware.
Even during the installation of my first GPS based NTP server I figured out this OS does not support 1PPS per default. So it's necessary to compile the kernel. How to do you can see  in this posting:

[Banana Pi compile kernel and header](/2016/01/01/bananapi-compile-kernel.html){:target="_blank"}

Once you have done this successfully you can modify the kernel to use 1PPS. Details you can find here

[Kernel with 1PPS support](/2017/06/12/kernel-with-1PPS.html){:target="_blank"}

During my test I figured out that a second 1PPS driver would be useful. This driver can be loaded only once. So a second one has to be compiled. This is not a necessary step. But it makes the live easier, especially during the phase of adjusting the rubidium frequency standard. How to do you can find here at GitHub:

[GitHub: pps2gpio - a second 1PPS](https://github.com/hans-mayer/pps2gpio){:target="_blank"}

If done the drive must be loaded. A typical way is to add a line in file

<pre>
/etc/modules
</pre>

with the following content

<pre>
pps-gpio
</pre>

As you can see this is the default driver above, which I use if there is a GPS module installed. For the rubidium 1PPS I load the driver via a script in /etc/init.d

<pre>
if test -z "`lsmod | grep pps_2gpio`"
  then
    echo insmod pps-2gpio.ko gpio_pin=17 1>&2
    insmod pps-2gpio.ko gpio_pin=17
  else
    echo pps-2gpio.ko already loaded 1>&2
fi
lsmod | grep pps
</pre>

This must be executed in the directory where you compile the driver source. Out put should be

<pre>
pps_2gpio               3376  1
pps_gpio                3357  1
pps_core               10107  4 pps_gpio,pps_2gpio
</pre>

As I said, output may be different, if you don't use pps_2gpio. Now you should test if 1PPS is really available. There is a tool called "ppstest". You can download at

[GitHub: ppstest](https://github.com/ago/pps-tools.git){:target="_blank"}

Running it, it should show you each second a timestamp.

<pre>
# ls -ld /dev/pps?
crw------- 1 root root 242, 0 May 14 22:22 /dev/pps0
crw------- 1 root root 242, 1 May 14 22:22 /dev/pps1

# ppstest /dev/pps1
trying PPS source "/dev/pps1"
found PPS source "/dev/pps1"
ok, found 1 source(s), now start fetching data...
source 0 - assert 1497522635.000049542, sequence: 2711411 - clear  0.000000000, sequence: 0
source 0 - assert 1497522636.000046069, sequence: 2711412 - clear  0.000000000, sequence: 0
source 0 - assert 1497522637.000043472, sequence: 2711413 - clear  0.000000000, sequence: 0
source 0 - assert 1497522638.000039958, sequence: 2711414 - clear  0.000000000, sequence: 0
source 0 - assert 1497522639.000037235, sequence: 2711415 - clear  0.000000000, sequence: 0
^C
</pre>

Not to get confused about the pin numbering of the GPIO here some documentation: [Banana Pi - GPIO - WiringBP](/2016/01/08/bananapi-gpio-wiringbp.html){:target="_blank"}

## Adjusting the EFRATOM rubidium standard.

Yes, the rubidium frequency standard must be adjusted. It is a high precisely source. Much better than everthing else once it's running really at 10000000.00 MHz.

Here some links:

* [EFRATOM LPRO-101 adjusting](/2016/01/03/efratom-lpro-101-adjusting.html){:target="_blank"}
* [my second Efratom LPRO-101](/2016/07/25/second-efratom.html){:target="_blank"}
* [EFRATOM LPRO-101 Pin 7 voltage vs. offset](/2016/01/31/efratom-lpro-101-pin7-voltage.html){:target="_blank"}

## Power supply

Normally it's not necessary to speak about power supplies. The modern way is to use a switched mode power supply. With such a power supply I made some bad experience. It took me a lot of time to figure out what's going wrong, why is the time source not stable. Until I found a solution with a classic three-terminal positive-voltage regulator LM317. Maybe a low pass filter could also solve this issue.


## Software and scripts

Here you can find all necessary scripts and configuration files for this stratum-1 server.

[GitHub: stratum1-scripts](https://github.com/hans-mayer/stratum1-scripts/){:target="_blank"}
Coming soon.


## Case

I had an old Sun Blade 100 workstation. From this I removed all the original electronic components. After that I mounted the EFRATOM LPRO-101 (left side), the Banana Pi M1 (right front) and the power supply (right back). With this I will not win a designer price but I am proud to build my own rubidium disciplined NTP server.

![rubi0_case.png](/images/rubi0_case.png){: width="800px"}
