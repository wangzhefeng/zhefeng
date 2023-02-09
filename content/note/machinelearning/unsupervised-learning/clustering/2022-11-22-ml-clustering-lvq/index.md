---
title: 学习向量量化聚类
author: 王哲峰
date: '2022-11-22'
slug: ml-clustering-lvq
categories:
  - machinelearning
tags:
  - model
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

- [算法介绍](#算法介绍)
- [算法实现](#算法实现)
</p></details><p></p>

# 算法介绍

LVQ(Learning vector Quantization) 假设数据样本带有类别标记, 
学习过程利用样本的这些监督信息来辅助聚类. 

给定样本集 `$D=\{(x_{1}, y_{1}), (x_{2}, y_{2}), \ldots, (x_{n}, y_{n})\}$`, 
`$x_{i} \in R^{p}$, $y_{i} \in \mathcal{Y}$` 是样本的类别标记. 
LVQ的目标是学得一组 `$n$` 维原型向量 `$\{p_{1}, p_{2}, \ldots, p_{q}\}$`, 
每个原型向量代表一个聚类簇, 簇标记为 `$t_{i}\in \mathcal{Y}$`.

**算法**

> **输入:**
> 
> 样本集: `$D=\{(x_{1}, y_{1}), (x_{2}, y_{2}), \ldots, (x_{n}, y_{n})\}$`;
>
> 原型向量个数 `$q$`, 各原型向量预设的类标记 `$\{t_{1}, t_{2}, \ldots, t_{q}\}$`;
>
> 学习率 `$\eta \in (0,1)$`.
> 
> **过程:**
> 
> 1. 初始化一组原型向量: `$\{p_{1}, p_{2},\ldots,p_{q}\}$`
> 2. **repeat**
> 3. 从样本集中随机选取样本 `$(x_{j}, y_{j})$`;
> 4. 计算样本 `$x_{j}$` 与 `$p_{i} (1 \leqslant i \leqslant q)$` 的距离: `$d_{ji}=\|x_{j}-p_{i}\|_{2}$`;
> 5. 找出与 `$x_{j}$` 距离最近的原型向量 `$p_{i^{*}}$`, `$i^{*}=arg min_{i \in \{1, 2, \ldots, q\}}d_{ji}$`;
> 6. **if** `$y_{j}=t_{i^{*}}$`
> 7. `$p' = p_{i^{*}} + \eta \cdot (x_{j}-p_{i^{*}})$`
> 8. **else**
> 9. `$p' = p_{i^{*}} - \eta \cdot (x_{j}-p_{i^{*}})$`
> 10. **end if**
> 11. 将原型向量 `$p_{i^{*}}$` 更新为 `$p'$`
> 12. **until** 满足停止条件
> 
> **输出:**
> 
> 原型向量 `$\{p_{1}, p_{2}, \ldots, p_{q}\}$`,

**算法解释**

* 算法第1行: 对原型向量进行初始化. 例如: 对第 `$i,i=(1,2,\ldots,q)$` 个簇,从类别标记为 `$t_{i}$` 的样本中随机选取一个作为原型向量. 
* 算法第2-12行: 对原型向量进行迭代优化. 在每一轮迭代中, 算法随机选取一个有标记训练样本, 
  找出与其距离最近的原型向量, 并根据两者的类别标记是否一致来对原型向量进行相应的更新. 
    - 第6-10行: 如何更新原型向量. 对样本 `$x_{j}$`,
        - 若距离`$x_{j}$`最近的原型向量 `$p_{i^{*}}$` 与 `$x_{j}$` 的标记相同, 
          则令 `$p_{i^{*}}$` 向 `$x_{j}$` 的方向靠拢, 此时新的原型向量为
          $$p' = p_{i^{*}} + \eta \cdot (x_{j}-p_{i^{*}})$$
          `$p^{'}$` 与 `$x_{j}$` 之间的距离为
          $$\|p'-x_{j}\|_{2}=(1-\eta) \cdot \|p_{i^{*}}-x_{j}\|_{2}$$
          原型向量 `$p_{i^{*}}$` 更新为 `$p'$` 之后将更接近 `$x_{j}$`.
        - 若距离`$x_{j}$`最近的原型向量 `$p_{i^{*}}$` 与 `$x_{j}$` 的标记不同, 
          则令 `$p_{i^{*}}$` 向 `$x_{j}$` 的方向远离, 此时新的原型向量为 
          $$p' = p_{i^{*}} - \eta \cdot (x_{j}-p_{i^{*}})$$
          `$p^{'}$` 与 `$x_{j}$` 之间的距离为
          $$\|p'-x_{j}\|_{2}=(1+\eta) \cdot \|p_{i^{*}}-x_{j}\|_{2}$$
          原型向量 `$p_{i^{*}}$` 更新为 `$p'$`之后将更远离 `$x_{j}$`.
* 算法第12行: 若算法的停止条件已满足(例如已达到最大迭代轮数, 或原型向量更新很小甚至不再更新), 则将当前原型向量作为最终结果返回. 
* 在学得一组原型向量 `$\{p_{1}, p_{2}, \ldots, p_{q}\}$` 后即可实现对样本空间 `$\mathcal{X}$` 的簇划分. 
    - 对任意样本 `$x$`, 他将被划入与其距离最近的原型向量所代表的簇中, 
      每个原型向量 `$p_{i}$` 定义了与之相关的一个区域 `$R_{i}$`, 
      该区域中每个样本与 `$p_{i}$` 的距离不大于他与其他原型向量 `$p_{i'} (i \neq i')$`, 即
      $$R_{i}=\{x \in \mathcal{X}|\|x-p_{i}\|_{2} \leqslant \|x-p_{i'}\|_{2}, i \neq i'\}$$
      由此形成了对样本空间 `$\mathcal{X}$` 的簇划分 `$\{R_{1}, R_{2}, \ldots, R_{q}\}$`, 该划分通常称为"Voronoi剖分"(Voronoi tessellation).


# 算法实现
