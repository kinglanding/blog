---
layout: post
title: "Machine Learning :linear regression"
date: 2014-01-05 12:00
comments: true
categories: C++ Armadillo Machine-Learning octave linear-regression
---

机器学习中，总体来说是分为两类问题：

1.有监督的学习方法
2.无监督的学习方法

其他是这两者的综合，比如说半监督学习方法，强化学习（这个还未接触过）。

本文呢，先从有监督的学习方法开始讲起，主要是记载学习过程中个人认为最重要的地方。

对于监督学习中的两类问题，或者说三类吧，分别是：回归问题，分类问题和标注问题（tagging）。后面这个很有意思，不过在这里现说一下回归和分类的区别，假如我们要
做一个连续变量的预测，比如说房价的预测，或者明日气温的预测，都是属于回归问题；而对于离散变量的预测，比如判断一个病人是否得了癌症，良性还是恶心，则是一个明显的分
类问题。

接下来的文章，大概是对Andrew Ng视频的一个简单的总结，会结合变成实例（octave和C++）来插叙。
<!--more-->

###线性回归

好吧，先从一个简单的例子讲起，假设我们要为一个房子售价做个数学模型，价格和什么有关系？当然因素很多，比如房间的大小，离商业区的距离，嗯，房子几坪，奥，看起来不是
个简单事儿～，那好吧，遵循我们先从最简单做起的原则，现假设相同尺寸的房子价格和城市人口多少有关系，其他的先抛到一边去，我喜欢做甩手掌柜==
，你看这很合理！北京上海的房子价格能和三四线城市的比么=。=

那么好，我们会看到下面这个图！图先不上！！！假设你装了octave，并执行ex1的话就会看到它的！！

![价格-人口关系图](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2014-01-05-machine-learning-linear-regression/linear_regression_f1.png?raw=true "价格-人口关系图")

在那之前，先让我们约定几个问题，恩恩：

#####注释
* $m$ ：是训练实例的个数
* $x$ ：是输入的特征向量,很有可能是这样子：$x=(x_1,...,x_k)$
* $y$ ：是输出结果
* $(x,y)$ ：是一个训练实例
* $(x^{(i)},y^{(i)})$ ：表示第i个训练样例

好了，让我们接着开始吧。那我们应该如何表示我们的假设（hypothesis）呢？既然只有一个变量，这样表示好了：

$$
h_\theta(x) = \theta_0 + \theta_1x_1 ， \Theta={ (\theta_0,\theta_1) }
$$

那应该如何选择参数$\theta$呢？机器学习不就是干这活的么=。=

直观的感受就是：“嘿，干嘛不用LMS最小二乘法？无脑流，简单又实惠！统计课上的入门案例。。"就他了。。。

所以，总结如下：

假设：

$$
h_\theta(x) = \theta_0 x_0 + \theta_1 x_1 ， 
\Theta = \left( \begin{array}{c}
        \theta_0 \\
        \theta_1
        \end{array} \right), 
x = (x_0,x_1),
x_0 \equiv 1 \\
h_\theta(x) = x \cdot \Theta
$$

费用函数：

$$
J{(\Theta)}=\frac{1}{2m} \sum\limits_{i=1}^m \left(h_\theta(x^{(i)})-y^{(i)}
\right)^2 
$$

目标：

$$
\min\limits_{\Theta} J{(\Theta)}
$$

回想下我们学过的数学知识吧，给定一个函数，求函数的最值，导数？梯度？那一套东西想起来了吧，OK。那好办了。要是还不是很清楚，那看一下[梯度](http://zh
.wikipedia.org/wiki/%E6%A2%AF%E5%BA%A6)以及[梯度下降法](http://zh.wikipedia.org/wiki/%
E6%A2%AF%E5%BA%A6%E4%B8%8B%E9%99%8D%E6%B3%95)在此就不罗嗦了。Andrew
Ng在视频中讲的很形象，只要我们沿着山最陡的方向向下走，就会有可能找到最小值，翻译成数学语言就是沿着梯度相反的方向$- \nabla F(x)$,
就可以下降最快。（我们不是要找最小值么，当然是水往低处流！所以就是负值了）

