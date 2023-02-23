---
title: 集成学习模型
subtitle: Ensemble Learning Models
list_pages: true
order_by: title
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

- [TODO](#TODO)
</p></details><p></p>

## 集成学习

所谓集成学习(ensemble learning)，是指利用训练数据集构建多个分类器(弱分类器)，并分别对测试数据集进行预测，
然后用某种策略将多个分类器预测的结果集成起来，作为最终预测结果. 通俗比喻就是"三个臭皮匠赛过诸葛亮”，
或一个公司董事会上的各董事投票决策，它要求每个弱分类器具备一定的"准确性”、分类器之间具备"差异性”

集成学习根据各个弱分类器之间有无依赖关系，分为 Boosting 和 Bagging 两大流派: 

* Boosting 流派
    - 各分类器之间有依赖关系，必须串行，比如: Adaboost、GBDT(Gradient Boosting Decision Tree)、
      Xgboost、LightGBM、CatBoost等
* Bagging 流派
    - 各分类器之间没有依赖关系，可各自并行，比如: Bagging、随机森林(Random Forest)

### Boosting

* GBM
    - Adaboost
    - GBDT
    - XGBoost
    - LightGBM
    - CatBoost

早期的 GBDT 模型，在诸多传统表格型数据的建模问题上表现都非常不错，
但早期版本的 GBDT 模型在模型训练预测方面的速度一般；

陈天奇对原始的 GBDT 模型进行了升级，将原先的一阶导数扩展为二阶导数近似的形式，
加入了新的正则并设计了近似算法来加速树的生成，此外，还在系统的设计上面进行了很多改进，
包括设计了并行计算的列模块；Cache-aware Access，内存外的计算等等来适用于目前的大数据的趋势

LightGBM 对 XGBoost 进行了进一步升级，将 XGBoost 中树模型的生成方式从 Level-wise 变为 Leaf-wise。
此外通过使用 histogram 的技巧加速了模型训练，在很多问题中也取得了更好的效果。不仅如此，
LightGBM 中提出的 GOSS 和 EFB 算法在不影响精度的情况下，分别减少了 GBDT 训练中所需的数据量和特征量。
另外 LightGBM 相较于 XGBoost 在特征和数据层面还进行了进一步的加速优化，所以在数据量大的情况下，
LightGBM 的效果和速度相较于 XGboost 往往更优

CatBoost 相较于 XGBoost 以及 LightGBM，在类别特征上进行了较大的优化，对类别特征引入了 Target Encoding，
还设计了贪心的特征组合策略。此外，通过 Ordered Boosting 的策略，CatBoost 缓解了预测漂移的问题，
在诸多小数据集的表现上也展现了非常好的效果

目前三个模型互相借鉴，也都在之前的基础上进行了不同程度的改进，但三个模型训练融合依旧是在传统数据建模竞赛中的一大利器

## 文档
