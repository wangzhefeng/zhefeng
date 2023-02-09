---
title: 时间序列聚类
author: 王哲峰
date: '2022-04-25'
slug: timeseries-clustering
categories:
  - timeseries
tags:
  - machinelearning
---

<style>
details {
    border: 1px solid #aaa;
    border-radius: 4px;
    padding: .5em .5em 0;
}
summary {
    font-weight: bold;
    margin: -.5em -.5em 0;
    padding: .5em;
}
details[open] {
    padding: .5em;
}
details[open] summary {
    border-bottom: 1px solid #aaa;
    margin-bottom: .5em;
}
</style>

<details><summary>目录</summary><p>

- [聚类](#聚类)
- [时间序列聚类算法](#时间序列聚类算法)
  - [基于距离的机器学习聚类算法](#基于距离的机器学习聚类算法)
    - [KMeans](#kmeans)
  - [基于相似性的机器学习聚类算法](#基于相似性的机器学习聚类算法)
    - [层次聚类](#层次聚类)
- [参考](#参考)
</p></details><p></p>


# 聚类

聚类分析(cluster analysis)简称聚类(clustering)，它是数据挖掘领域最重要的研究分支之一，
也是最为常见和最有潜力的发展方向之一。聚类分析是根据事物自身的特性对被聚类对象进行类别划分的统计分析方法，
其目的是根据某种相似度度量对数据集进行划分，将没有类别的数据样本划分成若干个不同的子集，
这样的一个子集称为簇（cluster)，聚类使得同一个簇中的数据对象彼此相似，
不同簇中的数据对象彼此不同，即通常所说的“物以类聚”

时间序列的聚类在工业生产生活中非常常见，大到工业运维中面对海量KPI曲线的隐含关联关系的挖掘，
小到股票收益曲线中的增长模式归类，都要用到时序聚类的方法帮助我们发现数据样本中一些隐含的、深层的信息

# 时间序列聚类算法

根据对时间序列的距离度量方法，可以采用机器学习中很多聚类算法，进行时间序列的聚类分析

## 基于距离的机器学习聚类算法

### KMeans

KMeans 算法是非常经典的距离度量聚类算法，其目的是把样本，
基于它们之间的距离公式，把它们划分成 K 个类别，
其中类别 K 的个数是需要在执行算法之前人为设定的

从数学语言上来说，假设已知距离空间点集为 `$\{x_{1}, x_{2}, \ldots, x_{n}\}$`，
事先设定的类别个数是 `$K$`，当然 `$K \leq n$` 是必须要满足的，
因为类别的数目不能够多于点集的元素个数。算法的目标是寻找到合适的集合 `$\{S_{i}\}_{1 \leq i \leq K}$` 使得

`$$\underset{S_{i}}{argmin} \underset{x \in S_{i}}{\sum} ||x-\mu_{i}||^{2}$$`

达到最小，其中 `$\mu_{i}$` 表示集合 `$S_{i}$` 中的所有点的均值

## 基于相似性的机器学习聚类算法

### 层次聚类

层次聚类是一种很直观的方法，就是一层一层的对数据进行聚类操作，可以自低向上进行合并聚类、也可以自顶向下进行分裂聚类

* 凝聚式：从点作为个体簇开始，每一个步合并两个最接近的簇
* 分裂式：从包含所有个体的簇开始，每一步分裂一个簇，直到仅剩下单点簇为止

所谓凝聚，其大体思想就是在一开始的时候，把点集集合中的每个元素都当做一类，
然后计算每两个类之前的相似度，也就是元素与元素之间的距离；
然后计算集合与集合之前的距离，把相似的集合放在一起，不相似的集合就不需要合并；
不停地重复以上操作，直到达到某个限制条件或者不能够继续合并集合为止

所谓分裂，正好与聚合方法相反。其大体思想就是在刚开始的时候把所有元素都放在一类里面，
然后计算两个元素之间的相似性，把不相似元素或者集合进行划分，
直到达到某个限制条件或者不能够继续分裂集合为止

在层次聚类里面，相似度的计算函数就是关键所在。在这种情况下，可以设置两个元素之间的距离公式，
距离越小表示两者之间越相似，距离越大则表示两者之间越不相似。
除此之外，还可以设置两个元素之间的相似度


# 参考

- [[1] 时间序列聚类](https://mp.weixin.qq.com/s?__biz=Mzg3NDUwNTM3MA==&mid=2247484837&idx=1&sn=cdc922e6a213064485113bbb9b8e911e&chksm=cecef050f9b9794672f0227b36212a1fcf8acb1916c6e12e923bcdf5e9ac71b0aa9e7bb7f58d&scene=21#wechat_redirect)
