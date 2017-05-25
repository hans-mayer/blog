---
layout: post
title:  NTPsec far-sighted decision for year 2038
date:   2017-05-25 17:13:00 CET
categories: ntp unix linux
---

### Do we have time to wait ? Definitely not.

This I call a really far-sighted decision from the NTPsec developer team. If we are looking now into the syslog of a 32-bit UNIX system than we can already see the end of all 32-bit systems in 2038.

<pre>
May 25 20:51:23 ntp ntpd[]: This system has a 32-bit time_t.
May 25 20:51:23 ntp ntpd[]: This ntpd will fail on 2038-01-19T03:14:07Z.
</pre>

Just to be sure there is no misunderstanding: now we have year <b>2017</b>.

But I am not sure, do we still have 32-bit systems in 21 years ?
