---
layout: post 
title: "On-Manifold Optimization Demo using Ceres Solver"
date: 2018-01-23 01:00:00
categories: en
tags: [robotics, SLAM, Ceres Solver]
---

__Contents__

* contents
{:toc}

### Introduction 

[Ceres Solver](http://ceres-solver.org/) is a C++ library by Google for solving least-square optimization problems,
providing a variety of functions and options for different types of problems.

Optimization problems on Euclidean-space states are common, and can be solved using a standardized workflow. 
States are incremented automatically by addition, and Jacobians of error functions w.r.t. to the states can be calculated using the built-in auto-diff utility, 
both of which are handled by the solver, and users need not worry about them.

However, in many applications, such as the bundle adjustment (BA) problem in computer vision,
the camera poses are non-Eucliean, and lie in the SE(3) manifold, which requires on-manifold optimization techniques. 
Although the auto-diff utility [can still be used](http://ceres-solver.org/nnls_tutorial.html#bundle-adjustment), we sometimes prefer manually handling the on-manifold structures.

Here we demonstrate a demo project on solving an on-manifold graph optimization problem with Ceres Solver. 
For a similar demo with g2o, another popular graph optimization library, please read [SLAM Implementation: Bundle Adjustment with g2o](/2016/03/15/g2o-demo/).

### Demo Project

Project repo: [izhengfan/ba_demo_ceres](https://github.com/izhengfan/ba_demo_ceres).

This project implements a simple bundle adjustment problem using Ceres Solver. 

The constraints are defined with customized cost functions and self-defined Jacobians.
In Ceres Solver, this can be achieved by inheriting the `ceres::SizedCostFunction` class,
and overloading the `Evaluate()` function:

```cpp
template<int PoseBlockSize>
class ReprojectionErrorSE3XYZ: public ceres::SizedCostFunction<2, PoseBlockSize, 3>
{
public:
    virtual bool Evaluate(double const* const* parameters,
                          double* residuals,
                          double** jacobians) const;

};
```

Two versions of on-manifold local parameterization of SE(3) poses (both of which using 6-dof Lie algebra for updating) are used, including

  - quaternion + translation vector (7d), and
  - rotation vector + translation vector (6d)

Local parameterization of states is implemented by inheriting the `ceres::LocalParameterization` class,
with the overloaded function `Plus()` to designate the state incrementing method,
`ComputeJacobian()` to compute the Jacobians of the incremented states w.r.t to the incremental Lie algebra,
`GlobalSize()` and `LocalSize()` to specify the dimension of the Lie group parameter block and the incremental Lie algebra respectively.

```cpp
template<int PoseBlockSize>
class PoseSE3Parameterization : public ceres::LocalParameterization {
public:
    PoseSE3Parameterization() {}
    virtual ~PoseSE3Parameterization() {}
    virtual bool Plus(const double* x,
                      const double* delta,
                      double* x_plus_delta) const;
    virtual bool ComputeJacobian(const double* x,
                                 double* jacobian) const;
    virtual int GlobalSize() const { return PoseBlockSize; }
    virtual int LocalSize() const { return 6; }
};
```

Please check `parametersse3.hpp` `parametersse3.cpp` for the implementation of customized cost functions and local parameterization.

### Jacobians

For convenience in expression, we denote the Jacobian matrix of the error function w.r.t. the Lie group by `J_err_grp`, 
the Jacobian matrix of the Lie group w.r.t. the local Lie algebra increment by `J_grp_alg`, 
and the Jacobian matrix of the error function w.r.t. the local Lie algebra increment by `J_err_alg`.

In many on-manifold graph optimization problems, only `J_err_alg` are required.
However, in Ceres Solver, one can only seperately assign `J_err_grp` and `J_grp_alg`, but cannot directly define `J_err_alg`. 
This may be redundant and cost extra computational resources:

  - One has to derive the explicit Jacobians equations of both `J_err_grp` and `J_grp_alg`.
  - The solver would spend extra time to compute `J_err_alg` by multiplying `J_err_grp` and `J_grp_alg`.

Therefore, for convenience, we use a not-that-elegant but effective trick.
Let's say that the error function term is of dimension `m`, the Lie group `N`, and the Lie algebra `n`.
We define the leading `m*n` block in  `J_err_grp` to be the actual `J_err_alg`, with other elements to be zero;
and define the leading `n*n` block in `J_grp_alg` to be identity matrix, with other elements to be zero.
Thus, we are free of deriving the two extra Jacobians, and the computational burden of the solver is reduced -- 
although Ceres still has to multiply the two Jacobians, the overall computational process gets simpler.

Still, I wish the Ceres team add the functionality of directly designating `J_err_alg` in a future version.
g2o can do this explicitly.
There are already some [issues](https://github.com/ceres-solver/ceres-solver/issues/303) concerning this problem.
 
### Other tips

One advice for self-defined Jacobians: do not access `jacobians[0]` and `jacobians[1]` in the meantime, which may cause seg-fault.

