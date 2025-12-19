---
title: OKVIS ImuError
author: ZHENG, Fan {fzheng@link.cuhk.edu.hk}
date: 2018-03-28
layout: math
---

> Must pre-read: [OKVIS IMU Error](https://blog.csdn.net/fuxingyin/article/details/53449209) by Xingyin Fu

## States

According to the two papers:

- Preintegration: $[p, R, v, b_g, b_a]$

- OKVIS: $[p, R, {}_S v, b_g, b_a]$.

This leads to different measurement model:

- Preintegration:

$$
\tilde{a} = C^T (\dot{v}-g)+b_a+\eta_a
$$

- OKVIS:

$$
\tilde{a} = C^T (\omega\times{}_Sv + C{}_S\dot{v} -g)+b_a+\eta_a
$$

The OKVIS version is more troublesome.

__However__, in OKVIS code, they still use the same $v$ as Preintegration.

The actual propogation:

$$
v_1 = v_0 + C_{0} \sum_k C_{0:k} (\tilde{a}_{k} - b_a) \Delta t_k - g \Delta t
$$

Note, the vector $g$ in OKVIS code is of the __opposite sign__ of that in Preintegration.
In OKVIS, $g=[0,0,9.8]$.
In contrast, in Preintegration,
${}_Wg=[0,0,-9.8]$, which is more physically accurate.
Imagine that in still state, the measured ${}_W\tilde{a}$ should be $-{}_Wg$.

Never mind, we only need to ensure the computation correctness in the implementation.

The actually used measurement model in OKVIS code:

$$
\tilde{a} = C^T (\dot{v}+g)+b_a+\eta_a
$$

In still state, the IMU gives measurement of $[0, 0, 9.8]$; and when $a=[0,0,-9.8]$ the measurement is $0$.
This equation is correct.


## SE3 Update

Preintegration: $p \leftarrow p+C\delta p$.

OKVIS: $p \leftarrow p+\delta p$

Therefore,

$$ \delta p_{\rm pre} = C^T \delta p_{\rm okvis} $$

Preintegration with right perturbation: $C \leftarrow C\, {\rm Exp}(\delta \alpha)$

OKVIS with left perturbation: $C \leftarrow {\rm Exp}(\delta \alpha) C$

$${\rm Exp}(\delta \alpha_{\rm pre}) = C^T{\rm Exp}(\delta \alpha_{\rm okvis}) C$$

$$\delta \alpha_{\rm pre} = C^T\delta \alpha_{\rm okvis}$$

Different forms of state incrementing would lead to different Jacobians.

## Jacobians

Calculate some intermiate variables:

$$
\begin{aligned}
& {}_W \delta_p = t_0 - t_1 + v_0 \Delta t - 0.5 g \Delta t^2 \\
& {}_W \delta_v = v_0 - v_1 - g \Delta t\\
& \Delta \tilde{q} = \prod q( (\tilde{w}_k - b_g) \Delta t_k ) \\
& \Delta \tilde{q} \leftarrow \delta q( - \frac{d\alpha}{d{b_g}}\Delta {b_g}) * \Delta \tilde{q} \\
& \int_a = \int_{a_{(0:1)}} = \sum_k C_{0:k}(\tilde{a}_{k} - b_a) \Delta t_k\\
& \iint_a = \sum_k \int_{a_{(0:k)}} \Delta t + 0.5 C_{0:k} (\tilde{a}_k - b_a) \Delta t_k^2\\
& \int_C = \int_{C_{(0:1)}} = \sum_k C_{0:k} \Delta t_k \\
& \iint_C = \sum_k \int_{C_{(0:k)}} \Delta t + 0.5 C_{0:k} \Delta t_k^2\\
\end{aligned}
$$

Here

- $\int_a$ corresponds to $\Delta \tilde{v}_{ij}$ in Preintegration;
- $\iint_a$ corresponds to $\Delta \tilde{p}_{ij}$ in Preintegration.
- $\int_C$ corresponds to $-\frac{\partial \Delta \bar{v}_{ij}}{\partial b_a}$ in Preintegration.
- $\iint_C$ corresponds to $-\frac{\partial \Delta \bar{p}_{ij}}{\partial b_a}$ in Preintegration.

Note that the calculation of $\frac{d \alpha}{db_g}$ in the code drops a minus symbol,
so an extra minus symbol appears in $\Delta\tilde{q}$ above.
Strange.

Jacobians of residuals w.r.t. parameter 0:

$$
\begin{aligned}
& F_0 = I_{15} \\
& F_{0_{(0:2,0:2)}} = C_0^{T} \\
& F_{0_{(0:2,3:5)}} = C_0^{T}*[{}_W\delta_p]_\times \\
& F_{0_{(0:2,6:8)}} = C_0^{T}*\Delta t \\
& F_{0_{(0:2,9:11)}} = \frac{dp}{d{b_g}}' \\
& F_{0_{(0:2,12:14)}} = - \iint_C \\
& F_{0_{(3:5,3:5)}} = (Q_l(\Delta\tilde{q}*q_1^{-1})  Q_r(q_{0}))_{(0:2,0:2)} \\
& F_{0_{(3:5,9:11)}} = (Q_r(q_1^{-1}*q_0)Q_r(\Delta\tilde{q}))_{(0:2,0:2)}*-\frac{d\alpha}{d{b_g}}' \\
& F_{0_{(6:8,3:5)}} = C_0^{T}*{}_W\delta_v \\
& F_{0_{(6:8,6:8)}} = C_0^{T} \\
& F_{0_{(6:8,9:11)}} = \frac{dv}{d{b_g}} \\
& F_{0_{(6:8,12:14)}} = -\int_C
\end{aligned}
$$


Jacobians of residuals w.r.t. parameter 1:

$$
\begin{aligned}
& F_1 = -I_{15} \\
& F_{1_{(0:2,0:2)}} = -C_0^T \\
& F_{1_{(3:5,3:5)}} = -(Q_l(\Delta\tilde{q} )Q_r(q_0)Q_l(q_1^{-1}))_{(0:2,0:2)} \\
& F_{1_{(6:8,6:8)}} = -C_0^T
\end{aligned}
$$

## Residuals

$$
\begin{aligned}
&r_p = e_{(0:2)}=C_0^{T}{}_W\delta_p+\iint_a+F_{0_{(0:2,9:14)}}*\Delta b \\
&r_R = e_{(3:5)}=2(\Delta\tilde{q}*(q_1^{-1}*q_0))_v\\
&r_v = e_{(6:8)}=C_{0}^T*{}_W\delta_v+\int_a+F_{0_{(6:8,9:14)}}*\Delta b\\
&r_b = e_{(9:14)}= b_0-b_1
\end{aligned}
$$

All residuals happend to be the inverse/negative version of those in Preintegration.
This does not affect the optimization process, but would invert the sign of the Jacobians.
