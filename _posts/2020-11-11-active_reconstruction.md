---
title: "激光雷达主动三维重建的原理"
tags: SLAM
---

## 背景

换课题了，换主动三维重建，今晚刚把两个Information gain调通，不打算写代码和看论文了，故总结之。

<!--more-->

## 论文

1. Isler, Stefan, et al. "An information gain formulation for active volumetric 3D reconstruction." 2016 IEEE International Conference on Robotics and Automation (ICRA). IEEE, 2016.
2. Wang, Yiduo, Milad Ramezani, and Maurice Fallon. "Actively Mapping Industrial Structures with Information Gain-Based Planning on a Quadruped Robot." arXiv preprint arXiv:2002.09710 (2020).

其中，第一篇论文是有关双目相机的视觉重建的。第二篇论文则是用VLP-16激光雷达结合ETH的ANYmal机器人完成的一个较大范围的三维重建。

![anymal](/pics/ig_active_slam/anymal-photo1-full.jpg)

由于我主要也在做的是基于激光的三维重建，因此我主要讲讲第二篇文章。

## 什么是主动重建

主动重建就是，当负责重建的机器人的传感器在当前视角下只能获得物体的部分点云的信息的时候，能够主动地算出来它应该下一次移动并对物体重建的位置与姿态。这样不断地移动机器人的位置，使得最终能够完成重建。

## 方法

### Next-best-view的评估函数

对于某个候选位置$c$，用以下函数来评估该位置的得分。

$$
\mathcal{U}_{c}=\mathcal{G}_{c} \times\left(1-\mathcal{P}_{c}\right) \times\left(1-\mathcal{T}_{c}\right)
$$

其中:

- $\mathcal{G}_c$为信息增益。 
- $\mathcal{P}_c$为在这个位置的位置损失。如果该位置已被访问过，或者特别接近物体，这个值将会比较大。
- $\mathcal{T}_c$为路程损失，当机器人当前位置到该位置的距离比较远的时候，该值比较大。


### 信息增益的定义

第二篇文章的信息增益实际上是直接用第一篇文章的信息增益的。主要分为两个信息增益。

#### Occlusion Aware

核心思想是，如果传感器要扫描的区域的未知性越高，则其信息增益也越高。

给定一个voxel的占据概率为$P_o(v)$，则这个voxel的信息熵可以用下式得出。

$$
H(v)=-P_{o}(v) \ln P_{o}(v)-\left(1-P_{o}(v)\right) \ln \left(1-P_{o}(v)\right)
$$

那么，对于某个voxel v，其Occlusion Aware　信息增益为

$$
\mathcal{I}_{O A}(v)=P_{v}(v) H(v)
$$

其中，$P_v(v)$为该voxel的能见性，这个能见性由下式定义：
$$
P_{v}\left(v_{n}\right)=\prod_{i=0}^{n-1}\left(1-P_{o}\left(v_{i}\right)\right)
$$
这个式子中的$v_n$为沿着ray $r$所经过的第n个voxel，而$v_i，i = 0...n-1$为ray $r$到达$v_n$之前经过的voxel。

根据这个定义，举例说明：

![](/pics/ig_active_slam/OA.png)

上图为根据OA 信息增益算出来的voxel的占据概率的信息增益图。其中，黑色表示该voxel已被占据，灰色表示该voxel未知，绿色表示该voxel为free。白色为待选的传感器未知。此外，边界区域的voxel被用白色斜线标出，信息增益的大小则用蓝色三角形的颜色深浅来表示。

#### Rear Side Entropy

Rear Side Entropy被定义为：

$$
\mathcal{I}_{R S E}(v)=\left\{\begin{array}{ll}
\mathcal{I}_{O A}(v) & v \text { is a Rear Side Voxel } \\
0 & \text { otherwise. }
\end{array}\right.
$$

其中, Rear Side Voxel的定义为，在已知的占据的voxel附近的未知的voxel。

### 位置损失与路程损失的定义

位置损失（Position cost)的定义为：
$$
\mathcal{P}_{c}=\left\{\begin{array}{ll}
1-d_{\text {thres}}^{-1} \times d_{c} & d_{\text {thres}} \geq d_{c} \geq 0 \\
0 & d_{c}>d_{\text {thres}}
\end{array}\right.
$$

其中 $d_c$为当前位置到任意已经访问过的位置的最小距离。$d_thres$则为自定义的一个threshold。

路程损失的定义为：

目前的定义是，如果这个位置，机器人可以到达，则损失为0，否则为1.

### 路径规划

首先根据输入的点云建立高程图(elevation map)。然后根据高程图，利用RRT扩展获得待选的位置，利用上述的评价函数选出最佳的next best view。最后再利用RRT* 获得到nbv的最短路径。

### 收敛条件

当next best view的评价函数的分数低于一个用户所选的阈值，则收敛。

### 代码笔记

![](/pics/ig_active_slam/IG_Active_Mapping.png)