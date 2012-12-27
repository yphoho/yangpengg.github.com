---
layout: post
title: "Linear Regression and the Theory"
date: 2012-12-16 22:30
comments: true
categories: [Machine Learning]
---

# 问题

简单来说，有一堆实数数据，数据的格式如下：

$$
x_1, x_2, x_3, \cdots, x_n, y
$$

所有的这些数据称为训练集，其中$x$称为feature，$y$称为target。

现在又来了一个数据：

$$
x_1, x_2, x_3, \cdots, x_n
$$

现在需要做的是根据这些$x$的值，推测出$y$的值。

对这个问题更详细的描述可以看[Stanford机器学习公开课](http://v.163.com/movie/2008/1/B/O/M6SGF6VB4_M6SGHJ9BO.html)中相关描述。


# 解决方法

## Overdetermined Equations

假设$y$是$x$的线性函数（顺便说一句lr中的linear是对于$\theta$而言的，并非针对$x$），表达为公式为：

$$
y = \theta_0x_0 + \theta_1x_1 + \theta_2x_2 + \cdots + \theta_nx_n
$$

其中$x_0$为截距(intercept term)，其值恒为1。

最容易想到的方法，可以把所有训练集的数据代入这个公式，得到方程组：

$$
\begin{eqnarray} 
\left\{ 
\begin{array}{lll}
y^{(1)} = \theta_0x_0^{(1)} + \theta_1x_1^{(1)} + \theta_2x_2^{(1)} + \cdots + \theta_nx_n^{(1)} \\
y^{(2)} = \theta_0x_0^{(2)} + \theta_1x_1^{(2)} + \theta_2x_2^{(2)} + \cdots + \theta_nx_n^{(2)} \\
\vdots \\
y^{(m)} = \theta_0x_0^{(m)} + \theta_1x_1^{(m)} + \theta_2x_2^{(m)} + \cdots + \theta_nx_n^{(m)}
\end{array} 
\right. 
\end{eqnarray}
$$

这个方程组有m个方程，n+1个未知数，实际问题中通常是训练集的个数大于feature个数，也就是说m > n+1，这种情况下的方程组称为[超定方程组](http://en.wikipedia.org/wiki/Overdetermined_system)，是不能直接求解的。当然可以像当年欧拉和拉普拉斯最初解决天文计算问题一样([here](http://www.52nlp.cn/%E6%AD%A3%E6%80%81%E5%88%86%E5%B8%83%E7%9A%84%E5%89%8D%E4%B8%96%E4%BB%8A%E7%94%9F%E4%BA%8C))，把m个方程组分成n+1组，然后每一组合并成一个方程，得到n+1个方程后再求解。不过问题是怎么分成n+1组，这个很是adhoc的。

## Cost Function

机器学习上解决这个问题的方法是定义一个损失函数：

$$
J(\theta) = \frac{1}{2}\sum_{i=1}^m(h_\theta(x^{(i)}) - y^{(i)})^2
$$

然后选择适当的$\theta$，使得$J(\theta)$最小。

## Gradient Descent

这个最小化的算法在机器学习中称为梯度下降：

* 随机初始化一组$\theta$值；
* 朝着减少cost function的方向，不断更新$\theta$值，直到收敛。更新公式为：

$$
\theta_j := \theta_j - \alpha\frac{\partial J(\theta)}{\partial \theta_j}
$$

其中$\alpha$为学习速率(learning rate)。

## Gradient Descent推导

假设训练集中只有一个数据，$\frac{\partial J(\theta)}{\partial \theta_j}$计算如下：

$$
\begin{align*}
\frac{\partial J(\theta)}{\partial \theta_j} &= \frac{\partial{(\frac{1}{2}(h_\theta(x) - y)^2)}}{\partial \theta_j} \\
&= 2 * \frac{1}{2}(h_\theta(x) - y) * \frac{\partial{(h_\theta(x) - y)}}{\partial \theta_j} \\
&= (h_\theta(x) - y) * \frac{\partial(h_\theta(x) - y)}{\partial \theta_j} \\
&= (h_\theta(x) - y) * \frac{\partial(\sum_{i=0}^n\theta_ix_i - y)}{\partial \theta_j} \\
&= (h_\theta(x) - y)x_j
\end{align*}
$$

代入更新公式：

$$
\begin{align*}
\theta_j &= \theta_j - \alpha(h_\theta(x) - y)x_j \\
&= \theta_j + \alpha(y - h_\theta(x))x_j
\end{align*}
$$

对于有m个数据集的情况可以得到如下公式：

$$
\begin{align*}
\theta_j := \theta_j + \alpha\sum_{i=1}^m(y^{(i)} - h_\theta(x^{(i)}))x_j^{(i)}
\end{align*}
$$

## Gradient Descent直观解释

$J(\theta)$是一个关于$\theta$的多元函数，高等数学的知识说，$J(\theta)$在点$$P(\theta_0, \theta_1, \cdots, \theta_n)$$延梯度方向上升最快。现在要最小化 $J(\theta)$，为了让$J(\theta)$尽快收敛，就在更新$\theta$时减去其在P点的梯度。

在最终推导出的更新公式中，可以得出以下直观结论：如果遇到一个数据使得$(y - h_\theta(x))$比较小，这时候$\theta$的更新也会很小，这也符合直观感觉。当一个数据使得差值比较大时，$\theta$的更新也会比较大。

## Stochastic Gradient Descent

以上的讨论的算法叫batch gradient descent，batch指的是，每次更新$\theta$的时候都需要所有的数据集。这个算法有两个缺陷：

* 数据集很大时，训练过程计算量太大；
* 需要得到所有的数据才能开始训练；

比如一个场景下，我们训练了一个lr模型，应用于线上环境，当这个模型跑在线上的时候我们会收集更多的数据。但是上面两个问题使得我们不能及时更新模型，而这正是随机梯度下降要解决的问题。

在之前的推导过程中已经给出了sgd的更新公式，只是没有指出，现正式提出sgd的更新公式：

loop for every (x, y) in training set until convergence:

$$
\theta_j := \theta_j + \alpha(y - h_\theta(x))x_j
$$

与bgd唯一的区别是，无论数据集有多少，每次迭代都只用一个数据。这样当有新的数据时，直接通过上式更新$\theta$，这就是所谓的online learning。又因为每次更新都只用到一个数据，所以可以显著减少计算量。

# sgd的Python实现

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import re
import numpy as np
                                                                       
                                                                       
def load_data(filename):                                                
    feature = []                                                        
    target = []                                                        
    f = open(filename, 'rb')                                            
    for line in f.readlines():                                          
        sample = re.split(' +', line.strip())                          
        feature.append([1, ] + sample[0:-1])                             
        target.append(sample[-1])                                      
                                                                       
    return np.array(feature, np.float), np.array(target, np.float)       
                                                                       
                                                                       
def sgd(feature, target, iter=200, step=0.001):                       
    theta = np.zeros(feature.shape[1])                                  
    # theta = np.ones(feature.shape[1])                                
                                                                       
    for it in range(iter):                                              
        for i in range(feature.shape[0]):                             
            error = target[i] - sum(theta * feature[i])                
            theta = theta + step * error * feature[i]                  
                                                                       
        predict = [sum(theta * sample) for sample in feature]            
        mse = sum((target - predict) ** 2) / feature.shape[0]            
        print it, 'mse:', mse
                                                                       
    return theta                                                        
                                                                       
                                                                       
def normalize(feature):                                                
    mu = feature.mean(0)                                                
    std = feature.std(0)                                                
                                                                       
    for j in range(1, feature.shape[1]):                                
        feature[:, j] = (feature[:, j] - mu[j]) / std[j]                 
                                                                       
    return feature, mu, std                                            
                                                                       
                                                                       
if __name__ == '__main__':                                              
    datafile = 'housing_data'
    x, y = load_data(datafile)
    x, mu, std = normalize(x)
    theta = sgd(x, y)
    print theta
```

代码中使用的数据集可以从[这里](http://amachinelearningtutorial.googlecode.com/files/housing_data)下载，描述在[这里](http://amachinelearningtutorial.googlecode.com/files/housing_description)。

代码中normalize函数用于对feature进行归一化处理，可以尝试一下去掉normalize过程，对于这个数据集会得出很出乎意料的结果。


# 概率解释

在以上的讨论中，得出$y$与$x$的关系是线性假设，使用梯度下降也可以从高数中得到依据，唯有损失函数好像是拍脑袋想出来的。有那么多的函数可以用，为什么单选择了一个二次式做为损失函数。其实这里选择二次函数是有其理论基础的。

$y$与$x$满足以下公式：

$$
y^{(i)} = \theta^{T}x^{(i)} + \varepsilon^{(i)}
$$

其中$\varepsilon^{(i)}$称为误差，可能由两个原因产生：

* feature选择的不合适；
* 随机噪声；

又假设$\varepsilon^{(i)}$独立同分布，且满足均值为0，方差为$\sigma^2$的高斯分布，即：

$$
p(\varepsilon^{(i)}) = \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(\varepsilon^{(i)})^2}{2\sigma^2}}
$$

也就是：

$$
p(y^{(i)} | x^{(i)}; \theta) = \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(y^{(i)} - \theta^Tx^{(i)})^2}{2\sigma^2}}
$$

以上是一个关于$y$, $X$的公式，可以定义一个似然函数，形式如同上式，但是似然函数是关于$\theta$的公式：

$$
L(\theta) = L(\theta; X,y) = p(y|X; \theta)
$$

根据之前$\varepsilon^{(i)}$的独立性假设，$L(\theta)$可以记做

$$
\begin{align*}
L(\theta) &= \prod_{i=1}^m{p(y^{(i)} | x^{(i)}; \theta)} \\
&= \prod_{i=1}^m \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(y^{(i)} - \theta^Tx^{(i)})^2}{2\sigma^2}}
\end{align*}
$$

现在已经观察到了很多数据($x$, $y$)，那么什么样的模型才能让这些数据出现的可能性最大。这就是最大似然估计的出发点，也就是求解$\theta$以最大化这些数据出现的概率，即最大化似然函数$L(\theta)$。

关于最大似然估计方法更多解释可以看[这里](http://www.cchere.com/article/1522559)。

当然更多时候最大化的是$logL(\theta)$，而不是直接最大化$L(\theta)$，因为log函数单调递增函数，所以这个转化不会影响$\theta$的最终取值。

$$
\begin{align*}
l(\theta) &= logL(\theta) \\
&= log\prod_{i=1}^m \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(y^{(i)} - \theta^Tx^{(i)})^2}{2\sigma^2}} \\
&= \sum_{i=1}^m log \frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(y^{(i)} - \theta^Tx^{(i)})^2}{2\sigma^2}} \\
&= mlog\frac{1}{\sqrt{2\pi}\sigma} - \frac{1}{\sigma^2}\frac{1}{2}\sum_{i=1}^m(y^{(i)} - \theta^Tx^{(i)})^2
\end{align*}
$$

因此最大化$l(\theta)$也就是最小化：

$$
\frac{1}{2}\sum_{i=1}^m(y^{(i)} - \theta^Tx^{(i)})^2
$$

也就是之前出现的$J(\theta)$。

至此，我们从概率和最大似然估计的角度解释了$J(\theta)$选择这个二次式是合理的。
