---
title: 机器学习概览
author: 王哲峰
date: '2023-02-24'
slug: ml
categories:
  - machinelearning
tags:
  - note
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

- [文章](#文章)
- [机器学习系统的种类](#机器学习系统的种类)
- [机器学习的主要挑战](#机器学习的主要挑战)
</p></details><p></p>

# 文章

* [机器学习入门清单及路线](https://mp.weixin.qq.com/s/KeD9kPG8PowlKrz69zSHNQ)
* [机器学习初学者易踩的5个坑](https://mp.weixin.qq.com/s/6r4FX7DEIUHE8pnYkftXSA)

# 机器学习系统的种类

1. 监督、无监督、半监督学习
2. 批量学习、在线学习
3. 基于实例、基于模型的学习

# 机器学习的主要挑战

1. 训练数据的数量不足
2. 训练数据不具有代表性
3. 数据质量差
4. 无关特征
5. 模型在训练数据上过度拟合
    - 改进模型过拟合的方法之一是提供更多的训练数据, 直到模型验证误差接近训练误差
    - 模型复杂度高, 对模型进行正则化: 
        - 多项式模型: 降低多项式的阶数
        - 岭回归, Ridge Regression
        - Lasso 回归, Lasso Regression
        - 弹性网络, Elastic Net
6. 模型在训练数据上欠拟合
    - 模型复杂度低：果模型对训练数据欠拟合，添加更多的训练样本也没用，
      此时需要使用更复杂的模型，或找到更好的特征
7. 偏差/方差权衡
    - 模型的泛华误差可以被表示为三个不同的误差之和：
        - 偏差：这部分泛化误差的原因在于错误的假设：比如假设数据是线性的：而实际上是二次的。
          高偏差模型最有可能对训练数据拟合不足
        - 方差：这部分误差是模型对训练数据的微小变化过度敏感导致的。
          具有高自由度的模型, 很可能有高方差, 所以很容易对训练数据过拟合
        - 不可避免的误差: 这部分误差是因为数据本身的噪声所致，减少这部分误差的唯一方法是清理数据
    - 增加模型的复杂度通常会显著提升模型的方差，减少偏差；反过来，降低模型的复杂度则会提升模型的偏差，降低方差