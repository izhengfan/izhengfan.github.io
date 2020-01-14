---
layout: post
title:  "Optical Flow Estimation With Horn-Schunck Method"
date:   2015-03-25 23:53:41
categories: en
tags: [computer vision, MATLAB]
---

__Contents__

* content
{:toc}


### Introduction

Estimating the motion of every pixel in a sequence of images is a problem with many applications in computer vision, such as image segmentation, object classification, and driver assistance.

In general, optical flow describes a dense vector field, where a displacement vector is assigned to each pixel, which points to where that pixel can be found in another image. An example is shown as below.

![l5qMbq.md.jpg](https://s2.ax1x.com/2020/01/11/l5qMbq.md.jpg)

This project is a solution for Assignment 3 Optical Flow Estimation of the course Image Processing and Computer Vision (ENGG 5104, CUHK, 2015 Spring). In this project, I implement an algorithm solving the optical flow map (u,v) between two image frames using Horn-Schunck Method.

### Horn-Schunck Method

Horn-Schunck method is a classical optical flow estimation algorithm. It assumes smoothness in the flow over the whole image. Thus, it tries to minimize distortions in flow and prefers solutions which show more smoothness. In this assignment, we will implement the modern version of Horn-Schunck method, using pyramid-based coarse-to-fine scheme to achieve better performance, similar to that of Lucas-Kanade method. Many current optical flow algorithms are built upon its framework.

The objective is formulated as a global energy functional which is then minimized. Let the image be p = (x,y) and the underlying flow field be w(p) = (u(p),v(p), 1), where u(p) and v(p) are the horizontal and vertical components of the flow field, respectively. Under the brightness constancy assumption, the pixel value should be consistent along the flow vector, and the flow field should be piecewise smooth. This results in an objective function in the continuous spatial domain

![l57TG4.jpg](https://s2.ax1x.com/2020/01/11/l57TG4.jpg)
<!-- <p><img  src="/images/cvproject/eq1.jpg"></p> -->


where ∇ is the gradient operator, λ weights the regularization, I1 and I2 are two corresponding images.

### Variational Framework

To solve Eq. (1), we use an iterative flow framework. It assumes that an estimate of the flow field is w, and one needs to estimate the best increment dw(dw=(du,dv)), to update w. The objective function in Eq. (1) is then changed to

![l57Hz9.jpg](https://s2.ax1x.com/2020/01/11/l57Hz9.jpg)
<!-- <p><img  src="/images/cvproject/eq2.jpg"></p> -->

Let

![l57Ls1.jpg](https://s2.ax1x.com/2020/01/11/l57Ls1.jpg)
<!-- <p><img  src="/images/cvproject/eq3.jpg"></p> -->

With first-order Taylor expansion:

![l577RJ.jpg](https://s2.ax1x.com/2020/01/11/l577RJ.jpg)
<!-- <p><img  src="/images/cvproject/eq4.jpg"></p> -->

In the above equation we add variable p into du and dv to index du and dv in images.

For easy computation, we vectorize u, v, du, dv into U, V, dU, dV, and Dx, Dy to denote derivative filters in x and y direction. The continuous function E(du,dv)in Eq. (2) can be discretized as E(dU,dV).

### Optimization

The main idea to solve the above equation is to find dU,dV so that the gradient

![l575IU.jpg](https://s2.ax1x.com/2020/01/11/l575IU.jpg)
<!-- <p><img src="/images/cvproject/eq5.jpg" width="20%"></p> -->

We can derive

![l57Oqx.jpg](https://s2.ax1x.com/2020/01/11/l57Oqx.jpg)
<!-- <p><img  src="/images/cvproject/eq6.jpg"></p> -->

L is a Laplacian filter defined as

![l57jZ6.jpg](https://s2.ax1x.com/2020/01/11/l57jZ6.jpg)
<!-- <p><img  src="/images/cvproject/eq7.jpg"></p> -->

The term of dU in gradient is derived similarly. Therefore, solving the gradient equation can be performed in the following linear system

![l57vdK.jpg](https://s2.ax1x.com/2020/01/11/l57vdK.jpg)
<!-- <p> <img  src="/images/cvproject/eq8.jpg"></p> -->

### Evaluation

Having built the pyramids and solve Eq. (8) in each level iteratively, the result in the finest level is obtained. We evaluate optical flow results by two measures: Average Angle Error (AAE) and End Point Error (EPE). They are defined by comparing flow result with the ground truth flow. The lower the AAE and EPE are, the better the optical flow performance is. Also we can visually estimate your result from visualization of the flow map (using flowToColor function).

### Algorithm Flowchart

```
Input: frame1,frame2,λ
Build image pyramid;
Initialize flow = 0;
For i= numPyramidLevel:1
    Initialize flow from previous level;
    Build gradient matrix and Laplacian matrix;
    For j=1:maxWarpingNum
        Warp image using flow vector;
        Compute image gradient Ix, Iy, and Iz;
        Build linear system to solve HS flow;
        Solve linear system to compute the flow;
        Use median filter to smooth the flow map;
    EndFor
EndFor
Output: flow
```

### Experimental Results

Five pairs of images are tested, the results as execution time are shown below:

<p><img src="https://s2.ax1x.com/2020/01/11/l570Vf.jpg"  ></p>
<p><center><img  src="https://s2.ax1x.com/2020/01/11/l57dqP.jpg"></center></p>

Evaluation measure of picture above: AAE 9.669 average EPE 1.010.<br><br>

<p><center>
<a href="https://imgchr.com/i/l5bVXR"><img src="https://s2.ax1x.com/2020/01/11/l5bVXR.md.jpg" alt="l5bVXR.md.jpg" border="0"></a>
</center></p>
<p><center>
<a href="https://imgchr.com/i/l5bA1J"><img src="https://s2.ax1x.com/2020/01/11/l5bA1J.jpg" alt="l5bA1J.jpg" border="0"></a><center></p>

Evaluation measure of picture above: AAE 4.901 average EPE 0.456.<br><br>

<a href="https://imgchr.com/i/l5bD3Q"><img src="https://s2.ax1x.com/2020/01/11/l5bD3Q.md.jpg" alt="l5bD3Q.md.jpg" border="0"></a>
<a href="https://imgchr.com/i/l5bw4S"><img src="https://s2.ax1x.com/2020/01/11/l5bw4S.jpg" alt="l5bw4S.jpg" border="0"></a>

Evaluation measure of picture above: AAE 9.812 average EPE 0.297.<br><br>

<a href="https://imgchr.com/i/l5bB9g"><img src="https://s2.ax1x.com/2020/01/11/l5bB9g.md.jpg" alt="l5bB9g.md.jpg" border="0"></a>
<p><center>
<img src="https://s2.ax1x.com/2020/01/11/l5bcBq.jpg" alt="l5bcBq.jpg" border="0" /></center></p>

Evaluation measure of picture above: AAE 4.673 average EPE 0.250.<br><br>

<a href="https://imgchr.com/i/l5b4CF"><img src="https://s2.ax1x.com/2020/01/11/l5b4CF.md.jpg" alt="l5b4CF.md.jpg" border="0"></a>
<p><center>
<a href="https://imgchr.com/i/l5bf4U"><img src="https://s2.ax1x.com/2020/01/11/l5bf4U.jpg" alt="l5bf4U.jpg" border="0"></a></center></p>

Evaluation measure of picture above: AAE 12.257 average EPE 1.312.<br><br>


Some regions in the image groups with visible changes are circled to show that the Warped Image is more similar to frame 1 than frame 2, which indicates that the estimated optical flow is considerately accurate, since the the Warped Image representing frame2 warped to frame1 accroding to the estimated optical flow.
