---
title: "贝叶斯估计、最大似然估计、最大后验概率与Baysian Matting"
tags: computer-vision math
---

## 背景

做cv project，题目是绿幕抠图，搜索到了baysian matting。于是便整理了一些有关贝叶斯公式、最大似然估计的知识，并简单介绍了Baysian Matting的思想。

<!--more-->

## 概念

### Likelihood(似然)

似然性与概率相近，都是指某种事件发生的可能性，但是在统计学中这两者又有明确的区分：

* **概率**用于在已知一些参数的情况下，预测接下来的观测所得到的结果。即已知参数B，求$P(A\mid B)$。
* **似然性**则是用于在已知某些观测所得到的结果时，对有关事物的性质的参数进行估计。即已知$P(A)$，根据贝叶斯定理$P(B \mid A)=\frac{P(A \mid B)P(B)}{P(A)}$，估计参数B的可能性。

在这种意义上，似然函数可以理解为条件概率的逆反。

### Likelihood function(似然函数)

似然函数是指这样一种函数，该函数的变量的概率密度取决于于某个可以观测的参数（先验知识）。写作(对于离散变量而言)：

$$ L(\theta \mid x) = p_\theta(x)=P_\theta(X=x)$$

其中$\theta$是先验知识，而$x$是随机变量**已取到的值**。$p_\theta(x)$表示已知某个参数$\theta$的情况下，$x$的概率密度。

虽然从式子上，似然函数和概率密度函数是相等的，但是它们是两个完全不同的**数学对象**。前者是关于$\theta$的函数，而后者是关于$x$的函数。所以理解这两个函数相等时，应理解为**函数值相等**。

那么如果有：

$$L(\theta_1 \mid x) = P_{\theta_1}(X=x) > P_{\theta_2}(X=x) = L(\theta_2 \mid x)$$

那么似然函数就反应出这样一个朴素推测：在参数$\theta_1$下随机向量$\textbf{X}$取到值$\textbf{x}$的可能性大于 在参数$\theta_2$下随机向量$\textbf{X}$取到值$\textbf{x}$的可能性。换句话说，我们更有理由相信，相对于$\theta_2$来说,$\theta_1$更有可能是真实值。这里的可能性由概率来刻画。

**Tips**: 在很多场合中竖线\mid符号表示条件概率或者条件分布，所以实际上似然函数的严格书写方式应该是$L(\theta \mid \textbf{x}) = f(\textbf{x} ; \theta)$，因为分号表示右侧的$\theta$只当做参数来理解。

### Maximum Likelihood Estimation-MLE(最大似然估计)

MLE是指给出上式观测值x，求使似然函数达到最大的$\theta$的值。即：

$$ \mathop{\arg\max}_{\theta}  L(\theta \mid x)$$ 

最大似然估计是似然函数最初也是最自然的应用。上文已经提到，似然函数取得最大值表示相应的参数能够使得统计模型最为合理。从这样一个想法出发，最大似然估计的做法是：首先选取似然函数（一般是概率密度函数或概率质量函数），整理之后求最大值。

实际应用中一般会取似然函数的对数作为求最大值的函数，这样求出的最大值和直接求最大值得到的结果是相同的。

### Maximum A Posteriori(最大后验概率)

最大似然估计属于统计学派。根据上面的表述，假设一枚硬币抛5次，5次全为正面，根据最大似然估计，抛掷该硬币正面朝上的概率就应该为1。但是这与生活中的经验不符，如果将生活中的经验（先验知识）代入，则引出了**最大后验概率**。

与统计学派不同，贝叶斯学派认为在做估计之前，人们对要估计的实物先有一个经验性的判断，然后根据数据调整对这个实物的判断。而这个经验性的判断就是先验概率，而经过调整之后的概率称作后验概率。最大后验估计的基础是贝叶斯公式：
$$P(\theta \mid x) = \frac{P(x \mid \theta)\times P(\theta)}{P(x)}$$
其中，$P(x \mid \theta)$为之前提到的似然函数，$P(\theta)$是先验概率。

求解MAP的一般推导步骤为：

$$
\begin{align*} 
\hat{\theta}_\text{MAP} &= \arg \max P(\theta | X) \\ &= \arg \min -\log P(\theta | X) \\ & = \arg \min -\log P(X|\theta) - \log P(\theta) + \log P(X) \\ &= \arg \min -\log P(X|\theta ) - \log P(\theta) 
\end{align*}
$$

由上式可见，MAP相比MLE多了一项$-\log P(\theta)$即加入的先验知识，也即是这一项使得对于抛硬币问题，给定先验知识$\theta$为正面概率为0.5,即使五次均正面朝上，算出的MAP概率$P(\theta \mid x)$也应该小于1而大于0.5。

19/2/13新增：可以把$P(\theta)$项想象成机器学习中的正则项。机器学习中引入正则是为了不让模型过于复杂，而这里引入该项是为了不让“经验主义”获胜。当然，$P(\theta)$一般由测量仪器的参数决定概率分布。

## Bayesian Matting (贝叶斯抠图)

论文原址在参考中。

