---
title: PyTorch 模型训练
author: 王哲峰
date: '2022-08-14'
slug: dl-pytorch-model-compile
categories:
  - deeplearning
  - pytorch
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
</style>

<details><summary>目录</summary><p>

- [权重初始化](#权重初始化)
- [损失函数](#损失函数)
  - [PyTorch 内置损失函数](#pytorch-内置损失函数)
    - [常用内置损失函数 API](#常用内置损失函数-api)
    - [常用内置损失函数解释](#常用内置损失函数解释)
  - [创建自定义损失函数](#创建自定义损失函数)
    - [自定义损失函数 FocalLoss](#自定义损失函数-focalloss)
    - [SCELoss](#sceloss)
  - [自定义 L1 和 L2 正则化项](#自定义-l1-和-l2-正则化项)
    - [数据准备](#数据准备)
    - [定义模型](#定义模型)
    - [模型训练](#模型训练)
    - [结果可视化](#结果可视化)
    - [通过优化器实现 L2 正则化](#通过优化器实现-l2-正则化)
- [优化器](#优化器)
- [评价指标](#评价指标)
- [模型训练](#模型训练-1)
  - [MNIST 数据集多分类示例](#mnist-数据集多分类示例)
    - [数据准备](#数据准备-1)
    - [数据查看](#数据查看)
  - [脚本风格](#脚本风格)
    - [模型构建](#模型构建)
    - [损失函数、优化器、评价指标](#损失函数优化器评价指标)
    - [模型训练](#模型训练-2)
  - [函数风格](#函数风格)
    - [模型构建](#模型构建-1)
    - [损失函数、优化器、评价指标](#损失函数优化器评价指标-1)
    - [模型训练](#模型训练-3)
  - [类风格](#类风格)
    - [类风格 torchkeras.KerasModel](#类风格-torchkeraskerasmodel)
      - [模型构建](#模型构建-2)
      - [损失函数、优化器、评价指标](#损失函数优化器评价指标-2)
      - [模型训练](#模型训练-4)
    - [类风格 torchkeras.LightModel](#类风格-torchkeraslightmodel)
      - [模型构建](#模型构建-3)
      - [损失函数、优化器、评价指标](#损失函数优化器评价指标-3)
      - [模型训练](#模型训练-5)
</p></details><p></p>

# 权重初始化

* TODO

# 损失函数

一般来说，监督学习的目标函数由损失函数和正则化项组成，

`$$Objective = Loss + Regularization$$`

PyTorch 中的损失函数一般在模型训练的时候指定。PyTorch 中内置的损失函数的参数和 TensorFlow 不同:

* PyTorch: `y_pred` 在前，`y_true` 在后
* TensorFlow: `y_true` 在前，`y_pred` 在后

PyTorch 内置损失函数的使用:

* 对于回归模型，通常使用的内置损失函数是均方损失函数 `torch.nn.MSELoss`
* 对于二分类模型，通常使用的是二元交叉熵损失函数 `torch.nn.BCELoss`(输入已经是 `torch.nn.Sigmoid` 激活函数之后的结果)，
  或者 `torch.nn.BCEWithLogitsLoss`(输入尚未经过 `torch.nn.Sigmoid` 激活函数)
* 对于多分类模型
    - 一般推荐使用交叉熵损失函数 `torch.nn.CrossEntropyLoss`(`y_true` 需要是一维的，
      是类别编码，`y_pred` 未经过 `torch.nn.Softmax` 激活)
    - 如果多分类的 `y_pred` 经过了 `torch.nn.LogSoftmax` 激活，可以使用 `torch.nn.NLLLoss` 损失函数(The Negative Log Likelihood Loss)，
      这种方法和直接使用 `torch.nn.CrossEntropyLoss` 等价

如果有需要，也可以自定义损失函数，自定义损失函数需要接收两个张量 `y_pred`、`y_true` 作为输入参数，
并输出一个标量作为损失函数

PyTorch 中的正则化项一般通过自定义的方式和损失函数一起添加作为目标函数。
如果仅仅使用 L2 正则化，也可以利用优化器的 `weight_decay` 参数来实现相同的效果

## PyTorch 内置损失函数

PyTorch 内置损失函数一般有类的实现和函数的实现两种形式。
例如：`torch.nn.BCE` 和 `torch.nn.functional.binary_cross_entropy` 都是二元交叉熵损失函数，
但前者是类的实现形式，后者是函数的实现形式。
实际上，类的实现形式通常是调用函数的实现形式并用 `torch.nn.Module` 封装后得到的

一般常用的是类的实现形式，它们封装在 `torch.nn` 模块下，并且类名以 `Loss` 结尾

### 常用内置损失函数 API

回归:

* `torch.nn.MSELoss`
    - 均方误差损失，也叫做 L2 损失
    - 用于回归
* `torch.nn.L1Loss`
    - L1 损失，也叫做绝对误差损失
    - 用于回归
* `torch.nn.SmoothL1Loss`
    - 平滑 L1 损失，当输入在 `$[-1, 1]$` 之间时，平滑为 L2 损失，
    - 用于回归

二分类:

* `torch.nn.BCELoss`
    - 二元交叉熵损失，输入已经过 `torch.nn.Sigmoid` 激活，
      对于不平衡数据集可以用 `weights` 参数调整类别权重
    - 用于二分类
* `torch.nn.BCEWithLogitsLoss`
    - 二元交叉熵损失，输入未经过 `torch.nn.Sigmoid` 激活
    - 用于二分类

多分类:

* `torch.nn.CrossEntropyLoss`
    - 交叉熵，要求 `label` 为稀疏编码，输入未经过 `torch.nn.Softmax` 激活，
      对于不平衡数据集可以用 `weight` 参数调整类别权重
    - 用于多分类
* `torch.nn.NLLLoss`
    - 负对数似然损失，要求 `label` 为稀疏编码，输入经过 `torch.nn.LogSoftmax` 激活
    - 用于多分类
* `torch.nn.KLDivLoss`
    - KL 散度，也叫相对熵，等于交叉熵减去信息熵，要求输入经过 `torch.nn.LogSoftmax` 激活
    - 用于多分类(标签为概率值)
* `torch.nn.CosineSimilarity`
    - 余弦相似度
    - 用于多分类
* `torch.nn.AdaptiveLogSoftmaxWithLoss`
    - 一种非常适合多类别且类别分布很不均衡的损失函数，会自适应地将多个小类别合成一个 cluster
    - 用于多分类

### 常用内置损失函数解释








## 创建自定义损失函数

如果有需要，也可以自定义损失函数，自定义损失函数需要接收两个张量 `y_pred`、`y_true` 作为输入参数，
并输出一个标量作为损失函数

也可以对 `torch.nn.Module` 进行子类化，重写 `forward` 方法实现损失的计算逻辑，
从而得到损失函数的类的实现

### 自定义损失函数 FocalLoss

Focal Loss 是一种对 `binary_crossentropy` 的改进损失函数的形式。
它在样本不均衡和存在较多易分类的样本时相比 `binary_crossentropy` 具有明显的优势

Focal Loss 的数学形式:

`$$\begin{split}
focal\_loss(y, p) =
\begin{cases}
-\alpha(1 - p)^{\gamma} \log(p) & \text{if } y = 1, \\\\
-(1 - \alpha)p^{\gamma} \log(1 - p) & \text{if } y = 0,
\end{cases}
\end{split}$$`


Focal Loss 有两个可调参数，从而让那模型更加聚焦在正样本和困难样本上，
这就是为什么这个损失函数叫做 Focal Loss:

* `alpha` 参数
    - 主要用于衰减负样本的权重
* `gamma` 参数
    - 主要用于衰减容易训练样本的权重

Focal Loss 介绍:

* https://zhuanlan.zhihu.com/p/80594704

Focal Loss 实现:

```python
import torch
from torch import nn


class FocalLoss(nn.Module):
    
    def __init__(self, gamma = 2.0, alpha = 0.75):
        super().__init__()
        self.gamma = gamma
        self.alpha = alpha
    
    def forward(self, y_pred, y_true):
        bce = torch.nn.BCELoss(reduction = "none")(y_pred, y_true)
        p_t = (y_true * y_pred) + ((1 - y_true) * (1 - y_pred))
        alpha_factor = y_true * self.alpha + (1 - y_true) * (1 - self.alpha)
        modulating_factor = torch.pow(1.0 - p_t, self.gamma)
        loss = torch.mean(alpha_factor * modulating_factor * bce)
        return loss

# 困难样本
y_pred_hard = torch.tensor([[0.5], [0.5]])
y_true_hard = torch.tensor([[1.0], [0.0]])
# 容易样本
y_pred_easy = torch.tensor([[0.9], [0.1]])
y_true_easy = torch.tensor([[1.0], [0.0]])

focal_loss = FocalLoss()
bce_loss = nn.BCELoss()

print("focal_loss(easy samples):", focal_loss(y_pred_easy, y_true_easy))
print("bce_loss(easy samples):", bce_loss(y_pred_easy, y_true_easy))

print("focal_loss(hard samples):", focal_loss(y_pred_hard, y_true_hard))
print("bce_loss(hard samples):", bce_loss(y_pred_hard, y_true_hard))
```

```
focal_loss(easy samples): tensor(0.0005)
bce_loss(easy samples): tensor(0.1054)
focal_loss(hard samples): tensor(0.0866)
bce_loss(hard samples): tensor(0.6931)
```

可见 focal_loss 让容易样本的权重衰减到原来的 0.0005/0.1054 = 0.00474。
而让困难样本的权重只衰减到原来的 0.0866/0.6931=0.12496。
因此相对而言，focal_loss 可以衰减容易样本的权重

### SCELoss

SCELoss(Symmetric Cross Entropy Loss) 也是一种对交叉熵损失的改进损失，
主要用在标签中存在明显噪声的场景

SCELoss 的数学形式:

`$$sce\_loss(y, p) = ace\_loss(y, p) + \beta rce\_loss(y, p)$$`

其中：

* `$ce\_loss(y, p) = -y log(p) - (1 - y) log(1 - p)$`
* `$rce\_loss(y, p) = -p log(y) - (1 - p) log(1 - y)$`
* `$rce\_loss(y, p) = ce\_loss(p, y)$`

SCELoss 介绍：

* https://zhuanlan.zhihu.com/p/420827592
* https://zhuanlan.zhihu.com/p/420913134

SCELoss 实现:

```python
import torch
from torch import nn
import torch.nn.functional as F

class SCELoss(nn.Module):

    def __init__(self, num_classes = 10, a = 1, b = 1):
        super(SCELoss, self).__init__()
        self.num_classes = num_classes
        self.a = a
        self.b = b
        self.cross_entropy = nn.CrossEntropyLoss()
    
    def forward(self, pred, labels):
        # CE 部分，正常的交叉熵损失
        ce = self.cross_entropy(pred, labels)
        # RCE
        pred = F.softmax(pred, dim = 1)
        pred = torch.clamp(pred, min = 1e-4, max = 1.0)
        label_one_hot = F.one_hot(labels, self.num_classes).float().to(pred.device)
        label_one_hot = torch.clamp(label_one_hot, min = 1e-4, max = 1.0)
        rce = (-1 * torch.sum(pred * torch.log(label_one_hot), dim = 1))
        # Loss
        loss = self.a * ce + self.b * rce.mean()
        return loss
```

## 自定义 L1 和 L2 正则化项

通常认为 L1 正则化可以产生稀疏权值矩阵，即产生一个稀疏模型，可以用于特征选择，
而 L2 正则化可以防止过拟合。一定程度上，L1 也可以防止过拟合

### 数据准备

```python
import numpy as np 
import pandas as pd 
from matplotlib import pyplot as plt
import torch
from torch import nn
import torch.nn.functional as F
from torch.utils.data import Dataset,DataLoader,TensorDataset
import torchkeras 
%matplotlib inline
%config InlineBackend.figure_format = 'svg'

#正负样本数量
n_positive,n_negative = 1000,6000

#生成正样本, 小圆环分布
r_p = 5.0 + torch.normal(0.0,1.0,size = [n_positive,1]) 
theta_p = 2*np.pi*torch.rand([n_positive,1])
Xp = torch.cat([r_p*torch.cos(theta_p),r_p*torch.sin(theta_p)],axis = 1)
Yp = torch.ones_like(r_p)

#生成负样本, 大圆环分布
r_n = 8.0 + torch.normal(0.0,1.0,size = [n_negative,1]) 
theta_n = 2*np.pi*torch.rand([n_negative,1])
Xn = torch.cat([r_n*torch.cos(theta_n),r_n*torch.sin(theta_n)],axis = 1)
Yn = torch.zeros_like(r_n)

#汇总样本
X = torch.cat([Xp,Xn],axis = 0)
Y = torch.cat([Yp,Yn],axis = 0)


#可视化
plt.figure(figsize = (6,6))
plt.scatter(Xp[:,0],Xp[:,1],c = "r")
plt.scatter(Xn[:,0],Xn[:,1],c = "g")
plt.legend(["positive","negative"]);


ds = TensorDataset(X,Y)

ds_train,ds_val = torch.utils.data.random_split(ds,[int(len(ds)*0.7),len(ds)-int(len(ds)*0.7)])
dl_train = DataLoader(ds_train,batch_size = 100,shuffle=True,num_workers=2)
dl_val = DataLoader(ds_val,batch_size = 100,num_workers=2)

features,labels = next(iter(dl_train))
```


### 定义模型

```python
class Net(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(2,4)
        self.fc2 = nn.Linear(4,8) 
        self.fc3 = nn.Linear(8,1)
        
    def forward(self,x):
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        y = self.fc3(x)
        return y
        
net = Net() 

from torchkeras import summary

summary(net,features);
```

### 模型训练

```python
# L2正则化
def L2Loss(model,alpha):
    l2_loss = torch.tensor(0.0, requires_grad=True)
    for name, param in model.named_parameters():
        if 'bias' not in name: #一般不对偏置项使用正则
            l2_loss = l2_loss + (0.5 * alpha * torch.sum(torch.pow(param, 2)))
    return l2_loss

# L1正则化
def L1Loss(model,beta):
    l1_loss = torch.tensor(0.0, requires_grad=True)
    for name, param in model.named_parameters():
        if 'bias' not in name:
            l1_loss = l1_loss +  beta * torch.sum(torch.abs(param))
    return l1_loss




from torchkeras import KerasModel
from torchkeras.metrics import AUCROC 

net = Net()

# 将L2正则和L1正则添加到FocalLoss损失，一起作为目标函数
def focal_loss_with_regularization(y_pred,y_true):
    y_probs = torch.sigmoid(y_pred)
    focal = FocalLoss()(y_probs,y_true) 
    l2_loss = L2Loss(net,0.0001) #注意设置正则化项系数
    l1_loss = L1Loss(net,0.0001)
    total_loss = focal + l2_loss + l1_loss
    return total_loss


optimizer = torch.optim.Adam(net.parameters(),lr = 0.002)
model = KerasModel(net=net,
                   loss_fn = focal_loss_with_regularization ,
                   metrics_dict = {"auc":AUCROC()},
                   optimizer= optimizer )


dfhistory = model.fit(train_data=dl_train,
      val_data=dl_val,
      epochs=20,
      ckpt_path='checkpoint.pt',
      patience=3,
      monitor='val_auc',
      mode='max')
```



### 结果可视化

```python
# 结果可视化
fig, (ax1,ax2) = plt.subplots(nrows=1,ncols=2,figsize = (12,5))
ax1.scatter(Xp[:,0],Xp[:,1], c="r")
ax1.scatter(Xn[:,0],Xn[:,1],c = "g")
ax1.legend(["positive","negative"]);
ax1.set_title("y_true");

Xp_pred = X[torch.squeeze(torch.sigmoid(net.forward(X))>=0.5)]
Xn_pred = X[torch.squeeze(torch.sigmoid(net.forward(X))<0.5)]

ax2.scatter(Xp_pred[:,0],Xp_pred[:,1],c = "r")
ax2.scatter(Xn_pred[:,0],Xn_pred[:,1],c = "g")
ax2.legend(["positive","negative"]);
ax2.set_title("y_pred");
```


### 通过优化器实现 L2 正则化

如果仅仅需要使用 L2 正则化，那么也可以使用优化器的 `weight_decay` 参数来实现，
`weight_decay` 参数可以设置参数在训练过程中的衰减，这和 L2 正则化的作用效果等价

PyTorch 的优化器支持一种称之为 Per-parameter options 的操作，
就是对每一个参数进行特定的学习率，权重衰减率指定

```python
weight_params = [
    param for name, param in model.named_parameters() 
    if "bias" not in name
]
bias_params = [
    param for name, param in model.named_parameters() 
    if "bias" in name
]

optimizer = torch.optim.SGD([
    {
        "params": weight_params, 
        "weight_decay": 1e-5,
    },
    {
        "params": bias_params,
        "weight_decay": 0,
    }
], lr = 1e-2, momentum = 0.9)
```

# 优化器



# 评价指标



# 模型训练

PyTorch 通常需要用户编写自定义训练循环，训练循环的代码风格因人而异。
有三类典型的训练循环代码风格:

* 脚本形式训练循环
* 函数形式训练循环
* 类形式训练循环

还有一种第三方用户编写的风格，通用的 Keras 风格的脚本形式训练循环，
需要安装相应的第三方库:

```bash
$ pip install torchkeras
```

##  MNIST 数据集多分类示例

```python
import torch
from torch import nn
import torchvision
from torchvision import transforms

print(f"torch.__version__ = {torch.__version__}")
```

### 数据准备

```python
# 数据转换
transform = transforms.Compose([transforms.ToTensor()])

# 数据
ds_train = torchvision.datasets.MNIST(
    root = "./data/minist/", 
    train = True,
    download = True,
    transform = transform
)
ds_val = torchvision.datasets.MNIST(
    root = "./data/minist/",
    train = False,
    download = True,
    transform = transform
)

dl_train =  torch.utils.data.DataLoader(
    ds_train, 
    batch_size = 128, 
    shuffle = True, 
    num_workers = 4
)
dl_val =  torch.utils.data.DataLoader(
    ds_val, 
    batch_size = 128, 
    shuffle = False, 
    num_workers = 4
)

print(len(ds_train))
print(len(ds_val))
```

### 数据查看

```python
import matplotlib
import matplotlib.pyplot as plt

%matplolib inline
%config InlineBackend.figure_format = "svg"

plt.figure(figsize = (8, 8))
for i in range(9):
    # data
    img, label = ds_train[i]
    img = torch.squeez(img)
    # plot
    ax = plt.subplot(3, 3, i + 1)
    ax.imshow(img.numpy())
    ax.set_title(f"label = {label}")
    ax.set_xticks([])
    ax.set_yticks([])
plt.show()
```

![img](images/mnist.png)

## 脚本风格

脚本风格的训练循环最为常见

```python
import os
import sys
import time

import numpy as np
import pandas as pd
import datetime
from tqdm import tqdm
from copy import deepcopy

import torch
from torch import nn
from torchmetrics import Accuracy
# 注：
#   多分类使用 torchmetrics 中的评估指标
#   二分类使用 torchkeras.metrics 中的评估指标

def printlog(info):
    nowtime = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    print("\n" + "==========" * 8 + "%s" % nowtime)
    print(str(info) + "\n")
```

### 模型构建

```python
net = nn.Sequential()
net.add_module("conv1", nn.Conv2d(in_channels = 1, out_channels = 32, kernel_size = 3))
net.add_module("pool1", nn.MaxPool2d(kernel_size = 2, stride = 2))
net.add_module("conv2", nn.Conv2d(in_channels = 32, out_channels = 64, kernel_size = 5))
net.add_module("pool2", nn.MaxPool2d(kernel_size = 2, stride = 2))
net.add_module("dropout", nn.Dropout2d(p = 0.1))
net.add_module("adaptive_pool", nn.AdaptiveMaxPool2d((1, 1)))
net.add_module("flatten", nn.Flatten())
net.add_module("linear1", nn.Linear(64, 32))
net.add_module("relu", nn.ReLU())
net.add_module("linear2", nn.Linear(32, 10))

print(net)
```

### 损失函数、优化器、评价指标

```python
loss_fn = nn.CrossEntropyLoss()

optimizer = torch.optim.Adam(net.parameters(), lr = 0.01)

metrics_dict = {
    "acc": Accuracy()
}
```

### 模型训练

```python
# epochs 相关设置
epochs = 20 
ckpt_path = 'checkpoint.pt'

# early_stopping 相关设置
monitor = "val_acc"
patience = 5
mode = "max"

# 训练循环
history = {}
for epoch in range(1, epochs + 1):
    printlog(f"Epoch {epoch} / {epochs}")
    # ------------------------------
    # 1.train
    # ------------------------------
    net.train()

    total_loss, step = 0, 0    
    loop = tqdm(enumerate(dl_train), total = len(dl_train))
    train_metrics_dict = deepcopy(metrics_dict) 
    for i, batch in loop: 
        features, labels = batch
        # forward
        preds = net(features)
        loss = loss_fn(preds, labels)
        # backward
        loss.backward()
        optimizer.step()
        optimizer.zero_grad()    
        # metrics
        step_metrics = {
            "train_" + name: metric_fn(preds, labels).item()
            for name, metric_fn in train_metrics_dict.items()
        }
        step_log = dict({
            "train_loss": loss.item()
        }, **step_metrics)
        # 总损失和训练迭代次数更新
        total_loss += loss.item()
        step += 1

        if i != len(dl_train) - 1:
            loop.set_postfix(**step_log)
        else:
            epoch_loss = total_loss / step
            epoch_metrics = {
                "train_" + name: metric_fn.compute().item() 
                for name, metric_fn in train_metrics_dict.items()
            }
            epoch_log = dict({
                "train_loss": epoch_loss
            }, **epoch_metrics)
            loop.set_postfix(**epoch_log)
            for name, metric_fn in train_metrics_dict.items():
                metric_fn.reset()
    for name, metric in epoch_log.items():
        history[name] = history.get(name, []) + [metric]
    # ------------------------------
    # 2.validate
    # ------------------------------
    net.eval()    

    total_loss, step = 0, 0
    loop = tqdm(enumerate(dl_val), total =len(dl_val))    
    val_metrics_dict = deepcopy(metrics_dict)     
    with torch.no_grad():
        for i, batch in loop: 
            features, labels = batch            
            #forward
            preds = net(features)
            loss = loss_fn(preds, labels)
            # metrics
            step_metrics = {
                "val_" + name: metric_fn(preds, labels).item() 
                for name,metric_fn in val_metrics_dict.items()
            }
            step_log = dict({
                "val_loss": loss.item()
            }, **step_metrics)
            # 总损失和训练迭代次数更新
            total_loss += loss.item()
            step9 += 1
            if i != len(dl_val) - 1:
                loop.set_postfix(**step_log)
            else:
                epoch_loss = (total_loss / step)
                epoch_metrics = {
                    "val_" + name: metric_fn.compute().item() 
                    for name, metric_fn in val_metrics_dict.items()
                }
                epoch_log = dict({
                    "val_loss": epoch_loss
                }, **epoch_metrics)
                loop.set_postfix(**epoch_log)
                for name, metric_fn in val_metrics_dict.items():
                    metric_fn.reset()
    epoch_log["epoch"] = epoch           
    for name, metric in epoch_log.items():
        history[name] = history.get(name, []) + [metric]
    # ------------------------------
    # 3.early-stopping
    # ------------------------------
    arr_scores = history[monitor]
    best_score_idx = np.argmax(arr_scores) if mode == "max" else np.argmin(arr_scores)
    
    if best_score_idx == len(arr_scores) - 1:
        torch.save(net.state_dict(), ckpt_path)
        print(f"<<<<<< reach best {monitor} : {arr_scores[best_score_idx]} >>>>>>", file = sys.stderr)
    
    if len(arr_scores) - best_score_idx > patience:
        print(f"<<<<<< {monitor} without improvement in {patience} epoch, early stopping >>>>>>", file = sys.stderr)
        break
    net.load_state_dict(torch.load(ckpt_path))

# 模型训练结果
dfhistory = pd.DataFrame(history)
```

## 函数风格

函数风格是在脚本风格形式的基础上做了进一步的函数封装

```python
import os
import sys
import time

import numpy as np
import pandas as pd
import datetime
from tqdm import tqdm
from copy import deepcopy

import torch
from torch import nn
from torchmetrics import Accuracy
# 注：
#   多分类使用 torchmetrics 中的评估指标
#   二分类使用 torchkeras.metrics 中的评估指标

def printlog(info):
    nowtime = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    print("\n" + "==========" * 8 + "%s" % nowtime)
    print(str(info) + "\n")
```

### 模型构建

```python
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.layers = nn.ModuleList([
            nn.Conv2d(in_channels = 1, out_channels = 32, kernel_size = 3),
            nn.MaxPool2d(kernel_size = 2, stride = 2),
            nn.Conv2d(in_channels = 32, out_channels = 64, kernel_size = 5),
            nn.MaxPool2d(kernel_size = 2, stride = 2),
            nn.Dropout2d(p = 0.1),
            nn.AdaptiveMaxPool2d((1, 1)),
            nn.Flatten(),
            nn.Linear(64, 32),
            nn.ReLU(),
            nn.Linear(32, 10),
        ])
    
    def forward(self, x):
        for layer in self.layers:
            x = layer(x)
        return x
    
net = Net()
print(net)
```

### 损失函数、优化器、评价指标

```python
loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(net.parameters(), lr = 0.01)
metrics_dict = {
    "acc": Accuracy(),
}
```

### 模型训练

```python
class StepRunner:

    def __init__(self,
                 net, 
                 loss_fn,
                 stage = "train",
                 metrics_dict = None,
                 optimizer = None,
                 lr_scheduler = None,
                 accelerator = None):
        self.net = net
        self.loss_fn = loss_fn
        self.metrics_dict = metrics_dict
        self.stage = stage
        self.optimizer = optimizer
        self.lr_scheduler = lr_scheduler
        self.accelerator = accelerator

    def setp(self, features, labels):
        # loss
        preds = self.net(features)
        loss = self.loss_fn(preds, labels)

        # backward
        if self.optimizer is not None and self.stage == "train":
            # backward
            if self.accelerator is None:
                loss.backward()
            else:
                self.accelerator.backward(loss)
            # optimizer
            self.optimizer.step()
            # learning rate scheduler
            if self.lr_scheduler is not None:
                self.lr_scheduler.step()
            # zero grad
            self.optimizer.zero_grad()
        
        # metrics
        step_metrics = {
            self.stage + "_" + name: metric_fn(preds, labels).item() 
            for name, metric_fn in self.metrics_dict.items()
        }

        return loss.item(), step_metrics

    def train_step(self, features, labels):
        """
        训练模式，dropout 层发生作用
        """
        self.net.train()
        return self.step(features, labels)

    @torch.no_grad()
    def eval_step(self, features, labels):
        """
        预测模式，dropout 层不发生作用
        """
        self.net.eval()
        return self.step(features, labels)

    def __call__(self, features, labels):
        if self.stage == "train":
            return self.train_step(features, labels)
        else:
            return self.eval_step(features, labels)


class EpochRunner:

    def __init__(self, steprunner):
        self.steprunner = steprunner
        self.stage = steprunner.stage
        self.steprunner.net.train() if self.stage == "train" else self.steprunner.net.eval()

    def __call__(self, dataloader):
        total_loss = 0
        step = 0
        loop = tqdm(enumerate(dataloader), total = len(dataloader))
        for i, batch in loop:
            if self.stage == "train":
                loss, step_metrics = self.steprunner(*batch)
            else:
                with torch.no_grad():
                    loss, step_metrics = self.steprunner(*batch)
            step_log = dict({
                self.stage + "_loss": loss
            }, **step_metrics)

            total_loss += loss
            step += 1
            if i != len(dataloader) - 1:
                loop.set_postfix(**step_log)
            else:
                epoch_loss = total_loss / step
                epoch_metrics = {
                    self.stage + "_" + name: metric_fn.compute().item()
                                       for name, metric_fn in self.steprunner.metrics_dict.items()
                }
                epoch_log = dict({
                    self.stage + "_loss": epoch_loss
                }, **epoch_metrics)
                loop.set_postfix(**epoch_log)

                for name, metric_fn in self.steprunner.metrics_dict.items():
                    metric_fn.reset()

        return epoch_log


def train_model(net, 
                optimizer, 
                loss_fn, 
                metrics_dict, 
                train_data, 
                val_data = None, 
                epochs = 10, 
                ckpt_path = 'checkpoint.pt',
                patience = 5, 
                monitor = "val_loss", 
                mode = "min"):
    history = {}
    for epoch in range(1, epochs+1):
        printlog("Epoch {0} / {1}".format(epoch, epochs))
        # ------------------------------
        # 1.train
        # ------------------------------
        train_step_runner = StepRunner(
            net = net,
            stage="train",
            loss_fn = loss_fn,
            metrics_dict = deepcopy(metrics_dict),
            optimizer = optimizer
        )
        train_epoch_runner = EpochRunner(train_step_runner)
        train_metrics = train_epoch_runner(train_data)
        for name, metric in train_metrics.items():
            history[name] = history.get(name, []) + [metric]
        # ------------------------------
        # 2.validate
        # ------------------------------
        if val_data:
            val_step_runner = StepRunner(
                net = net,
                stage = "val",
                loss_fn = loss_fn,
                metrics_dict = deepcopy(metrics_dict)
            )
            val_epoch_runner = EpochRunner(val_step_runner)
            with torch.no_grad():
                val_metrics = val_epoch_runner(val_data)
            val_metrics["epoch"] = epoch
            for name, metric in val_metrics.items():
                history[name] = history.get(name, []) + [metric]
        # ------------------------------
        # 3.early-stopping
        # ------------------------------
        arr_scores = history[monitor]
        best_score_idx = np.argmax(arr_scores) if mode=="max" else np.argmin(arr_scores)
        if best_score_idx == len(arr_scores) - 1:
            torch.save(net.state_dict(), ckpt_path)
            print(f"<<<<<< reach best {monitor} : {arr_scores[best_score_idx]} >>>>>>", file = sys.stderr)
        if len(arr_scores) - best_score_idx > patience:
            print(f"<<<<<< {monitor} without improvement in {patience} epoch, early stopping >>>>>>", file = sys.stderr)
            break 
        net.load_state_dict(torch.load(ckpt_path))

    return pd.DataFrame(history)
```

```python
df_history = train_model(
    net = net, 
    optimizer = optimizer, 
    loss_fn = loss_fn, 
    metrics_dict = metrics_dict, 
    train_data = dl_train,
    val_data = dl_val,
    epochs = 10,
    patience = 3,
    monitor = "val_acc",
    mode = "max"
)
```

## 类风格

### 类风格 torchkeras.KerasModel

使用 `torchkeras.KerasModel` 高阶 API 接口中的 `fit` 方法训练模型

`torchkeras` 安装:

```bash
$ pip install torchkeras
```

```python
import os
import sys
import time

import numpy as np
import pandas as pd
import datetime
from tqdm import tqdm
from copy import deepcopy

import torch
from torch import nn
from torchmetrics import Accuracy
from torchkeras import KerasModel
# 注：
#   多分类使用 torchmetrics 中的评估指标
#   二分类使用 torchkeras.metrics 中的评估指标

def printlog(info):
    nowtime = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    print("\n" + "==========" * 8 + "%s" % nowtime)
    print(str(info) + "\n")
```

#### 模型构建

```python
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.layers = nn.ModuleList([
            nn.Conv2d(in_channels = 1, out_channels = 32, kernel_size = 3),
            nn.MaxPool2d(kernel_size = 2, stride = 2),
            nn.Conv2d(in_channels = 32, out_channels = 64, kernel_size = 5),
            nn.MaxPool2d(kernel_size = 2, stride = 2),
            nn.Dropout2d(p = 0.1),
            nn.AdaptiveMaxPool2d((1, 1)),
            nn.Flatten(),
            nn.Linear(64, 32),
            nn.ReLU(),
            nn.Linear(32, 10),
        ])
    
    def forward(self, x):
        for layer in self.layers:
            x = layer(x)
        return x
    
net = Net()
print(net)
```

#### 损失函数、优化器、评价指标

```python
loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(net.parameters(), lr = 0.01)
metrics_dict = {
    "acc": Accuracy(),
}
```

#### 模型训练

```python
# 模型实例化
model = KerasModel(
    net = net, 
    loss_fn = loss_fn,
    metrics_dict = metrics_dict,
    optimizer = optimizer,
)

# 启动训练循环
model.fit(
    trian_data = dl_train,
    val_data = dl_val,
    epochs = 10,
    patience = 3,
    monitor = "val_acc",
    mode = "max",
)
```

### 类风格 torchkeras.LightModel

`torchkeras` 还提供了 `torchkeras.LightModel` 来支持跟多的功能。
`torchkeras.LightModel` 借鉴了 `pytorch_lightning` 库中的很多功能，
并演示了对 `pytorch_lightning` 的一种最佳实践


```python
import os
import sys
import time

import numpy as np
import pandas as pd
import datetime
from tqdm import tqdm
from copy import deepcopy

import torch
from torch import nn
from torchmetrics import Accuracy
from torchkeras import LightModel
import pytorch_lightning as pl
# 注：
#   多分类使用 torchmetrics 中的评估指标
#   二分类使用 torchkeras.metrics 中的评估指标

def printlog(info):
    nowtime = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    print("\n" + "==========" * 8 + "%s" % nowtime)
    print(str(info) + "\n")
```

#### 模型构建

```python
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.layers = nn.ModuleList([
            nn.Conv2d(in_channels = 1, out_channels = 32, kernel_size = 3),
            nn.MaxPool2d(kernel_size = 2, stride = 2),
            nn.Conv2d(in_channels = 32, out_channels = 64, kernel_size = 5),
            nn.MaxPool2d(kernel_size = 2, stride = 2),
            nn.Dropout2d(p = 0.1),
            nn.AdaptiveMaxPool2d((1, 1)),
            nn.Flatten(),
            nn.Linear(64, 32),
            nn.ReLU(),
            nn.Linear(32, 10),
        ])
    
    def forward(self, x):
        for layer in self.layers:
            x = layer(x)
        return x
    
net = Net()
print(net)
```

#### 损失函数、优化器、评价指标

```python
loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(net.parameters(), lr = 0.01)
metrics_dict = {
    "acc": Accuracy(),
}
```

#### 模型训练

```python
# 模型实例化
model = LightModel(
    net = net, 
    loss_fn = loss_fn,
    metrics_dict = metrics_dict,
    optimizer = optimizer,
)

# 设置回调函数
model_ckpt = pl.callbacks.ModelCheckpoint(
    monitor = "val_acc",
    save_top_k = 1,
    mode = "max",
)

# 设置早停
early_stopping = pl.callbacks.EarlyStopping(
    monitor = "val_acc",
    patience = 3,
    mode = "max",
)

# 设置训练参数
trainer = pl.Trainer(
    logger = True,
    min_epochs = 10,
    max_epochs = 20,
    gpus = 0,
    callbacks = [model_ckpt, early_stopping],
    enable_progress_bar = True,
)

# 启动训练循环
trianer.fit(
    model,
    dl_train,
    dl_val,
)
```
