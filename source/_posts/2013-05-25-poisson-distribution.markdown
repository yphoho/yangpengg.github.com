---
layout: post
title: "Poisson Distribution"
date: 2013-05-25 22:42
comments: true
categories: [Probability]
---

## 泊松分布

泊松分布是一个离散分布，其 pmf 如下：

$$
Pr(X = k) = \frac{\lambda^{k}e^{-\lambda}}{k!}
$$

其中参数 $\lambda$ 为分布的均值。

泊松分布描述的是：

* 事件发生次数的均值已知；

* 两次事件是否发生相互独立；

满足以上条件下，事件发生次数的概率。

比如，在商店里某种商品平均一天被购买 $\lambda$ 次，则该商品在一天中被购买 $k$ 次的概率就满足泊松分布。


## 泊松分布的由来

以下推导主要参考可汗学院的公开课\<统计学\>，可以在[这里](http://v.163.com/movie/2011/6/G/H/M82IC6GQU_M83JASUGH.html)观看相关视频。

大学概率与数理统计的课程讲到泊松分布的时候都是直接给出泊松分布的概率分布函数，很少会讲泊松分布的推导过程，其实泊松分布可以由二项分布推导而来。

为了方便推导，假设对如下过程进行建模：

求某个十字路口，在一小时内通过车的数量的概率。

为了对此建模，首先统计该路口一小时内平均通过的车辆，记为 $\lambda$。假设每小时通过车的数量独立同分布，并且每辆车通过也相互独立。

假如不知道泊松分布，只知道二项分布，则可以用二项分布建模，如下：

### 第一次尝试

把一小时分为60份，按照二项分布，$n = 60$，$E(X) = \lambda$ ，则在一分钟是否有车通过的概率：

$$
p = \frac{\lambda}{60}
$$

那么一小时内通过 $k$ 辆车的概率为：

$$
P(X = k) = (^{60} _k)p^k(1-p)^{60-k}
$$

以上的建模方法有一个明显的问题，就是当一分钟通过的车辆不止一辆时，以上方法失效。因为二项分布的基本事件只能表达两种状态，发生和未发生，不能表达发生多少次。

### 第二次尝试

当然可以通过再细分时间间隔的方法解决，比如把一小时分为3600份，则基本事件表示一秒内是否有车通过，概率公式如下：

$$
P(X = k) = (^{3600} _k)p^k(1-p)^{3600-k}
$$

此时

$$
p = \frac{\lambda}{3600}
$$

以上的过程依然存在问题，假如一秒内有不止一辆车通过，则以上方法又失效了。

### 两次尝试之后

通过以上两次尝试，可以发现，我们实际需要的是 $n$ 趋于无穷时，$X = k$ 的概率。

当 $n\to\infty$，由二项分布推导泊松分布如下：

$$
\begin{align*}
P(X = k) &= \lim\limits_{n\to\infty}(^n_k)(\frac{\lambda}{n})^{k}(1-\frac{\lambda}{n})^{n-k} \\
&= \lim\limits_{n\to\infty}\frac{n!}{(n-k)!k!}\frac{\lambda^k}{n^k}(1-\frac{\lambda}{n})^n(1-\frac{\lambda}{n})^{-k} \\
&= \lim\limits_{n\to\infty}\frac{n(n-1)(n-2)\cdots(n-k+1)}{n^k}\frac{\lambda^k}{k!}(1-\frac{\lambda}{n})^n(1-\frac{\lambda}{n})^{-k} \\
&= \lim\limits_{n\to\infty}\frac{n(n-1)(n-2)\cdots(n-k+1)}{n^k}\times\lim\limits_{n\to\infty}\frac{\lambda^k}{k!}\times\lim\limits_{n\to\infty}(1-\frac{\lambda}{n})^n\times\lim\limits_{n\to\infty}(1-\frac{\lambda}{n})^{-k} \\
&= 1\times\frac{\lambda^k}{k!}\times e^{-\lambda}\times1 \\
&= \frac{\lambda^{k}e^{-\lambda}}{k!}
\end{align*}
$$


## 泊松分布的应用

关于应用，阮一峰的博客有一篇很好的文章：[泊松分布与美国枪击案](http://www.ruanyifeng.com/blog/2013/01/poisson_distribution.html)。