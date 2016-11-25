---
layout: post
title:  disk free on casper-rw file system
date:   2016-11-25 13:36:00 CET
categories: linux
---

In good old days we had have "live CD's". It was possible to boot a Linux operating system from a CD or DVD without installing it on the builtin hard disk. The disadvantage all changes were lost after a reboot. Now we have USB sticks. USB sticks are read/writable. With the right tool changes are written back to the stick. For example I installed "GNUradio" ISO image with "UNetbootin" on such an USB stick. It works fine on my MacBook. 

All modified data are written to a "casper-rw" file system. Sometimes it is nice to know how much space is free on such a file system. I didn't find a way to figure out free space running the system directly. 

Here a way if there is an additional GNU/Linux system available. I used a Banana-Pi but I am sure each other hardware would do the job too. 

Insert the USB stick to an available running Linux system. You should be able to see an additional device.

<pre>
#  ls -lad /dev/sd*
brw-rw---- 1 root disk 8,  0 Nov 25 11:05 /dev/sda
brw-rw---- 1 root disk 8, 16 Nov 25 11:12 /dev/sdb
brw-rw---- 1 root disk 8, 17 Nov 25 11:13 /dev/sdb1
</pre>

"/dev/sdb" and "/dev/sdb1" are new in my case. Now mount "sdb1" if not already done automatically. 

<pre>
mount  /dev/sdb1 /mnt
</pre>

Looking into this directory you will find a big file. It has the size what I defined in "UNetbootin" for "space used to preserve files across reboots". In my case 2048 MB. 

<pre>
# ls -lda /mnt/ca*
drwxr-xr-x 2 root root      16384 Nov 14 00:53 /mnt/casper
-rwxr-xr-x 1 root root 2147483648 Nov 25 11:19 /mnt/casper-rw
</pre>

Now mount the file system on a different mount point

<pre>
# mkdir /mnt2
# mount -o loop /mnt/casper-rw /mnt2
</pre>

and verify what's free on it: 

<pre>
# df -h /mnt2

Filesystem      Size  Used Avail Use% Mounted on
/dev/loop0      2.0G  318M  1.6G  17% /mnt2
</pre>

Before unplugging the USB stick again do:

<pre>
# umount /mnt2
# umount /mnt
</pre>


