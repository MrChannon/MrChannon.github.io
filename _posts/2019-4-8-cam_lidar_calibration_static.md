---
title: "相机-激光雷达静态标定"
tags: computer-vision
---

## 背景

在图像上做Object detection 准确率目前可以达到90%以上，而在点云上做端到端的Object detection准确率却很低--车辆大概70%,行人大概40%，并且这样的网络换个不同线数的雷达就不行了。（点云有其特殊性， 近处和远处稠密程度不一样，不同线数的激光雷达稠密程度也不一样）。

但是图像做物体识别却没有办法得到深度信息，于是很自然地，便有了将激光雷达点云上的点投影到相机图像上，借而得到深度的想法。而这样做的第一步就是激光雷达和相机的标定，借而得到相机与激光雷达的坐标系间的转换。
<!--more-->
## Paper与源码

- [Arxiv上的文章：LiDAR-Camera Calibration using 3D-3D Point correspondences](https://arxiv.org/abs/1705.09785)
- [Github源码](https://github.com/ankitdhall/lidar_camera_calibration)

## 原理

总结：**ICP就完事了。**

### 实验环境

![setup](/assets/images/post_images/cam_lidar_calib_static/setup.png)

如图，左边是zed双目相机（当单目用），右边是VLP-16。两块硬纸板上装的是[Aruco](https://github.com/pal-robotics/aruco_ros)，类似于April tag，单目相机可以很容易通过PnP获得它自身坐标系到相机坐标系的R丨t。

### 参数

运行程序前需要提前知道的参数：

- 相机分辨率
- 相机内参
- 初始旋转的估计
- ![setup](/assets/images/post_images/cam_lidar_calib_static/cardboard.jpg)该图标注的距离。

### 标定细节

1. 根据Aruco这个marker确定相机到Aruco坐标系的转换（原理为PnP），然后根据上图那些s1, s2, b1, b2, e等等量出来的距离，确定硬纸板四个角点在相机坐标系的三维坐标。
2. 在激光雷达点云中，手点，框出硬纸板的四条边，然后用直线拟合这四条边，每两条边的交点为硬纸板的定点。
3. 由1,2得到的四组对应3d点，由于有两个硬纸板，所以有八组对应点，随后用ICP求解两坐标系之间的R丨t。

## 其他

文章一开始是直接用硬纸板的四个像素点和硬纸板做PnP的，后来发现效果不好（可能是因为他们像素点是手点的），后来才改成这个方法。