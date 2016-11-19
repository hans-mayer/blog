---
layout: post
title:  web stick - send SMS from command line
date:   2016-11-19 09:56:00 CET
categories: 
---

Web sticks or Internet sticks are USB devices to connect a notebook to the internet if there is no wireless available. With the driver there comes typically a GUI where one can connect the network and send SMS too. Here I describe how to send a SMS from the command line. My environment 

* MacBook Pro with macOS Sierra version 10.12.1 
* Web Stick from T-Mobile, actually hardware is from Huawei Technologies Co., Ltd - Version 1.02 

But it should work with any Linux like operating system very similar. If you connect such a web stick you will find additional entries in "/dev". In my case it looks like this: 

<pre>
crw-rw-rw-   1 root   wheel           20,   2 Nov 19 13:54 tty.HUAWEIMobile-Modem
crw-rw-rw-   1 root   wheel           20,   3 Nov 19 13:54 cu.HUAWEIMobile-Modem
crw-rw-rw-   1 root   wheel           20,   6 Nov 19 13:54 tty.HUAWEIMobile-Pcui
crw-rw-rw-   1 root   wheel           20,   4 Nov 19 13:54 tty.HUAWEIMobile-Diag
crw-rw-rw-   1 root   wheel           20,   7 Nov 19 13:54 cu.HUAWEIMobile-Pcui
crw-rw-rw-   1 root   wheel           20,   5 Nov 19 13:54 cu.HUAWEIMobile-Diag
</pre>

Now we can connect to the device. We do this either with "screen" command or with "cu" command: 

<pre>
screen  /dev/tty.HUAWEIMobile-Modem 115200 8N1
</pre>

or

<pre>
sudo cu -l  /dev/cu.HUAWEIMobile-Modem -s 115200
</pre>

Such a device does typically not echo characters from input. Therefore we type in 

<pre>
at e1
</pre>

for easier communication in the future. Now we should see an "OK". And if wee press any key we will see it on the screen. 

Maybe you have to set the GSM modem to text mode SMS
<pre>
AT +CMGF=1
OK
</pre>

Probably you have to authenticate with a PIN. PIN and PUK are delivered by the Internet service provider. 

<pre>
AT+CPIN=1234
OK
</pre>

Now we are able to send a SMS. End of text is done by pressing "ctrl" + z. I symbolized this with the string ctrl-z.

<pre>
AT+CMGS="0123456789"
> hello world
> ctrl-z
+CMGS: 9

OK
</pre>

Within some seconds the owner of the phone with number 0123456789 should receive a SMS with text "hello world". 

To quit the session press ctrl-a k within the screen session or ~. within cu. 

If you want to run a SMS gateway permanently for receiving and sending SMS from your Linux box you should search for SMS Server Tools 3. Home page is http://smstools3.kekekasvi.com/ 


