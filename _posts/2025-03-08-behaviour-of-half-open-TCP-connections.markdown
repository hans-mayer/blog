---
layout: post
title:  behavior of half open TCP connections
date:   2025-03-08 11:56:00 CET
categories: 
---

### This document describes my experience with "Scapy", "synflood.c" and half open TCP sessions. 

These days I stumbled over a description of half opened TCP sessions. <br/>

Very soon I realized that Scapy is a very nice tool to experiment with it. To create a half opened session the sender will initiate a SYN packet but will not complete the full handshake. 

A full TCP handshake is normally: <code>SYN -> SYN ACK -> ACK </code>

So with "Scapy" it shouldn't be an issue to send out just a single packet with a SYN flag and to see on the destination the half open session with ```netstat -nat | grep SYN_RECV```

So I run Scapy with 

```send(IP(src="192.168.231.9",dst="192.168.231.89")/TCP(dport=80,sport=1234,flags="S")) ```

But it failed. Debuging with tcpdump shows the reason: 

<pre><code> IP 192.168.231.9.1234 > 192.168.231.89.80: Flags [S],
 IP 192.168.231.89.80 > 192.168.231.9.1234: Flags [S.], 
 IP 192.168.231.9.1234 > 192.168.231.89.80: Flags [R], 
</code></pre>

Here we can see that the source is sending a RST and therefore the destination doesn't keep the session open. My first assumption was, the operating system is sending out a RST if the process is dead. But this assumption is wrong as I learned later. 

Therefore I was looking for a synflood program written in C which should actually work. There are many sources at GitHub available. I found one from Hemanth V. Alluri. He did really a nice job but I missed some features. As he archived his repository I was not able to create a pull request. Therefore I created my own repository. You can find it at [https://github.com/hans-mayer/synflood.c](https://github.com/hans-mayer/synflood.c)

I improved it by some options which are usefull for testing. <br>
There is the option ```--wait-time```. It waits before the attack and after the attack for a given time. The default is 3 seconds, good enough the switch to another window and start for example a ```tcpdump``` or another tool. And I realized that the source is sending a RST even if the process is still running. So the RST is coming from the OS, in my case the latest Debian version running on Intel. 

Scapy gives the possibility to send packets with a fake source address. If the source is an up and running server it will reply

```send(IP(src="192.168.231.190",dst="192.168.231.89")/TCP(dport=80,sport=1234,flags="S"))``` 

The frame above was sent out from source 192.168.231.9 and the tcpdump below is made at the destination server 192.168.231.89

<pre><code> IP 192.168.231.190.1234 > 192.168.231.89.80: Flags [S], 
 IP 192.168.231.89.80 > 192.168.231.190.1234: Flags [S.], 
 IP 192.168.231.190.1234 > 192.168.231.89.80: Flags [R.], 
</code></pre>

A server sends back a RST if there is no established session, which makes sense. To accept a SYN-ACK ip-address, port and sequence number must fit. 

"snflood.c" has an option ```--enable-spoofing``` and as described, packets with a fake source are mostly not routed over the Internet. And it has another bad behaviour if your target is in Internet. If your local firewall is well configured you will get flooded with FW messages as all these dropped fake IP adresses are coming from the wrong interface. And if you are not looking close enough you can think you get attacked from many different IP addresses. <br>

So I decided to create a new option ```--class-c-network-spoofing``` .
The difference is it will create fake IP addresses only out of the range where the server is located. Indeed it does not care about the network boundaries. It takes, as the name says, a complete class-C with 256 IP addresses. 
What happens can also be simulated with Scapy on server 192.168.231.9

```send(IP(src="192.168.231.2",dst="192.168.231.89")/TCP(dport=80,sport=1234,flags="S"))```

"tcpdump" at destination server shows again what happens. 

<pre><code> IP 192.168.231.2.1234 > 192.168.231.89.80: Flags [S], 
 ARP, Request who-has 192.168.231.2 tell 192.168.231.89, length 28
 ARP, Request who-has 192.168.231.2 tell 192.168.231.89, length 28
 ARP, Request who-has 192.168.231.2 tell 192.168.231.89, length 28
</code></pre>

The server is looking for a layer 2 MAC address with ARP. During this time it can't sent back a SYN-ACK. 

With this little script you can see there is a half opened session for 3 seconds. 
<pre><code> while true
  do  
    netstat -nat | grep SYN_RECV
    sleep 0.25 
    date
  done
</code></pre>

Between the time stamps you should get 12 lines with this information <br>
```tcp6       0      0 192.168.231.89:80       192.168.231.2:1234      SYN_RECV ```

3 seconds time until this SYN_RECV state is removed. An easy job for synflood.c 

``` synflood -v -h 192.168.231.89  -p 80 --enable-sniffer --attack-time 10 ``` 

On destination I run the following script: 

<pre><code> while true
  do  
    netstat -nat | grep SYN_RECV | wc -l
    sleep 0.25 
    date
  done
</code></pre>

The destination I used is an Respberry PI4. When the attack start the number of SYN_RECV states jumps up immediatelly to 511. When the attack finished it goes down to 500 and it takes almost 1 minute until the number goes down to zero. 

I checked also the responsibility of the target with the following script using check_tcp from Nagios. 
<pre><code> while true
  do  
    /usr/lib/nagios/plugins/check_tcp -H 192.168.231.89 -p 80
    sleep 0.25 
  done
</code></pre>

I got continuously replies within some 100 micro seconds independant if the attack was running or not. 

What I can say I couldn't bring down my server with a synflood attack. 

My conclusion, modern OS are save and healthy enough to prohibit a sucessfull simple synflood attack. There is probably a different situation if such an attack is done from many sources as DDOS. 