####梯度下降法

选定了回归模型，那就要确定参数$\Theta$了，$\Theta$只有在$J{(\Theta)}
$最小的情况下才能确定，所以问题归结为了求极小值的问题，梯度下降法是个不错的选择。当然，它会遇到找到的值只是个局部最小值。

这是示意图：

![最小值](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2014-01-05-machine-learning-linear-regression/linear_regression_f4.png?raw=true)

![局部极小值](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2014-01-05-machine-learning-linear-regression/linear_regression_f5.png?raw=true)

流程如下：

1. 对$\Theta$赋予初始值，可随机，可为零向量。
2. 同步改变$\Theta$值，使得$J{(\Theta)}$沿着梯度下降的方向走，直到学习曲线平滑，也就是收敛。

用公式来描述就是,对于$j=1$和$j=0$，同时重复以下操作，直到$J{(\Theta)}$收敛。


$$
\theta_j := \theta_j - \alpha \frac{\partial}{\partial \theta_j}
J{(\theta_0,\theta_1) } \\
\theta_j := \theta_j - \alpha \frac{1}{m} \sum\limits_{i=1}^m 
\left(h_\theta(x^{(i)})-y^{(i)}
\right) \cdot x_j^{(i)}
$$

这是octave实现，向量形式,代码[详见](https://github.com/aluenkinglee/mlclass/blob/master/
mlclass-ex1/gradientDescent.m)

```octave
function [theta, J_history] = gradientDescent(X, y, theta, alpha, num_iters)
%GRADIENTDESCENT Performs gradient descent to learn theta
%   theta = GRADIENTDESENT(X, y, theta, alpha, num_iters) updates theta by 
%   taking num_iters gradient steps with learning rate alpha

% Initialize some useful values
m = length(y); % number of training examples
J_history = zeros(num_iters, 1);
for iter = 1:num_iters
    %theta1 = theta(1) - alpha * X(:,1)' *(X * theta - y) / m;
    %theta2 = theta(2) - alpha * X(:,2)' *(X * theta - y) / m;
    %theta = [theta1; theta2]
    theta = theta - alpha / m * (X' * (X * theta - y));
    % Save the cost J in every iteration    
    J_history(iter) = computeCost(X, y, theta);
end
end
```

对应的C++实现，向量形式，代码[详见](https://github.com/aluenkinglee/mlclass/blob/master/mlclass
-ex1/gradientDescent.cpp)

```cpp
#include "gradientDescent.h"
#include "computeCost.h"
using namespace mlclass::ex1;
namespace mlclass{
namespace ex1{
    //Performs gradient descent to learn theta
    mat gradientDescent(mat X, vec y, mat& theta, double alpah,long num_inters){
        //number of training examples
        long m = y.n_rows;
        
        mat J_history = zeros<mat>(num_inters,1);
        long i = 0;
        for (;i < num_inters; i++){
            theta = theta - alpah/m* (X.t()* (X*theta - y));
            J_history(i) = computeCost(X, y, theta);    
        }
        return J_history;
    }
}
}
```

**有一个事情需要说明一下**

梯度下降发的收敛速度比较慢，相比于直接用公式求解$\theta$来说，尤其是当m较小的时候，比如说$m<10000$,
这个时候用公式求解$\theta$比较快，但是大于这个值之后，计算矩阵的逆是花费较大的，此时使用梯度下降法比较理想，而且可以做到分布式计算值，加快求解速度。

$$
\Theta=(X^TX)^-1X^Ty
$$

关于线性回归就先到这，接下来会记述关于logistic回归等的文章。

> reference

1.[Machine Learning by Andrew Ng(1-2)](https://class.coursera.org/ml-004/lecture)

2.[常用数学符号的 LaTeX 表示方法](http://mohu.org/info/symbols/symbols.htm)