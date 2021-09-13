---
title: "速腾激光雷达与velodyne激光雷达的数据转换"
tags: computer-vision
---

## 背景

上次提到了激光雷达与相机的标定，由于我毕设用的是速腾的激光雷达，发现标定时直接报错，故在此记录下问题与解决措施，借此详细分析下速腾激光雷达与velodyne激光雷达的异同。
<!--more-->

## 问题分析

标定时接上了robosense的激光雷达，提示点云中的点没有"ring"属性。
那么什么是点的"ring"属性？

## 有序点云与无序点云

首先介绍下什么是有序点云和无序点云。

> 有序点云数据集，意味着点云是类似于图像（或者矩阵）的结构，数据分为行和列。这种点云的实例包括立体摄像机和tof摄像机生成的数据。有序数据集的优势在于，预先了解相邻点（和像素点类似）的关系，邻域操作更加高效，这样就加速了计算并降低了PCL中某些算法的成本。

在PCL库中，每个pcl::Pointcloud类型都有两个属性，width与height。借CSDN的话来解释下。
> WIDTH –用点的数量表示点云数据集的宽度。根据是有序点云还是无序点云，WIDTH有两层解释：
1)它能确定无序数据集的点云中点的个数（和下面的POINTS一样）；
2)它能确定有序点云数据集的宽度（一行中点的数目）。

也就是说，对于无序点云，width等于点云中点的个数，而对于有序点云，width等于每行点云的点的个数。

很显然，激光雷达的点云属于有序点云。

## Velodyne 数据类型

然而对有序点云进行操作（降采样、刚体变换）后，有序点云就会变成无序点云。velodyne选择给每个点多加了一个属性ring，详细定义如下：

```cpp
namespace velodyne_pointcloud
{
  struct PointXYZIR
  {
  PCL_ADD_POINT4D;                    // quad-word XYZ
  float    intensity;                 
  uint16_t ring;                      
  EIGEN_MAKE_ALIGNED_OPERATOR_NEW     // ensure proper alignment
  } EIGEN_ALIGN16;
}; // namespace velodyne_pointcloud
```

这个ring表示每个点之前属于哪个线的，16线的话，这个值为0~15。并且高度自低向高排列。

## 为什么会用到ring

之前相机标定时提到过，在点云上画出四条边，拟合四条边为四条直线等等。我们运行程序时，显示的直接就是硬纸板的轮廓让你去框（见下图），怎么做到的呢？
![setup](/assets/images/post_images/rslidar_to_velodyne/calib.png)

其实就是对激光雷达同一ring的点进行遍历，检测每条线上的点如果有z方向上的突变则保存下来，其余删除。然后就得到了上图。

## robosense激光雷达数据类型

然而robosense的点云并没有ring属性，它采用的就是上图说的有序点云的方法，height=16，故没办法直接运行标定程序——它的点云中的点没有ring属性。

此外，它的线的排列顺序也和velodyne规定的不一样。见下表。
![setup](/assets/images/post_images/rslidar_to_velodyne/rslidar.png)

## rslidar数据类型向velodyne数据类型转换

为了继续用潘师兄的那个包进行标定，我写了个包将rslidar数据转换为velodyne数据，主要代码见下图：

```cpp
    pcl::PointCloud<pcl::PointXYZI> rs_pc;
    pcl::fromROSMsg(*cloud, rs_pc);
    rs_pc.height = 16;
    pcl::PointCloud<velodyne_pointcloud::PointXYZIR> velo_pc;
    for(int i = 0; i < 16; i++)
    {
        for(int j = 0; j < rs_pc.size() / 16; j++)
        {
            velodyne_pointcloud::PointXYZIR velo_point;
            pcl::PointXYZI& rs_point =  rs_pc[j + i * rs_pc.size() / 16];
            velo_point.x = rs_point.x;
            velo_point.y = rs_point.y;
            velo_point.z = rs_point.z;
            velo_point.intensity = rs_point.intensity;
            velo_point.ring = i < 8 ? i : 16 - (i - 8);
            velo_pc.push_back(velo_point);
        }
    }
```
代码详见[这里](https://github.com/EpsAvlc/rslidar_to_velodyne)。

即接受rslidar的点云，给其加ring属性，转发出去。