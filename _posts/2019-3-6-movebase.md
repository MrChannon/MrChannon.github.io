---
title: "Movebase在实践中的总结"
tags: navigation
---

## 背景

[**movebase**](http://wiki.ros.org/move_base?distro=melodic)是ROS 2d导航栈(navigation stack)的一部分。由于比赛需要所以花了许多时间去学习，然后我想分享下我所理解的入门的movebase使用方法。

<!--more-->

## 导航下的坐标系

导航主要是要厘清以下几个坐标系：world, odom, base_link, sensor_frame(s)。

* world:世界坐标系，小车在该坐标系中是真实的位置，也是定位想要知道的位置。
* odom:里程计坐标系，在该坐标系中小车的位置是由里程计信息或者里程计与其他传感器融合得到的位置值决定的。当小车没有启动时，odom坐标系与world坐标系是重合的，然而当