### 关于Digital Matting

Digital Matting描述的是这样的一类问题：

1. 给定某张图片C（如图）。
   
   ![teddy]({{site.baseurl}}/assets/images/post_images/2018-12-22/teddy.jpg)
2. 给定先验知识。
   
   以trimap为例。下图为一张trimap。

   ![trimap]({{site.baseurl}}/assets/images/post_images/2018-12-22/trimapT.png)

   trimap图为一张单通道，且像素值范围为0-1的double型图像。其中像素值为1(也就是图像中白色的部分)表示图片C中对应位置的像素为前景，像素值为0(也就是图像中黑色的部分)表示图片C中对应位置的像素为背景。像素值在0-1之间（也就是图像中灰色的部分）表示尚未确认是前景与背景的部分。
3. Digital Matting的任务就是将上文所述的未确定部分，通过某种方法确定为前景或者背景，从而得到以下的mask图片。
   
    ![alpha]({{site.baseurl}}/assets/images/post_images/2018-12-22/alpha.png)

另外，在Digital Matting中有一个经典的**compositing equation**:

$$C=\alpha F + (1-\alpha)B$$

意思是一幅图像$C$是由前景$F$与背景$B$按照一定比例$\alpha$组成的。

这个式子可以看做是针对整个图像而言，此时，$C, F, B$都是尺寸相同，通道数相同的图像，而$\alpha$是尺寸与上者相同，但通道数为1的，元素值为0～1的矩阵。

该式也可以看做是针对图像的一个特定像素而言。此时$C,F,B$都是单个像素，而$\alpha$为单个double型的值。

### 利用MAP构建Matting问题

问题可以转化为式子：
$$\arg\max_{F,B,\alpha}P(F, B, \alpha \mid C)$$
运用贝叶斯公式有：
$$\arg\max_{F,B,\alpha}P(F, B, \alpha \mid C)\\
     = \arg \max_{F, B, \alpha}P(C \mid F, B, \alpha)P(F)P(B)P(\alpha)/P(C)\\
     = \arg \max_{F, B, \alpha}L(C \mid F, B, \alpha) + L(F) + L(B) + L(\alpha) $$
其中$L(.)$表示log likelihood $L(.)=logP(.)$。那么下一步就应该是去研究式子$L(C \mid F, B, \alpha)$，$L(F)$与$L(B)$在Matting问题中的具体定义（$L(\alpha)$由于与$F, B, \alpha$无关，所以在式子中可以舍去）。

### MAP中各式在Matting问题中的定义

1. $L(C \mid F, B, \alpha)$
   
   假设$C$满足均值为 $\overline{C} = αF + (1 − α)B$ 的高斯分布，则有
   $$L(C \mid F, B, \alpha) = -\mid\mid C-\alpha F - (1-\alpha)B\mid \mid ^ 2 / \sigma^2_C$$ 
   
   其中$\sigma ^2_C$为$C$的标准差
2. $L(F)$与$L(B)$
   
   以$F$为例
   $$L(F) = −(F − \overline F)^T \Sigma_F^{-1}(F − \overline F) / 2$$
   其中$\Sigma_F$为前景F的某个像素在一定邻域内的协方差矩阵。

### 求解过程

由于将上面得到的$L(C \mid F, B, \alpha)$，$L(F)$与$L(B)$代入后，该问题不是一个**quadratic equation**(二次方程)，因此要将问题分布解决。先认为$\alpha$为常数，对$F,B$求偏导。之后认为$F,B$为常数，再对$\alpha$求偏导。通过多次迭代后得到的值为较为准确的值。

1. 遍历trimap中每个像素，遇到不为0或者1的alpha则进入步骤2。
2. 用3*3的邻域的alpha的均值初始化当前像素alpha。根据论文中的对$F,B$求偏导等于0后的方程求解出$F,b$，再求出$alpha$
3. 重复2的步骤直到达到迭代次数或者$\alpha$变化与上次迭代差距不大。
4. 3步骤执行完后该像素算求解完成，继续找下一个像素。

### C++实现

我用Eigen + OpenCV实现了Bayesian Matting的算法，具体地址在参考中。目前来说我的实现待改进的地方有：

1. 求$F,B$的协方差矩阵时没有用当前像素的一定邻域内的前景与背景的像素算协方差，而是算了整幅图的前景与背景的协方差。这样会导致，抠图时万一背景不是处处都相似，会导致抠图效果变差。
2. 算法运算时间太慢。没有优化导致抠一幅图约需要6s。

## 参考

1. [wikipedia--似然函数](https://zh.wikipedia.org/wiki/%E4%BC%BC%E7%84%B6%E5%87%BD%E6%95%B0)
2. [知乎--如何理解似然函数](https://www.zhihu.com/question/54082000)
3. [知乎--最大后验估计](https://zhuanlan.zhihu.com/p/32616870)
4. [A Bayesian Approach to Digital Matting](https://grail.cs.washington.edu/projects/digital-matting/papers/cvpr2001.pdf)
5. [Bayesian Matting 的 C++ 实现](https://github.com/EpsAvlc/GreenScreenMatting)
