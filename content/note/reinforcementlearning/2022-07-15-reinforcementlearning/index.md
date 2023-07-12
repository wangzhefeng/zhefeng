---
title: 强化学习概览
author: 王哲峰
date: '2022-07-15'
slug: reinforcementlearning
categories:
  - deeplearning
tags:
  - tool
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
img {
    pointer-events: none;
}
</style>

<details><summary>目录</summary><p>

- [强化学习基础](#强化学习基础)
  - [强化学习的定义](#强化学习的定义)
  - [强化学习、监督学习和无监督学习的区别](#强化学习监督学习和无监督学习的区别)
  - [强化学习的使用场景](#强化学习的使用场景)
  - [强化学习损失函数](#强化学习损失函数)
  - [有模型和免模型](#有模型和免模型)
- [强化学习示例](#强化学习示例)
  - [CartPole](#cartpole)
    - [开发环境](#开发环境)
    - [CartPole 示例](#cartpole-示例)
</p></details><p></p>

# 强化学习基础

## 强化学习的定义

> 强化学习包含环境、动作和奖励三部分，其本质是智能体通过与环境的交互，使其做出的动作对应的决策得到的总奖励最大，或者说是期望最大

强化学习(Reinforcement Learning, RL)讨论的问题是 **智能体(agent)** 怎么在复杂、不确定的 **环境(environment)** 中最大化它能获得的奖励

![img](images/rl.png)

强化学习由两部分组成：**智能体(agent)** 和 **环境(environment)**。在强化学习过程中，智能体与环境一直在交互。
智能体在环境中获取某个 **状态(state)** 后，它会利用该状态输出一个 **动作(action)**，这个动作也称为 **决策(decision)**。
然后这个动作会在环境中被执行，环境会根据智能体采取的动作，输出下一个状态以及当前这个动作带来的 **奖励(award)**。
智能体的目的就是尽可能地从环境中获得奖励

## 强化学习、监督学习和无监督学习的区别

强化学习、监督学习和无监督学习的区别：

* 首先强化学习和无监督学习是不需要有标签样本的，而监督学习需要许多有标签样本来进行模型的构建和训练
* 其次对于强化学习与无监督学习，无监督学习直接基于给定的数据进行建模，寻找数据或特征中隐藏的结构，
  一般对应聚类问题；强化学习需要通过延迟奖励学习策略来得到模型与目标的距离，这个距离可以通过奖励函数进行定量判断，这里我们可以将奖励函数视为正确目标的一个稀疏、延迟形式。
* 另外，强化学习处理的多是序列数据，样本之间通常具有强相关性，但其很难像监督学习的样本一样满足独立同分布条件。

## 强化学习的使用场景

多序列决策问题，或者对应的模型未知，需要通过学习逐渐逼近真实模型的问题。并且当前的动作会影响环境的状态，即具有马尔科夫性的问题。
同时应满足所有状态是可重复到达的条件，即满足学习条件

## 强化学习损失函数

深度学习中的损失函数的目的是使预测值和真实值之间的差距尽可能小，而强化学习中损失函数的目的是使总奖励的期望尽可能大

## 有模型和免模型

两者的区别主要在于是否需要对真实的环境进行建模，免模型方法不需要对环境进行建模，直接与真实环境进行交互即可，
所以其通常需要较多的数据或者采样工作来优化策略，这也使其对于真实环境具有更好的泛化性能；

而有模型方法需要对环境进行建模，同时在真实环境与虚拟环境中进行学习，如果建模的环境与真实环境的差异较大，
那么会限制其泛化性能。现在通常使用有模型方法进行模型的构建工作

# 强化学习示例

## CartPole

以下示例使用深度强化学习玩 CartPole(倒立摆)游戏，倒立摆是控制论中的经典问题，
在这个游戏中，一根杆的底部与一个小车通过轴相连，而杆的重心在轴之上，因此是一个不稳定的系统。
在重力的作用下，杆很容易倒下。需要控制小车在水平的轨道上进行左右运动，使得杆一直保持竖直平衡状态。

### 开发环境

OpenAI 推出的 Gym 环境库中的 CartPole 游戏环境，和 Gym 的交互过程很像一个回合制游戏：

* 首先，获得游戏的初始状态(比如杆的角度、小车位置)
* 然后，在每个回合，都需要在当前可行的动作中选择一个并交由 Gym 执行
    - 比如：向左或者右推动小车，每个回合中二者只能选择一
* Gym 在执行动作后，会返回动作执行后的下一个状态和当前回合所获得的奖励值
    - 比如：选择向左推动小车并执行后，小车位置更加偏左，而杆的角度更加偏右，Gym 将新的角度和位置返回，
      而如果在这一回合杆没有倒下，Gym 同时返回一个小的正奖励
* 上述过程可以一直迭代下去，直到游戏结束
    - 比如：杆倒下了

Gym 环境库中的 CartPole 游戏环境库安装

```bash
$ pip install gym
```

Gym 的 Python 基本调用方法

```python
import gym

# 实例化一个游戏环境，参数为游戏名称
env = gym.make("CartPole-v1")

# 初始化环境，获得初始状态
state = env.reset()

while True:
    # 对当前帧进行渲染，绘图到屏幕
    env.render()
    
    # 假设我们有一个训练好的模型，能够通过当前状态预测出这时应该进行的动作
    action = model.predict(state)
    
    # 让环境执行动作，获得执行完动作的下一个状态，动作的奖励，游戏是否已结束以及额外信息
    next_state, reward, done, info = env.step(action)
    
    # 如果游戏结束，则退出循环
    if done:
        break
```

### CartPole 示例

任务：训练出一个模型，能够根据当前的状态预测出应该进行的一个好的动作。
粗略地说，一个好的动作应当能够最大化整个游戏过程中获得的奖励之和，
这也是强化学习的目标

CartPole 游戏中的目标是，希望做出的合适的动作使得杆一直不倒，
即游戏交互的回合数尽可能多，而每进行一回合，都会获得一个小的正奖励，
回合数越多则积累的奖励值也越高。因此，最大化游戏过程中的奖励之和与最终目标是一致的

使用深度强化学习中的 Deep Q-Learning 方法来训练模型

1. 首先，引入常用库，定义一些模型超参数

```python
import tensorflow as tf
```

2.使用 `tf.keras.Model` 建立一个 Q 函数网络，用于拟合 Q-Learning 中的 Q 函数

使用较简单的多层全连接神经网络进行拟合，该网络输入当前状态，输入各个动作下的 Q-Value(CartPole 下为二维，即向左和向右推动小车)

```python
class QNetwork(tf.keras.Model):
    pass
```