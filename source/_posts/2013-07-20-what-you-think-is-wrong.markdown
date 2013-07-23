---
layout: post
title: "What You Think is Wrong"
date: 2013-07-20 00:01
comments: true
categories: [Probability, Machine Learning]
---

最近看东西的时候，发现两个很有意思。之所以有意思，是因为实际的结论跟初看上去时得到的结论很不一样。

一个来自Udacity的[Introduction to Statistics](https://www.udacity.com/course/st101)；一个来自[An Introduction to Statistical Learning](http://www.amazon.com/books/dp/1461471370)，著名的ESL的兄弟篇，只是更偏向于应用。

## Gender Bias

如果有Udacity的帐户，可以在[这里](https://www.udacity.com/course/viewer#!/c-st101/l-48759015/e-48686794/m-48676955)观看。如果没有的话，youtube上也可以看到：
[1](https://www.youtube.com/watch?v=CLgVLQAEYw8)
[2](https://www.youtube.com/watch?v=f3y_weFskL4)
[3](https://www.youtube.com/watch?v=pJrwiukN3Ls)
[4](https://www.youtube.com/watch?v=o91iPvtqt78)
[5](https://www.youtube.com/watch?v=iKTYAsZLbhc)
[6](https://www.youtube.com/watch?v=rDw0TIpwJ-c)
[7](https://www.youtube.com/watch?v=-GMhV1twy6Y)
[8](https://www.youtube.com/watch?v=GD6cQhkoqS4)
[9](https://www.youtube.com/watch?v=DeWp0hnRq4g)
[10](https://www.youtube.com/watch?v=JWl8lPGhlbY)
[11](https://www.youtube.com/watch?v=55eZrE82TqA)
[12](https://www.youtube.com/watch?v=8j5hria6Rc8)
[13](https://www.youtube.com/watch?v=YkaVgZ-yFrM)
[14](https://www.youtube.com/watch?v=tPSj6_m-0_M)
[15](https://www.youtube.com/watch?v=-GMhV1twy6Y)
[16](https://www.youtube.com/watch?v=GD6cQhkoqS4)
[17](https://www.youtube.com/watch?v=4YY-hmqSz30)
[18](https://www.youtube.com/watch?v=dOa4Cl0wM0s)

该问题统计了男生和女生申请两门课程的数据，如下：

男生：申请  接受  接受率

课程A：900  450  50%

课程B：100  10  10%


女生：申请  接受  接受率

课程A：100  80  80%

课程B：900  180  26%

现在的问题是，该学校在接受学生时，有没有性别倾向？

通过以上的数据，第一感觉是学校明显更青睐女生，因为两个课程上女生的接受率都比男生要高。

但是计算一下整体接受率，男生 46%，女生 26%。结论完全相反。

课程的最后一节提到一句话，正好可以放这里：

```
I never believe in Statistics. I didn't doctor myself.
```


## 逻辑回归的系数变化

具体请参考 _An Introduction to Statistical Learning_ 的 4.3.3 和4.3.4节内容。

这两节中使用的数据集是Default，是一个逻辑回归问题。记录了信用卡用户是否延期还款(default)，其中涉及到的主要feature包括帐单金额(balance)，是否是学生(student)，收入(income)。

如果只用balance作为feature训练，则可以得到如下训练参数：

![Coefficient balance](/images/blogpng/coef-balance.png)

如果只用student，结果如下：

![Coefficient student](/images/blogpng/coef-student.png)

如果同时使用balance, income, student，结果如下：

![Coefficient Multiple](/images/blogpng/coef-multi.png)


在第二个图和第三个图中最大的变化在于student的系数发生了根本变化(符号改变)。

在第二个图中student的系数是正数，这符合常情，因为学生没有收入来源，更有可能申请分期还款。第三个图中student的系数却表达了相反的意思，学生反而更不倾向于分期还款。

用更数学的方式表达是，在没有任何信息的情况下，学生更倾向于分期(p(student_default))；但是在知道用户收入和帐单的时候(p(student_default \| income, balance))，学生不倾向于分期。

其实我想说的是，在多变量回归的时候，因为因变量间的相互作用，训练出来的系数可能已经跟直觉不一致。