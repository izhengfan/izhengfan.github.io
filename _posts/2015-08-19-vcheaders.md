---
layout: cnpost
title:  "C/C++: 预编译头文件"
date: 2015-08-19 22:00:51 
categories: cn
tags: C/C++ Windows
---

当一个项目里有多个头文件和源文件时，每个文件开始都重复 include 一堆头文件看起来很傻，有没有优雅高效一点的做法？

Visual C++ 里提供了这种机制，即预编译头文件 Stdafx.h。在 Visual Studio 的 solution 资源管理器中，Stdafx.h 位于头文件文件夹，Stdafx.cpp 位于源文件文件夹。

可以在 Stdafx.h 中 include 标准库文件（比如 iostream）、系统 API 头文件（比如 windows.h）和经常使用但不经常变化的项目头文件（比如 OpenCV 的头文件）。 这样，项目里面的其他文件要 include 这一堆常用头文件时，只需要在一开始 include Stdafx.h 就可以了。这样也方便统一管理 include 列表，一处修改，处处更新。

Stdafx.h一定要被 Stdafx.cpp 源文件 include 。这个源文件会被最先编译。一旦完成，会生成 `.pch `文件，用于之后的编译。因而之后的项目编译速度会被大大加快，尤其当预编译的头文件很多的时候。

如何快速生成预编译文件？实际上，在 Visual Studio 创建 Win32 Console Application 的项目引导中，附加选项里 Precompiled header 打勾就好了。看下图。

<!-- ![appset](/images/noempty.png) -->
[![lXH6nf.png](https://s2.ax1x.com/2020/01/15/lXH6nf.png)](https://imgchr.com/i/lXH6nf)

总结：

<!-- ![msdn](/images/yubianyi.jpg) -->
[![lXHcB8.jpg](https://s2.ax1x.com/2020/01/15/lXHcB8.jpg)](https://imgchr.com/i/lXHcB8)

参考：

[msdn](https://msdn.microsoft.com/zh-cn/library/h552b3ca.aspx)