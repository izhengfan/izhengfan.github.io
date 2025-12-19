---
layout: post
title: "Fix Synaptics Touchpad in Ubuntu 16.04"
date: 2017-05-11 01:00:00
categories: en
tags: Ubuntu Linux 
---

The touchpad in my Alienware 13 R2 used to work normally in Ubuntu 14.04 and 15.04, but it gets totally unresponsive when upgraded to Ubuntu 16.04. It seems this is a frequently-met problem on laptops that using Synaptics touchpad. After searching for solutions for a while, I found a solution from a [dicussion thread](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1523738) in the Ubuntu bug report center:

> Hi guys
> I found an solution:
>
> Type in the terminal
>
>```
>sudo su
>echo 'blacklist i2c_hid' >> /etc/modprobe.d/blacklist.conf
>depmod -a
>update-initramfs -u
>```
>
> and reboot
>
> Hope did work for you!!

And this truely works for my laptop.
