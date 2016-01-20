---
layout: post
title:  Solaris Runtime Linking Environment
date:   2016-01-20 17:29:00 CET
categories: operating system 
---


If a program terminates with a message like the following: 
<pre>
ld.so.1: fwbuilder: fatal: lib_some_library_name: open failed: No such file or directory 
Killed 
Then you should configure the runtime linking environment 
</pre>

There are 2 methodes to setup $LD_LIBRARY_PATH correctly,
the recommended way and the manual way 


### using command "crle" 

Key in as root "crle" to see, wether the environment $LD_LIBRARY_PATH is usable. 
if not, update the environment with: 

<pre>
 crle -u -l /lib:/usr/lib:/usr/local/lib:/opt/sfw/lib:/usr/sfw/lib:/usr/local/qt/lib 
</pre>

Above is jsut an eaxmple. Make sure that all pathes are available you need. 

Check again the result with command crle 
the listed directories above should be in your linking environment 
see also "man crle" 
using command crle has permanent effects, file /var/ld/ld.config is created or modified 


### doing manually 

<pre>
# change $LD_LIBRARY_PATH ( more or less ) 
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/sfw/lib 
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/sfw/lib 
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/qt/lib
</pre>

you could change $HOME/.profile or /etc/profile to make this changes permanent ( not recommended ) 

