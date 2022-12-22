---
title: 反向传播算法
author: 王哲峰
date: '2022-07-15'
slug: dl-model-backforward
categories:
  - deeplearning
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

- [计算图(computational graph)](#计算图computational-graph)
- [链式法则(chain rule)](#链式法则chain-rule)
- [反向传播](#反向传播)
- [梯度爆炸与梯度消失](#梯度爆炸与梯度消失)
  - [梯度爆炸与梯度消失的问题提出](#梯度爆炸与梯度消失的问题提出)
  - [梯度爆炸与梯度消失现象解释](#梯度爆炸与梯度消失现象解释)
  - [梯度爆炸和梯度消失出现的机制](#梯度爆炸和梯度消失出现的机制)
  - [梯度爆炸与梯度消失的解决方法](#梯度爆炸与梯度消失的解决方法)
</p></details><p></p>

# 计算图(computational graph)

计算图将计算过程用图形表示出来, 这里的图形是指数据结构图, 通过多个节点和边表示

用计算图解题时, 需要按如下流程进行:

- 构建计算图; 
- 在计算图上从左到右进行计算(正向传播, forward propagation); 
- 在计算图上从右到左进行计算(反向传播, backward propagation); 
   - 将上游传过来的值E乘以节点的局部导数 `$\frac{\partial y}{\partial x}$`, 
     然后将结果传递给下一个节点

计算图的优点:

- 局部计算
   - 无论全局多么复杂的计算, 都可以通过局部计算使各个节点致力于简单的计算, 从而简化问题
- 利用计算图可以将中间的计算结果保存起来
- 可以通过反向传播高效计算导数
- 综上:可以通过正向传播和反向传播高效地计算各个变量的导数值

# 链式法则(chain rule)

计算图的正向传播将计算结果正向(从左到右)传递, 
而反向传播将局部导数向正方向的反方向(从右到左)传递; 

* **复合函数:**
    - 复合函数是由多个函数构成的函数; 
* **链式法则:**
    - 如果某个函数由复合函数表示, 则该复合函数的导数可以用构成复合函数的各个函数的导数的乘积表示; 

# 反向传播

- 加法节点的反向传播
    - 加法的反向传播只是将上游的值传给下游, 并不需要正向传播的输入信号; 
- 乘法节点的反向传播
    - 乘法的反向传播会将上游的值乘以正向传播时的输入信号的“翻转值”后传递给下游, 翻转值表示一种翻转关系; 
    - 乘法的反向传播需要正向传播时的输入信号值; 因此, 实现乘法节点的反向传播时, 要保存正向传播的输入信号; 

# 梯度爆炸与梯度消失

## 梯度爆炸与梯度消失的问题提出

在神经网络训练的过程中, 当网络结构变深时, 
神经网络在训练时会碰到梯度爆炸或者梯度消失的情况。
那么什么是梯度爆炸和梯度消失呢？它们又是怎么产生的呢？

## 梯度爆炸与梯度消失现象解释

- 由于神经网络的训练机制, 不管是哪种类型的神经网络, 其训练都是通过反向传播计算梯度来实现权重更新的。
  我们通过设定损失函数, 建立损失函数关于各层网络输入输出的梯度计算, 当网络训练开动起来的时候, 
  系统便按照反向传播机制来不断更新网络各层参数直到停止训练。但当网络层数加深时, 这个训练系统并不是很稳, 
  经常会出现一些问题。其中梯度爆炸和梯度消失便是最大的问题之二
- 梯度爆炸
    - 在神经网络训练过程中, 梯度变得越来越大, 使得神经网络权重得到疯狂更新的情形, 这种情况很容易发现, 
      因为梯度过大, 计算更新得到的参数也会大到崩溃, 这时候我们可能看到更新的参数值中有很多的 NaN, 
      这说明梯度爆炸已经使得参数更新出现数值溢出
- 梯度消失
    - 与梯度爆炸相反的是, 梯度消失就是在神经网络训练过程中梯度变得越来越小以至于梯度得不到更新的一种情形。
      当网络加深时, 网络深处的误差因为梯度的减小很难影响到前层网络的权重更新, 一旦权重得不到有效的更新计算, 
      神经网络的训练机制也就失效了

## 梯度爆炸和梯度消失出现的机制

神经网络训练过程中梯度怎么就会变得越来越大或者越来越小呢？以神经网络反向传播推导公式为例来解释:

![img](images/gradient_explosion_disappear.png)

上述公式是一个两层网络的反向传播参数更新公式推导过程。离输出层相对较远的是输入到隐藏层的权重参数, 
可以看到损失函数对于隐藏层输出 a1 输入到隐藏层权重 W1 和偏置 b1 的梯度计算公式, 一般而言都会转
换从下一层的权重乘以激活函数求导后的式子。如果激活函数求导后的结果和下一层权重的乘积大于 1 或者
说远远大于 1 的话, 在网络层数加深时, 层层递增的网络在做梯度更新时往往就会出现梯度爆炸的情况。
如果激活函数求导和下一层权重的乘积小于 1 的话, 在网络加深时, 浅层的网络梯度计算结果会越来越小往往
就会出现梯度消失的情况。所以可是说是反向传播的机制本身造就梯度爆炸和梯度消失这两种不稳定因素。
比如说一个 100 层的深度神经网络, 假设每一层的梯度计算值都为 1.1, 经过由输出到输入的反向传播梯度计
算可能最后的梯度值就变成 1.1^100 = 13780.61234, 这是一个极大的梯度值了, 足以造成计算溢出问题。
若是每一层的梯度计算值为 0.9, 反向传播输入层的梯度计算值则可能为 0.9^100 = 0.000026561398, 
足够小到造成梯度消失。 (例子只是一个简化的假设情况, 实际反向传播计算要更为复杂。) 

所以总体来说, 神经网络的训练中梯度过大或者过小引起的参数过大过小都会导致神经网络失效, 那我们的目的就
是要让梯度计算回归到正常的区间范围, 不要过大也不要过小, 这也是解决这两个问题的一个思路。
下图是一个 4 层网络的训练过程各参数的取值范围:

## 梯度爆炸与梯度消失的解决方法

那么梯度爆炸和梯度消失怎么解决呢？梯度爆炸并不麻烦, 在实际训练的时候对梯度进行修剪即可, 
但是梯度消失的处理就比较麻烦了, 由上述的分析我们知道梯度消失一个关键在于激活函数。
sigmoid 激活函数本身就更容易产生这种幺蛾子, 所以一般而言, 
我们换上更加鲁棒的 ReLu 激活函数以及给神经网络加上归一化激活函数层, 一般问题都能得到很好的解决, 
但也不是任何情形下都管用, 比如说咱们的 RNN 网络。