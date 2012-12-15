---
layout: post
title: "Minhash and LSH"
date: 2012-09-05 09:05
comments: true
categories: [Algorithm, Recommender System]
---

本文主要参考[Mining of Massive Datasets](http://i.stanford.edu/~ullman/mmds.html)中第三章相关内容。

# Jaccard Similarity

在推荐系统经常要做相似度计算，其中比较简单的一种是[Jaccard Similarity](http://en.wikipedia.org/wiki/Jaccard_index)。Jaccard Similarity描述如下：  
设A,B为两个特征向量，比如为两个user投过票的电影，则  

$$
J(A, B) = \frac{|A \cap B|}{|A \cup B|}
$$

J(A,B)的值介于0-1之间，数值越大，相似度越高。

Jaccard相似度计算很简单，但是当特征向量维数很高的时候(比如文档的特征向量经常会上万维)，这个计算也会很耗时，这时通用的方法是降维，Minhash也可以算作针对Jaccard相似度的降维方法。

# Minhash

Minhash思想是用一个维度比较小的*signatures*代替原来维度很大的特征向量，而且可以通过计算这个signatures向量Jaccard相似度去估计原来特征向量的Jaccard相似度。

## 特征向量的矩阵表示

如S1 = {a, d}, S2 = {c}, S3 = {b, d, e}, S4 = {a, c, d}，则表示为矩阵：

![Matrix Representation](/images/blogpng/matrix-representation.png)

以电商为例，这个矩阵可以解释为：用户S1购买过a,d两个商品，S2购买过c，S3购买过b,d,e，...

## Minhash计算原理

为了推荐物品给S1，思路可以是计算出和S1相似的用户，查看这个(些)用户的购买记录，推荐其中S1没有购买过的商品。可以使用Jaccard相似度计算S1的相似用户，但是现在用户的特征向量是5维的，是不是可以使用更少的维度就可以得到Jaccard相似度。

取Element的一个全排列，以beadc为例，这个排列定义了一个hash函数，这个函数的矩阵可以表示：

![a permutation](/images/blogpng/permutation.png)

根据这个hash函数，每个特征向量的minhash值定义为：第一个value不为0的Element的值。

从矩阵中可以的得到：  
h(S1) = a, h(S2) = c, h(S3) = b, h(S4) = a

可以取另一个Element的不同的全排列，再进行一次以上的步骤，可以得到一组不同的minhash值。将所有的minhash值组合成矩阵，则新生成的矩阵可以作为新的特征向量。如果只取两个排列的话，则原来的5维特征向量降到了2维。

## 计算Minhash

实际应用中，如果特征维度很高的话，产生一次全排列是很费时的，所以通常并不会通过产生全排列计算minhash值。

如果特征向量为k维，则可以取n个hash函数，将0,1,...,k-1重新映射到0,1,...,k-1。类比于全排列，这些hash函数的意义相当于将第r维的特征放到了h(r)维，相当于产生了n个全排列。当然hash函数不可避免会有冲突，但是只要k足够大(如果不够大的话，就不需要minhash了)，而冲突不太多的时候，这个影响可以忽略。

如果以h1,h2,...,hn表示n个随机hash函数，r表示原矩阵的行数，SIG(i, c)表示为新生成矩阵第i个hash函数(也就是第i行)，第c列的元素。首先设所有SIG中的元素为MAXINT，然后对每一行r作如想处理：

![algo minhash](/images/blogpng/algo-minhash.png)

Minhash的python实现：

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-


def minhash(data, hashfuncs):
    '''
        see mining-of-massive-datasets.pdf ch3 minhash for detail
    '''

    # DEBUG = True
    DEBUG = False

    rows, cols, sigrows = len(data), len(data[0]), len(hashfuncs)

    # fucking the shadow copy
    # sigmatrix = [[1000000] * cols] * sigrows
    sigmatrix = []
    for i in range(sigrows):
        sigmatrix.append([10000000] * cols)

    for r in range(rows):
        hashvalue = map(lambda x: x(r), hashfuncs)
        if DEBUG: print hashvalue
        for c in range(cols):
            if DEBUG: print '-' * 2, r, c
            if data[r][c] == 0:
                continue
            for i in range(sigrows):
                if DEBUG: print '-' * 4, i, sigmatrix[i][c], hashvalue[i]
                if sigmatrix[i][c] > hashvalue[i]:
                    sigmatrix[i][c] = hashvalue[i]
                if DEBUG: print '-' * 4, sigmatrix

        if DEBUG:
            for xxxxxxx in sigmatrix:
                print xxxxxxx
            print '=' * 30

    return sigmatrix


if __name__ == '__main__':
    def hash1(x):
        return (x + 1) % 5

    def hash2(x):
        return (3 * x + 1) % 5

    data = [[1, 0, 0, 1],
            [0, 0, 1, 0],
            [0, 1, 0, 1],
            [1, 0, 1, 1],
            [0, 0, 1, 0]]

    print minhash(data, [hash1, hash2])
```

# Locality-Sensitive Hashing

在推荐系统中，通常需要的是与S1 n个最相似的，而不是所有的。[Locality-Sensitive Hashing](http://en.wikipedia.org/wiki/Locality_sensitive_hashing)的提出就是为了过滤明显不相似的，以此来减少计算。

假设向量的维度为n，将n维分为b份，称为bands，每份为n/b维。同时存在b个hash函数，每个bands对应一个hash函数，每个hash函数接受n/b个参数，产生一个数值，把相同数值的元素分为一组。

对每个binds中的n/b维特征向量进行hash计算，比如对于{S1, S2, S3, S4}，3个binds而言，第一个binds可能产生的数据为{S1, S2}, {S3}, {S4}，第二个binds为{S1}, {S2, S4}, {S3}，第三个binds为{S1, S3}, {S2, S4}。则为了计算得到和S1最相似的n个，则只需要计算J(S1, S2), J(S1, S3)，这样达到了减少了计算的目的。


