---
layout: post
title:  Solaris 11 find PID for established TCP session
date:   2016-04-25 16:34:00 CET
categories: solaris
---


Assume you found an established TCP session with "lsof" or "netstat" and the port is &lt;PORTNUMBER&gt;

How to find the process ID for this connection ?

<pre>
pfiles `ls /proc` | ggrep -A 10 -B 64  &lt;PORTNUMBER&gt;
</pre>

With the command above you should see 10 lines above the end a line starting with "peername" and the given &lt;PORTNUMBER&gt;. Several lines above ( hopefully not more than 64 ) one can find the process ID. 



