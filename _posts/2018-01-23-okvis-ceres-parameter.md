---
layout: cnmath
title: "OKVIS 笔记：后端状态量参数结构"
date: 2018-01-23 02:00:00
categories: cn
tags: [robotics, VIO, SLAM, OKVIS, Ceres Solver]
---

> OKVIS 系列文章：
> - [OKVIS 笔记：位姿变换及其局部参数类](/2018/01/23/okvis-transformation)
> - OKVIS 笔记：后端状态量参数结构
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

主要涉及源文件：

```
okvis_ceres/include/okvis/ceres/ParameterBlock.hpp
okvis_ceres/include/okvis/ceres/ParameterBlockSized.hpp
okvis_ceres/include/okvis/ceres/PoseParameterBlock.hpp
okvis_ceres/include/okvis/ceres/HomogeneousPointParameterBlock.hpp
okvis_ceres/include/okvis/ceres/SpeedAndBiasParameterBlock.hpp
```

### ParameterBlock

OKVIS 状态量的基础抽象类是 `ParameterBlock`，结构如下：

```
▼ okvis::ceres : namespace
  ▼ ParameterBlock : class

     [prototypes]
    +dimension() const
    +liftJacobian(const double* x0, double* jacobian) const
    +minimalDimension() const
    +minus(const double* x0, const double* x0_plus_Delta, double* Delta_Chi) const
    +parameters()
    +parameters() const
    +plus(const double* x0, const double* Delta_Chi, double* x0_plus_Delta) const
    +plusJacobian(const double* x0, double* jacobian) const
    +setParameters(const double* parameters)
    +typeInfo() const

     [functions]
    +ParameterBlock()
    +~ParameterBlock()
    +fixed() const
    +id() const
    +localParameterizationPtr() const
    +setFixed(bool fixed)
    +setId(uint64_t id)
    +setLocalParameterizationPtr( const ::ceres::LocalParameterization* localParameterizationPtr)

     [members]
    #fixed_
    #id_
    #localParameterizationPtr_
```

可以看到，数据方面，只有三个成员（`[members]`）：是否 fix、id、局部参数类指针。
该基础类不存储具体的状态估计数据。

接口设计上，有很多虚函数（`[prototypes]`）需要被继承，
比如增量函数、增量 Jacobian、参数维度、局部最小参数维度等等，
和 Ceres Solver 的接口设计相仿。

### ParameterBlockSized

`ParameterBlockSized` 继承自 `ParameterBlock`：

```cpp
template<int Dim, int MinDim, class T>
class ParameterBlockSized : public okvis::ceres::ParameterBlock
{
  /// ...

  protected:
  double parameters_[Dimension];
}
```

顾名思义，该类可用于表示参数 size（即维度）固定的状态量
（我们所要处理的大部分状态量，如 pose、landmark、bias 等，皆是如此），
故其 members 中包含参数维度、局部最小参数维度，以及长度为参数维度的状态参数。
也就是说，`ParameterBlockSized` 就是盛放状态量参数的类。
其具体结构如下：

```
▼ okvis::ceres : namespace
  ▼ ParameterBlockSized : class

     [prototypes]
    +estimate() const
    +setEstimate(const T& estimate)

     [functions]
    +ParameterBlockSized()
    +~ParameterBlockSized()
    +dimension() const
    +minimalDimension() const
    +parameters()
    +parameters() const
    +setParameters(const double* parameters)

     [members]
    +Dimension
    +MinimalDimension
    #parameters_
```

有两个 prototypes 需要被继承：获得当前估计，和设定当前估计；
加上参数维度 `Dim`、局部最小参数维度 `MinDim` 和估计量的类 `T` 作为模板，
可被不同子类的继承，为不同状态量进行自定义。

至此，我们就可以定义系统里各种状态量了。

### PoseParameterBlock

SE(3) 位姿状态量由 `PoseParameterBlock` 定义，继承自 `ParameterBlockSized`:

```cpp
class PoseParameterBlock : 
  public ParameterBlockSized<7,6,okvis::kinematics::Transformation>
{
/// ...
private:
  okvis::Time timestamp_; 
}
```

估计量用的类是 `okvis::kinematics::Transformation`；
members 多了一个时间戳。
具体结构不再罗列了，把 `ParameterBlock` `ParameterBlockSized` 两个基类中的 prototypes 都给继承了就是。
值得一提的是，其内部各种参数变换是基于 `PoseLocalParameterization` 类的，
请参看《[位姿变换及其局部参数类](/2018/01/23/okvis-transformation/)》。

### HomogeneousPointParameterBlock

齐次 3D 点座标由 `HomogeneousPointParameterBlock` 定义，继承自 `ParameterBlockSized`:

```cpp
class HomogeneousPointParameterBlock :
  public ParameterBlockSized<4, 3, Eigen::Vector4d>
{
  /// ...
private:
  bool initialized_;  ///< Whether or not the 3d position is considered initialised.
}
```

估计量用的是四维向量；
members 多了一个是否已被初始化的 flag。
内部参数变换基于 `HomogeneousPointLocalParameterization` 类，都比较简单，不赘述。

### SpeedAndBiasParameterBlock

速度和 bias 归并为一个状态量，由 `SpeedAndBiasParameterBlock` 定义，继承自 `ParameterBlockSized`:

```cpp
typedef Eigen::Matrix<double, 9, 1> SpeedAndBias;

class SpeedAndBiasParameterBlock :
  public ParameterBlockSized<9, 9, SpeedAndBias> 
{
  /// ...
private:
  okvis::Time timestamp_; 
}
```

估计量用的是九维向量（速度项 3，两个 bias 各 3）；
members 多了一个时间戳。
这个状态量不是非欧流形，内部参数变换都是线性变换。

### 小结

第一层：`ParameterBlock` 为不包括状态估计数据的基础抽象类；

第二层：`ParameterBlockSized` 为包括状态估计数据的固定维度的基础类；

第三层：`PoseParameterBlock` `HomogeneousPointParameterBlock` `SpeedAndBiasParameterBlock`
为针对不同种类状态量的具体实现。
