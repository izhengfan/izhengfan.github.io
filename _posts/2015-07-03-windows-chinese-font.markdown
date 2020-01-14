---
layout: cnpost
title: "英文版 Windows 下 non-Unicode 程序中正常显示汉字"
date: 2015-07-03 09:46:00
categories: cn
tags: Windows type
---

英文版 Windows 下， non-Unicode 程序（例如Sublime Text 2）中，汉字可能显示异常。比如我的 Sublime Text 2 中每个汉字都只有半个字的显示空间，看起来像右半边被砍掉一样。

这是因为英文版 Windows 下 non-Unicode 程序的默认显示字体仍是英文。

所以要解决这个问题，在控制中心的地区和语言设置项里修改就行了。在 Administrative 这个 tab 下，把 Language for non-Unicode programs 从 English 改到 Chinese Simplified 即可。

![lLnxk4.jpg](https://s2.ax1x.com/2020/01/14/lLnxk4.jpg)
<!-- ![settings](/images/fontwindows.jpg) -->

Then enjoy input Chinese in Sublime Text 2!


![lLnj7F.jpg](https://s2.ax1x.com/2020/01/14/lLnj7F.jpg)
<!-- ![settings](/images/st2chi.jpg) -->

