---
layout: post
title:  Scapy - kill an established TCP session
date:   2025-02-09 08:08:08 CET
categories: network
---

Scapy is a great tool to learn and to understand the Internet protocol. In this proof of concept I want to show how to kill an established TCP connection.
To kill an established TCP connection we need 5 information:
* source IP address and source port
* destination IP address and destination port
* and also the sequence number.

In my environment I have 2 systems:
* a server 192.168.241.9 with a web service running on port 80 and where I run Scapy
* a client 192.168.241.89 with a telnet session to the web server

To make the example easy for guessing the sequence I use a long running but idle session to the web server. But good enough to demonstrate how to kill a session.

So lets start on the web server a script which will finally execute Scapy:
<pre>
192.168.241.9# tcpdump -i eno1 -c 1 -n  host 192.168.241.89 and src port 80 2> /dev/null | \
  awk '{ print ( $3 , $5 , $9 + 1 ) }' | tr -d ':,' | tr "." " " | \
  awk '{ print ( "send (IP(src=\""$1"."$2"."$3"."$4"\",dst=\""$6"."$7"."$8"."$9"\")/TCP(sport="$5",dport="$10",flags=\"R\",seq="$11"))" ) }' | \
  awk '{ print ( "scapy -H <<< \x27" $0 "\x27"  ) }' | \
  bash -x
</pre>

So lets start now a telnet session against the web service on port 80.

<pre>
192.168.241.89# telnet 192.168.241.9 80
Trying 192.168.241.9...
Connected to 192.168.241.9.
Escape character is '^]'.
Connection closed by foreign host.
</pre>

It takes about 2 seconds and the session is closed again. Under normal conditions the session would timeout after 45 seconds. In the Scapy session we can see

<pre>
Welcome to Scapy (2024.05.15) using IPython 8.5.0
>>> .
Sent 1 packets.
>>>
</pre>

How does it work ?  We need tcpdump to get the necessary information. The telnet session creates a three way handshake: SYN -> SYN ACK -> ACK

<pre>
18:17:00.817436 IP 192.168.241.89.54686 > 192.168.241.9.80: Flags [S], seq 348254769
18:17:00.817511 IP 192.168.241.9.80 > 192.168.241.89.54686: Flags [S.], seq 2464209222, ack 348254770
18:17:00.817671 IP 192.168.241.89.54686 > 192.168.241.9.80: Flags [.], ack 1, win 502
</pre>

What we need is the sequence from the frame sending the webserver back: 2464209222

Therefore for tcpdump we use `src port 80`. Option `-c 1` says terminate if the first matching frame is arrived which is the second one on the wire at 18:17:00.817511.

We need for Scapy a line like this <br>
<pre>send (IP(src="192.168.241.9",dst="192.168.241.89")/TCP(sport=80,dport=54686,flags="R",seq=2464209223))</pre>

Of course this could typed in in a Scapy session but has to be done within the timeout. So we use the next two awk's to do it faster and without typos. To "guess" the sequence we add one the sequence we got from tcpdump by doing $9 + 1. The third awk puts the string in single quote (\x27) and prepares the program call. This we push finally to a shell.

And finally we can see about 2 seconds later the packet with the R-Flag.

<pre>
18:17:02.550497 IP 192.168.241.9.80 > 192.168.241.89.54686: Flags [R], seq 2464209223
</pre>
