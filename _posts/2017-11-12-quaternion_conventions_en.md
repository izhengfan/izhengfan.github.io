---
layout: math
title: "Quaternion Conventions: Hamilton and JPL"
date: 2017-11-12 01:00:00
categories: en
tags: robotics
---

Quaternion is a commonly used 3D rotation parameterization. It is written like $\mathbf{q}=q_0 + q_1 i + q_2 j + q_3 k$, in which $i, j, k$ forms the three bases of the imaginary part (analogous to the imaginary part of a complex number) and $i^2=j^2=k^2=-1$. Usually a rotation is represented by a unit quaternion (a quaternion whose norm is 1).

I used to think there is only one notation for quaternions, like the one from Wikipedia [\[1\]](https://en.wikipedia.org/wiki/Quaternion):

$$ i^2=j^2=k^2=ijk=-1 $$

With such definition, the transformation from a unit quaternion to a rotation matrix is like:

$$ {\bf R}=\begin{bmatrix} 1-2q_2^2-2q_3^2 & 2q_1q_2-2q_0q_3 & 2q_1q_3+2q_0q_2 \\ 2q_1q_2+2q_0q_3 & 1-2q_1^2-2q_3^2 & 2q_2q_3-2q_0q_1 \\ 2q_1q_3-2q_0q_2 & 2q_2q_3+2q_0q_1 & 1-2q_1^2-2q_2^2 \end{bmatrix} $$

However, I recently noticed that in some literature the transformation from quaternion to rotation matrix is not the one above, but its transpose. This confused me. My confusion was cleared when I read a tutorial _Indirect Kalman Filter for 3D Attitude Estimation_ [\[2\]](http://mars.cs.umn.edu/tr/reports/Trawny05b.pdf)  by Stergios I. Roumeliotis etc. The truth is that there are two conventions for quaternions, _Hamilton_ and _JPL_. The key difference between the two conventions lies in the relation between the three imaginary bases. In Hamilton convention, $ijk=-1$, while JPL defines $ijk = 1$. As consequences, the multiplication of quaternions and the transformation between quaternions and other rotation parameterizations differ with different quaternion conventions.

The co-existence of two conventions for quaternions is still confusing for me. Quaternion is introduced by mathematician William Rowan Hamilton, so I guess Hamilton convention has been used from the beginning by him. Then where the hell is JPL convention from? According to the materials I read that use JPL (like the tutorial by Roumeliotis), it comes from the well-known NASA Jet Propulsion Laboratory (JPL), and the cited original document seems to be a tech report titled _Quaternions - Proposed Standard Conventions_, which, unfortunately, I failed to search and find. 

One thing for sure, the co-existence of two conventions has resulted in some confusion and arguments among researchers. 

For instance, oddly enough, although the _Quaternion_ entry in Wikipedia uses Hamilton, and the _Quaternions and spatial rotation_ entry [\[3\]](https://en.wikipedia.org/wiki/Quaternions_and_spatial_rotation) uses Hamilton too, the _Conversion between quaternions and Euler angles_ entry [\[4\]](https://en.wikipedia.org/wiki/Conversion_between_quaternions_and_Euler_angles), on the contrary, uses JPL. I have to say, inconsistency and impreciseness and is a major weakness of free encyclopedias that everyone can edit like Wikipedia.

Nicolas Rotella also mentioned this problem in his report [\[5\]](http://www-clmc.usc.edu/~nrotella/IROS2014_linearization.pdf), and complained that "unfortunately, the quaternion algebra used in these conventions is often mixed up in the literature". He finally chose JPL.

There are also some concerning arguments and discussions in GitHub. One such [issue](https://github.com/ethz-asl/ethzasl_msf/issues/19) from a project of ETH-ASL:

<!-- ![](/images/quaternion_convention_issue.png) -->
![lXOZ8J.png](https://s2.ax1x.com/2020/01/15/lXOZ8J.png)

It seems Hamilton receives more favor from their group.

In another report _Quaternion kinematics for the error-state KF_ [\[6\]](http://www.iri.upc.edu/people/jsola/JoanSola/objectes/notes/kinematics.pdf), Joan Sola also voted for Hamilton.

My personal opinion: the current situation of two quaternion conventions co-existing is awful. I cannot understand what is the necessity of proposing another notation when there is Hamilton convention well used. ~~I know you guys from JPL (the lab) are amazing in creation and engineering, but arbitrarily introducing new standards is evil~~. In current situation, Any reliable material should at least point out the convention they use at the beginning. From a realistic point of view, the widely used linear algebra library Eigen uses Hamilton; Matlab uses Hamilton ([MathWorks: Quaternion](https://www.mathworks.com/discovery/quaternion.html)); ROS, Google Ceres Solver use Hamilton; let's use Hamilton to avoid any chaos in our development work. ~~JPL to hell~~.

__Updated 2018-03-19__: As for the Wikipedia entry \[4\], after confirmation, it actually just claims to use JPL, but still uses Hamilton throughout the equations... Don't know why someone just added this mismatched claim. Thanks to [Hannes](https://github.com/HannesSommer) for reminding me of this case.

Moreover, Hannes wrote a paper on this problem with more extensive historical investigation and implementation suggestions.
Read his paper on [arxiv](https://arxiv.org/abs/1801.07478).
A shorter version is also available at [ResearchGate](https://www.researchgate.net/publication/323426570_Why_and_How_to_Avoid_the_Flipped_Quaternion_Multiplication_-_Shorter_and_Less_Formal_Version).

### References:

[1] Wikipedia: Quaternion [↗](https://en.wikipedia.org/wiki/Quaternion)

[2] Nikolas Trawny and Stergios I. Roumeliotis, _Indirect Kalman Filter for 3D Attitude Estimation_ [↗](http://mars.cs.umn.edu/tr/reports/Trawny05b.pdf)

[3] Wikipedia: Quaternions and spatial rotation
 [↗](https://en.wikipedia.org/wiki/Quaternions_and_spatial_rotation)

[4] Wikipedia: Conversion between quaternions and Euler angles [↗](https://en.wikipedia.org/wiki/Conversion_between_quaternions_and_Euler_angles)

[5] Nicholas Rotella, _Quaternion Review and Conventions_ [↗](http://www-clmc.usc.edu/~nrotella/IROS2014_linearization.pdf)

[6] Joan Sola, _Quaternion kinematics for the error-state KF_ [↗](http://www.iri.upc.edu/people/jsola/JoanSola/objectes/notes/kinematics.pdf)

