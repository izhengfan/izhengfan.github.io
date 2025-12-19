---
layout: cnmath
title: "OKVIS 笔记：边缘化原理和策略"
date: 2018-03-22 01:00:00
categories: cn
tags: robotics VIO SLAM OKVIS
---

> OKVIS 系列文章：
> - [OKVIS 笔记：位姿变换及其局部参数类](/2018/01/23/okvis-transformation)
> - [OKVIS 笔记：后端状态量参数结构](/2018/01/23/okvis-ceres-parameter)
> - [OKVIS 笔记：后端架构简述](/2018/03/06/okvis-estimator)
> - OKVIS 笔记：边缘化原理和策略
> - [OKVIS 笔记：边缘化实现](/2018/03/23/okvis-marginalization)
> 
> 建议阅读：
> - [Xingyin-Fu：OKVIS 代码框架](https://blog.csdn.net/fuxingyin/article/details/53428523)
> - [Xingyin-Fu：OKVIS 笔记](https://blog.csdn.net/fuxingyin/article/details/53368649)

__目录__

* content
{:toc}

### 边缘化原理

边缘化在很多文献里都提到了。
对于 VIO 系统来说，边缘化的目的是把旧的状态量从状态估计窗口中移除，保证运行效率；
同时，需要把移除的状态量的信息保留下来，作为当下窗口的先验，尽可能避免信息丢失。
边缘化主要利用的技巧是 Schur Complement（舒尔补）。
这套技巧在有些不需要剔除旧变量的场合也会用到，比如 g2o 论文 [1]
中的 _Systems Having Special Structure_ 一节，
通过将高度稀疏的 landmark 部分 marginalize，先求解 dense 部分，回头再求解 landmark 部分，可以大幅提高运算速度。

下面的分析中，式子的编号将与 OKVIS 论文 [4] 中保持一致，但形式上有改动。

图优化中，状态估计总可以转换为求解一个 H-b 方程系统；将当下状态估计窗口下的 H-b 系统记为

$$
\begin{bmatrix}
\rm H_{11} & \rm H_{12} \\ \rm H_{21} & \rm H_{22}
\end{bmatrix}
\begin{bmatrix}
\delta \chi_1 \\ \delta \chi_2
\end{bmatrix} = 
\begin{bmatrix}
\rm b_1 \\ \rm b_2
\end{bmatrix} \tag{21}
$$

其中，$\rm H_{11}, b_1$ 对应保留的状态量， $\rm H_{22}, b_2$ 对应需要边缘化的量。
利用舒尔补，可以将边缘化的量的信息「归并到」保留量的信息中：

$$
\rm H_{11}^{*} = H_{11} - H_{12}H_{22}^{-1}H_{21} \tag{22a}
$$

$$
\rm b_{1}^{*} = b_{1} - H_{12}H_{22}^{-1}b_{2} \tag{22b}
$$

于是，对于移除边缘化状态量之后的系统，只需基于 $\rm H_{11}^{\ast}$ 和 $\rm b_{1}^{\ast}$ 求解即可。

不过，为了保持系统的一致性，需要在优化迭代的过程中维持相关量的最初线性点，使用 First Estimate Jacobian （FEJ）。
关于 FEJ 的原理和深入分析，请参看 [2] [3] 。
简而言之，就是对于同一批状态量，未开始优化迭代时，计算其相关的 Jacobians（最初线性点）；
进入优化迭代后，必须一直使用这些 Jacobians。
这意味着，优化迭代过程中 H 矩阵一直不变。
稍微麻烦一点的是 b 向量。因 $\rm b=-J^T\Omega e$，而误差量 $\rm e$ 受估计值 $\rm \bar{x}$ 影响，
所以每一步迭代之后，$\rm \bar{x}$ 改变，b 也会改变。

OKVIS 使用一阶线性化来估计 b 的更新，避免重算 $\rm e$ 值。
记最初线性点的状态估计值为 $\rm x_{0}$，到当下估计量的偏移值为 $\rm \Delta\chi$，即

$$
\rm \bar{x} = Exp(\Delta \chi)\boxplus x_{0}
$$

这里的 $\boxplus$ 为广义上的两个状态量的叠加，与《[位姿变换及其局部参数类](/2018/01/23/okvis-transformation/)》文中的有所不同。
于是，$\rm \bar{x}$ 时的 b 可以估计为：

$$
\begin{aligned}
\rm b
&= \rm b_{0} + \frac{\partial \rm b}{\partial\Delta\chi}\Big|_{\rm x_0} \Delta\chi\\
&= \rm b_{0} - J^T\Omega \frac{\partial\rm e}{\partial\Delta\chi}\Big|_{\Delta\chi=0} \Delta\chi \\
&= \rm b_{0} - H\Delta\chi \tag{24}
\end{aligned}
$$

即

$$
\begin{bmatrix}
\rm b_{1} \\ \rm b_{2}
\end{bmatrix} =
\begin{bmatrix}
\rm b_{1,0} \\ \rm b_{2,0}
\end{bmatrix} -
\begin{bmatrix}
\rm H_{11} & \rm H_{12}\\
\rm H_{21} & \rm H_{22}
\end{bmatrix} 
\begin{bmatrix}
\Delta\chi_{1} \\ \Delta\chi_{2}
\end{bmatrix} \tag{25}
$$

将其代入式 (22b)，结合式 (22a)，有

$$
\rm b_{1}^{\ast} = b_{1,0}-H_{12}H_{22}^{-1}b_{2,0}-H_{11}^{\ast}\Delta\chi_{1} \tag{26}
$$

定义 $\rm b_{1,0}^{\ast}=b_{1,0}-H_{12}H_{22}^{-1}b_{2,0}$，$\rm b_{1,0}^{\ast}$ 在初次线性化时就可以求出。
于是，边缘化过程实际上包括两个步骤：初次线性化时的式 (22a)，以及 $\rm b_{1,0}^{\ast}$；优化过程中每一步迭代后的式 (26)。
其代码实现，见《[边缘化实现](/2018/03/23/okvis-marginalization/)》一文，
尤其是 `marginalizeOut()` 和 `EvaluateWithMinimalJacobians()` 两个函数。

### OKVIS 边缘化策略

接下来介绍 OKVIS 的边缘化策略，即状态量估计窗口管理策略。
此部份 [5] 中亦有介绍。

首先，OKVIS 会维持 _S_ 个最新 Frame 和 _M_ 个最新 Keyframe，如下图所示。

<!-- ![time-window](https://ftp.bmp.ovh/imgs/2020/01/33b434e77ae364cf.png) -->
![time-window](https://s2.ax1x.com/2020/01/07/lcgrG9.png)

最新的 _S_ 个 Frame 称为 temporal window。
当不断有新的 Frame 加进来，或是普通 Frame，或是 Keyframe，就需要移除掉一些 Frame 或 Keyframe，
保持 _S_ 和 _M_ 的数目不变。

边缘化一开始时，旧的 _M_ 个 Keyframe 都处在 marginalization window 中
（marg window 中最末还有第 _M+1_ 个 Frame，稍后可能被 marg 掉），
先 marg 掉它们的 speed/Bias 项，如下图所示：

<!-- ![okvis-fig7](https://ftp.bmp.ovh/imgs/2020/01/5d19da7981ef99ae.png) -->
![okvis-fig7](https://s2.ax1x.com/2020/01/07/lcgqqf.png)

当新进来一个 Frame 时，记为 $\rm x^c$；
这时，$\rm x^{c-S}$ 帧既是既有 marg window 的最后一帧，又是 temporal window 中的最先一帧；
如果它不是 Keyframe，这意味着 temporal window 中的普通 Frame 多了一帧，
于是 marg 掉 $\rm x^{c-S}$，如下图：

<!-- ![okvis-fig8](https://ftp.bmp.ovh/imgs/2020/01/ff85f50d40cec811.png) -->
![okvis-fig8](https://s2.ax1x.com/2020/01/07/lc2Zi4.png)

这种情况下不会有 landmark 被 marg 掉。
不过，与 $\rm x^{c-S}$ 相关的 observation 需要丢掉，
并且需要在 marginalization 之前丢掉，避免边缘化后的 fill-in 破坏系统中 landmark 部分的稀疏度（参看 [2]）。

如果 $\rm x^{c-S}$ 是 Keyframe，新的 Keyframe 不宜被 marg，于是就 marg 最老的一帧 Keyframe $\rm x^{k_1}$，如下图：

<!-- ![okvis-fig9](https://ftp.bmp.ovh/imgs/2020/01/e231bef31f7896f0.png) -->
![okvis-fig9](https://s2.ax1x.com/2020/01/07/lc28oD.png)

同时，会把被 $\rm x^{k_1}$ 观测到、且不被 $\rm x^{c-S}$ 或 $\rm x^{c-S}$ 之后的 Frame 观测到的 landmark 也 marg 掉。
当然，$\rm x^{k_1}$ 对不会被 marg 的 landmark 的 observation 项也要在 marginalization 前丢掉，避免 fill-in。

当有一帧被 marg 掉，marg window 就往后扩容一帧，保持边缘化窗口大小始终一致，如上面 Fig 8、Fig 9 两图所示。

边缘化策略的代码实现，见《[边缘化实现](/2018/03/23/okvis-marginalization/)》一文，
尤其是 `applyMarginalizationStrategy()` 函数。

### 参考文献

[1] R. Kümmerle, G. Grisetti, H. Strasdat, K. Konolige and W. Burgard, "G2o: A general framework for graph optimization,"
2011 IEEE International Conference on Robotics and Automation, Shanghai, 2011, pp. 3607-3613.
[\[PDF\]](http://ais.informatik.uni-freiburg.de/publications/papers/kuemmerle11icra.pdf)

[2] 白巧克力亦唯心：SLAM 中的 marginalization 和 Schur complement [↗](https://blog.csdn.net/heyijia0327/article/details/52822104)

[3] 知乎：如何理解 SLAM 中的 First-Estimates Jacobian？[↗](https://www.zhihu.com/question/52869487)

[4] S. Leutenegger, S. Lynen, M. Bosse, R. Siegwart, P. Furgale, 
"Keyframe-based visual-inertial odometry using nonlinear optimization", Int. Journal of Robotics Research (IJRR), 2014.

[5] Xingyin-Fu：OKVIS 笔记 [↗](https://blog.csdn.net/fuxingyin/article/details/53368649)
