---
layout: cnmath
title: "从零开始的 IMU 状态模型推导"
date: 2016-11-20 01:00:00
categories: cn
tags: robotics VIO
---


__目录__

* content
{:toc}


提示：请使用 Firefox，Chrome，Edge 等较新的浏览器阅读，以获得完整的公式排版。IE 请使用 IE 11。

本文已授权「泡泡机器人 SLAM」微信公众号（paopaorobot_slam）发表。


### 0. 总览

IMU 是移动机器人、移动智能设备上常见的传感器。常见的 IMU 为六轴传感器，配备输出三轴加速度的加速度计和输出三轴角速度的陀螺仪。九轴 IMU 还会配备输出三轴姿态角的磁力计。我们这里只讨论六轴 IMU。

IMU 的状态量通常表示为：

$$ {\bf X}_{IMU} = [ ^I_G \bar{q}^T \quad {\bf b}_g^T \quad ^G{\bf v}_I^T \quad {\bf b}_a^T \quad ^G{\bf p}_I^T] \tag{0.0} $$

这里我们使用和 MSCKF [1] 一样的 notation。用 {I} 表示 IMU 坐标系，{G} 表示参考坐标系。IMU 的姿态由旋转量 $^I_G \bar{q}$ 和平移量 $^G{\bf p}_I$ 表示。更具体来说，前者为将任意向量从 {G} 坐标映射到 {I} 坐标的旋转量，用单位四元数表示；后者为 IMU 在 {G} 下的三维位置。$^G{\bf v}_I$ 表示 IMU 在 {G} 下的平移速度。另外两个量 ${\bf b}_g$ 和 ${\bf b}_a$ 表示陀螺仪（角速度计）和加速度计的 bias。可以注意一下这里除了 bias 之外的状态量的时间维度：平移量表达到速度（p 和 v，对时间的一阶导），因为 IMU 只提供到加速度（对时间的二阶导）的测量；旋转量只表达姿态量（对时间的零阶导），因为 IMU 提供到角速度（对时间的一阶导）。状态量的估计可以由 IMU 测量积分得到。

对于 IMU 状态估计问题，需要提供运动模型、观测（噪声）模型、估计误差模型：

$$ \dot{\bf x} = f({\bf x}) \tag{0.1} $$

$$ {\bf z} = g({\bf x}) + {\bf n} \tag{0.2}$$

$$ \delta {\bf x} = e(\hat{\bf x},{\bf x}) \tag{0.3}$$

这是一个通用模型，我们用 $\bf x$ 表示真实状态量（待估计，不可知），用 ${\bf z}$ 表示观测量，${\bf n}$ 表示观测噪声，$\hat{\bf x}$ 表示当前的状态估计量。这篇小文主要讲 IMU （即 ${\bf x} := {\bf X}_{IMU}$ 时）这三个模型的推导。

### 1. IMU 运动模型

#### 1-1. 前置(1)： 旋转量求导

这部分讲刚体动力学相关的前置知识，熟悉的读者可以跳过。

众所周知，一个刚体在同一个惯性坐标系下进行平移运动，其平移量对时间的一阶导和二阶导即速度和加速度：

$$ \dot{\bf p}={\bf v}, \dot{\bf v} = {\bf a} $$

对于旋转量以及非惯性系参考坐标系，情况稍微复杂些。

首先，如下图（左）所示，考虑一个从原点出发的向量 $\bf r$ 绕单位轴 $\bf u$ 旋转，角速度大小为 $\dot{\theta}$。

