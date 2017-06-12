---
layout: post
title:  Kernel with 1PPS support
date:   2017-06-12 11:30:00 CET
categories:
---

First take a look how to compile the kernel.

[Banana Pi compile kernel and header](/2016/01/01/bananapi-compile-kernel.html){:target="_blank"}


I recommend to compile the kernel without modifications in the first step to see if it compiles without any issues. It take much time. In my situation more than 7 hours. But don't worry a modification goes much faster.

Follow the instructions how to enable 1PPS. (If it's not already done in your kernel.)

Before you start make a copy of file <b>.config</b>

For example

    cp .config .config_original_2017

<pre>
# make menuconfig
 .config - Linux/arm 3.4.108 Kernel Configuration
 ──────────────────────────────────────────────────────────────────────────────
  ┌──────────────── Linux/arm 3.4.108 Kernel Configuration ─────────────────┐
  │  Arrow keys navigate the menu.  .Enter. selects submenus ---..          │  
  │  Highlighted letters are hotkeys.  Pressing .Y. includes, .N. excludes, │  
  │  .M. modularizes features.  Press .Esc..Esc. to exit, .?. for Help,     │  
  │  for Search.  Legend: [*] built-in  [ ] excluded  .M. module  . .       │  
  │ ┌─────────────────────────────────────────────────────────────────────┐ │  
  │ │        General setup  ---.                                          │ │  
  │ │    [*] Enable loadable module support  ---.                         │ │  
  │ │    -*- Enable the block layer  ---.                                 │ │  
  │ │        System Type  ---.                                            │ │  
  │ │    [ ] FIQ Mode Serial Debugger                                     │ │  
  │ │        Bus support  ---.                                            │ │  
  │ │        Kernel Features  ---.                                        │ │  
  │ │        Boot options  ---.                                           │ │  
  │ │        CPU Power Management  ---.                                   │ │  
  │ │        Floating point emulation  ---.                               │ │  
  │ │        Userspace binary formats  ---.                               │ │  
  │ │        Power management options  ---.                               │ │  
  │ │    -*- Networking support  ---.                                     │ │  
  │ │        Device Drivers  ---.                                         │ │  
  │ │        File systems  ---.                                           │ │  
  │ │        Kernel hacking  ---.                                         │ │  
  │ │        Security options  ---.                                       │ │  
  │ │    -*- Cryptographic API  ---.                                      │ │  
  │ │        Library routines  ---.                                       │ │  
  │ │    ---                                                              │ │  
  │ │        Load an Alternate Configuration File                         │ │  
  │ │        Save an Alternate Configuration File                         │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ └─────────────────────────────────────────────────────────────────────┘ │  
  ├─────────────────────────────────────────────────────────────────────────┤  
  │                    .Select.    . Exit .    . Help .                     │  
  └─────────────────────────────────────────────────────────────────────────┘  

</pre>

Scroll down to the "Device Drivers" section and press the "return" button if the "Select" field is high lighted. Now you see

<pre>
 .config - Linux/arm 3.4.108 Kernel Configuration
 ──────────────────────────────────────────────────────────────────────────────
  ┌──────────────────────────── Device Drivers ─────────────────────────────┐
  │  Arrow keys navigate the menu.  .Enter. selects submenus ---..          │  
  │  Highlighted letters are hotkeys.  Pressing .Y. includes, .N. excludes, │  
  │  .M. modularizes features.  Press .Esc..Esc. to exit, .?. for Help, ./. │  
  │  for Search.  Legend: [*] built-in  [ ] excluded  .M. module  . .       │  
  │ ┌─────────────────────────────────────────────────────────────────────┐ │  
  │ │        Generic Driver Options  ---.                                 │ │  
  │ │    {*} Connector - unified userspace .-. kernelspace linker  ---.   │ │  
  │ │    .M. Memory Technology Device (MTD) support  ---.                 │ │  
  │ │    .M. Parallel port support  ---.                                  │ │  
  │ │    [*] Block devices  ---.                                          │ │  
  │ │        Misc devices  ---.                                           │ │  
  │ │        SCSI device support  ---.                                    │ │  
  │ │    .*. Serial ATA and Parallel ATA drivers  ---.                    │ │  
  │ │    [*] Multiple devices driver support (RAID and LVM)  ---.         │ │  
  │ │    .M. Generic Target Core Mod (TCM) and ConfigFS Infrastructure  --│ │  
  │ │    [*] Network device support  ---.                                 │ │  
  │ │    [ ] ISDN support  ---.                                           │ │  
  │ │        Input device support  ---.                                   │ │  
  │ │    [*] Gsensor support  ---.                                        │ │  
  │ │        Character devices  ---.                                      │ │  
  │ │    {*} I2C support  ---.                                            │ │  
  │ │    [*] SPI support  ---.                                            │ │  
  │ │    . . HSI support  ---.                                            │ │  
  │ │        PPS support  ---.                                            │ │  
  │ │        PTP clock support  ---.                                      │ │  
  │ │    [*] GPIO Support  ---.                                           │ │  
  │ │    .*. Dallas's 1-wire support  ---.                                │ │  
  │ │    {*} Power supply class support  ---.                             │ │  
  │ │    .*. Hardware Monitoring support  ---.                            │ │  
  │ │    .*. Generic Thermal sysfs driver  ---.                           │ │  
  │ │    [*] Watchdog Timer Support  ---.                                 │ │  
  │ │        Sonics Silicon Backplane  ---.                               │ │  
  │ └────v(+)─────────────────────────────────────────────────────────────┘ │  
  ├─────────────────────────────────────────────────────────────────────────┤  
  │                    .Select.    . Exit .    . Help .                     │  
  └─────────────────────────────────────────────────────────────────────────┘  

</pre>

Now go down to "PPS support" and press "return" again.

I selected the following kernel modules with "M"

<pre>
 .config - Linux/arm 3.4.108 Kernel Configuration
 ──────────────────────────────────────────────────────────────────────────────
  ┌────────────────────────────── PPS support ──────────────────────────────┐
  │  Arrow keys navigate the menu.  .Enter. selects submenus ---..          │  
  │  Highlighted letters are hotkeys.  Pressing .Y. includes, .N. excludes, │  
  │  .M. modularizes features.  Press .Esc..Esc. to exit, .?. for Help, ./. │  
  │  for Search.  Legend: [*] built-in  [ ] excluded  .M. module  . .       │  
  │ ┌─────────────────────────────────────────────────────────────────────┐ │  
  │ │    .M. PPS support                                                  │ │  
  │ │    [ ]   PPS debugging messages                                     │ │  
  │ │    [ ]   PPS kernel consumer support                                │ │  
  │ │          *** PPS clients support ***                                │ │  
  │ │    .M.   Kernel timer client (Testing client, use for debug)        │ │  
  │ │    .M.   PPS line discipline                                        │ │  
  │ │    .M.   Parallel port PPS client                                   │ │  
  │ │    .M.   PPS client using GPIO                                      │ │  
  │ │        *** PPS generators support ***                               │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ │                                                                     │ │  
  │ └─────────────────────────────────────────────────────────────────────┘ │  
  ├─────────────────────────────────────────────────────────────────────────┤  
  │                    .Select.    . Exit .    . Help .                     │  
  └─────────────────────────────────────────────────────────────────────────┘  

</pre>

Finally save the new configuration. You can verfy the changes with the diff command.

    diff .config .config.old

You could build a new kernel. But probably you need also a patch for this driver. It is from Chen Wei weichen302@gmail.com and you can find it here:

[https://github.com/infinet/bananapi/blob/master/pps-over-gpio.patch](https://github.com/infinet/bananapi/blob/master/pps-over-gpio.patch){:target="_blank"}
