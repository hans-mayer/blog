---
layout: post
title:  NTP and ADSL - theoretical aspects 
date:   2016-05-02 20:29:00 CET
categories: 
---

Here I want to show some theoretical aspects using NTP over a network connection which has different delays in both directions. This is typical given with ADSL. Before continuing reading here you should read my [first blog about NTP and ADSL](/2016/03/16/ntp-and-adsl.html){:target="_blank"}.


## Symmetric situation 

First lets look at a situation where delay from server to client is identical to the delay from client to server. This is also the situation where NTP works correct. For a detailed description see [NTP Timestamp Calculations](https://www.eecis.udel.edu/~mills/time.html){:target="_blank"}. 

Let's assume both time server are in sync ( have the same time ) 

![ntp_timestamp1.png](/images/ntp_timestamp1.png)

As we can easily see delay between T1 and T2 is identical with delay between T3 and T4. With the formula for offset 

> offset = [(T2 - T1) + (T3 - T4)] / 2

we get the result 0. 

## Non symmetrical situation 

Now the non symmetrical situation as it is given with ADSL for example. This ntp frame reaches the server later based on the lower upload rate in opposite to the higher download speed. We can say 

> T2' = T2 + x 

And "x" is the additional delay given by asymmetry. 

![ntp_timestamp2.png](/images/ntp_timestamp2.png)

The formula for offset is now 

> 0 = [(T2 + x - T1) + (T3 - T4)] / 2

We can set this value to 0 as we know both time server are in sync. Rewriting the mathematical formula we get  

> x = T1 - T2 - T3 + T4 

for the additional delay. 

I tracked the packet with "tcpdump" running "ntpdate -d -d -d -d -u ip.address" and got the following values: 

<pre>
T1=0.410634 ; T2=0.415284 ; T3=0.415471 ; T4=0.430596
</pre>

Of course I dropped the values left of the decimal point. The values for timestamp 2 and 3 I read out from the answer packet coming from the server. The values for timestamp 1 and 4 I took from "wireshark". I know this is not exactly but it is also not sooo bad. 

Doing the math to verify the results I got 

<pre>
# echo -n "delay " ; dc <<< "7k $T4 $T1 -  $T3 $T2 - - p"
delay .019775
</pre>

The value for delay coming from "ntpq -p" is 19.664 - so quite near my hand measured result. 

<pre>
# echo -n "offset "  ; dc <<< "7k $T2 $T1 - $T3 $T4 - + 2/p "
offset -.0052375
</pre>

Also offset fits well. -5.342 is the value from ntpq. 

Now calculating x: 

<pre>
# echo -n "x = "  ; dc <<< "7k $T1 $T2 - $T3 - $T4 +p"
x = .010475
</pre>

Splitting x into two parts each 50% we have the symmetrical situation. And of course ' x / 2 = offset ' respectively ' x = 2 * offset '. This is not really surprising as the formula for "x" is coming out of the formula for offset. It is quite similar. Some signs are different and we lost factor 2. But it is also what we have seen in my [previous blog](/2016/03/16/ntp-and-adsl.html){:target="_blank"}. 

So we know the additional delay is twice the offset we see with "ntpq" for a 90 byte UDP packet. But even with a symmetric connection to the internet there are a lot of components which produces different delays from A to B and from B to A. Therefore we never know how accurate our time information is. 


