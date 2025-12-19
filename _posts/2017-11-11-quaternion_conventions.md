---
layout: cnmath
title: "四元数的两种定义形式：Hamilton 和 JPL"
date: 2017-11-11 01:00:00
categories: cn
tags: robotics
---

Quaternion（四元数）是一种常用的三维空间旋转的表示法。四元数由一个实部和三个虚部构成，写如 $\mathbf{q}=q_0 + q_1 i + q_2 j + q_3 k$, 其中 i, j, k 为虚部的三个基，满足 $i^2=j^2=k^2=-1$，有点类似复数中的虚部。通常用单位四元数（即模为 1 的四元数）来表示旋转量。

很长一段时间里，我都以为四元数只有一种定义法，截取维基百科定义 [\[1\]](https://en.wikipedia.org/wiki/Quaternion)：

$$ i^2=j^2=k^2=ijk=-1 $$

在这种定义下，单位四元数转化为旋转矩阵的公式为：

$$ {\bf R}=\begin{bmatrix} 1-2q_2^2-2q_3^2 & 2q_1q_2-2q_0q_3 & 2q_1q_3+2q_0q_2 \\ 2q_1q_2+2q_0q_3 & 1-2q_1^2-2q_3^2 & 2q_2q_3-2q_0q_1 \\ 2q_1q_3-2q_0q_2 & 2q_2q_3+2q_0q_1 & 1-2q_1^2-2q_2^2 \end{bmatrix} $$

不过，在我阅读高翔博士的《视觉 SLAM 十四讲》一书 [7] 时，我发现其中的四元数到旋转矩阵的转换公式和上式不同，是上式的转置。这使我怀疑，是我之前看过的材料有错，还是高博的书有错。在翻阅 Stergios I. Roumeliotis 等撰写的 _Indirect Kalman Filter for 3D Attitude Estimation_ [\[2\]](http://mars.cs.umn.edu/tr/reports/Trawny05b.pdf) 这篇流传甚广的关于四元数的 tutorial 时发现了答案。原来，四元数并不只有一种定义，常见的 convention 有两种，Hamilton 和 JPL。维基百科的 quaternion 条目使用的是 Hamilton，而  Roumeliotis 的 tutorial 则使用了 JPL。Hamilton 和 JPL 两者的根本差别在于三个虚部的相互关系。在 Hamilton convention 里，$ijk=-1$；而 JPL 则取 $ijk = 1$。 这直接导致两者在四元数乘法、与其他旋转表示法相互转换时的公式都有所不同。而高博书中四元数的定义采用的是 Hamilton，但四元数转旋转矩阵使用的是 JPL convention 下的公式——猜想是在写书时两处公式刚好参考了两份使用不同 convention 的文献造成。这个问题高博表示已经发现，相信新一版的书里会更正。

关于书的疑惑解除了，但四元数的表示存在两种 convention 这件事又让我疑惑。四元数是数学家 William Rowan Hamilton 引入的，那么 Hamilton convention 应该是他一开始使用的。那 JPL convention 又是从哪儿冒出来的呢？从我阅读的使用了 JPL convention 的文献（如 Roumeliotis 的 tutorial）来看，这个标准是由著名的 NASA Jet Propulsion Laboratory (JPL) 搞的，参阅文献是一份题为 _Quaternions - Proposed Standard Conventions_ 的 Tech Report。遗憾的是，我搜索不到 JPL 的这份报告。

唯一可以确定的是，两种 convention 的存在，确实给学界造成了一些困惑或争论。

比如，一件吊诡的事情就是，虽然维基百科的 quaternion 主条目使用了 Hamilton 风格的定义，Quaternions and spatial rotation 条目 [\[3\]](https://en.wikipedia.org/wiki/Quaternions_and_spatial_rotation) 也用了 Hamilton，另一个四元数与其他旋转表示法相转换的条目里 [\[4\]](https://en.wikipedia.org/wiki/Conversion_between_quaternions_and_Euler_angles) 却使用了 JPL 的定义。不能不说，混乱和不严谨不统一是维基百科这种大杂烩式的自由百科的缺点。

学者 Nicolas Rotella 在一篇报告里也提到这个问题 [\[5\]](http://www-clmc.usc.edu/~nrotella/IROS2014_linearization.pdf)，对学界往往混用两种 convention 表示很心塞：“unfortunately, the quaternion algebra used in these conventions is often mixed up in the literature”。最后这个作者使用的是 JPL。

Github 上也能看见不少的关于四元数不同 convention 的问题和争论。比如知名的 ETH-ASL 的一个项目下就有这样一个 [issue](https://github.com/ethz-asl/ethzasl_msf/issues/19)：

<!-- ![](/images/quaternion_convention_issue.png) -->
![lXOZ8J.png](https://s2.ax1x.com/2020/01/15/lXOZ8J.png)

看起来他们组支持 Hamilton 的比较多。


Joan Sola 在另一篇报告 _Quaternion kinematics for the error-state KF_ [\[6\]](http://www.iri.upc.edu/people/jsola/JoanSola/objectes/notes/kinematics.pdf) 里也投了 Hamilton 一票。


我个人意见是，四元数存在两种 convention 的现状实在糟糕透顶，如果已经存在 Hamilton，大家都用得好好的，实在无法理解 JPL  提出一个新的标准到底有什么必要。~~嗯，你们 JPL 是很厉害，但另立标准、混淆视听实在是异端，烧死烧死！~~在当前情况下，任何一份可靠的材料，在使用四元数时至少都应明示使用哪种 convention。从现实的角度讲，Eigen 这种通用线性代数运算库用的是 Hamilton，Matlab 使用的是 Hamilton（[MathWorks: Quaternion](https://www.mathworks.com/discovery/quaternion.html)），ROS、Google Ceres Solver 也用的是 Hamilton， 那么大家还是尽量用 Hamilton，免除烦恼。~~愿异端 JPL 尽早进入历史的垃圾堆。~~


__2018-03-19 更新__: 经查证，维基百科条目 \[4\] 只是声称使用了 JPL，实际公式用的还是 Hamilton…… 不知道为什么会出现这种错误。感谢 [Hannes](https://github.com/HannesSommer) 向我指出了这个问题。

此外，Hannes 还写了一篇论文，对此问题进行了更详尽的追根溯源，并给出实践上的合理倡议。
论文放在 [arxiv](https://arxiv.org/abs/1801.07478) 上。
也可以在 [ResearchGate](https://www.researchgate.net/publication/323426570_Why_and_How_to_Avoid_the_Flipped_Quaternion_Multiplication_-_Shorter_and_Less_Formal_Version) 上获取其精简版。


### 参考文献:

[1] Wikipedia: Quaternion [↗](https://en.wikipedia.org/wiki/Quaternion)

[2] Nikolas Trawny and Stergios I. Roumeliotis, _Indirect Kalman Filter for 3D Attitude Estimation_ [↗](http://mars.cs.umn.edu/tr/reports/Trawny05b.pdf)

[3] Wikipedia: Quaternions and spatial rotation
 [↗](https://en.wikipedia.org/wiki/Quaternions_and_spatial_rotation)

[4] Wikipedia: Conversion between quaternions and Euler angles [↗](https://en.wikipedia.org/wiki/Conversion_between_quaternions_and_Euler_angles)

[5] Nicholas Rotella, _Quaternion Review and Conventions_ [↗](http://www-clmc.usc.edu/~nrotella/IROS2014_linearization.pdf)

[6] Joan Sola, _Quaternion kinematics for the error-state KF_ [↗](http://www.iri.upc.edu/people/jsola/JoanSola/objectes/notes/kinematics.pdf)

[7] 高翔，张涛，刘毅，颜沁睿：视觉 SLAM 十四讲，电子工业出版社，2017. [↗](https://www.amazon.cn/dp/B071YF9SLK/)
