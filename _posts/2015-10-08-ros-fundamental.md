---
layout: cnpost
title: "ROS 基本概念和配置"
date: 2015-10-08 12:00:00
categories: cn
tags: ROS robotics
---

转载自： [http://my.phirobot.com/blog/2013-12-overlay_catkin_and_rosbuild.html](http://my.phirobot.com/blog/2013-12-overlay_catkin_and_rosbuild.html)


### Summary


本文通过一系列问题，解释 ROS 里面 node, package, workspace, source setup.bash, build system, overlay 等概念。并以最常用的 workspace 搭配方式，记录如何配置一个 catkin 和一个 rosbuild 工作空间，并 overlay 它们。

环境：ROS Hydro。


### 概念

ROS 里面有一系列概念，作为初学者，最先接触的概念非 node, package 和 workspace 莫属了。

#### node

node 是 ROS 里面最小的执行单位，你可以把 node 看成是一个 main 函数，当你启动一个 node，就相当于启动了一个 main 函数，通常这个 main 函数会不停循环监听某个消息，或者执行一系列操作，直至你关闭它，或者执行完退出。

#### package

package 是 ROS 里面最小的编译单位，也是 ROS 里面的搜索单位，你可以把 package 看成是一个有结构的文件夹，它包含多个 node 以及一些结构性质文件。ROS 里面的 node，是以 package 为单位进行编译的，一次编译 package 里面多个 node。

通常一条 ROS 的 node 执行命令具有以下结构：

    rosrun <package_name> <node_name>

即执行某个 node 的时候，要标识这个 node 是属于哪个 package 的，ROS 会先通过 package 的名字检索到 package，再通过 node 名字找这个 package 下对应的 node 去执行。

#### workspace

首先思考一个问题： package 都被放在什么地方？

package 存在于两个地方。一个是 ROS 库，即当你安装 ROS 的地方，通常在 `opt/ros/<distro>/share/` (`<distro>` 指你的 ROS 版本，如 hydro, groovy)里面（hydro 以前的版本还会有 stack），你无需关心，这些都是从 ROS 远程仓库下载下来的。另外一个地方，就是我们自己建的 workspace 了，我们自己的 package，或者网上下载的别人的 package 都必须放在 workspace 里面编译，才可以使用。

workspace 是 ROS 里面最小环境配置单位，你可以把 workspace 看成是一个有结构的文件夹，它包含多个 package 以及一些结构性质文件。一次将一个 workspace 配置进环境变量里面，你才能使用 ROS 命令执行与这个 workspace 里面的 package 相关的操作。

#### build system

前面提到的 package 和 workspace 都是有结构的，那么这些结构到底是怎么规定的呢？

这些结构，就是由 build system（编译系统）规定的。当你运行 workspace 相关的命令，它会在一个你指定的空文件夹下面放很多功能性质的文件，并将这个空文件夹变成一个 workspace。当你运行 package 相关的命令，它会在你指定的路径下面创建出 package。

目前，ROS 的 build system 有两种，一种叫做 catkin，另外一种叫做 rosbuild。利用不同的 build system 创建出来的 workspace 和 package 因此也分为两种。

rosbuild 是 ROS 传统的编译系统，从最初沿用至今，但面临被抛弃的状态。catkin 源于 ROS  fuerte，当时只是被一小部分人使用，在 fuerte 的下一个版本 groovy 开始被正式使用，用于取代 rosbuild。为什么要用 catkin 取代 rosbuild？当然是因为 catkin 比 rosbuild 好很多，在实际应用过程中会感受到的，具体理由可以看看这个 [catkin_conceptual_overview](http://my.phirobot.com/blog/2013-12-overlay_catkin_and_rosbuild.html#catkin-conceptual-overview) 。基于 catkin 编写的 package 叫做 wet package，基于 rosbuild 编写的 package 叫做 dry package，在 ROS 相关的问答里面会常常看到这种说法的。

#### source setup.bash

在 workspace 部分介绍了 package 存在的两个地方，那么，是不是我们打开 terminal，就能使用ROS命令，系统就能找到那些 package 呢？

这里，就要了解 `source setup.bash` 的功能。 `source setup.bash` 就是执行 `setup.bash` 脚本，而这个脚本，就会在环境中配置一些数据。

通常我们安装完 ROS 后，要先 `source /opt/ros/<distro>/setup.bash`，这条语句的目的就是将ROS相关的命令配置在当前 terminal 工作的环境中，我们在这个环境中就能使用 ROS 命令了，否则系统是无法知道 ROS 命令的存在的。同时，这条语句也会让 ROS 库中的 package 能够被找到。

当我们创建一个 workspace 后，同样系统也是不会知道这个 workspace 存在的，一切与这个 workspace 相关的 ROS 命令都会失效——这个 workspace 里面的 package 不会被 ROS 命令发现。要想将这个 workspace 的信息配置到环境里面，就必须执行这个 workspace 里面的 `setup.bash` 脚本，例如 catkin 的就是 `source ~/catkin_ws/devel/setup.bash`，rosbuild 的就是 `source ~/rosbuild_ws/setup.bash`。

#### overlay

再来思考一个问题：你有一个或者多个 catkin workspace，同时你也有一个或者多个 rosbuild workspace；而某个名字叫 example\_package 的 package 在 ROS库，在每个 workspace 里面都存在，当你运行 rosrun example\_package example\_package 的时候，到底会运行哪个位置的 example\_package呢？

这就关系到 overlay 的概念。overlay就是一种操作，可以让不同的 workspace 层层覆盖，最底层的是 ROS 库。通过这种覆盖关系，当寻找某个 package 的时候，ROS 会先从顶层的 workspace 找，如果找不到再依次往下找。也可以看成是一个路径链条，当我们想把通过 overlay 链接起来的 workspace 和 ROS 库的路径配入环境的时候，我们只需要 source overlay 链条顶层的 workspace 的 setup.bash 脚本即可。

### overlay rosbuild_ws->catkin_ws->ROS库

最常见的配置是一个 rosbuild workspace 和一个 catkin workspace 了（对应的文件夹分别为 rosbuild\_ws 和 catkin\_ws ）。因为有时候我们需要使用这两种 workspace 里面的 package，而往往是自己的 package 优先于 ROS 库里面自带的 package 的。

#### overlay catkin_ws->ROS库

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
catkin_init_workspace
cd ~/catkin_ws/
catkin_make
```

第一条语句创建 catkin\_ws 文件夹，以及 src 文件夹；第二条和第三条语句是在 src 路径下初始化 catkin workspace，并指定 src 路径是 package 存放的路径；第四条和最后一条语句是在 catkin_ws 路径下编译整个 workspace，将会生成 catkin workspace 的结构，以及编译 src 下所有的 package。


> Note
>
> 上面的操作会自动overlay ROS库，即catkin\_ws->ROS库。如果不需要再加入rosbuild workspace，则执行 `echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc` 就行了，跳过后面的所有步骤。

#### overlay rosbuild_ws->catkin_ws

```bash
sudo apt-get install python-rosinstall
```

在新装的系统第一次运行 rosbuild 相关操作前，还要先安装一下 rosinstall。

```bash
mkdir ~/rosbuild_ws
cd ~/rosbuild_ws
rosws init . ~/catkin_ws/devel
mkdir ~/rosbuild_ws/sandbox
rosws set ~/rosbuild_ws/sandbox
```

前两条语句创建并进入 rosbuild\_ws 文件夹；第三条语句初始化 rosbuild_ws 并 overlay 前面创建的 catkin workspace；第四条和最后一条语句创建文件夹 sandbox 并将其设置为 package 存放的文件夹。

#### source setup.bash

```bash
echo "source ~/rosbuild_ws/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

每次打开 terminal，都将会执行 `~/.bashrc` 脚本。因此，第一条语句将引号内的内容写入 `~/.bashrc` 后，每次打开 terminal，overlay 顶层的 rosbuild_ws 的 setup.bash 都会被 source，就不用我们手动 source了。

### 检查overlay路径

```bash
echo $ROS_PACKAGE_PATH
```

确认是否显示了下面4个路径：

```bash
/home/<user>/rosbuild_ws/sandbox:/home/<user>/catkin_ws/src:/opt/ros/<distro>/share:/opt/ros/<distro>/stacks
```

如果不全，说明之前的overlay出了问题。再进一步做检查：

```bash
gedit ~/rosbuild_ws/.rosinstall
```

看看有没有下面的内容：

	- setup-file: {local-name: /home/<user>/catkin_ws/devel/setup.sh}
	- other: {local-name: sandbox}

第一条指被 overlay 的 catkin workspace 路径，第二条指该 rosbuild workspace 的 package 存放目录。

更多的 overlay 概念，可以参考 [catkin_workspace_overlaying](http://my.phirobot.com/blog/2013-12-overlay_catkin_and_rosbuild.html#catkin-workspace-overlaying) 和 [using_rosbuild_with_catkin](http://my.phirobot.com/blog/2013-12-overlay_catkin_and_rosbuild.html#using-rosbuild-with-catkin) 。

### Reference

[catkin_conceptual_overview](http://wiki.ros.org/catkin/conceptual_overview)

[catkin_workspace_overlaying](http://wiki.ros.org/catkin/Tutorials/workspace_overlaying)

[using_rosbuild_with_catkin](http://wiki.ros.org/catkin/Tutorials/using_rosbuild_with_catkin)