<!-- ![](/images/rot_time_der.png)  -->
![](https://ftp.bmp.ovh/imgs/2020/01/60c776f513f7b409.png)

角速度矢量可以表示为 ${\boldsymbol \omega}=\dot{\theta}\bf u$。易得向量 $\bf r$ 末端点 P 的速度矢量，即 $\bf r$的时间一阶导为

$$ \frac{d{\bf r}}{dt} = {\boldsymbol \omega} \times {\bf r} $$

现在考虑上图（右），坐标系 {B} 绕单位轴 $\bf u$ 旋转，如上所述，其三个轴的时间一阶导同样为

$$ \frac{d{\bf i}_B}{dt} = {\boldsymbol \omega} \times {\bf i}_B, \frac{d{\bf j}_B}{dt} = {\boldsymbol \omega} \times {\bf j}_B, \frac{d{\bf k}_B}{dt} = {\boldsymbol \omega} \times {\bf k}_B $$

我们知道，$ \[ {\bf i}_B \quad {\bf j}_B \quad {\bf k}_B \]$ 实际上就是坐标系 {B} 相对于参考坐标系的旋转矩阵 $\bf R$。所以 $\bf R$ 的时间一阶导为

$$ \dot{\bf R} =  [ {\boldsymbol \omega} \times {\bf i}_B \quad {\boldsymbol \omega} \times {\bf j}_B \quad {\boldsymbol \omega} \times {\bf k}_B ] = {\boldsymbol \omega} \times {\bf R} \tag{1.0}$$ 

我们知道上面的叉乘运算可以转化为负对称矩阵的乘法：

$$\dot{\bf R} = {\boldsymbol \omega}^{\land} {\bf R} \tag{1.1}$$ 

其中负对称矩阵为

$$ \quad {\boldsymbol \omega}^{\land}= \begin{bmatrix}0 & -\omega_3 & \omega_2\\ \omega_3 & 0 & -\omega_1 \\ -\omega_2 & \omega_1 & 0\end{bmatrix} $$

注意这里的角速度 ${\boldsymbol \omega}$ 是在参考坐标系下表达的。角速度也经常表达在体坐标系 {B} 下，记为 ${}^B{\boldsymbol \omega} = {\bf R}^T{\boldsymbol \omega}$，即 ${\boldsymbol \omega} = {\bf R}{}^B{\boldsymbol \omega}$，于是 $(1.1)$ 可以写作

$$ \dot{\bf R} = ({\bf R}{}^B{\boldsymbol \omega})^{\land} {\bf R} \tag{1.2}$$ 

这里我们要利用负对称矩阵的一个很好的性质：对任意旋转矩阵 $\bf R$ 和三维向量 $\bf v$，都有 $({\bf R v})^{\land} = {\bf Rv^{\land}R}^T$（参看《[(Rv)^ = Rv^R' 的简单证明](/2017/12/10/Rvhat/)》），于是 $(1.2)$ 可以写成

$$ \dot{\bf R} = {\bf R}({}^B{\boldsymbol \omega})^{\land} \tag{1.3} $$

比较一下 $(1.1)$ 和 $(1.3)$，可以发现一个很有趣的事实，角速度如果表达在参考坐标系下，负对称矩阵写在左边；如果表达在体坐标系下，负对称矩阵写在右边。这点微小的区别，读者在阅读文献时可以特别留意。


#### 1-2. 前置(2)： 四元数

这部分讲四元数如何表示旋转的前置知识，熟悉的读者可以跳过。

用旋转矩阵来表示旋转很直观，但过于冗余，因为旋转只有三个自由度，而旋转矩阵有九个量。表征旋转还可以用欧拉角，但有万向锁问题，而且计算也不方便。旋转向量（即李代数 so(3)）和四元数是更常用的表征方法，在惯性导航中四元数似乎更普遍些。这里采用四元数。

一个四元数由一个实部和三个虚部构成，书写顺序各家不同，这里和 MSCKF [1] 一样，虚部在前实部在后：

$$ {\bf q} = q_1 i + q_2 j + q_3 k + q_4  = [{\boldsymbol v}^T \quad q_4]^T$$

虚部 ${\boldsymbol v}=[q_1 \quad q_2 \quad q_3]^T$。虚部三个基 $i,j,k$ 满足 $i^2=j^2=k^2=ijk=-1$。 四元数仍是一种冗余表达法，为了更紧凑，通常使用使用单元四元数 $\bar{\bf q}$，通过将四元数的模直为 1 得到。

四元数和旋转向量有很直接的转换关系。绕单位轴 $\bf u$ 转了 $\theta$ 角度，用四元数表达为

$$ {\bf q} = [{\bf u}\sin\frac{\theta}{2} \quad \cos\frac{\theta}{2}] \tag{1.4}$$


四元数乘法 $\otimes$ 为类似于多项式乘法的逐项相乘：

$$
\begin{aligned}
 {\bf q}\otimes {\bf p} 
 =  & (q_1 i + q_2 j + q_3 k + q_4)(p_1 i+p_2j+p_3k+p_4) \\
 = & (q_1p_4+q_2p_3-q_3p_2+q_4p_1)i+(-q_1p_3+q_2p_4+q_3p_1+q_4p_2)j+\\
 &(q_1p_2-q_2p_1+q_3p_4+q_4p_3)k + (-q_1p_1-q_2p_2-q_3p_3+q_4p_4)
\end{aligned}
$$

这个计算结果可以表达为多种形式：

$$
\begin{aligned}
 {\bf q}\otimes {\bf p} 
 & = \begin{bmatrix}
 q_4{\bf I}_3+{\boldsymbol v}_q^{\land} & {\boldsymbol v}_q \\
 -{\boldsymbol v}_q^T & q_4 
 \end{bmatrix} \begin{bmatrix}
 {\boldsymbol v}_p \\ p_4 
 \end{bmatrix} \\
 & = \begin{bmatrix}
 p_4{\bf I}_3-{\boldsymbol v}_p^{\land} & {\boldsymbol v}_p \\
 -{\boldsymbol v}_p^T & p_4 
 \end{bmatrix} \begin{bmatrix}
 {\boldsymbol v}_q \\ q_4 
 \end{bmatrix} 
\end{aligned}
$$

四元数乘法和其对应的两个旋转矩阵相乘物理意义是一样的，即 ${\bf R}({\bf q\otimes p})={\bf R}({\bf q}){\bf R}({\bf p})$。四元数对应的旋转矩阵为：

$$ {\bf R}({\bf q}) = (2q_4^2-1){\bf I}_3 +2q_4{\boldsymbol v}^{\land} + 2{\boldsymbol{vv}}^T \tag{1.5}$$

四元数的逆为 ${\bf q}^{-1} = [-{\boldsymbol v}^T \quad q_4]^T$。易得 ${\bf q}\otimes{\bf q}^{-1}=[0\quad 0\quad 0\quad 1]^T:={\bf q}_I$，故 ${\bf q}_I$ 表示旋转量为零。

四元数对时间一阶导为

$$\begin{aligned}
 \dot{\bf q} 
 &= \frac{1}{2} \begin{bmatrix} {\boldsymbol \omega}^{\land} & {\boldsymbol\omega} \\ -{\boldsymbol \omega}^T & 0 \end{bmatrix} {\bf q} := \frac{1}{2}{\boldsymbol \Omega}({\boldsymbol \omega}){\bf q} \tag{1.6} \\
 &= \frac{1}{2} \begin{bmatrix}
 q_4{\bf I}_3-{\boldsymbol v}^{\land} \\
 -{\boldsymbol v}^T 
 \end{bmatrix} {\boldsymbol \omega}
\end{aligned}$$

读者可能注意到了 $(1.6)$ 和 $(1.1)$ 形式上的相似。这里 $\boldsymbol \omega$ 的意义也是一样的。$(1.6)$ 的推导可以参考 [2]，这里不赘述。


#### 1-3. IMU 运动模型

有了前置知识的铺垫之后，我们可以给出 IMU 的运动模型：

$$ \begin{aligned}
{}^I_G \dot{\bar{q}} &= \frac{1}{2}{\boldsymbol \Omega}({\boldsymbol \omega}){}^I_G \bar{q}\\
 \dot{\bf b}_g &= {\bf n}_{wg}\\
{}^G\dot{\bf v}_I &= {}^G{\bf a} \tag{1.7}\\
 \dot{\bf b}_a &= {\bf n}_{wa} \\
 {}^G\dot{\bf p}_I &= {}^G{\bf v}_I 
 \end{aligned}
$$

${}^I_G \dot{\bar{q}}$ 由 $(1.6)$ 直接得到。注意这里角速度 $\boldsymbol \omega$ 是在体坐标系 {I} 下表达的，与 $(1.1)$ 处相反。原因是 ${}^I_G \bar{q}$ 表示的旋转方向与 $(1.1)$ 处的 $\bf R$ 是相反的。其他的四项，速度和加速度都很简单，bias 两项在下面观测模型部分讲。


### 2. IMU 观测和噪声模型

#### 2-1 前置(1)： 科氏加速度

这部分在 1-1 的基础上，讨论参考坐标系不是惯性系的情况，熟悉科氏加速度的读者可以跳过。我们仍利用 1-1 中的图，但这次把绕惯性系 {A} 中固定单位轴 $\bf u$ 旋转的 {B} 作为参考坐标系。考虑下图， 点 P 相对于 {B} 运动，记 $^B{\bf r}$ 分别为 P 在 {B} 下的坐标，$\bf r$ 为 P 的绝对坐标（即 {A} 下坐标）， $\bf R$ 仍为 {B} 相对于 {A} 的旋转矩阵，易知 $ {\bf r}={\bf R}^B{\bf r}$。

<!-- ![](/images/acc_with_rot.png) -->
![](https://ftp.bmp.ovh/imgs/2020/01/b73a03bd1c798e2e.png)

求一阶时间导，并利用公式 $(1.1)$：

$$ {\bf v} = \dot{\bf r} = \dot{\bf R} {}^B{\bf r} + {\bf R}^B\dot{\bf r} = {\boldsymbol \omega}^{\land}{\bf R}{}^B{\bf r}+ {\bf R}^B\dot{\bf r} $$

记 P 在 {B} 下速度为 $^B\bf v$，于是

$$\begin{aligned}
  {\bf v} 
  & = {\boldsymbol \omega}^{\land}{\bf r} + {\bf R}^B{\bf v} \\
  & = {\boldsymbol \omega}\times{\bf r}+ {\bf v}_r
   \tag{2.0}
\end{aligned}$$

请注意，这里用 ${\bf v}_r$ 来表达「相对速度」的概念，准确定义为 P 相对于 {B} 的速度，在惯性系 {A} 下的表达。请分清 ${\bf v}_r$、$\bf v$ 以及 $^B\bf v$ 三者之间的区别和联系。

再对 $(2.0)$ 求时间导：

$$
\begin{aligned}
 {\bf a} = \dot{\bf v} 
 & = \dot{\boldsymbol \omega}\times {\bf r} + {\boldsymbol \omega} \times \dot{\bf r}+ \dot{\bf R} {}^B{\bf v} + {\bf R}{}^B\dot{\bf v} \\
 & = {\boldsymbol \alpha}\times {\bf r}+{\boldsymbol \omega}\times（{\boldsymbol \omega}\times{\bf r}+ {\bf v}_r）+ {\boldsymbol \omega}\times{\bf R}{}^B{\bf v}+{\bf R}{}^B{\bf a} \\
 & = {\boldsymbol \alpha}\times {\bf r}+{\boldsymbol \omega}\times({\boldsymbol \omega}\times{\bf r})+2{\boldsymbol \omega}\times{\bf v}_r+{\bf a}_r \tag{2.1}
\end{aligned} 
$$

我们来逐项分析上面这个式子。第一项中 ${\boldsymbol \alpha}$ 为 {B} 的角加速度，所以第一项的物理意义是 {B} 旋转所造成的 P 的切向加速度。第二项是 {B} 旋转所造成的向心加速度。第四项为 P 相对于 {B} 的加速度，但在惯性系 {A} 下表达——类似于 ${\bf v}_r$，定义相对加速度 ${\bf a}_r$。第三项比较特殊，为 {B} 的旋转运动与 P 相对 {B} 的平移运动耦合产生的加速度，称为「科氏加速度」。可以看到，除了第四项外，另外三项都和 {B} 的旋转有关。


#### 2-2 前置(2)： 惯性导航相关坐标系定义

这部分讲惯性导航中经常出现的几个坐标系的定义 [5]。

**Earth-Centered-Earth-Fixed (ECEF) Frame**：地心地固坐标系 ECEF。以地心为坐标原点，向北为 z 轴，x-y 平面为赤道平面，x 轴指向经纬度 (0,0) 点。ECI 固连在地球上，跟随地球自转，非惯性坐标系。MSCKF 一代 [1] 使用 ECEF 为参考坐标系 {G}。

**Earth-Centered-Inertial (ECI) Frame**：地心惯性坐标系 ECI。以地心为坐标原点，向北为 z 轴，x-y 平面为赤道平面，x 轴指向春分点（vernal equinox point，即每年春分时日心-地心连线与赤道的交点）。ECI 不跟随地球自转，在惯性导航中视为惯性坐标系。MSCKF 二代 [3] 使用 ECI 为参考坐标系 {G}。

**Body Frame**：体坐标系。原点在导航体的质心，固连在导航体上，用来表示导航体的姿态。在本文前置推导部分为 {B}，在 MSCKF 中为 {I}。

#### 2-3 前置(3)： 高斯白噪声与随机游走

这部分讲高斯白噪声和随机游走(random walk)模型，及其离散化。这部分在kalibr 库中的 IMU noise model [4] 有简单的介绍，这里在其基础上添加了离散化的推导，因为离散化中部分内容还是有些令人疑惑的。离散化的推导部分参考自 [5]。

先讲高斯白噪声。一个连续时间的高斯白噪声 $n(t)$，满足以下两个条件

$$ E[n(t)]=0 \\ E[n(t_1)n(t_2)] = \sigma_g^2 \delta(t_1-t_2) $$

其中 $\delta()$ 表示狄拉克函数。可以看出，不同时刻的高斯白噪声相互独立。$\sigma_g^2$ 为方差，值越大，表示噪声程度越大。

将高斯白噪声离散化，可得到：

$$ n_d[k]=\sigma_{gd} w[k] $$

其中

$$ w[k] \sim \mathcal{N}(0,1) \\ 
\sigma_{gd}=\frac{\sigma_g}{\sqrt{\Delta t}}  $$

其中 $\Delta t$ 为采样时间。为什么离散化后分母会多出 $\sqrt{\Delta t}$ 这一项呢？我们假定在一个采样周期内 $n(t)$ 为常数，于是

$$ n_d[k] \triangleq n(t_0+\Delta t)\simeq\frac{1}{\Delta t}\int_{t_0}^{t_0+\Delta t}n(\tau)dt $$

$$\begin{aligned}
E(n_d[k]^2) 
&= E(\frac{1}{\Delta t^2}\int_{t_0}^{t_0+\Delta t}\int_{t_0}^{t_0+\Delta t}n(\tau)n(t)d\tau dt) \\
&= E( \frac{\sigma_g^2}{\Delta t^2}\int_{t_0}^{t_0+\Delta t}\int_{t_0}^{t_0+\Delta t}\delta(t-\tau)d \tau dt)\\
&= E(\frac{\sigma_g^2}{\Delta t})
\end{aligned}$$

所以有　$\sigma_{gd}^2=\frac{\sigma_g^2}{\Delta t}$，即 $\sigma_{gd}=\frac{\sigma_g}{\sqrt{\Delta t}}$。

接下来讨论随机游走模型。准确地讲，随机游走其实是一个离散模型，其连续模型称为维纳过程（Wiener Process）。维纳模型是高斯白噪声的积分：

$$ \dot{b}_g(t)=n(t)=\sigma_{bg}w(t) $$

其中 $w$ 为单位高斯白噪声。将其离散化后得到随机游走模型：

$$ b_d[k] = b_d[k-1]+\sigma_{bgd}w[k] $$

其中

$$ w[k] \sim \mathcal{N}(0,1) \\ 
\sigma_{gd}=\sigma_{bg}\sqrt{\Delta t}  $$

这里多出来的 $\sqrt{\Delta t}$ 又是哪来的呢？仍假定一个采样周期内高斯白噪声为常数，有：

$$ b_d[k] \triangleq b(t_0) + \int_{t_0}^{t_0+\Delta t}n(t)dt $$

$$ \begin{aligned}
E((b_d[k]-b_d[k-1])^2) 
&=E(\int_{t_0}^{t_0+\Delta t}\int_{t_0}^{t_0+\Delta t}n(t)n(\tau)d \tau dt)\\
&= E({\sigma_{bg}^2}\int_{t_0}^{t_0+\Delta t}\int_{t_0}^{t_0+\Delta t}\delta(t-\tau)d \tau dt)\\
&= E(\sigma_{bg}^2\Delta t)
\end{aligned}
$$

所以有　$\sigma_{bgd}^2=\sigma_{bg}^2\Delta t$，即 $\sigma_{bgd}=\sigma_{bg}\sqrt{\Delta t}$。

于是我们得到随机游走模型的完整表达。实际上，观察离散模型的表达式，可以发现它生动阐释了「随机游走」的含义：每一时刻都是上一个采样时刻加上一个高斯白噪声得到的，犹如一个游走的粒子，踏出的下一步永远是随机的。在我们前面给出的 IMU 的运动模型中，bias 就设定为服从随机游走模型。

#### 2-4. IMU 观测模型

根据上述前置知识，现在我们可以给出 IMU 的观测模型。需要注意的是，观测在不同参考坐标系下形式不同。

**以 ECEF 为参考坐标系**：这是 MSCKF 一代 [1] 的做法。因为 ECEF 不是惯性系，需要考虑地球自转，于是加速度模型中将会引入科氏加速度。记 ${\boldsymbol \omega}_G$ 为地球自转角速度， ${}^G{\bf g}$ 为重力加速度， ${\boldsymbol \omega}_m,{\bf a}_m$ 为陀螺仪和加速度计的观测量，观测模型由以下公式给出：

$$ {\boldsymbol \omega}_m = {\boldsymbol \omega}+{\bf R}({}^I_G \bar{q}){\boldsymbol \omega}_G+{\bf b}_g+{\bf n}_g \tag{2.2}$$

$${\bf a}_m = {\bf R}({}^I_G \bar{q})({}^G{\bf a} -{}^G{\bf g} +2{\boldsymbol \omega}_G^{\land}{}^G{\bf v}_I+({\boldsymbol \omega}_G^{\land})^2{}^G{\bf p}_I)+{\bf b}_a+{\bf n}_a \tag{2.3}
$$

观测量都是在体坐标系 {I} 下表达的，所以在参考坐标系 {G} 下表达的量都需要左乘一个旋转矩阵转化到体坐标系。每个观测量的不确定量都用一个随机游走的 bias 和一个高斯白噪声之和来表达。陀螺仪的观测模型是比较易懂的。加速度计的观测模型，我们先将其改写为形如 $(2.1)$ 的形式：

$$ {\bf R}^T({}^I_G \bar{q})({\bf a}_m-{\bf b}_a-{\bf n}_a)=({\boldsymbol \omega}_G^{\land})^2{}^G{\bf p}_I+2{\boldsymbol \omega}_G^{\land}{}^G{\bf v}_I+{}^G{\bf a}-{}^G{\bf g}$$

但这还不够，因为各个量只是在 ECEF 坐标系 {G} 下的表达，而 $(2.1)$ 中的量都是表达在惯性坐标系下的。记 ${\bf R}_G$ 为将 {G} 下坐标映射到惯性坐标系下坐标的旋转矩阵。由于 ECEF 绕固定的 z 轴匀速转动，易得 ${\bf R}_G{\boldsymbol \omega}_G = {\boldsymbol \omega}_G $。于是上式两边左乘 ${\bf R}_G$，可得

$$ {\bf R}_G{\bf R}^T({}^I_G \bar{q})({\bf a}_m-{\bf b}_a-{\bf n}_a)={\boldsymbol \omega}_G\times({\boldsymbol \omega}_G\times{\bf R}_G{}^G{\bf p}_I) + 2{\boldsymbol \omega}_G\times({\bf R}_G{}^G{\bf v}_I)+{\bf R}_G({}^G{\bf a}-{}^G{\bf g})$$

这里我们还利用了 ${\bf R}(\bf a\times b)={\bf Ra}\times{\bf Rb}$ 的性质。上式对应到 $(2.1)$ 中各项，左边为绝对加速度 $\bf a$；因为地球自转是匀速的，故切向加速度项 ${\boldsymbol \alpha} \times\bf r$ 为零。其余各项，依次为向心加速度项 ${\boldsymbol \omega}\times({\boldsymbol \omega}\times{\bf r})$，科氏加速度项 $2{\boldsymbol \omega}\times{\bf v}_r$，以及相对加速度项 ${\bf a}_r$。

**以 ECI 为参考坐标系**：这是 MSCKF 二代 [3] 的做法。由于 ECI 为惯性系，不需要考虑地球自转，于是观测模型简单很多：

$$ {\boldsymbol \omega}_m = {\boldsymbol \omega}+{\bf b}_g+{\bf n}_g \tag{2.4} $$

$$
{\bf a}_m = {\bf R}({}^I_G \bar{q})({}^G{\bf a} -{}^G{\bf g} )+{\bf b}_a+{\bf n}_a \tag{2.5}
$$

因为比较简单，就不多做解释了。从文献上看，现在移动机器人领域 ECI 用得更多些。

至此，我们推导完了 IMU 的观测模型。


### 3. IMU 状态估计误差模型

#### 3-1. 前置：四元数误差小量

旋转量是非线性的，不宜像线性量那样使用 $\tilde{ \bf x}= x-\hat{x}$ 来定义误差量。这里我们使用四元数误差小量来定义误差量。根据 $(1.4)$，四元数可以用旋转向量经简单的转换得到。假定绕单位轴 $\bf u$ 旋转了一个角度小量 $\delta \theta$，用四元数表达为：

$$\begin{aligned} 
\delta {\bf q} 
&= \begin{bmatrix} {\bf u}\sin{\frac{\delta \theta}{2}} \\ \cos{\frac{\delta \theta}{2}} \end{bmatrix}\\
&\simeq \begin{bmatrix} {\bf u}\frac{\delta \theta}{2} \\ 1 \end{bmatrix} \triangleq 
\begin{bmatrix} \frac{\boldsymbol{\delta \theta}}{2} \\ 1 \end{bmatrix}
\end{aligned} $$

于是，可以用 $\delta {\bf q} $ 来表示旋转的真实值和估计值之间的误差，具体关系为

$$ {\bf q} =  \delta {\bf q} \otimes\hat{\bf q}$$

直接使用 $\boldsymbol{\delta \theta}$，可以实现参数最小化，适用于优化问题中的目标函数。

#### 3-2. IMU 状态估计误差模型

我们直接给出和 MSCKF 一样的 IMU 状态估计误差模型：

$$ \tilde{\bf X}_{IMU} = [ \boldsymbol{\delta \theta}_I^T \quad \tilde{\bf b}_g^T \quad ^G\tilde{\bf v}_I^T \quad \tilde{\bf b}_a^T \quad ^G\tilde{\bf p}_I^T] \tag{3.0}$$ 

其中旋转量按照四元数误差小量给出，其余直接由真实值和估计值相减得到。

### 4. 小结

本文从基础出发推导了 IMU 的运动模型$(1.7)$、观测和噪声模型$(2.2)-(2.5)$、估计误差模型$(3.0)$，适用于用 IMU 来做状态估计的场合。至于以上这些模型如何再经过线性化、离散化等处理进入具体状态估计问题的框架中，这里不做赘述，留待读者阅读和探索。 


### 参考文献

[1] Mourikis, Anastasios I., and Stergios I. Roumeliotis. "A multi-state constraint Kalman filter for vision-aided inertial navigation." Proceedings 2007 IEEE International Conference on Robotics and Automation. IEEE, 2007.

[2] Trawny, Nikolas, and Stergios I. Roumeliotis. "Indirect Kalman filter for 3D attitude estimation." University of Minnesota, Dept. of Comp. Sci. & Eng., Tech. Rep 2 (2005): 2005.

[3] Li, Mingyang. "Visual-inertial odometry on resource-constrained systems." (2014).

[4] IMU noise model [https://github.com/ethz-asl/kalibr/wiki/IMU-Noise-Model](https://github.com/ethz-asl/kalibr/wiki/IMU-Noise-Model)

[5] Crassidis, John L., and John L. Junkins. Optimal estimation of dynamic systems. CRC press, 2011.
