---
title: "c++的反射"
tags: c++
---

## 背景
c++ 如何实现反射？

<!--more-->

## 什么是反射？
我所理解的反射，即根据类的字符串名，可以生成类的实例。

## 为什么要用反射
在写自己的代码的时候，希望能够复用代码，举个例子，针对主动探索而言：

在获取候选的view的位置的时候，需要不同的view space finder，在评估某个view的信息增益时，需要考虑不同的传感器的光线投射模型的区别……

如果将这些东西硬编码到代码里，那么想要更换一种view space finder或者一个传感器模型的时候，就需要重新修改代码并且重新编译，浪费时间且可能造成前后代码不一致，导致不同方法间的比较会出现问题。

而如果有反射机制，则只需要在yaml文件里写好运行时所需要的传感器类型或者view space finder，代码可以自动生成对应的实例。

Java或者c#是有反射机制的：
> (JAVA)编译器会登记类型的一切信息，包括全部字段的详细信息，全部方法的详细信息，全部接口的详细信息等，不管猿猴是否需要，就算是临时辅助类，甚至是虚拟机生成的匿名类，完全压根就看不到其反射信息在代码上的作用，都会一一备案。

而c++没有，但是我们可以通过一些方法去实现。

## 反射的实现方式
### 工厂函数+反射器

我参考了几篇CSDN的文章：
- [C++ 实现反射机制](https://blog.csdn.net/q1007729991/article/details/56012253) 

其中，GCC编译器似乎不支持写关于模板类的宏。详见[Concatenation](https://gcc.gnu.org/onlinedocs/gcc-4.3.3/cpp/Concatenation.html#Concatenation)
>However, two tokens that don't together form a valid token cannot be pasted together.

而在实现函数的时候，需要在类名后连接模板的类型名，类似于
```cpp
    virtual base_name ##<NODE_TYPE>* getInstance()
```

会提示
> pasting "base_nameFactory" and "<" does not give a valid preprocessing token
这表示因为base_name< 不能够形成一个有效的c++ token名，因此GCC编译器是不能通过的。(但是MSVC可以)

### C++14及以上的方法

在编译期间实现反射。
涉及到许多奇技淫巧。可能相关的repo为：

[ bytemaster/boost_reflect ](https://link.zhihu.com/?target=https%3A//github.com/bytemaster/boost_reflect)

目前而言我暂时还没有用这些库的需求，暂时就用最简单的工厂类反射