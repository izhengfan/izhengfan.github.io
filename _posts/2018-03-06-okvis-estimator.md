---
layout: cnmath
title: "OKVIS 笔记：后端架构简述"
date: 2018-03-06 01:00:00
categories: cn
tags: [robotics, VIO, SLAM, OKVIS, Ceres Solver]
---

> OKVIS 系列文章：
> - [OKVIS 笔记：位姿变换及其局部参数类](/2018/01/23/okvis-transformation)
> - [OKVIS 笔记：后端状态量参数结构](/2018/01/23/okvis-ceres-parameter)
> - OKVIS 笔记：后端架构简述
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
okvis_ceres/include/okvis/Estimator.hpp
okvis_ceres/include/okvis/implementation/Estimator.hpp
okvis_ceres/src/Estimator.cpp
okvis_ceres/include/okvis/ceres/Map.hpp
okvis_ceres/src/Map.cpp
```

### 封装结构

OKVIS 系统顶层 `ThreadedKFVio` 这个类里封装了一个 `Estimator` 类，这就是 OKVIS 后端的最顶层。`Estimator` 里封装了一个 `Map` 类的 `shared_ptr`，这是优化问题的顶层；同时存储了一些窗口数据，用于管理 sliding-window。`Map` 类封装了一个 `ceres::Problem` 及相关的操作接口。由外及里的结构就是：

```
ThreadedKFVio
|
|__ Estimator
    |
    |__ Sliding-window states
    |__ Map
        |
        |__ ceres::Problem
```

- `ThreadedKFVio` 会在初始化时构建 `Estimator`，在 `matchingLoop()` 里往 `Estimator` 添加后端所需数据；在 `optimizationLoop()` 里调用 `Estimator` 的接口进行优化和边缘化。

- `Estimator` 实现后端逻辑流，将前面塞进来的数据按后端逻辑流添加给 `Map`；调用 `Map` 的优化接口进行优化；实现 sliding window 和状态边缘化。

- `Map` 实现了一些基于 `ceres::Problem` 的接口函数，包括添加和删除 ParameterBlock、添加和删除 Residual、Parameterization 管理、Id 关联管理等，将 `Estimator` 添加过来的数据喂给 `ceres::Problem`；调用 `ceres::Solve()` 求解优化问题。

### 主要接口和函数

#### Estimator::statesMap_

`Estimator::statesMap_`：
当前窗口下的状态维护信息，包括各种 id、时间戳、各种布尔值等等，不包括状态的实际数据。实际数据都存在 `Map` 里。实际操作中，通过 `Estimator::statesMap_` 维护状态身份信息，通过 `Map` 维护实际状态数据。`Estimator::statesMap_` 结构稍微有点复杂（吐个槽，感觉 OKVIS 的多重封装看起来有点累，时不时需要跳转查看接口定义）：

```
statesMap_ [ std::map<uint64_t, States> ]
|
|__ [ uint64_t ] //等于对应 Frame 的 id
|__ [ States ]
    |
    |__ timestamp
    |__ isKeyframe [ bool ]
    |__ id [ uint_64_t ] //等于对应 Frame 的 id
    |__ global [ GlobalStatesContainer i.e. array<StateInfo, 6> ]
    |   |
    |   |__ StateInfo
    |       |
    |       |__ id [ uint64_t ] //如果对应 GlobalStates::T_WS，则等于对应 Frame 的 id
    |       |__ exits [ bool ]
    |       |__ isRequired [ bool ]
    |
    |__ sensors [ AllSensorStatesContainer i.e. array<vector<SpecificSensorStatesContainer>, 7> ]
        |
        |__ [ SpecificSensorStatesContainer i.e. vector<StateInfo> ]
        |__ StateInfo
            |
            |__ id [ uint64_t ]
            |__ exits [ bool ]
            |__ isRequired [ bool ]
```

注意到 `States.global` 和 `States.sensors` 分别是长度为 6 和 7 的定长数组，这是对应于：

```cpp
/// \brief GlobalStates The global states enumerated
enum GlobalStates
{
    T_WS = 0, ///< Pose.
    MagneticZBias = 1, ///< Magnetometer z-bias, currently unused
    Qff = 2, ///< QFF (pressure at sea level), currently unused
    T_GW = 3, ///< Alignment of global frame, currently unused
};

