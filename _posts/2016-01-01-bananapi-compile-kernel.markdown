---
layout: post
title:  Banana Pi compile kernel and header 
date:   2016-01-01 22:53:00 CET
categories: software 
---

## How-to compile kernel and header for Banana Pi 

Check if the necessary modules are available. If not install it. 

<pre>
build-essential libncurses5-dev u-boot-tools uboot-mkimage
</pre>

As unpriviledged user 

<pre>
git clone https://github.com/Bananian/linux-bananapi.git --depth 1
cd linux-bananapi
make sun7i_defconfig
make menuconfig              # exit & save or make necessary changes 
make -j2 uImage modules      # several hours working
</pre>

As super user "root"

<pre>
make modules_install
mount /dev/mmcblk0p1 /mnt
mv /mnt/uImage /mnt/uImage_old  # save precedent uImage, can be restored 
cp arch/arm/boot/uImage /mnt
sync 
umount /mnt
reboot
</pre>


Later you can verify if updates are available with 

> git pull 

To verify from which repository you cloned the tree run 

> git remote show -n origin

