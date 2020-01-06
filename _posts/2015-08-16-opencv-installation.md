---
layout: post
title:  "OpenCV: Installation and Comfiguration"
date: 2015-08-16 10:00:51 
categories: en
tags: OpenCV
---

__Contents__

* content
{:toc}

### Introduction

OpenCV is a well-known library for computer vision computation. Its website is [opencv.org](http://opencv.org/). This post discusses how to install OpenCV. While there are many versions and 3.0 is the latest, here we choose version 2.4.11.

### Installation in Windows

Installation of OpenCV in Windows is simple. In the official download [page](http://opencv.org/downloads.html), download the corresponding version for Windows, run the downloaded `.exe` file, and just follow the instructions to finish the installation process. 

An advisable strategy here is to arrange an individual directory for one version of OpenCV under a parent `opencv` directory, for convinience of version management. For example, in my Windows machine, I have installed two versions of OpenCV, 2.4.11 and 3.0.0, and the directory structure is:

	-opencv
	  --opencv2411
	  --opencv300
	 
All header files and lib files of version 2.4.11 are put in directory `opencv2411` and the case is similar for 3.0.0. Whenever I use the version 2.4.11 or 3.0.0, I choose the files in the corresponding directory. And whenever I feel one version useless and want to remove it, I just delete the directory of that version. If I want to install a new version, I simply create a new directory under `opencv`, and the new installation will not affect the old versions. On the other hand, if all versions are in one public directory, it would become a total mess and bring much difficulty in usage and management. 



### Installation in Linux

There are many Linux distributions, among which Ubuntu is the most popular. Here we only consider Ubuntu. Moreover, as mentioned in Ubuntu [documentation](https://help.ubuntu.com/community/OpenCV), installation of OpenCV may depend on Ubuntu versions since the latest Ubuntu incarnation comes with a new version of libav, and some previously standard installation methods will fail  in version newer than Ubuntu 14.04. I don't know why those guys in open source community always switch core libraries casually only to create more difficulty and annoyance in the experience of using their products.

As far as I know, there are mainly three methods to install OpenCV in Ubuntu, and the third one below is what I mostly recommend. Please note that you'd better install the [dependencies](http://docs.opencv.org/doc/tutorials/introduction/linux_install/linux_install.html#linux-installation) before installing OpenCV itself.

#### Apt-get

This is the easiest way, with just one command line in terminal:

```bash
sudo apt-get install libopencv-dev
```
	
The drawback of this method is: you cannot choose the version of OpenCV you want, and individual version management is therefore nearly impossile (it's even implicit where the header and lib files are located). Also, according to my experience, `nonfree` branch in OpenCV library is not included if installed via `apt-get`. The `nonfree` branch includes some feature detection implementation like SIFT and SURF.

#### Opencv.sh from Ubuntu Community

The method is stated clearly in the [Ubuntu community](https://help.ubuntu.com/community/OpenCV), you can simply execute the opencv.sh file provided. Be careful of the Version Note in the page: the provided script may not work in version after Ubuntu 14.10.

#### Local building from source

In principle, local building from source is the same to running `opencv.sh` as mentioned above, since the `opencv.sh` script just assembles all required commands into one .sh file. However, local building from source gives us more freedom to choose and manage OpenCV versions and to overcome detailed problems we may meet with in the installation process. Below is the local building process.

First, download the source file of the the version you like from the [website](http://docs.opencv.org/doc/tutorials/introduction/linux_install/linux_install.html#linux-installation), and extract the compressed file to local disk. Assume that you download OpenCV 2.4.11 and extract it to `~/opencv2411/`.

Second, in the extracted directory, create a directory `release`, and then in `release` configure Cmake like:

```bash
cd ~/opencv2411
mkdir release
cd release
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local/opencv/opencv2411 -D BUILD_PYTHON_SUPPORT=ON ..
```

Note that in cmake configuration `CMAKE_INSTALL_PREFIX` declares the directory we want OpenCV to be installed in, and we adopt the similar strategy to installtion in Windows mentioned above: allocate a directory to each version seperately under the parent directory `opencv`. Moreover, if you are using latest version of Ubuntu (14.04 or newer), there exists the problem of libav support. In order to complete the installtion of OpenCV, you can either install `ffmpeg` (its website [here](https://www.ffmpeg.org/download.html#build-linux)) beforehand, or disabled the  `ffmpeg` dependency in Cmake configuration with `-D WITH_FFMPEG=OFF` added in the command line.

Third, run the make (this takes quite some time):

```bash
make
sudo make install
```

### Installation in Mac

Installation in Mac is similar to that in Linux.

### Test Installation

Choose any one of the samples provided by OpenCV (eg. the [show image sample](http://docs.opencv.org/2.4/doc/tutorials/introduction/display_image/display_image.html)), compile and run it to see whether your installation is successful.

To use OpenCV in your project in Windows (here we only consider C++ project), just like using other libraries, you need to declared the header files directory and the lib files in your project's property. In Visual Studio, the most convinient way is to use property sheet. You can declare all headers and libs in one property sheet and add it to your project in property manager. I have put my property sheets in my [Github](https://github.com/izhengfan/OpenCVConfigForVS2013). If you want to use my property sheets, be aware of the OpenCV version you use, and the directory where you install it. Also, remember to put the `bin` directory of OpenCV into emvironment variables.

To use OpenCV in your project in Linux, you can compile with Cmake, or more conviniently if you compile only one .cpp file, with `pkg-config`. To compile with `pkg-config`, make sure you have installed it and put the configuration file of OpenCV for pkg-config `opencv.pc` (you can find it in `OPENCV_LIB_DIR/pkg-config/`) in the PATH of `pkg-config`. Then, compile the source file `filename.cpp` with this command:

```bash
g++ filename.cpp -o projectname `pkg-config --cflags --libs opencv`
```
	
If your installation and configuration is correct, this will produce a executable file named `projectname` and you can run it by

	./projectname
	
