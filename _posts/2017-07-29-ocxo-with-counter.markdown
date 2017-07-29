---
layout: post
title:  10 MHz OCXO with 40 MHz frequency counter
date:   2017-07-29 17:28:00 CET
categories:
---

## The idea

Not all my hardware projects get a case. Some years ago I bought one and currently I can't remember for what it was intended. But last year I decided to install my OCX0 and a frequency counter into this case.

I used the OCXO oscillator for two reasons:  <br>
* During my tests with NTP it was a valuable tool. <br>
* As ham radio operator I need a good refence to fine tune my radios. <br>

The frequency counter is not designed by me. This is a project I found at https://www.mikrocontroller.net/topic/200279 <br> I bought the empty pc-board from the author and soldered all the components and programmed the Atmel chip. 

As this case is too big for each device alone I found both fit very well together into this housing. The case is about 20 cm x 17 cm.  

Here some pictures about this project which I finished more than a year ago.

## The circuit

The circuit is simple. I bought a cheap 12 V switching power supply. This I used for the OCXO together with 2 coils and a capacitor as low pass filter. This 12 V line is also source for a three-terminal positive-voltage regulator 7805.

![ocxo_circuit](/images/ocxo_circuit.png)

## Front view

In the front view you can see the BNC connectors. "Out" is from the OCXO oscillator and "IN" is for the 40 MHz frequency counter. With the 10-turn poti it's possible to adjust the oscillator and to change it for about 8.4 PPB per 100 scale lines. So the change from one marker to the next is theoretically less than 0.1 PPB or 0.001 Hz. But 0.2 PPB is realistic.

![ocxo_front](/images/ocxo_front.png)

## Top view

On the top side of the image you can see the Isotemp OCXO124-10. Below there is the 12 V power supply. On the left part there is the 40 MHz frequency counter.

![ocxo_top](/images/ocxo_top.png)

## Back view

I made the experience - and I share this opinion with others - that an OCXO needs about 9 month until it runs quite stable. Quite stable is less than 0.5 PPB.
Switching off for long time will reset the "memory" of this OCXO and you need again long time to be stable. A short time outage is less critical. With short time I mean much less than an hour. Especially if the quartz doesn't cool down. <br>
Therefore I designed the possibility to connect a 12 V battery in case of a power outage.

![ocxo_back](/images/ocxo_back.png)

## Data sheet

[Isotemp OCXO 134-10](/images/isotemp_ocxo_134-10.pdf){:target="_blank"}
