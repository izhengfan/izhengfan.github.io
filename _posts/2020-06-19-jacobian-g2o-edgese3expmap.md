---
layout: math
title: "Derivation of Jacobians in g2o::EdgeSE3Expmap"
date: 2020-06-19 01:00:00
categories: en
tags: [robotics, SLAM, g2o]
---

I has [been asked](https://github.com/izhengfan/se2lam/issues/26) about how to derive the Jacobians in [`g2o::EdgeSE3Expmap`](https://github.com/RainerKuemmerle/g2o/blob/20160424_git/g2o/types/sba/types_six_dof_expmap.cpp#L271).
Here is my derivation (note that $T_i$ and $T_j$ are the state of pose $i$ and $j$, $e$ is the error function, and $\bar{T}_{ij}$ is the measurment):

$$
\begin{aligned}
    e &= Log(T_j^{-1} \bar{T}_{ij} T_i) \\
    e(\delta \xi_i) 
    &= Log(T_j^{-1} \bar{T}_{ij} Exp(\delta \xi_i) T_i) \\
    &= Log(T_j^{-1} \bar{T}_{ij} Exp(\delta \xi_i) (T_j^{-1} \bar{T}_{ij})^{-1}T_j^{-1} \bar{T}_{ij} T_i) \\
    &= Log(Exp(Adj( T_j^{-1} \bar{T}_{ij}) \delta \xi_i ) T_j^{-1} \bar{T}_{ij} T_i) \\
    &= Log(Exp(Adj( T_j^{-1} \bar{T}_{ij}) \delta \xi_i ) Exp(e)) \\
    &= Adj( T_j^{-1} \bar{T}_{ij}) \delta \xi_i +  e \\
  \frac{\partial e}{\partial \delta \xi_i}  &=  Adj( T_j^{-1} \bar{T}_{ij})  \\
    e(\delta \xi_j) 
    &= Log(T_j^{-1} Exp(-\delta \xi_j)  \bar{T}_{ij}T_i) \\
    &= Log(T_j^{-1}\bar{T}_{ij} T_i \, (\bar{T}_{ij}T_i)^{-1} Exp(-\delta \xi_j)  \bar{T}_{ij}T_i) \\
    &= Log(Exp(e) \, T_i^{-1}\bar{T}_{ij}^{-1} Exp(-\delta \xi_j)  (T_i^{-1}\bar{T}_{ij}^{-1})^{-1}) \\
    &= Log(Exp(e) Exp(- Adj(T_i^{-1}\bar{T}_{ij}^{-1})\delta \xi_j)) \\
    &= e - Adj(T_i^{-1}\bar{T}_{ij}^{-1})\delta \xi_j\\
    \frac{\partial e}{\partial \delta \xi_j}  &= - Adj(T_i^{-1}\bar{T}_{ij}^{-1})
\end{aligned}
$$
