---
layout: cnmath
title: "OKVIS 笔记：边缘化实现"
date: 2018-03-23 01:00:00
categories: cn
tags: [robotics, VIO, SLAM, OKVIS, Ceres Solver]
---

> OKVIS 系列文章：
> - [OKVIS 笔记：位姿变换及其局部参数类](/2018/01/23/okvis-transformation)
> - [OKVIS 笔记：后端状态量参数结构](/2018/01/23/okvis-ceres-parameter)
> - [OKVIS 笔记：后端架构简述](/2018/03/06/okvis-estimator)
> - [OKVIS 笔记：边缘化原理和策略](/2018/03/22/okvis-marginalization-base)
> - OKVIS 笔记：边缘化实现
> 
> 建议阅读：
> - [Xingyin-Fu：OKVIS 代码框架](https://blog.csdn.net/fuxingyin/article/details/53428523)
> - [Xingyin-Fu：OKVIS 笔记](https://blog.csdn.net/fuxingyin/article/details/53368649)

__目录__

* content
{:toc}

主要涉及源文件：

```
okvis_ceres/include/okvis/ceres/MarginalizationError.hpp
okvis_ceres/src/MarginalizationError.cpp
okvis_ceres/src/Estimator.cpp
```

### 封装结构

OKVIS 的边缘化在 `MarginalizationError` 类中实现，在 `Estimator` 类的 `applyMarginalizationStrategy()` 函数中调用。

先看看 `MarginalizationError` 内部封装数据：

```cpp
/// @name The internal storage of the linearised system.
/// lhs and rhs: (left hand side / right hand side)
/// H_*delta_Chi = _b - H_*Delta_Chi .
/// the lhs Hessian matrix is decomposed as _H = J^T*J = _U*S*_U^T ,
/// the rhs is decomposed as _b - _H*Delta_Chi = -J^T * (-pinv(J^T) * _b + J*Delta_Chi) ,
/// i.e. we have the ceres standard form with weighted Jacobians _J,
/// an identity information matrix, and an error
/// _e = -pinv(J^T) * _b + J*Delta_Chi .
/// _e = _e0 + J*Delta_Chi .
/// @{
Eigen::MatrixXd H_;  ///< lhs - Hessian
Eigen::VectorXd b0_;  ///<  rhs constant part
Eigen::VectorXd e0_;  ///<  _e0 := pinv(J^T) * _b0
Eigen::MatrixXd J_;  ///<  Jacobian such that _J^T * J == _H
Eigen::MatrixXd U_;  ///<  H_ = _U*_S*_U^T lhs Eigen decomposition
Eigen::VectorXd S_;  ///<  singular values
Eigen::VectorXd S_sqrt_;  ///<  cwise sqrt of _S, i.e. _S_sqrt*_S_sqrt=_S; _J=_U^T*_S_sqrt
Eigen::VectorXd S_pinv_;  ///<  pseudo inverse of _S
Eigen::VectorXd S_pinv_sqrt_;  ///<  cwise sqrt of _S_pinv, i.e. pinv(J^T)=_U^T*_S_pinv_sqrt
```

显然，这是一个最小二乘的 H-b 系统。根据注释，大概思路如下。首先，考虑边缘化系统如下：

$$
\rm H \delta \chi = b - H \Delta\chi
$$

其中的 H 矩阵可以分解为 $\rm H=J^TJ=USU^T$，其中的 $\rm S$ 为奇异值矩阵（对半正定实方阵来说，奇异值和特征值相同）。
于是上式可以变换为

$$
\rm J^TJ \delta \chi = -J^T(-J^{T+}b+J \Delta\chi) \tag{1.1}
$$

$$
\rm e:=-J^{T+}b+J \Delta\chi \tag{1.2}
$$

此处 $()^+$ 为伪逆。于是，我们得到一个 Jacobian 为 $\rm J$，信息矩阵为单位阵，error 为 $\rm e$ 的最小二乘标准形式，可由 Ceres 来处理。


### 主要接口和函数

#### MarginalizationError::addResidualBlock()

`MarginalizationError::addResidualBlock()`
把一个 Residual 及其对应的 ParameterBlock 加入现有的 Marginalization 系统（即 `MarginalizationError` 类中封装的 H-b 系统）中，同时将其从 Map 中剔除。
此后 Marginalization 系统会一直维持该 Residual 在加入 Marginalization 系统时的线性点（即 FEJ），直至其被重新线性化或从当前窗口中剔除。

代码中的重要步骤：

获取 Residual 对应的 ParameterBlock：

```cpp
Map::ParameterBlockCollection parameters = mapPtr_->parameters(
      residualBlockId);
```

针对每一个 ParameterBlock，取其 `minimalDimension` 作为 `additionalSize`，即加入 H-b 系统时增加的行（列）数。
当一个 ParameterBlock 不是 landmark 时，将其加入到 H 矩阵和 b 向量中间，即 dense 部分的尾巴上。示意如下：

```
| H_00         H_01 |      |  b_0  |
|       H_new       |      | b_new |
| H_01         H_11 |      |  b_1  |
```

上面的 `H_00` 和 `b_0` 对应原本的 dense 部分（Pose、Extrinsics、SpeedAndBias 等不是 landmark 的状态量）；`H_11` `b_1` 对应原本的 sparse 部分（landmark 状态量）。
`H_new` `b_new` 即对应新加入的 ParameterBlock。

当一个 ParameterBlock 是 landmark 时，直接将其 append 到 H 矩阵和 b 向量的尾巴上：

```
| H_00  H_01        |      |  b_0  |
| H_01  H_11        |      |  b_1  |
|             H_new |      | b_new |
```

上述操作（为新的 ParameterBlock 在 H 和 b 中开辟相应区域）的代码为：

```cpp
if(additionalSize>0) {
  if (!isLandmark) {
    // insert
    // lhs
    Eigen::MatrixXd H01 = H_.topRightCorner(denseSize, origSize - denseSize);
    Eigen::MatrixXd H10 = H_.bottomLeftCorner(origSize - denseSize, denseSize);
    Eigen::MatrixXd H11 = H_.bottomRightCorner(origSize - denseSize, origSize - denseSize);
    // rhs
    Eigen::VectorXd b1 = b0_.tail(origSize - denseSize);

    conservativeResize(H_, origSize + additionalSize, origSize + additionalSize);  // lhs
    conservativeResize(b0_, origSize + additionalSize);  // rhs

    H_.topRightCorner(denseSize, origSize - denseSize) = H01;
    H_.bottomLeftCorner(origSize - denseSize, denseSize) = H10;
    H_.bottomRightCorner(origSize - denseSize, origSize - denseSize) = H11;
    H_.block(0, denseSize, H_.rows(), additionalSize).setZero();
    H_.block(denseSize, 0, additionalSize, H_.rows()).setZero();

    b0_.tail(origSize - denseSize) = b1;
    b0_.segment(denseSize, additionalSize).setZero();
  } else {
    conservativeResize(H_, origSize + additionalSize, origSize + additionalSize);  // lhs
    conservativeResize(b0_, origSize + additionalSize);  // rhs
    // just append
    b0_.tail(additionalSize).setZero();
    H_.bottomRightCorner(H_.rows(), additionalSize).setZero();
    H_.bottomRightCorner(additionalSize, H_.rows()).setZero();
  }
}
```

上面的代码只是 resize，未完成计算。

接下来，先取新加的 ParameterBlock 初始值，计算 Residual 和 Jacobians：

```cpp
errorInterfacePtr->EvaluateWithMinimalJacobians(parametersRaw, residualsRaw,
                                                jacobiansRaw,
                                                jacobiansMinimalRaw);
```

之后还会根据 loss function 去调整刚求的 residual，略过。

最后就是计算新添加的 `H_new` 和 `b_new` 了，其实很简单，就是 $\rm H=J^TJ$ 和 $\rm b=-J^Te$（以下代码有精简）：

```cpp
for (size_t i = 0; i < parameters.size(); ++i) {

  ParameterBlockInfo parameterBlockInfo_i = parameterBlockInfos_.at(
      parameterBlockId2parameterBlockInfoIdx_[parameters[i].first]);

  /// 计算主对角
  H_.block(parameterBlockInfo_i.orderingIdx, parameterBlockInfo_i.orderingIdx,
           parameterBlockInfo_i.minimalDimension,
           parameterBlockInfo_i.minimalDimension) += jacobiansMinimalEigen.at(
      i).transpose().eval() * jacobiansMinimalEigen.at(i);
  b0_.segment(parameterBlockInfo_i.orderingIdx,
              parameterBlockInfo_i.minimalDimension) -= jacobiansMinimalEigen
      .at(i).transpose().eval() * residualsEigen;
  
  /// 计算右上和左下角
  for (size_t j = 0; j < i; ++j) {
    ParameterBlockInfo parameterBlockInfo_j = parameterBlockInfos_.at(
        parameterBlockId2parameterBlockInfoIdx_[parameters[j].first]);

    // upper triangular:
    H_.block(parameterBlockInfo_i.orderingIdx,
             parameterBlockInfo_j.orderingIdx,
             parameterBlockInfo_i.minimalDimension,
             parameterBlockInfo_j.minimalDimension) += jacobiansMinimalEigen
        .at(i).transpose().eval() * jacobiansMinimalEigen.at(j);
    // lower triangular:
    H_.block(parameterBlockInfo_j.orderingIdx,
             parameterBlockInfo_i.orderingIdx,
             parameterBlockInfo_j.minimalDimension,
             parameterBlockInfo_i.minimalDimension) += jacobiansMinimalEigen
        .at(j).transpose().eval() * jacobiansMinimalEigen.at(i);
  }
}
```

#### MarginalizationError::marginalizeOut()

`MarginalizationError::marginalizeOut()`
实现了根据 Schur Complement 来修改 H-b 系统，将给定 ParameterBlock 边缘化。

一开始是一堆整理索引值的工作，不提。

然后是计算部分。先考虑 sparse 部分（landmark）：

- 先对 H 和 b0 （b0 即初始的 b）进行 scale，保持计算尺度正规化
- H 分解出 U 和 V，b0 分解出 b_a 和 b_b；U 和 b_a 对应不被边缘化的部分，V 和 b_b 对应将被边缘化的部分
- 计算 delta\_b 和 delta\_H（代码有精简）：

  ```cpp
  for (size_t i = 0; int(i) < V.cols(); i += sdim) {
    Eigen::MatrixXd M = W.block(0, i, W.rows(), sdim) * V_inv_sqrt;
    Eigen::MatrixXd M1 = W.block(0, i, W.rows(), sdim) * V_inv_sqrt*V_inv_sqrt.transpose(); 
    
    delta_H.at(idx) = delta_H.at(idx - 1) + M * M.transpose();
    delta_b.at(idx) = delta_b.at(idx - 1) + M1 * b_b.segment<sdim>(i);
  }
  ```

  我们知道在舒尔补的计算中，$\rm H_{11}$ 需要减去的量为 $\rm H_{12} H_{22}^{-1} H_{21}$，$\rm b_1$ 需要减去的量为 $\rm H_{12} H_{22}^{-1} b_{2}$。
  在上面的代码中，`W` 对应 $\rm H_{12}$，`V` 对应 $\rm H_{22}$，`V_inv_sqrt` 可以近似地理解为 $\rm H_{22}^{-1/2}$，即满足 `V_inv_sqrt`*`V_inv_sqrt'`=$\rm H_{22}^{-1}$。
  于是，`M` 就是 $\rm H_{12}H_{22}^{-1/2}$，`M1` 就是 $\rm H_{12}H_{22}^{-1}$。

- 完成舒尔补的更新：

  ```cpp
  b0_ -= delta_b.at(idx - 1);
  H_ -= delta_H.at(idx - 1);
  ```
 
  此处 b0 的更新仅对应论文中 eq(26) 的左半步。
  右半步则是在 `EvaluateWithMinimalJacobians()` 函数中完成。

- 最后对 H 和 b0 进行 unscale

以上是对 sparse 部分的计算。对 dense 部分的计算步骤差不多，不提。

处理完一些本地维护的信息的更新之后，将边缘化的状态量从 Map 中移除，完毕：

```cpp
mapPtr_->removeParameterBlock(parameterBlockIdsCopy[i]);
```

#### MarginalizationError::updateErrorComputation()

`MarginalizationError::updateErrorComputation()`
这个函数比较简单，就是计算上面 (1.1) 式中的 $\rm J$, 和 (1.2) 式中的 $\rm e$ 的初始值（$\rm -J^{T+}b_0$）。
这个函数需要在添加状态量（AddResidualBlock）和边缘化状态量（marginalizeOut）之后、执行优化之前调用。

先算 $\rm H=USU^T=J^TJ$ 中的 S, 及相应的 $\rm S^{-1}, S^{1/2}, S^{-1/2}$。
注意，代码中的 `S` 及相关量都是 Vector 而非 Matrix，这是因为它们都是对角阵，取其对角元素即可：

```cpp
S_ = Eigen::VectorXd(
      (saes.eigenvalues().array() > tolerance).select(
          saes.eigenvalues().array(), 0));
S_pinv_ = Eigen::VectorXd(
      (saes.eigenvalues().array() > tolerance).select(
          saes.eigenvalues().array().inverse(), 0));
S_sqrt_ = S_.cwiseSqrt();
S_pinv_sqrt_ = S_pinv_.cwiseSqrt();
```

于是 $\rm J=(US^{1/2})^T$：

```cpp
// assign Jacobian
J_ = (p.asDiagonal() * saes.eigenvectors() * (S_sqrt_.asDiagonal())).transpose();
```

$\rm J^{T+}=S^{-1/2}U^{-1}=S^{-1/2}U^T$：

```cpp
Eigen::MatrixXd J_pinv_T = (S_pinv_sqrt_.asDiagonal())
      * saes.eigenvectors().transpose()  *p_inv.asDiagonal();
```

此处多了一个 `p_inv`，是尺度元素，因为我们为了效果更好的特征值分解，预先对 H 做了 normalize。

最后 $\rm e_0 = -J^{T+}b_0$：

```cpp
e0_ = (-J_pinv_T * b0_);
```

#### MarginalizationError::EvaluateWithMinimalJacobians()

`MarginalizationError::EvaluateWithMinimalJacobians()` 这个函数在 `Evaluate()` 中调用，
`Evaluate()` 是 Ceres 内置虚函数，用于自定义 contraint，
根据现有状态量计算 residual 误差量和 Jacobian，由 Ceres 在求解过程中自动调用。

我们需要在这里指定 MarginalizationError 的 residual 误差量和 Jacobians 计算方法。

Jacobians 我们在 `updateErrorComputation()` 中已经算好，绑定到输出数据即可，不提。

residual 误差量根据上面 (1.2) 式计算。
首先计算估计值更新量 $\Delta \rm \chi$：

```cpp
Eigen::VectorXd Delta_Chi;
computeDeltaChi(parameters, Delta_Chi);
```

最后计算误差量：

```cpp
e = e0_ + J_ * Delta_Chi;
```

对于 MarginalizationError 的 H-b 系统，这相当于计算误差量，对应上面的 (1.2) 式；
对于 VIO 的边缘化操作，这对应论文中 eq(26) 的右半步（左半步已经在 `marginalizeOut()` 中完成）。

另外，上面计算的 Jacobians 是 Minimal Jacobians，即 residual 相对于状态更新量的 Jacobians；
如果还需要计算 residual 相对于状态量本身的 Jacobians，则再乘以相应的 liftJacobians 即可：

```cpp
if (jacobians != NULL) {
  if (jacobians[i] != NULL) {
    /// ....
    J_i = Jmin_i * J_lift; // J_err_group = J_err_algebra * J_algebra_group
  }
}
```

关于这两种 Jacobians 的区别，请参考 [izhengfan/ba_demo_ceres](https://github.com/izhengfan/ba_demo_ceres)。

#### Estimator::applyMarginalizationStrategy()

`Estimator::applyMarginalizationStrategy()`
实现了边缘化策略，哪些 states 和 residuals 要 marg 掉，哪些要保留等等。
整个逻辑比较烦，文字描述已经无力，下面直接贴伪代码。
看起来可能有点乱，实际代码更乱。
一旦真搞起这种工程逻辑，OKVIS 作者也就顾不上 OOP 了 :)

～～～～ 伪代码开始 ～～～～

Declare:

- `removeFrames`: states to remove (frame id, or oldest keyframe id)

- `removeAllButPose`: all in marg window (frame id)

- `allLinearizeFrames`: all in marg window (frame id)

  `removeFrames` contains $\rm x^{c-S}$ or $\rm x^{k_1}$.
  The newest one in `allLinearizeFrames` / `removeAllButPose` is $\rm x^{c-S}$.
  Check the [strategy](/2018/03/22/okvis-marginalization-base/#okvis-边缘化策略).

_For each in_ `removeAllButPose`:

- __Add SpeedAndBias__ states to `parameterBlockToBeMarg`.
- __Add residuals concerning SpeedAndBias__ states to `marginalizationErrorPtr`. They are never ReprojectionError.

_end For_


_For each in_ `removeFrames`:

- __Add Pose__ state to `parameterBlockToBeMarg`.
- Get residuals concerning Pose state. 
  - _If_ a residual is PoseError:

    - it is an initial pose prior, remove it from `mapPtr_`;
    - set `reDoFixation` true

    _Else_:
    - it must be a ReprojectionError. 

    _End If_

- __Add Extrinsics__ state to `parameterBlockToBeMarg`.
  - Residuals concerning Extrinsics state must be ReprojectionError.

- Now let's handle the landmarks

  _For each_ in `landmarksMaps`: 

  - Get related `residuals`

    Declare:
    - `marginalize` = true, indicating if this landmark should be marg
    - `errorTermAdded` = false

    _If_ this landmark is connected to $\rm x^{c-S}$ or newer Frames:

    - set `marginalize` false

    _End If_

    _If_ this landmark has no connected Frame in `removedFrames`:

    - _Continue_

    _End If_


    _If_ `residuals` is empty:

    - Remove this landmark
    - _Continue_

    _End If_

    _If_ `marginalize` is false:

    - Remove residuals whose related Frames are in `removedFrames` 

    _End if_ 

    _If_ `marginalize`:

    - Remove residuals whose related Frames are not in marg window 
    - If this landmark has < 2 connected Frames in the marg window, remove residuals even if their related Frames are in marg window
    - If this landmark has >= 2 connected Frames in the marg window, and if a residual's related Frame is in the marg window, __add the ReprojectionError__ to `marginalizationErrorPtr`, and set `errorTermAdded` true

    _End if_  

    _If_ after residual removal, `residuals` is empty:

    - Remove this landmark
    - _Continue_

    _End If_

    _If_ `marginalize` _And_ `errorTermAdded`:

    - __Add this landmark__ to  `parameterBlockToBeMarg`
    - Remove this landmark from `landmarksMaps`

    _End If_

  _End For_  

- Remove this Frame from `statesMap_`

_End For_

_If_ `parameterBlockToBeMarg` is not empty:

- `marginalizationErrorPtr`->`marginalizeOut()`
- `marginalizationErrorPtr`->`updateErrorComputation()`

_End If_

Add `marginalizationErrorPtr` together with `parameterBlockToBeMarg` as a residual to `mapPtr_`

_If_ `reDoFixation`:

- Re-add a PoseError prior to `mapPtr_`

_End If_

～～～～ 伪代码结束 ～～～～

