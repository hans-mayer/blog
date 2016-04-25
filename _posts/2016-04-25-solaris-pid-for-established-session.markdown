---
layout: post
title:  Solaris 11 find PID for established TCP session
date:   2016-04-25 16:34:00 CET
categories: solaris
---


Assume you found an established TCP session with "lsof" or "netstat" 

How to find the process ID for this connection ?

### pfiles `ls /proc` | ggrep -A 64 -B 64  <PID> 



