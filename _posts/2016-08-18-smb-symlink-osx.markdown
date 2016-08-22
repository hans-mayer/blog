---
layout: post
title:  SMB file systems with symbolic link and OS X
date:   2016-08-18 17:28:00 CET
categories: IT 
---

Environment: An EMC filer VNX5400 and an OS X client <br>
Protocol: Server Message Block (SMB) <br>
The issue: The client can see empty directories only

The VNX filer has the ability that file systems can be access vie SMB protocol as well via NFS. It's a so called "mixed type" volume. Accessing the NFS share from a Linux/UNIX/Solaris client gives the possibility to create symbolic links. Symbolic links can point to directories or to files. Accessing such a sym-link to a directory from OS X with SMB protocol results in a way that the directory is shown but it's empty. As default OS X is using SMB version 3. A solution is to use version 1. Obviously this version doesn't care about sym-links or doesn't "know" about it. 

To achieve this result one has to create a file _~/Library/Preferences/nsmb.conf_ with the following content:

<pre>
[default]
smb_neg=smb1_only
</pre>

Now you can mount the share in the usual way with "Finder" and "Connect to Server..." : 

> smb://server/share 

A very similar situation exists also for Windows based operating systems. One can query the settings with <br>
  fsutil behavior query SymlinkEvaluation <br> 
and all 4 settings should be enabled. 