/// \brief SensorStates The sensor-internal states enumerated
enum SensorStates
{
    Camera = 0, ///< Camera
    Imu = 1, ///< IMU
    Position = 2, ///< Position, currently unused
    Gps = 3, ///< GPS, currently unused
    Magnetometer = 4, ///< Magnetometer, currently unused
    StaticPressure = 5, ///< Static pressure, currently unused
    DynamicPressure = 6 ///< Dynamic pressure, currently unused
};
```

可以看到 `States.global` 的长度其实只需要是 4，不知为什么设为 6。同时，现有框架下 `GlobalStates` 里有三个索引没用到，`SensorStates` 里有 5 个索引没用到。

注意一下 `States.sensors` 的内部结构，是一个 `array<vector<vector<StateInfo> >, 7>`，最外层的 `array` 上一段已经说了，对应不同种类的传感器；第二层的 `vector` 是给多目相机用的，如果双目则长度为 2，如果单目则长度为 1；最里层的 `vector` 也是给相机用的，可以用来放外参和内参：

```cpp
/// \brief CameraSensorStates The camera-internal states enumerated
enum CameraSensorStates
{
    T_SCi = 0, ///< Extrinsics as T_SC
    Intrinsics = 1, ///< Intrinsics
};
```

而这里面的两层 `vector` 对于 IMU 来说是无意义的。首先，系统只支持一个 IMU，故中间层的 `vector` 无用。其次，对于 IMU，其对应的量也只有一个（speedAndBias），索引如下：

```cpp
/// \brief ImuSensorStates The IMU-internal states enumerated
/// \warning This is slightly inconsistent, since the velocity should be global.
enum ImuSensorStates
{
    SpeedAndBias = 0 ///< Speed and biases as v in S-frame, then b_g and b_a
};
```

故最底层的 `vector` 也无用。所以，这种封装虽然很 OOP，但对于实际系统来说，如果没有扩展传感器种类的计划，可以考虑减少层级结构，避免不必要的资源开销，减少心智负担。

#### Estimator::addStates()

`Estimator::addStates()`
实现了添加一个新的 Frame （主要是 pose 和 speedAndBias）到后端状态量的逻辑流。这个函数有三个输入参数：

```cpp
okvis::MultiFramePtr multiFrame,
const okvis::ImuMeasurementDeque & imuMeasurements,
bool asKeyframe
```

这里的 multiFrame 并不是来自不同时间点的多个 Frame， 而是一个 multi-camera frame。如果是双目，就是两个 Camera Frame；如果是单目，就是单个 Frame。

首先声明主要处理对象：

```cpp
okvis::kinematics::Transformation T_WS;
okvis::SpeedAndBias speedAndBias;
```

根据存储在`statesMap_` 中的状态维护信息、 `Map` 中的前次状态信息。如果 `statesMap_` 尚为空，说明现在是第一帧，则利用 IMU measurements 进行初始化：

```cpp
initPoseFromImu(imuMeasurements, T_WS);
```

如果 `statesMap_` 不为空，利用 IMU measurements 进行更新：

```cpp
// propagate pose and speedAndBias
int numUsedImuMeasurements = ceres::ImuError::propagation(
    imuMeasurements, imuParametersVec_.at(0), T_WS, speedAndBias,
    statesMap_.rbegin()->second.timestamp, multiFrame->timestamp());
```

更新后的 `T_WS` 创建为新的 ParameterBlock 加进 `Map`：

```cpp
mapPtr_->addParameterBlock(poseParameterBlock,ceres::Map::Pose6d)
```

把新的状态信息添加到 `statesMap_` 里：

```cpp
// create a states object:
States states(asKeyframe, multiFrame->id(), multiFrame->timestamp());
states.global.at(GlobalStates::T_WS).exists = true;
states.global.at(GlobalStates::T_WS).id = states.id;

