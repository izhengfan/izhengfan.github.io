---
layout: cnmath
title: "OKVIS 笔记：位姿变换及其局部参数类"
date: 2018-01-23 01:00:00
categories: cn
tags: [robotics, VIO, SLAM, OKVIS, Ceres Solver]
---

> OKVIS 系列文章：
> - OKVIS 笔记：位姿变换及其局部参数类
> - [OKVIS 笔记：后端状态量参数结构](/2018/01/23/okvis-ceres-parameter)
> - [OKVIS 笔记：后端架构简述](/2018/03/06/okvis-estimator)
> - [OKVIS 笔记：边缘化原理和策略](/2018/03/22/okvis-marginalization-base)
> - [OKVIS 笔记：边缘化实现](/2018/03/23/okvis-marginalization)
> 
> 建议阅读：
> - [Xingyin-Fu：OKVIS 代码框架](https://blog.csdn.net/fuxingyin/article/details/53428523)
> - [Xingyin-Fu：OKVIS 笔记](https://blog.csdn.net/fuxingyin/article/details/53368649)

__目录__

* content
{:toc}

### 位姿变换类 

主要涉及源文件：

```
okvis_kinematics/include/okvis/kinematics/Transformation.hpp
```

OKVIS 的 `Trawnsformation` 类实现了 SE3 变换。

OKVIS 的变换类虽然也采用了类似预积分论文中用最小表示的局部小量来做更新的 local parameterization（预积分论文称之为 lift-solve-retract），但不同于预积分论文的 $\rm (R\,Exp(\delta\alpha), p+R\delta p)$，更不同于常规的 Exp(se3)，OKVIS 的更新策略是将旋转量和平移量完全解耦，对旋转量使用左扰动，对平移量则直接加法：

$$
\rm x = \bar{x}\boxplus \Delta x: \, q \leftarrow \delta q \otimes \bar{q}, \, p \leftarrow \delta p+\bar{p}.
$$

其中四元数小量与李代数小量的关系可以近似为 $\rm \delta q \approx [\frac{1}{2}\delta \alpha;1]$。其实现代码在 `Transformation::oplus()` 函数中（省略琐碎细节，下同）：

```cpp
// apply small update:
template<typename Derived_delta>
inline bool Transformation::oplus(
    const Eigen::MatrixBase<Derived_delta> & delta) {
  r_ += delta.template head<3>();
  Eigen::Vector4d dq;
  double halfnorm = 0.5 * delta.template tail<3>().norm();
  dq.template head<3>() = sinc(halfnorm) * 0.5 * delta.template tail<3>();
  dq[3] = cos(halfnorm);
  q_ = (Eigen::Quaterniond(dq) * q_);
  return true;
}
```

旋转和平移完全解耦的好处是计算雅克比矩阵非常方便。比如，计算 oplusJacobian：

$$
\begin{aligned}
&\rm \frac{\partial x}{\partial \Delta x}:\\
&\rm \frac{\partial q}{\partial \delta \alpha} = \frac{\partial Q_{\bar{q}}\,\delta q}{\partial \delta \alpha} = \frac{1}{2}Q_{\bar{q}}I_{4\times 3}\\
&\rm \frac{\partial p}{\partial \delta p} = I_{3}\\
&\rm \frac{\partial q}{\partial \delta p} = \frac{\partial p}{\partial \delta \alpha} = 0
\end{aligned}
$$


其中使用到四元数的 Q 矩阵，其定义是表示对于任意四元素 q 和 p，都有 $\rm q\otimes p=Q_{p}\,q$。OKVIS 代码中 `okvis::kinematics::oplus(Quaterniond)` 函数实现了 Q 矩阵。oplusJacobian 实现代码在 `Transformation::oplusJacobian()` 函数中：

```cpp
template<typename Derived_jacobian>
inline bool Transformation::oplusJacobian(
    const Eigen::MatrixBase<Derived_jacobian> & jacobian) const 
{
  Eigen::Matrix<double, 4, 3> S = Eigen::Matrix<double, 4, 3>::Zero();
  const_cast<Eigen::MatrixBase<Derived_jacobian>&>(jacobian).setZero();
  const_cast<Eigen::MatrixBase<Derived_jacobian>&>(jacobian)
      .template topLeftCorner<3, 3>().setIdentity();
  S(0, 0) = 0.5;
  S(1, 1) = 0.5;
  S(2, 2) = 0.5;
  const_cast<Eigen::MatrixBase<Derived_jacobian>&>(jacobian)
      .template bottomRightCorner<4, 3>() = okvis::kinematics::oplus(q_) * S;
  return true;
}
```


计算 liftJacobian 也类似：

$$
\begin{aligned}
&\rm \frac{\partial \Delta x}{\partial x}:\\
&\rm \frac{\partial \delta \alpha}{\partial q}=\frac{\partial\delta\alpha}{\partial\delta q} \frac{\partial q\otimes\bar{q}^{-1}}{\partial q} = 2I_{3\times 4}\,Q_{\bar{q}^{-1}}\\
&\rm \frac{\partial \delta p}{\partial p} = I_3\\
&\rm \frac{\partial \delta \alpha}{\partial p} = \frac{\partial \delta p}{\partial q} = 0
\end{aligned}
$$

其实现代码在 `Transformation::liftJacobian()` 函数中：

```cpp
template <typename Derived_jacobian>
inline bool Transformation::liftJacobian(const Eigen::MatrixBase<Derived_jacobian> & jacobian) const
{
  const_cast<Eigen::MatrixBase<Derived_jacobian>&>(jacobian).setZero();
  const_cast<Eigen::MatrixBase<Derived_jacobian>&>(jacobian).template topLeftCorner<3,3>()
      = Eigen::Matrix3d::Identity();
  const_cast<Eigen::MatrixBase<Derived_jacobian>&>(jacobian).template bottomRightCorner<3,4>()
      = 2*okvis::kinematics::oplus(q_.inverse()).template topLeftCorner<3,4>();
  return true;
}
```

### 位姿局部参数类

主要涉及源文件：

```
okvis_ceres/include/okvis/ceres/PoseLocalParameterization.hpp
okvis_ceres/src/PoseLocalParameterization.cpp
```

基于 `Transformation` 类，OKVIS 实现了用于 Ceres 优化的局部位姿参数类 `PoseLocalParameterization`。群的参数排列顺序为：
```
x y z qx qy qz qw
```

增量的参数排列顺序为
```
dx dy dz dalphax dalphay dalphaz
```

其增量函数 `Plus()`，增量雅克比函数 `ComputeJacobian()`，lift 雅克比函数 `ComputeLiftJacobian()`，都是基于上述 `Transformation` 类的 `oplus()` `oplusJacobian()` `liftJacobian()` 或与其一致，此处不再重复。

同时，针对不同场景，OKVIS 还实现了 PoseLocalParameterization 的几个衍生类：PoseLocalParameterization3d、PoseLocalParameterization4d 和 PoseLocalParameterization2d。其中，\*3d 类仅对平移量扰动；\*4d 类仅对平移量和 yaw 角进行扰动；\*2d 类仅扰动 roll 角和 pitch 角。
