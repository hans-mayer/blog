---
layout: post
title:  iftop without garbage characters
date:   2016-08-12 11:39:00 CET
categories:
---

Sometimes it happens that this ASCII based semigraphic of ncurses based applications bring some garbage characters. This happens especially running "putty" on a Windows client connecting to a Linux box. Here an example with "iftop" 

<pre>
                   195Kb               391Kb              586Kb               781Kb          977Kb
mqqqqqqqqqqqqqqqqqqvqqqqqqqqqqqqqqqqqqqvqqqqqqqqqqqqqqqqqqvqqqqqqqqqqqqqqqqqqqvqqqqqqqqqqqqqqqqqqq
</pre>

To solve the problem one has to set the variable 

> NCURSES_NO_UTF8_ACS 

For example with bash syntax

> export NCURSES_NO_UTF8_ACS=1 ; iftop

and than it looks much better

<pre>
                   195Kb               391Kb              586Kb               781Kb          977Kb
+------------------+-------------------+------------------+-------------------+-------------------
</pre>


