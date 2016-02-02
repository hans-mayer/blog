---
layout: post
title:  Banana Pi - GPIO - WiringBP
date:   2016-01-08 20:01:00 CET
categories: 
---

This document describes how-to use a PIN of the GPIO module of the BananaPi M1

Download the source from **[https://github.com/LeMaker/WiringBP.git](https://github.com/LeMaker/WiringBP.git){:target="_blank"}** <br />
or fetch it with **git clone git@github.com:LeMaker/WiringBP.git**

Compile the package and install the binaries. 

After that installation one should be able to run command "gpio readall" 

<pre>
# gpio readall
 +-----+-----+---------+------+---+--Banana Pro--+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 |     |     |    3.3v |      |   |  1 || 2  |   |      | 5v      |     |     |
 |   2 |   8 |   SDA.1 | ALT5 | 0 |  3 || 4  |   |      | 5V      |     |     |
 |   3 |   9 |   SCL.1 | ALT5 | 0 |  5 || 6  |   |      | 0v      |     |     |
 |   4 |   7 | GPIO. 7 | ALT2 | 0 |  7 || 8  | 1 | IN   | TxD     | 15  | 14  |
 |     |     |      0v |      |   |  9 || 10 | 0 | IN   | RxD     | 16  | 15  |
 |  17 |   0 | GPIO. 0 | ALT2 | 0 | 11 || 12 | 0 | IN   | GPIO. 1 | 1   | 18  |
 |  27 |   2 | GPIO. 2 | ALT4 | 0 | 13 || 14 |   |      | 0v      |     |     |
 |  22 |   3 | GPIO. 3 | ALT4 | 0 | 15 || 16 | 0 | IN   | GPIO. 4 | 4   | 23  |
 |     |     |    3.3v |      |   | 17 || 18 | 0 | IN   | GPIO. 5 | 5   | 24  |
 |  10 |  12 |    MOSI |   IN | 0 | 19 || 20 |   |      | 0v      |     |     |
 |   9 |  13 |    MISO |   IN | 0 | 21 || 22 | 0 | ALT4 | GPIO. 6 | 6   | 25  |
 |  11 |  14 |    SCLK |   IN | 0 | 23 || 24 | 0 | IN   | CE0     | 10  | 8   |
 |     |     |      0v |      |   | 25 || 26 | 0 | IN   | CE1     | 11  | 7   |
 |   0 |  30 |   SDA.0 | ALT4 | 0 | 27 || 28 | 0 | ALT4 | SCL.0   | 31  | 1   |
 |   5 |  21 | GPIO.21 |   IN | 0 | 29 || 30 |   |      | 0v      |     |     |
 |   6 |  22 | GPIO.22 | ALT4 | 0 | 31 || 32 | 0 | ALT4 | GPIO.26 | 26  | 12  |
 |  13 |  23 | GPIO.23 |   IN | 0 | 33 || 34 |   |      | 0v      |     |     |
 |  19 |  24 | GPIO.24 |   IN | 0 | 35 || 36 | 0 | IN   | GPIO.27 | 27  | 16  |
 |  26 |  25 | GPIO.25 |   IN | 0 | 37 || 38 | 0 | IN   | GPIO.28 | 28  | 20  |
 |     |     |      0v |      |   | 39 || 40 | 0 | IN   | GPIO.29 | 29  | 21  |
 +-----+-----+---------+------+---+----++----+---+------+---------+-----+-----+
 | BCM | wPi |   Name  | Mode | V | Physical | V | Mode | Name    | wPi | BCM |
 +-----+-----+---------+------+---+--Banana Pro--+---+------+---------+-----+-----+
</pre>

BCM is the number one can use in the scripts.

For example I want to use BCM port 22 which is physical pin 15 ( the 8th pin from top in this row of pins ) 

Prepare this pin to use 

<pre>
# echo 22 > /sys/class/gpio/export
</pre>

Define the direction 

<pre>
# echo out > /sys/class/gpio/gpio22/direction 
</pre>

Now set the pin to LOW 

<pre>
# gpio -g write 22 0 
</pre>

or to HIGH state 

<pre>
# gpio -g write 22 1 
</pre>

Check the result on pin 15  with a voltmeter or an oscilloscope. 


