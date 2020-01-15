---
layout: post
title: "Brief Review on Visual SLAM: A Historical Perspective"
date: 2016-05-30 08:00:00
categories: en
tags: SLAM robotics
---

__Contents__

* contents
{:toc}

### Earlier Inspirations


Inspirations for visual SLAM comes from two categories: basic SLAM and robot vision.

According to H. D.-Whyte et al.[1], the research on SLAM dates back to 1986, when the idea of using estimation-theoretic methods for robot localization and mapping were first discussed in IEEE Robotics and Automation Conference held in San Francisco. Yet it was until 1995 International Symposium on Robotics Research that the structure of the SLAM problem,
the convergence result and the coining of the acronym SLAM were presented.

Early SLAM works mostly relied on odometry and laser/ultrasonic sensors as perception input, such as J. Leonard et al. [2] (with ultrasonic), and J. S. Gutmann et al. [3] (with laser). The theoretical fundamental and prototype of traditional Bayesian filtering based SLAM framework emerged in 1990s, and the notable work was from S. Thrun , D. Fox and W. Burgard [4], (as well as P. Newman et al. [5] sooner), all of which together with concerning basic theory and specific topics are thoroughly discussed in their later published book, _Probabilistic Robotics_ [6].

Robot vision also began to boom in this period, important topics including visual odometry (VO) and structure from motion (SFM). Visual odometry is the process of estimating the ego-motion of a robot using only the input of a single or multiple cameras attached to it [7]. The history of VO, interestingly, dates back to early works [8] [9] motivated by the NASA Mars exploration program to provide all-terrain rovers with the capability to measure their 6-degree-of-freedom (DoF) motion in the presence of wheel slippage in uneven and rough terrains. SFM, investigating the problem of recovering relative camera poses and 3-D structure from a set of calibrated or un-calibrated camera images, is a larger and more general problem than VO, and can be considered as an early off-line version of visual SLAM [10]. Important SFM works were from A. Fitzgibbon in automatic camera pose recovery [11] and automatic 3-D model construction [12], M. Pollefeys [13] [14], etc. Both VO and SFM are based on the principles of visual multi-view geometry, which are well expounded in the famous book _Multiple View Geometry in Computer Vision_ [15] by R. Hartley and A. Zisserman.

Before 2000s, the research of SLAM in mobile robotics community and the research of SFM in computer vision community developed individually and almost turned away from each other.


### EKF Monocular SLAM

Landmark work of bringing vision into SLAM came from A. Davison's Mono-SLAM in early 2000s [16] [17] [18]. Mono-SLAM uses image features (local image patches) to represent landmarks in the map, iteratively updates the probability density of the feature depth by frame-to-frame matching to recover their 3-D positions and hence initilize feature-based sparse map, and updates the full state vector (robot pose plus feature 3-D locations) within an Extended Kalman Filter (EKF) framework. Davison's work in some way set the standard framework for traditional Bayesian filtering based visual SLAM implementation.

Meanwhile in early 2000s, there were several SLAM or related works also worth mentioning. M. Pupilli and A. Calway investigated utilizing particle filter instead of Gaussian filter framework in camera pose tracking and visual SLAM [19] [20]. Although more famous work of particle filter based SLAM is FastSLAM proposed by M. Montemerlo et al. [21], they used laser sensors instead of vision. Another significant work worth mentioning is in visual odometry from D. Nister et al. [22], who proposed methodology that sooner became almost 'golden method' for feature based VO and SFM, and are still frequently used in front-end of visual SLAM today. Another work that sooner became 'golden method' was in feature parameterisations from J. Montiel et al. [23], which used inverse depth instead of depth in feature position filter process.

### PTAM: New Standard for Local Tracking and Mapping


The problem of traditional EKF based visual SLAM mainly lies in computation effort as the map grows, since computation grows quadratically with the number of landmarks. The implementation of putting all landmarks together with the robot pose in a long state vector and a huge covariance matrix seems not making sense (because not all observed features have direct constraints with each other), and, as the covariance matrix is not sparse, computation resource spent on the iterative calculation of the covariance matrix is huge.

