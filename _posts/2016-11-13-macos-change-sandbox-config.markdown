---
layout: post
title:  macOS - change sandbox config
date:   2016-11-13 18:55:00 CET
categories: 
---

Apple's operating system macOS is running in a "rootless" version. ( SIP ) A great idea. In traditional UNIX systems almost everything is allowed except those things which are forbidden. macOS goes a different approach. Everything is denied except those activities which are explicitly allowed. Each processes is running in a so called "sandbox". 

But this generates some issues too. For example I tried to run "ntpd" daemon with a config to write statistic files. This is typically done in /etc/ntp.conf with the additional lines 

<pre>
statistics loopstats peerstats sysstats
statsdir /var/ntp/
filegen peerstats file peers type day link enable
filegen loopstats file loops type day link enable
filegen sysstats file sys type day link enable
</pre>

But directory "/var/ntp/" isn't defined in sandbox config. Therefore ntpd cannot write to this directory or files. Each daemon has a sandbox config file. For "ntpd" it is "/usr/share/sandbox/ntpd.sb". We have to modify this file. And of course it's not possible to change this file either as unprivileged user nor as "root". 

The solution is to boot from an external device. For example from a boot-able USB stick or the disk created by "Time Machine". Typically the internal disk is now already mounted. Now it's possible to change files in /usr/share/sandbox.

In my case I add 2 lines: 

<pre>
       (regex "^/private/var/ntp/.*"))
       (regex "^/private/etc/ntp/.*"))
</pre>

The first line is for directory /var/ntp/ the second for all files in /etc/ntp/ where I put the leap seconds file. Now reboot the system from the original disk and ntp can write statistic files. 

Beware the fact that after a system upgrade sandbox files are modified and changed to default. 

This is the complete file "/usr/share/sandbox/ntpd.sb" 

<pre>
;;
;; ntpd - sandbox profile
;; Copyright (c) 2006-2009, 2014 Apple Inc.  All Rights reserved.
;;
;; WARNING: The sandbox rules in this file currently constitute 
;; Apple System Private Interface and are subject to change at any time and
;; without notice. The contents of this file are also auto-generated and not
;; user editable; it may be overwritten at any time.
;;
(version 1)

(deny default)

(allow process-fork)
(allow signal (target same-sandbox))

(allow iokit-open (iokit-user-client-class "RootDomainUserClient")
                  (iokit-user-client-class "AppleSMCClient"))

(allow file-read-data file-read-metadata
       (literal "/private/etc/ntp-restrict.conf")
       (literal "/private/etc/ntp_opendirectory.conf")
       (literal "/private/var/run/resolv.conf")
       (regex "^/private/etc/ntp\\.(conf|keys)$")
       (regex "^/private/etc/(services|hosts)$")
       (regex "^/private/var/run/tmpntp.conf.*"))

(allow file-write* file-read-data file-read-metadata
       (literal "/private/var/run/ntpd.pid")
       (regex "^/private/var/db/ntp\\.drift(\\.TEMP)?$"))
       (regex "^/private/var/ntp/.*"))
       (regex "^/private/etc/ntp/.*"))

(allow network-inbound
       (local udp "*:123"))

(allow network-outbound
       (control-name "com.apple.netsrc")
       (control-name "com.apple.network.statistics")
       (literal "/private/var/run/mDNSResponder")
       (remote udp))

(allow mach-lookup
       (global-name "com.apple.networkd")
       (global-name "com.apple.SystemConfiguration.configd")
       (global-name "com.apple.SystemConfiguration.DNSConfiguration")
       (global-name "com.apple.SystemConfiguration.SCNetworkReachability"))

(allow system-set-time)
(allow system-socket)
(import "bsd.sb")

</pre>



