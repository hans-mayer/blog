---
layout: post
title:  macOS - verify certificate with openssl
date:   2016-11-05 18:06:00 CET
categories: IT security 
---

After installing an application with a certificate one should verify if this was well done. For example for a mail gateway - running SMTP on port 25 - this is typically done with the following command 

<pre>
echo QUIT |  openssl s_client -connect mail.gateway.domain:25 -starttls smtp 
</pre>

If everything is fine you get lines like this ( as an example ) 

<pre>
CONNECTED(00000003)
depth=2 /C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
verify return:1
depth=1 /C=US/O=GeoTrust Inc./CN=RapidSSL SHA256 CA - G3
verify return:1
depth=0 /CN=mail.gateway.domain
verify return:1
</pre>

... and many other lines. Return code "1" means success.

But not so with macOS ( In my case version 10.12.1 ) 
Trying the same it could be a result like this 

<pre>
CONNECTED(00000003)
depth=2 /C=IL/O=StartCom Ltd./OU=Secure Digital Certificate Signing/CN=StartCom Certification Authority
verify error:num=19:self signed certificate in certificate chain
verify return:0
</pre>

or this 

<pre>
CONNECTED(00000003)
depth=1 /C=US/O=GeoTrust Inc./CN=RapidSSL SHA256 CA - G3
verify error:num=20:unable to get local issuer certificate
verify return:0
</pre>

Return code "0" says there is an error. 

The issue is "openssl" doesn't know where the root certificates are located in the file system. To overcome this problem "openssl" has the option 

<pre>
-CApath /path/to/certs
</pre>

This is in most UNIX systems "/etc/ssl/certs"

Unfortunately macOS doesn't have it. The certificates are stored in key-chains. You can find all of them if you open the "Keychain Access" application. If you select "System Roots" and "Certificates" you can find all of them. But "openssl" doesn't use this. 

One solution could be to export the certificates with the "Keychain Access" application. But this is a very painful work. Another solution could be to copy all these files from another UNIX / Linux / Solaris system to any directory of your choice and run "openssl" with the "-CApath" option. This is the approach I used. 


