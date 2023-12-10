---
layout: post
title:  GPSD and NTP on Raspberry PI4 with Debian OS 11.3 Bullseye
date:   2022-06-19 11:07:00 CET
categories:
---


# OS installation

After 6 years running a GPS disciplined NTP server on old hardware I thought it's time to setup a new system. I took a Banana PI4 and I installed this image

```
2022-04-04-raspios-bullseye-arm64.img

```
I used the GPS-hat I already have since about 6 years.

<img src="/images/pi4_with_gps.jpg" width="400">

As you can see the vendor is M0UPU. You can buy the GPS modules at uputronics.com <br> At that time an ublox MAX-M8Q was used as chip on the gps hat.

I decided to compile GPSD and NTPD by myself and not to use a possible available package. To do so a lot of packages are needed:

```
scons libncurses5 libncurses5-dev ncurses-doc python3-matplotlib python3-matplotlib-dbg pps-tools libusb-1.0-0 libusb-1.0-0-dev libusb-1.0-doc asciidoctor asciidoctor-doc cu minicom libgtk-3-dev gtk+-3.0 python3-gi-cairo libcap-dev ksh

```

# GPSD

For complete documentation I want to refer to the Internet

> https://gpsd.io/building.html
>
> https://gpsd.gitlab.io/gpsd/installation.html

But compiling is quite straight forward:

`scons ; scons check ; scons install
`

If you decide later on to update to the latest version run:

```
git pull origin master --rebase
scons --config=force

```

# NTPD

