---
title: "机器人常用的三维地图表达形式"
tags: active_slam
---

介绍机器人建图中常见的三维地图表达形式。

<!--more-->

## OcTree 八叉树

> 论文：Hornung, Armin, et al. “OctoMap: An efficient probabilistic 3D mapping framework based on octrees.” Autonomous robots 34.3 (2013): 189-206.

有关八叉树我已经讲很多了，详见：
- [八叉树地图的构建](../../../2020/11/05/Octomap.html)
- [再谈octomap——一些细节](../../../2021/02/25/octomap_revisit.html)

## $N^3$ tree
> 论文：J. Chen, D. Bautembach, and S. Izadi, “Scalable real-time volumetric surface reconstruction,” ACM Trans. Graph., vol. 32, no. 4, pp. 113:1– 113:16, Jul. 2013.

$N^3$ tree的数据结构如下图所示：

![](/pics/map_representation/n3.png)

其思想为：认为空间中大部分区域为free的栅格，因此free的栅格就不用细分。而那些包含surface的栅格可以细分为Level1 与Level2的栅格。Level2的栅格为叶子栅格，保存了tsdf距离。

N3 tree不用octree的原因在于，他认为octree不容易在GPU上实现并行化。

## flat hash-tables(Voxel Hashing)

> 论文：M. Nießner, M. Zollh¨ofer, S. Izadi, and M. Stamminger, “Real-time 3D reconstruction at scale using voxel hashing,” ACMTrans. Graph., vol. 32, 2013, Art. no. 169.

- [Voxel Hashing Repo](https://github.com/niessner/VoxelHashing)
- [基于Voxel Hashing的InfiniTAM Repo](https://github.com/victorprad/InfiniTAM)

Voxel hashing主要是解决这样一个问题：当在大规模建图的时候，如果事先就给所有的voxel分配内存，则会导致内存爆炸。因此，Voxel hashing选择只保存相机观测到的voxel，将其存入Hash表中。

Voxel hashing的数据结构为：

![](/pics/map_representation/voxel_hashing.png)

具体而言，将不同的坐标的voxel，用一个hash函数映射到不同的bucket中去。该论文使用了许多技巧来解决发生哈希碰撞时如何提高效率，以及如何根据voxel hashing的地图形式来解决TSDF更新时的问题。

其最终还是类似于Kinect-fusion，TSDF更新的方式与raycast得到重建表面的方式都类似。

## SuperEight
> 论文：Vespa, Emanuele, et al. "Efficient octree-based volumetric SLAM supporting signed-distance and occupancy mapping." IEEE Robotics and Automation Letters 3.2 (2018): 1144-1151.

- [SuperEight repo](https://github.com/emanuelev/supereight)

SuperEiget的数据结构为：
![](/pics/map_representation/supereight.png)

有点类似于Voxel Hashing，不同于八叉树的是，每个Leaf node里有8x8x8个栅格。这些栅格通过莫顿编码的形式来实现快速查找与分配内存。

相较于八叉树地图，他的优势在于能够更快更方便地查找到某个Voxel的邻节点。

相较于使用TSDF表示表面的地图表征，他的优势在于，能够准确区分以下两种voxel:
- free voxel
- unknown voxel
换言之，TSDF表征的地图里，没有区分free与unknown栅格的概念。
