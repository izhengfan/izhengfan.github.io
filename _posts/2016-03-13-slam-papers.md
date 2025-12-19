---
layout: post
title: "SLAM Materials: Literature Collection"
date:   2016-03-13 09:53:41
categories: en
tags: SLAM robotics
---

__Contents__

* content
{:toc}


### Introduction

This is collection of literature on SLAM (mainly Visual SLAM). Keep updating.

### Books

- _Probabilistic Robotics_ by S Thrun, W Burgard, D Fox - 2005 - MIT press

- _State Estimation for Robotics_ by TD Barfoot - 2017 

- _Multiple View Geometry in Computer Vision_ by A Harltey, A Zisserman - 2006 - Cambridge University Press

- _视觉 SLAM 十四讲_ by 高翔 etc - 2017 - 电子工业出版社

- _Estimation with Applications to Tracking and Navigation_ by Y Bar-Shalom, XR Li, T Kirubarajan

- _Optimal Estimation of Dynamic Systems, Second Edition_ by JL Crassidis, JL Junkins

### Tutorials and Basics

- SLAM Tutorial 1 [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=1638022)

- SLAM Tutorial 2 [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=1678144)

- Visual Odometry 1 [↗](http://ieeexplore.ieee.org/xpl/freeabs_all.jsp?arnumber=6096039)

- Visual Odometry 2 [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=6153423)

- Bundle Adjustment [↗](http://link.springer.com/chapter/10.1007/3-540-44480-7_21)

- A tutorial on graph-based SLAM [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=5681215)

- Motion and structure from motion in a piecewise planar environment [↗](https://www.researchgate.net/profile/Olivier_Faugeras/publication/243764888_Motion_and_structure_from_motion_in_a_piecewise_planar_environment/links/541810b70cf25ebee987ff2e.pdf) 


### Parameterization

- Inverse Depth Parametrization for Monocular SLAM [↗](http://ieeexplore.ieee.org/abstract/document/4637878/)

- Pose parameterization using Lie groups

    - On-Manifold Preintegration for Real-Time Visual--Inertial Odometry [↗](http://ieeexplore.ieee.org/abstract/document/7557075/)

	- _State Estimation for Robotics_ by TD Barfoot, Chapter 7.


- Pose estimation using linearized rotations and quaternion algebra [↗](http://www.sciencedirect.com/science/article/pii/S0094576510002407)

### Batch Optimization

- g2o [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=5979949)

- Google Ceres Solver [↗](http://ceres-solver.org/nnls_tutorial.html)

- GTSAM [↗](https://bitbucket.org/gtborg/gtsam)

- SBA [↗](http://users.ics.forth.gr/~lourakis/sba/sba-toms.pdf)

- Relative Bundle Adjustment [↗](http://www.robots.ox.ac.uk/~gsibley/Personal/Papers/rba.pdf)


### Graph SLAM

- Olson 2006 [↗](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=1642040)

- Square Root SAM 2006 [↗](http://ijr.sagepub.com/content/25/12/1181.short)

- iSAM 2008 [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=4682731)

- Gresetti 2009 [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=5164927)

- iSAM2 2011 [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=5979641)

- Johannsson 2013 Temporally scalable visual SLAM using a reduced pose graph [↗](http://ieeexplore.ieee.org/xpls/icp.jsp?arnumber=6630556)

- Survey of Geodetic Method for SLAM [↗](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=6894692)

- Generalized graph SLAM: Solving local and global ambiguities through multimodal and hyperedge constraints [↗](http://ijr.sagepub.com/content/35/6/601.abstract?rss=1)

- COP-SLAM: Pose Chain Graph [↗](http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7272096)

- Towards a robust back-end for pose graph SLAM [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=6224709&tag=1)


### Batch vs filter

- The iterated Kalman filter as a Gauss–Newton method [↗](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=250476&tag=1)

- Why filter? [↗](http://ieeexplore.ieee.org/abstract/document/5509636/)

- Introduction part in OKVIS paper [↗](http://ijr.sagepub.com/content/34/3/314.short)



### Mapping

- Robot Mapping: A Survey [↗](http://robots.stanford.edu/papers/thrun.mapping-tr.pdf)


### Important SLAM Works

- EKF Based: P. Newman's group's work [↗](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=938381)

- Partical Filter Based: FastSLAM [↗](http://www.aaai.org/Papers/AAAI/2002/AAAI02-089.pdf)


### Impressive Visual SLAM Works

- EKF - Sparse Feature Based

	- MonoSLAM [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=4160954)

- Keyframe Graph Optimization - Sparse Feature Based

	- PTAM [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=4538852)

- Relocalization: [↗](http://ieeexplore.ieee.org/xpls/icp.jsp?arnumber=4409115)

- Keyframe Graph Optimization - Direct Dense Based

	- DTAM [↗](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=6126513)

- Direct Dense with Surface Construction

	- KinectFusion [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=6162880)

### Loop Closing

- Appearance Based: Ulrich 2000 [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=844734)

- TREE-MAP: Frese 2006 Closing a Million-Landmarks Loop [↗](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=4059219)

- Bags of Words Bases: Angeli 2008 [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=4543475)

- Appearance Based: FAB-MAP 2008 [↗](http://ijr.sagepub.com/content/27/6/647.short)

- Appearance Based: FAB-MAP 2.0 2010 [↗](http://ijr.sagepub.com/content/30/9/1100.short)

- Vocabulary tree [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=1641018&tag=1)

- Bag of Binary Words Based 2011 [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=6094885) 

- Down-sampled Binarized Images Based: H. ZHANG 2014 [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=6906955)

- Gist
	
	- implementation of gist [↗](http://people.csail.mit.edu/torralba/code/spatialenvelope/)
	
	- loop close with gist in in Manhattan World [↗](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.680.9042&rep=rep1&type=pdf) 
	
- Visual Place Recognition A Survey [↗](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=7339473)


### State of Art Works

- Direct dense method: LSD-SLAM [↗](http://link.springer.com/chapter/10.1007/978-3-319-10605-2_54)

- Semi-Direct method: SVO [↗](http://ieeexplore.ieee.org/abstract/document/6906584/) and SVO 2.0 [↗](http://rpg.ifi.uzh.ch/docs/TRO17_Forster-SVO.pdf)

- Feature based method: ORB-SLAM [↗](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=7219438)

- Direct sparse method: DSO [↗](http://ieeexplore.ieee.org/abstract/document/7898369/)


### Visual Inertial SLAM

- MSCKF

	- 1.0 [↗](http://ieeexplore.ieee.org/abstract/document/4209642/)

	- 2.0 in Li's dissersion: _Visual-Inertial Odometry on Resource-Constrained Systems_ [↗](https://search.proquest.com/docview/1656483484?pq-origsite=gscholar)

- Others from Mourikis and Li's group:
	
	- Decoupled Representation of the Error and Trajectory Estimates for Efficient Pose Estimation [↗](http://www.roboticsproceedings.org/rss11/p09.pdf)

	- Vision-aided Inertial Navigation with Rolling-Shutter Cameras [↗](http://journals.sagepub.com/doi/abs/10.1177/0278364914538326)

	- Online Temporal Calibration for Camera-IMU Systems: Theory and Algorithms [↗](http://journals.sagepub.com/doi/abs/10.1177/0278364913515286)

- Keyframe-based visual–inertial odometry using nonlinear optimization [↗](http://ijr.sagepub.com/content/34/3/314.short)

	- Source Code [OKVIS](http://ethz-asl.github.io/okvis/)

- ROVIO [↗](http://journals.sagepub.com/doi/abs/10.1177/0278364917728574)
	
- Inertial ORB-SLAM [↗](http://ieeexplore.ieee.org/abstract/document/7817784/)

- On-Manifold Preintegration for VIO [↗](http://ieeexplore.ieee.org/abstract/document/7557075/)

- Asynchronous adaptive conditioning for visual–inertial SLAM [↗](http://ijr.sagepub.com/content/34/13/1573.abstract)

- S. Jones IJRR 2010 [↗](http://ijr.sagepub.com/content/30/4/407.short)

- Kelly IJRR 2011 [↗](http://journals.sagepub.com/doi/abs/10.1177/0278364910382802)

- A Hesch IJRR 2014 [↗](http://ijr.sagepub.com/content/33/1/182.full.pdf+html)

- A Hesch TRO 2014 [↗](http://ieeexplore.ieee.org/abstract/document/6605544/)


### Fusing Odometry And Other Sensor into V-SLAM

- Weighted Local BA [↗](http://maxime.lhuillier.free.fr/pBmvc10.pdf)

- Fast Odometry Integration in Local Bundle Adjustment-Based Visual SLAM [↗](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=5597789)

- Bi-Objective Bundle Adjustment with Application to Multi-Sensor SLAM [↗](http://michot.julien.free.fr/drupal/sites/default/files/3DPVT_Michot_etal.pdf)

### Combining More Image Information

- combining visual SLAM and dense scene flow [↗](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=6224690) 

- Incorparating edges:

    - PL-SLAM [↗](https://github.com/rubengooj/pl-slam)


### Semantic SLAM and Learning for SLAM

- SLAM++ [↗](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=6619022)

- Special Issues IJRR on RSS 2014 [↗](http://ijr.sagepub.com/content/35/1-3.toc)

- Probabilistic Data Association for Semantic SLAM (Best Paper in ICRA-2017) [↗](http://ieeexplore.ieee.org/abstract/document/7989203/)

- Learning for Localization and Mapping, workshop at IROS 2017 [↗](https://sites.google.com/site/learningforslam/home)


### Slides

- The Problem of Mobile Sensors (Workshop in RSS 2015) [↗](http://ylatif.github.io/movingsensors/)

- From author of ORB-SLAM [↗](http://wp.doc.ic.ac.uk/thefutureofslam/wp-content/uploads/sites/93/2015/12/ICCV15_SLAMWS_RaulMur.pdf)

### Good blogs and discussions

- The Future of Real-Time SLAM and "Deep Learning vs SLAM" [↗](http://www.computervisionblog.com/2016/01/why-slam-matters-future-of-real-time.html)

- 半闲居士 [↗](http://www.cnblogs.com/gaoxiang12/)

- 白巧克力亦唯心 [↗](http://my.csdn.net/heyijia0327)

- 刚刚开始做机器人，打算做SLAM，不知道机器人定位领域现在有哪些比较新的算法？希望大家推荐推荐 [↗](http://zhihu.com/question/29434085/answer/47314765)

- 去美国读CS博士，方向是机器人导航，视觉方面，推荐一下相关编程方面准备？还有相关算法需要学习哪些？[↗](https://www.zhihu.com/question/28025607)