For NTP I took the tar ball from ntp.org [http://www.ntp.org/downloads.html](http://www.ntp.org/downloads.html). Currently the latest version is 4.2.8p17
With the following commands compilation was easy

```
export CC=gcc
./configure  '--with-sntp' '--enable-RAWDCF' --enable-SHM --enable-ATOM -enable-NMEA '--enable-autokey' '--enable-simulator' --with-crypto --enable-clockctl --enable-linuxcaps
make

```

# OS configuration

To make it runnable some minor changes are necessary in the system. And this is maybe the most tricky part.

## /boot/cmdline.txt

This was the original content:

```
console=serial0,115200 console=tty1 root=PARTUUID=09abe25e-02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles

```
Strip away which makes troubles. Then it looks like this


```
root=PARTUUID=09abe25e-02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash

```
## /boot/config.txt

Add two lines:

```
dtoverlay=pps-gpio
enable_uart=1
```

## /etc/modules

Add one line:

```
pps-gpio
```

I also stoppped hciuart

`systemctl disable hciuart`

I modified /etc/group to be sure not having permission issues:

`dialout:x:20:admin,root,nobody`

## reboot

Now reboot the PI4 and check the logs. With

`dmesg | egrep 'gpio|pps|AMA|serial|gps'`

you should see something like this:

```
[    0.172164] pps_core: LinuxPPS API ver. 1 registered
[    0.172177] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    1.288574] gpiomem-bcm2835 fe200000.gpiomem: Initialised: Registers at 0xfe200000
[    1.415873] uart-pl011 fe201000.serial: there is not valid maps for state default
[    1.416154] uart-pl011 fe201000.serial: cts_event_workaround enabled
[    1.416301] fe201000.serial: ttyAMA0 at MMIO 0xfe201000 (irq = 19, base_baud = 0) is a PL011 rev2
[    1.424233] bcm2835-aux-uart fe215040.serial: there is not valid maps for state default
[    1.424976] fe215040.serial: ttyS0 at MMIO 0xfe215040 (irq = 20, base_baud = 62500000) is a 16550
[    3.209970] pps pps0: new PPS source pps@12.-1
[    3.210095] pps pps0: Registered IRQ 65 as PPS source
[    9.483539] pps_ldisc: PPS line discipline registered
[    9.485867] pps pps1: new PPS source serial0
[    9.485911] pps pps1: source "/dev/ttyS0" added
```

Check if the pps_gpio module is loaded

```
pi4:root# lsmod | grep pps_gpio
pps_gpio               16384  2
```

`modinfo pps_gpio` gives also some information

There are new devices

```
pi4:mayer# ls -ld /dev/serial? /dev/pps0
crw-rw---- 1 root root 250, 0 Jun 19 13:28 /dev/pps0
lrwxrwxrwx 1 root root      5 Jun 19 13:28 /dev/serial0 -> ttyS0
lrwxrwxrwx 1 root root      7 Jun 19 13:28 /dev/serial1 -> ttyAMA0
```

Checking pps0 is useable. It could also be that it is pps1. Of course a GPS-hat must be connected now. 

```
pi4:root# ppstest /dev/pps0
trying PPS source "/dev/pps0"
found PPS source "/dev/pps0"
ok, found 1 source(s), now start fetching data...
source 0 - assert 1655239442.996967522, sequence: 49 - clear  0.000000000, sequence: 0
source 0 - assert 1655239443.996906114, sequence: 50 - clear  0.000000000, sequence: 0
source 0 - assert 1655239444.996843175, sequence: 51 - clear  0.000000000, sequence: 0
source 0 - assert 1655239445.996787475, sequence: 52 - clear  0.000000000, sequence: 0
```

If this is the result then it's fine.

To see if data are received run

`minicom -b 9600 -o -D /dev/serial0`

You should see a lot of clear text data coming from the GPS module. Of course the speed could be a different. If you see some wired character then it's the wrong speed. In my case it's working with 9600 bd.

Now it's time to start gpsd and see if it is working

`/usr/local/sbin/gpsd -n -s 9600 /dev/serial0 /dev/pps0`

Now the following test should bring some results:

```
pi4:root# ntpshmmon
ntpshmmon: version 3.24.1~dev
#      Name  Seen@                 Clock                 Real                 L Prc
sample NTP0  1655647858.246635278  1655647858.161288573  1655647858.000016114 0  -1
sample NTP2  1655647858.246747648  1655647858.000497546  1655647858.000000000 0 -20
sample NTP2  1655647859.000767703  1655647859.000496185  1655647859.000000000 0 -20
sample NTP0  1655647859.152066030  1655647859.151809902  1655647859.000018310 0  -1
sample NTP2  1655647860.001379136  1655647860.000496214  1655647860.000000000 0 -20
```

To verify the working daemon run `cgps -s -u m`. You should see some satellite information.

Now it's time for ntpd

The relevant part in /etc/ntp.conf is the following

```
# pps needs another server as peer
# for easy start I take another stratum-1 in my network
server 192.168.241.190 minpoll 4 maxpoll 4 prefer

# Enabling PPS/ATOM support
server 127.127.22.0 minpoll 4 maxpoll 4
fudge 127.127.22.0 refid PPS time1 0.000500
fudge 127.127.22.0 flag3 1 flag4 1  # enable kernel PLL/FLL clock discipline and clockstats

# gpsd shared memory clock
server 127.127.28.0 minpoll 4 maxpoll 4 # PPS requires at least one preferred peer
fudge 127.127.28.0 refid GPS
fudge 127.127.28.0 time1 +0.15 flag4 1 # coarse processing delay offset
```

The time parameter "time1" you have to adjust for your situation. And of course several other configuration lines are necessary. If ntp.conf is ready start the deamon.

After some time you should see:

```
pi4:root# ntpq -pn
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
o127.127.22.0    .PPS.            0 l    5   16  377    0.000   +0.001   0.001
+127.127.28.0    .GPS.            0 l    3   16  377    0.000   +5.908   2.857
*192.168.241.190 .GPS.            1 u   10   16  377    0.712   +0.243  21.012
```

### That's it.

#### Just a word about NTP classic.

As you can see  the current and latest version of NTP is very old. I am not sure how long it will be available. There are alternatives, for example ntpsec.

But there is one reason. For this see my other blog https://blog.mayer.tv/2022/06/20/NTP-classic-with-.ntprc-file.html
