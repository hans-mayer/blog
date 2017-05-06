---
layout: post
title:  NTPsec on macOS Sierra with Homebrew
date:   2017-05-06 07:45:00 CET
categories: time OSX
---

My impression is that NTP Classic from ntp.org will die. Maybe not immediately but the days will be counted. To deal with NTPsec can't be a wrong decision. And to have the tools on the private MacBook delivered by NTPsec is a nice feature. I describe here how to compile NTPsec on OS X.

My environment:

* macOS Sierra 10.12.4
* Homebrew from https://brew.sh/

Which formulas from brew are actually needed I can't say exactly. I installed "brew" with a lot of packages - sorry formulas - long time before. The recommended packages you can find in file "INSTALL" once you download the source tree. Definitely you need

* python
* openssl

And here are the necessary steps:

<pre>
git clone https://github.com/ntpsec/ntpsec.git
cd ntpsec
export LDFLAGS="-L/usr/local/opt/openssl/lib"
export CFLAGS="-I/usr/local/opt/openssl/include"
./waf configure
./waf build
./waf install
</pre>

That's it. Enjoy.

How to run ntpd on macOS is a different story.

Some important external links:

* [www.ntpsec.org](https://www.ntpsec.org/){:target="_blank"}
* [docs.ntpsec.org](https://docs.ntpsec.org/latest/){:target="_blank"}
* [gitlab.com/NTPsec/ntpsec](https://gitlab.com/NTPsec/ntpsec/){:target="_blank"}
* [github.com/ntpsec](https://github.com/ntpsec/ntpsec){:target="_blank"}
