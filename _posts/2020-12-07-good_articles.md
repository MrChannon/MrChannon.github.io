---
title: "《一文读懂系列》汇总"
tags: 
---
## 背景

搜集了自己在读论文过程中发现的很好的科普性论文。

<!--more-->

## 文章

### 协方差矩阵的理解
[A geometric interpretation of the covariance matrix](https://www.visiondummy.com/2014/04/geometric-interpretation-covariance-matrix/)

我的协方差矩阵理解入门。之后在点云的地面提取等多个方面用到了协方差矩阵。

### 高斯过程

#### 高斯过程的理解
[Understanding Gaussian Process, the Socratic Way](https://towardsdatascience.com/understanding-gaussian-process-the-socratic-way-ba02369d804)

事无巨细的介绍了高斯过程的原理。其中我认为比较重要的一点是

> Gaussian Process does not find a function body that only needs a new x and returns a y in the traditional sense of $f(x)=ax+b$, like what linear regression gives you. Instead, it gives you a mapping between every test location $x_*$ to a function mean value (together with its variance of course). This mapping involves not only the test location x_* but also all the training data X and Y.

[A Visual Exploration of Gaussian Processes](https://www.jgoertler.com/visual-exploration-gaussian-processes/)

#### 高斯过程的微分

[stackexchange](https://stats.stackexchange.com/questions/373446/computing-gradients-via-gaussian-process-regression)

### 贝塞尔曲线与B样条

####　贝赛尔曲线
[曲线篇: 贝塞尔曲线(知乎)](https://zhuanlan.zhihu.com/p/136647181)

#### B样条
[曲线篇：深刻理解B 样条曲线（上）](https://zhuanlan.zhihu.com/p/139759835)

[曲线篇：深刻理解B 样条曲线（下）](https://zhuanlan.zhihu.com/p/140921657)

### 卡方分布

卡方分布经常用来检验SLAM问题中的数据关联的正确与否。

[ t分布, 卡方x分布，F分布 ](https://www.cnblogs.com/think-and-do/p/6509239.html)

### MPC(模型预测控制)

模型预测控制是在学习主动探索的时候碰到的。MPC即模型-预测-控制。

其主要思想为，给定一个参考未来N个时刻的预计输出，MPC有一个predicting horizon，使得未来的N个时刻都尽量与参考输出接近，同时又要满足系统的各个限制。优化出来控制变量后，仅将N个时刻中的离当前时刻最近的控制量赋值给被控机器人，随后等该控制量执行之后，再按照上述的策略继续算控制量。

[bilibili 视频](https://www.bilibili.com/video/av24625694?p=2%29)

### 软件开发相关：什么是CI/CD

[CI/CD是什么？如何理解持续集成、持续交付和持续部署](https://www.redhat.com/zh/topics/devops/what-is-ci-cd)