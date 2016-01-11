---
layout: post
title:  Solaris port 80 access for non privileged user
date:   2016-01-11 11:18:00 CET
categories:
---

Sometimes it's necessary to run a process as non priviledged user on a port lower than 1024. 

Assuming there is a non priviledged user called "usernonpriv"

On Solaris 10 and 11 you can run 

> usermod -K defaultpriv=basic,net_privaddr usernonpriv 

This will allow that user "usernonpriv" can start a process listeing on port 80 or 443 

It adds a line to /etc/user_attr 

> usernonpriv::::defaultpriv=basic,net_privaddr


