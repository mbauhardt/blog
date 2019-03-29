---
title: "Change Size Of Partition On SDcard To Get More Space For My Raspberry Pi"
date: 2013-03-24
tags: ["raspberrypi", "linux"]
---

I'm using my [raspberry pi](http://www.raspberrypi.org/) as a router on my private network.
Yesterday i'm getting the _no space left on device_ error message. 
I recognized that only the half size of the sd card was used. So i had to resize the root partition.

These steps you have todo to resize your /root partition. I did this on my arch linux arm installation.

* Open fdisk `fdisk /dev/mmcblk0`
* Hit option `p` to get a list of partitions. You should see two partitions. The first is the /boot partition the second is the linux partition
* Delete the second partition with option `d`
* Create a new partition wit option `n` and press enter to use the default answer on all following questions
* Exit *fdisk* with `w` to write the partition to disk
* reboot your raspberry with `reboot`
* Resize filesystem with `resize2fs -p /dev/mmcblk0p2`

Thats it!

Here are the steps i did as screenshots.

![fdisk](/screenshot_1.png)

![fdisk](/screenshot_2.png)

![fdisk](/screenshot_3.png)

![fdisk](/screenshot_4.png)

![fdisk](/screenshot_5.png)

![fdisk](/screenshot_6.png)
