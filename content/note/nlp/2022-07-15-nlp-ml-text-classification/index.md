---
title: NLP--文本分类
author: 王哲峰
date: '2022-04-05'
slug: nlp-utils-spacy
categories:
  - NLP
tags:
  - tool
---

NLP--文本分类
================

1.文本分类任务简介
---------------------------------------

一般, 文本分类大致分为以下几个步骤:

    - 1.定义阶段
        - 定义数据以及分类体系, 具体分为哪些类别, 需要哪些数据
    - 2.数据预处理
        - 对文档做分词、去停用词等准备工作
    - 3.数据提取特征
        - 对文档矩阵进行降维, 提取训练集中最有用的特征
    - 4.模型训练阶段
        - 选择具体分类模型以及算法, 训练出文本分类器
    - 5.评测阶段
        - 在测试集上测试并评价分类器性能
    - 6.应用阶段
        - 应用性能最高的分类模型对待分类文档进行分类

2.朴素贝叶斯分类模型
-----------------------------------------

2.1 理论
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


2.2 实现
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~









2.使用 fastText 构建用户评论模型
----------------------------------------

    - 构建用户评论模型
    - 使用 Facebook 的 `fastText`, 一个很强大的文本分类模型库

2.1 下载训练数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    - Yelp 提供了一个拥有 470万 用户评论的研究数据集

