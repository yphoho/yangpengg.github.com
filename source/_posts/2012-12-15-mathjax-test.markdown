---
layout: post
title: "MathJax Test"
date: 2012-12-15 23:12
comments: true
categories: [Octopress, MathJax]
---

# Instruction

[Here](http://www.idryman.org/blog/2012/03/10/writing-math-equations-on-octopress/) is a good instructions to set the MathJax for Octopress.

There are two things to point:

  * 'gem install kramdown' is not needed, cause of the installed when setup Octopress;
  * I have not found something useful about 'change Gemfile to kramdown';


# Latex Test

## Gaussian Function

$$
f(x; \mu, \sigma^2) = \frac{1}{\sigma\sqrt{2\pi}}e^{-\frac{1}{2}(\frac{x-\mu}{\sigma})^2}
$$

## Equation Used in Gradient Descent 

$$
h_\theta(x) = \sum_{i=0}^n\theta_ix_i = \Theta^Tx
$$

$$
J(\theta) = \frac{1}{2}\sum_{i=1}^m(h_\theta(x^i) - y^i)^2
$$

$$
\theta_j := \theta_j + \alpha\sum_{i=1}^m(y^i - h_\theta(x^i))x_j^i
$$

## A Long Equation

$$
\begin{align*}
  & \phi(x,y) = \phi \left(\sum_{i=1}^n x_ie_i, \sum_{j=1}^n y_je_j \right)
  = \sum_{i=1}^n \sum_{j=1}^n x_i y_j \phi(e_i, e_j) = \\
  & (x_1, \ldots, x_n) \left( \begin{array}{ccc}
      \phi(e_1, e_1) & \cdots & \phi(e_1, e_n) \\
      \vdots & \ddots & \vdots \\
      \phi(e_n, e_1) & \cdots & \phi(e_n, e_n)
    \end{array} \right)
  \left( \begin{array}{c}
      y_1 \\
      \vdots \\
      y_n
    \end{array} \right)
\end{align*}
$$