<!-- ![](/images/monovsptam.png)<br> -->
[![lXLzgs.md.png](https://s2.ax1x.com/2020/01/15/lXLzgs.md.png)](https://imgchr.com/i/lXLzgs)<br>
__Figure 1__: _Mono-SLAM vs PTAM. Left: visualized result of MonoSLAM. Middle: visualized result of keyframe-based tracking and mapping with PTAM. Right: demo of PTAM._

Interesting enough, inspiring work that set a new standard framework for visual SLAM did not come from the mobile robotics community, but from two researchers on Augmented Reality (AR). In 2007, Georg Klein and David Murray proposed Parallel Tracking and Mapping (PTAM) for small AR workspace [24]. PTAM is also a feature-based SLAM algorithm that tracks and maps many (hundreds of) features to achieve robustness. Simultaneously, it runs in real-time by creatively parallelizing the motion estimation and mapping tasks and by relying on efficient keyframe-based Bundle Adjustment (BA) instead of Bayesian filtering for pose and map refinement, which are two main reasons to make PTAM outperform MonoSLAM and the like in both efficiency and precision. Although PTAM is designed specifically for AR application and only works well in small desktop space without global map management, its implementation style of parallelizing tracking and mapping and keyframe-based map management is used by most of modern feature based visual SLAM systems (like ORB-SLAM [25]) or VO systems (like SVO [26]).

In fact, keyframe-based map refinement with BA belongs to so-called `Graph SLAM' techniques, as keyframes and map points are treated as nodes in a graph and optimized to minimize their measurement errors [27]. Graph SLAM is better than EKF SLAM in the sparsity of the graph and thus computational efficiency. Meanwhile in 2007, E. Eade and T. Drummond proposed an information filter based method of graph SLAM [28]. Information filter method is essentially similar to BA method in theory, but PTAM obviously achieved greater influence in visual SLAM research due to its creative implementation of parallel tracking and mapping.

### Effort towards Large Scale Mapping

PTAM achieves considerately satisfactory result in small local space, while the problem of large scale mapping remains to be solved. The solution to large scale map management basically includes two parts: 1) efficient map representation and refinement 2) and loop closing. Efficient map representation and and refinement is actually a necessary condition for good loop closing.

Graph SLAM shows its advantages in large scale mapping, since a graph can be easily sparsified to reduce redundant information. M. Kaess, F. Dellaert et al.'s works, including Square Root SAM (Smoothing and Mapping) [29], iSAM [30] and iSAM2 [31],  contributed a lot in theory for this problem. They investigated the theory and performance of graphical model for map representation and matrix factorization for valuable elimination and graph sparsification. Another famous work is FAB-MAP [32] by M. Cummins and P. Newman, which utilizes visual bag-of-words to do image retrieval for loop closure detection.


The problem of loop closure detection is still far from solved, with challenges mainly in place recognition in long-term SLAM [33], due to the illumination change, view-point change, dynamic or semi-dynamic objects and many other `pollution' factors within one scene at different visited moments.

### Modern State of Art Systems

As robotics and computer vision both become considerately prominent research fields to the public, visual SLAM attracts a large number of researchers to devote their energy and intelligence for visual SLAM system development, and share their works in open source communities. Modern state of art visual SLAM systems are more like `systematic engineering', building complete and practical systems with both trivial and non-trivial improvement techniques in theoretical and implementation level. They are built with many former researchers' work results as foundation, for example the improved image feature detector and descriptors like ORB [34] provide real-time performance with desktop CPU which SIFT or SURF cannot provide; some nonlinear least square optimization libraries like g2o [27] provide utilities to do bundle adjustment, etc.  Among them ORB-SLAM [25] by R. Mur-Artal et al. and LSD-SLAM (Large-Scale Direct Monocular SLAM) [35] by J. Engel et al. received much appreciation from the community.

ORB-SLAM is a more traditional feature based system, and quite similar to PTAM in some way, yet attains much more impressive performance in practice. Its main improvement compared to PTAM includes but not limited to: 1) implement 3 parallel threads, namely tracking, mapping, and loop closing to achieve consistent localization and mapping, while PTAM does not have loop closing; 2) automatic map initialization with a model selection on two paralleling thread calculating Homography and Fundamental ego-motion with RANSAC, while PTAM requires manual operation to finish initialization; 3) use ORB feature detector and descriptor instead of image patches used in PTAM, improving robustness of image tracking and feature matching under scale and orientation change; 4) multi-scale mapping, including local graph for pose bundle adjustment, co-visibility graph for local bundle adjustment, and essential graph for global bundle adjustment after loop closure detection.

<!-- ![](/images/lsdorb.png)<br> -->
[![lXLx3j.md.png](https://s2.ax1x.com/2020/01/15/lXLx3j.md.png)](https://imgchr.com/i/lXLx3j)<br>
__Figure 2__: _State of art visual SLAM systems. Left: result of LSD-SLAM. Right: result of ORB-SLAM_.


LSD-SLAM is quite different from PTAM or ORB-SLAM. It should be classified into so-called direct methods, that is, doing state estimation based on image pixels directly, rather than relying on image features. The tracking is performed by se(3) image alignment using a coarse-to-fine algorithm with a robust Huber loss. Depth estimation is just like many other SLAM systems, using an inverse depth parametrization with a bundle of relatively small baseline image pairs. Map optimization is also executed in commonly used graph optimization, with existing keyframe poses expressed in sim(3) space just like ORB-SLAM does. LSD-SLAM actually recovers a 'semi-dense' map, since  it only estimates depth at pixels solely near image boundaries, which may be the main reason for it to be the first direct visual SLAM system that can run in real-time on a CPU. In practice, processing every pixel over all image sequences is very computationally consuming, which is why many dense visual SLAM systems, like DTAM (Dense Tracking and Mapping) [36] by A. Davison's group, require a GPU to attain real-time performance.

The maturity of visual SLAM can also be reflected from their application in commercial products, among which the Google Tango and Microsoft Hololens attract most interest from the public. It can be predicted that there will be more visual SLAM driven products appear in the near future.

### What's Next?

#### More Semantic Information: From CV to Robotics

As mentioned above,  the main challenge in large scale mapping remains in long term visual place recognition. This problem is extremely difficult to solve based only on geometric methods, which are what most current visual SLAM systems use. These several years has seen some effort in utilizing semantic information in image to do object detection for mapping and scene recognition, such as SLAM++ [38] by R. F. Salas-Moreno et al. Due to complicated nature of place recognition problem, increasing number of researches on the utilization of visual semantic information in visual SLAM is predictable in recent future. In fact, this has already been happening, reflected by the frequent appearance of this topic in CVPR, RSS and ICRA workshops in recent years, as shown in Figure 3.

<!-- ![](/images/robotvision.png)<br> -->
[![lXLvCQ.md.png](https://s2.ax1x.com/2020/01/15/lXLvCQ.md.png)](https://imgchr.com/i/lXLvCQ)<br>
__Figure 3__: _Workshops on visual place recognition in CV and robotics conferences recent years_ [37].


Historically, robotic researchers on visual SLAM benefited a lot from research work of computer vision community. These years, as the booming of deep learning in computer vision field, it is also of high possibility that deep learning style methods will soon take their part in visual SLAM or other research fields in robotics. Let's see.

#### More Sensor Fusion: Towards Robustness for Practical Use

On the other hand, as geometric methods for visual SLAM gradually become mature, the day of its application in civil and industrial environment is approaching (or has approached, in some way, especially in VR/AR), which bring forward the issues of robustness. The state of art visual SLAM system mentioned above are far from being truly usable for mobile robot navigation (even if not consider the unrecovered scale problem of monocular vision). The leading limitation lies in their poor robustness. Robustness issue is an inherent problem of pure vision-based methods, due to the fact that image tracking is easy to fail in many motion modes and many environment structures. Therefore, research work on fusing image with other sensors to achieve robust mobile robot navigation is highly demanded.



### References

[1] H. Durrant-Whyte and T. Bailey. Simultaneous localization and mapping:
part i. Robotics Automation Magazine, IEEE, 13(2):99–110, June 2006.

[2] J. J. Leonard and H. F. Durrant-Whyte. Simultaneous map building and lo-
calization for an autonomous mobile robot. In Intelligent Robots and Systems
’91. ’Intelligence for Mechanical Systems, Proceedings IROS ’91. IEEE/RSJ
International Workshop on, pages 1442–1447 vol.3, Nov 1991.

[3] J. S. Gutmann and K. Konolige. Incremental mapping of large cyclic envi-
ronments. In Computational Intelligence in Robotics and Automation, 1999.
CIRA ’99. Proceedings. 1999 IEEE International Symposium on, pages 318–
325, 1999.

[4] S. Thrun, W. Burgard, and D. Fox. A probabilistic approach to concurrent
mapping and localization for mobile robots. Autonomous Robots, 5(3-4):253–
271, 1998.

[5] M. W. M. G. Dissanayake, P. Newman, S. Clark, H. F. Durrant-Whyte, and
M. Csorba. A solution to the simultaneous localization and map building
(slam) problem. IEEE Transactions on Robotics and Automation, 17(3):229–
241, Jun 2001.

[6] S. Thrun, W. Burgard, and D. Fox. Probabilistic robotics. MIT press, 2005.

[7] D. Scaramuzza and F. Fraundorfer. Visual odometry [tutorial]. Robotics &
Automation Magazine, IEEE, 18(4):80–92, 2011.

[8] H. P. Moravec. Obstacle avoidance and navigation in the real world by a
seeing robot rover. Technical report, DTIC Document, 1980.

[9] S. Lacroix, A. Mallet, R. Chatila, and L. Gallo. Rover self localization in
planetary-like environments. In Artificial Intelligence, Robotics and Automa-
tion in Space, volume 440, page 433, 1999.

[10] T. Malisiewicz.
The Future of Real-Time SLAM and "Deep Learning
vs SLAM" . http://www.computervisionblog.com/2016/01/why-slam-matters-future-of-real-time.html, 2016.

[11] A. Fitzgibbon and A. Zisserman. Automatic camera recovery for closed or
open image sequences. Computer VisionECCV’98, pages 311–326, 1998.

[12] A. Fitzgibbon, G. Cross, and A. Zisserman. Automatic 3d model construction
for turn-table sequences. 3D Structure from Multiple Images of Large-Scale
Environments, pages 155–170, 1998.

[13] M. Pollefeys, R. Koch, and L. Van Gool. A simple and efficient rectification
method for general motion. In Computer Vision, 1999. The Proceedings of
the Seventh IEEE International Conference on, volume 1, pages 496–501.
IEEE, 1999.

[14] M. Pollefeys. Self-calibration and metric 3D reconstruction from uncali-
brated image sequences. PhD thesis, 1999.

[15] R. Hartley and A. Zisserman. Multiple view geometry in computer vision.
Cambridge university press, 2003.

[16] A. J. Davison. Real-time simultaneous localisation and mapping with a single
camera. In Computer Vision, 2003. Proceedings. Ninth IEEE International
Conference on, pages 1403–1410. IEEE, 2003.

[17] A. J. Davison and D. W. Murray. Simultaneous localization and map-building
using active vision. Pattern Analysis and Machine Intelligence, IEEE Trans-
actions on, 24(7):865–880, 2002.

[18] A. J. Davison, I. D. Reid, N. D. Molton, and O. Stasse. Monoslam: Real-
time single camera slam. Pattern Analysis and Machine Intelligence, IEEE
Transactions on, 29(6):1052–1067, 2007.

[19] M. Pupilli and A. Calway. Real-time camera tracking using a particle filter.
In BMVC, 2005.

[20] M. Pupilli and A. Calway. Real-time visual slam with resilience to erratic
motion. In 2006 IEEE Computer Society Conference on Computer Vision and
Pattern Recognition (CVPR’06), volume 1, pages 1244–1249, June 2006.

[21] M. Montemerlo, S. Thrun, D. Koller, B. Wegbreit, et al. Fastslam: A fac-
tored solution to the simultaneous localization and mapping problem. In
AAAI/IAAI, pages 593–598, 2002.

[22] D. Nister, O. Naroditsky, and J. Bergen. Visual odometry. In Computer Vision
and Pattern Recognition, 2004. CVPR 2004. Proceedings of the 2004 IEEE
Computer Society Conference on, volume 1, pages I–652–I–659 Vol.1, June
2004.

[23] J. Montiel, J. Civera, and A. J. Davison. Unified inverse depth parametrization
for monocular slam. analysis, 9:1, 2006.

[24] G. Klein and D. Murray.
Parallel tracking and mapping for small ar
workspaces. In Mixed and Augmented Reality, 2007. ISMAR 2007. 6th IEEE
and ACM International Symposium on, pages 225–234. IEEE, 2007.

[25] R. Mur-Artal, J. M. M. Montiel, and J. D. Tards.
Orb-slam: A versa-
tile and accurate monocular slam system. IEEE Transactions on Robotics,
31(5):1147–1163, Oct 2015.

[26] C. Forster, M. Pizzoli, and D. Scaramuzza. Svo: Fast semi-direct monocular
visual odometry. In Proc. IEEE Intl. Conf. on Robotics and Automation, 2014.

[27] G. Grisetti, H. Strasdat, K. Konolige, and W. Burgard. g2o: A general frame-
work for graph optimization. In IEEE International Conference on Robotics
and Automation, 2011.

[28] E. Eade and T. Drummond. Monocular slam as a graph of coalesced obser-
vations. In 2007 IEEE 11th International Conference on Computer Vision,
pages 1–8, Oct 2007.

[29] F. Dellaert and M. Kaess. Square root sam: Simultaneous localization and
mapping via square root information smoothing. The International Journal
of Robotics Research, 25(12):1181–1203, 2006.

[30] M. Kaess, A. Ranganathan, and F. Dellaert. isam: Incremental smoothing
and mapping. Robotics, IEEE Transactions on, 24(6):1365–1378, Dec 2008.

[31] M. Kaess, H. Johannsson, R. Roberts, V. Ila, J. Leonard, and F. Dellaert.
isam2: Incremental smoothing and mapping with fluid relinearization and
incremental variable reordering. In Robotics and Automation (ICRA), 2011
IEEE International Conference on, pages 3281–3288, May 2011.

[32] M. Cummins and P. Newman. Fab-map: Probabilistic localization and map-
ping in the space of appearance. The International Journal of Robotics Re-
search, 27(6):647–665, 2008.

[33] S. Lowry, N. Snderhauf, P. Newman, J. J. Leonard, D. Cox, P. Corke, and
M. J. Milford. Visual place recognition: A survey. IEEE Transactions on
Robotics, 32(1):1–19, Feb 2016.

[34] E. Rublee, V. Rabaud, K. Konolige, and G. Bradski. Orb: An efficient alter-
native to sift or surf. In 2011 International Conference on Computer Vision,
pages 2564–2571, Nov 2011.

[35] J. Engel, T. Sch ̈ops, and D. Cremers. Lsd-slam: Large-scale direct monocular
slam. In Computer Vision–ECCV 2014, pages 834–849. Springer, 2014.

[36] R. A. Newcombe, S. J. Lovegrove, and A. J. Davison. Dtam: Dense tracking
and mapping in real-time. In Computer Vision (ICCV), 2011 IEEE Interna-
tional Conference on, pages 2320–2327. IEEE, 2011.

[37] Australian centre for robotic vision public. https://roboticvision.atlassian.net/wiki/display/PUB/RVSS2015+Robotic+
Vision+Summer+School+Presentations, 2016

[38] R. F. Salas-Moreno, R. A. Newcombe, H. Strasdat, P. H. J. Kelly, and A. J.
Davison. Slam++: Simultaneous localisation and mapping at the level of
objects. In Computer Vision and Pattern Recognition (CVPR), 2013 IEEE
Conference on, pages 1352–1359, June 2013.