statesMap_.insert(std::pair<uint64_t, States>(states.id, states));
```

更新 sensor states （包括相机外参和 IMU），并添加相应的 ParameterBlock 给 `Map`（注意代码注释里添加的文字）：

```cpp
// initialize new sensor states
// cameras:
for (size_t i = 0; i <  extrinsicsEstimationParametersVec_.size(); ++i) {
   // 注意：
   // 相机外参维护了几个 sigma 参数，代表不确定度；
   // 不确定度包括 relative 和 absolute；
   // 如果 relative 的 sigma 很小，则沿用旧的外参；
   // 如果 relative 的 sigma 不是很小，则创建新的外参，添加进 Map
   // 目前的 OKVIS 代码中，sigma 都是 0.
}
// IMU states are automatically propagated.
for (size_t i=0; i<imuParametersVec_.size(); ++i){
   // 把 speedAndBias 加进 Map
   mapPtr_->addParameterBlock(speedAndBiasParameterBlock);
}
```

接下是添加约束信息（Residual）给 `Map`。首先，如果是第一帧，则添加 prior，包括

1. 第一个 pose 的 prior

   ```cpp
   std::shared_ptr<ceres::PoseError> poseError(new ceres::PoseError(T_WS, information));
   mapPtr_->addResidualBlock(poseError,NULL,poseParameterBlock);
   ```

2. 外参的 prior。注意，相机外参维护的 sigma 参数里，如果 absolute sigma 的值不是很小，则作为普通 ParameterBlock 添加进 `Map`，相当于待标定外参：

   ```cpp
   std::shared_ptr<ceres::PoseError> cameraPoseError(
      new ceres::PoseError(T_SC, translationVariance, rotationVariance));
   mapPtr_->addResidualBlock(
      cameraPoseError,
      NULL,
      mapPtr_->parameterBlockPtr(
          states.sensors.at(SensorStates::Camera).at(i).at(CameraSensorStates::T_SCi).id));
   ```

   如果 absolute sigma 的值很小，则作为 constant ParameterBlock 添加进 `Map`，相当于已标定外参：

   ```cpp
   mapPtr_->setParameterBlockConstant(
      states.sensors.at(SensorStates::Camera).at(i).at(CameraSensorStates::T_SCi).id);
   ```

3. 基于 IMU 的 constraint，用来约束 SpeedAndBias：

   ```cpp
   mapPtr_->addResidualBlock(
      speedAndBiasError,
      NULL,
      mapPtr_->parameterBlockPtr(
          states.sensors.at(SensorStates::Imu).at(i).at(ImuSensorStates::SpeedAndBias).id));
   ```

另一方面，如果目前不是第一帧，则添加为前一帧与新一帧的相对约束。包括

1. 基于 IMU 的 constraint，注意每个 imuError 会关联四个 ParameterBlock，包括上一帧的 Pose、上一帧的 speedAndBias、这一帧的 pose、这一帧的 speedAndBias：

   ```cpp
   std::shared_ptr<ceres::ImuError> imuError(
      new ceres::ImuError(imuMeasurements, imuParametersVec_.at(i),
                          lastElementIterator->second.timestamp,
                          states.timestamp));
   mapPtr_->addResidualBlock(
      imuError,
      NULL,
      mapPtr_->parameterBlockPtr(lastElementIterator->second.id),
      mapPtr_->parameterBlockPtr(
          lastElementIterator->second.sensors.at(SensorStates::Imu).at(i).at(
              ImuSensorStates::SpeedAndBias).id),
      mapPtr_->parameterBlockPtr(states.id),
      mapPtr_->parameterBlockPtr(
          states.sensors.at(SensorStates::Imu).at(i).at(
              ImuSensorStates::SpeedAndBias).id));
   ```

2. 如果存在新的外参状态（即 relative sigma 不是很小时，参看前面），添加外参的 error term：

   ```cpp
   mapPtr_->addResidualBlock(
      relativeExtrinsicsError,
      NULL,
      mapPtr_->parameterBlockPtr(
          lastElementIterator->second.sensors.at(SensorStates::Camera).at(
              i).at(CameraSensorStates::T_SCi).id),
      mapPtr_->parameterBlockPtr(
          states.sensors.at(SensorStates::Camera).at(i).at(
              CameraSensorStates::T_SCi).id));
   ```

#### Estimator::addObservation()

可以注意到，上面 `addStates()` 函数中添加的 ParameterBlock 和 Residual 都是和 vehicle states 相关的，和 landmark 与视觉观测无关。这主要由于视觉信息和 IMU、外参等信息不同，需要先经过视觉前端处理再加到后端来，所以单独处理。`Estimator::addObservation()` 这个函数实现了添加基于视觉观测的 Residual，其定义专门放在了 `okvis_ceres/include/okvis/implementation/Estimator.hpp` 文件里（这种架构太繁琐，简直是在刁难 IDE）：

```cpp
// create error term
std::shared_ptr< ceres::ReprojectionError<GEOMETRY_TYPE> > reprojectionError(
              new ceres::ReprojectionError<GEOMETRY_TYPE>(
              multiFramePtr->template geometryAs<GEOMETRY_TYPE>(camIdx),
              camIdx, measurement, information));
::ceres::ResidualBlockId retVal = mapPtr_->addResidualBlock(
    reprojectionError,
    cauchyLossFunctionPtr_ ? cauchyLossFunctionPtr_.get() : NULL,
    mapPtr_->parameterBlockPtr(poseId),
    mapPtr_->parameterBlockPtr(landmarkId),
    mapPtr_->parameterBlockPtr(
        statesMap_.at(poseId).sensors.at(SensorStates::Camera).at(camIdx).at(
            CameraSensorStates::T_SCi).id));
```

这个函数由视觉前端 `VioKeyframeWindowMatchingAlgorithm` 类调用。

#### Estimator::addLandmark()

`Estimator::addLandmark()`
这个函数实现添加 landmark ParameterBlock：

```cpp
mapPtr_->addParameterBlock(pointParameterBlock, okvis::ceres::Map::HomogeneousPoint)；
```

这个函数也由视觉前端 `VioKeyframeWindowMatchingAlgorithm` 类调用。


