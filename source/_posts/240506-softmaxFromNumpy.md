---
title: numpy实现softmax回归
toc: true
date: 2024-05-06 22:20:05
updated: 2024-05-06 22:20:05
cover:
thumbnail:
categories:
tags: Python
---

从**线性代数**到**概率论与数理统计**
<!-- more -->


花了点时间仔细推导了公式，实现一个简单的softmax回归模型，用于手写数字识别。该模型是一个单层神经网络，通过梯度下降法不断迭代更新参数，使得损失函数最小化。

也就单层神经网络可以手动计算梯度，多层神经网络不得不借助自动微分工具。如若用上Pytorch等高级框架的话，整个模型几行代码就可实现。

参考链接
- [Softmax回归](https://zh-v2.d2l.ai/chapter_linear-networks/softmax-regression.html)
- [一文搞懂极大似然估计](https://zhuanlan.zhihu.com/p/26614750)


## 算法实现

### 导入相关库
```python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
```

### 模型框架
```python
def softmax(x):
    x_exp = np.exp(x)
    return x_exp / np.sum(x_exp, axis=1, keepdims=True)

def net(x, w, b):
    return softmax(np.dot(x, w) + b)

# Loss function (cross-entropy)
def loss(y, y_hat):
    return -np.sum(y * np.log(y_hat))

# Gradient of the loss function with respect to w and b
def gradient(x, y, y_hat):
    return np.dot(x.T, y_hat - y), np.sum(y_hat - y, axis=0)
```

### 小批量梯度下降法

```python 代码较长 >folded
def batch_gradient_descent(
    x: np.ndarray, 
    y: np.ndarray, 
    learning_rate=0.1, 
    epochs=10, 
    batch_size=32
):

    def get_batch(batch_size=batch_size):
        for i in range(0, len(x), batch_size):
            if i + batch_size < len(x):
                yield x[i:i + batch_size], y[i:i + batch_size]
            else:
                yield x[i:], y[i:]

    input_feat, output_feat = x.shape[1], y.shape[1]
    w = np.random.randn(input_feat, output_feat)
    b = np.zeros(output_feat)
    loss_lst = []

    for _ in range(epochs):
        loss_epoch = 0
        for x_batch, y_batch in get_batch(batch_size):
            y_hat = net(x_batch, w, b)
            w_grad, b_grad = gradient(x_batch, y_batch, y_hat)
            w -= (1 / x_batch.shape[0]) * learning_rate * w_grad
            b -= (1 / x_batch.shape[0]) * learning_rate * b_grad
            loss_epoch += loss(y_batch, y_hat)
        loss_lst.append(loss_epoch / batch_size)

    plt.plot(loss_lst)
    plt.xlabel('Epochs')
    plt.ylabel('Loss')
    plt.show()

    return w, b
```

## 测试
在经典的手写数字识别(MNIST)数据集上进行测试，该数据集可从[kaggle](https://www.kaggle.com/competitions/digit-recognizer/data?select=train.csv)下载

### 数据处理
- 随机选取95%的数据作训练集，剩余5%作测试集
- 特征向量中每个元素取值为-128到127，将其归一化到0-1
- 标签为0-9的整数，将其转化为独热编码
```python
mnist_data = pd.read_csv(r'your\path\to\train.csv')

train_data = mnist_data.sample(frac=0.95)
test_data = mnist_data.drop(train_data.index)

train_X = train_data.drop('label', axis=1).to_numpy(dtype=np.float32)
test_X = test_data.drop('label', axis=1).to_numpy(dtype=np.float32)
# normalize
train_X = (train_X + 128) / 255.0
test_X = (test_X + 128) / 255.0

train_y = train_data['label'].to_numpy()
test_Y = test_data['label'].to_numpy()
# convert to one-hot encoding
train_y = np.eye(10)[train_y].astype(np.float32)
test_Y = np.eye(10)[test_Y].astype(np.float32)
```

### 训练
```python
w, b = gradient_descent(train_X, train_y)

y_hat = net(test_X, w, b)
y_pred = np.argmax(y_hat, axis=1)
y_true = np.argmax(test_Y, axis=1)

accuracy = np.mean(y_pred == y_true)
print(f"Accuracy: {accuracy:.6f}")
```

训练耗时2s，超参没有细调，最终准确率在**0.86**左右

<div align="center">
<a href="https://imgse.com/i/pkZa8zD"><img src="https://s21.ax1x.com/2024/05/10/pkZa8zD.png" alt="pkZa8zD.png" style="zoom:70%"/></a>
</div>


## 数学建模

### 分类问题与独热编码
样本空间特征向量 $x \in \mathbb{R}^d$ 和标签 $y \in \mathbb{R}$，如果标签仅0或1两种，则其概率分布用数学公式表达为
<div>
$$
P(y | x) = 
\begin{cases}
    p & \text{if } y = 1 \\
    1 - p & \text{if } y = 0
\end{cases}
$$
</div>

也可以用一个简单公式 $P(y|x) = yp + (1-y)(1-p)$ 来表示。那么对于标签数多于2的情况呢？

独热编码将标签转化为一个长度为 $q$ 的向量 $y \in \mathbb{R}^q$，其中只有一个元素为1，其他元素为0，值为1元素的索引就是原标签的值。于是，给定一个样本 $x$，其标签 $y$ 的概率分布可以表示为
$$
P(y | x) = \prod_{i=1}^q p_j y_j
$$

代码中，通过左乘单位矩阵 `np.eye(q)` 实现独热编码
```python
# convert to one-hot encoding
train_y = np.eye(10)[train_y]
```

### softmax回归
所谓回归，就是找出一个函数，使其代表因变量和自变量之间的关系。softmax回归就是在单层神经网络上叠加了一个softmax函数，将输出转化为概率分布。

单层神经网络是一个线性函数：
$$
o = x W + b
$$
其中 $W \in \mathbb{R}^{q\times d}$ 为权重，$b \in \mathbb{R}^q$ 为偏置，$o \in \mathbb{R}^q$ 为该层网络的输出。


softmax函数将 $o$ 转化为概率分布：
$$
p_j = \text{softmax}(o) = \frac{\exp(o_j)}{\sum_{k=1}^q \exp(o_k)}
$$
这时输出 $p$ 的每个元素都在0和1之间，且和为1，“看起来”才像一个概率分布。

代码实现：
```python
def softmax(x):
    x_exp = np.exp(x)
    return x_exp / np.sum(x_exp, axis=1, keepdims=True)

def net(x, w, b):
    return softmax(np.dot(x, w) + b)
```


### 最大似然估计
从一个装有红球和白球的袋中，不放回地抽取100个球，最终抽出70个红球，那么袋中红球的比例最可能是多少？凭直觉，这个比例应该是0.7。但是，袋中红球的比例完全可能是其他值，比如0.6或0.8，那么如何从数学上证明0.7是最合理的呢？

抽到红球的个数 $X$ 服从二项分布 $B(100, p)$，其中 $p$ 为红球的比例，那么“抽出70个红球”这一事件的概率为 $P(X=70) = C_{100}^{70} p^{70} (1-p)^{30}$，这个概率称为似然函数，我们的目标是找到一个 $p$ 使得似然函数最大，即最大似然估计。随后只需求一次导，就可证明 $p=0.7$ 时似然函数最大。

对于具有n个样本的数据集，其似然函数为

$$
P(Y|X) = P(y^{(1)}, \ldots, y^{(n)} | x^{(1)}, \ldots, x^{(n)}) = \prod_{i=1}^n P(y^{(i)} | x^{(i)})
$$

对数似然函数为
<div>
$$
\begin{split} 
    \log P(Y|X) &= \sum_{i=1}^n \log P(y^{(i)} | x^{(i)}) \\
    &= \sum_{i=1}^n \log \prod_{j=1}^q p_j{y_j^{(i)}} \\
    &= \sum_{i=1}^n y^{(i)} \log p^{(i)} \\
    &\triangleq - \sum_{i=1}^n l^{(i)}(y, p) 
\end{split} 
$$
</div>

注意推导中利用了 $y^{(i)}$ 是独热编码的性质，即仅有一个元素为1。最大化似然函数，等价于最小化对数似然函数的负数，即上式最后一行定义的**交叉熵损失函数**：
<div>
$$
\begin{split}
    l^{(i)}(y, p) &\triangleq -\sum_{j=1}^q  y_j \log p_j \\
    &= -\sum_{j=1}^q y_j \log \frac{\exp(o_j)}{\sum_{k=1}^q \exp(o_k)} \\
    &= -\sum_{j=1}^q y_j o_j + \sum_{j=1}^q y_j\log \sum_{k=1}^q \exp(o_k) \\
    &= -\sum_{j=1}^q y_j o_j + \log \sum_{k=1}^q \exp(o_k) \\
\end{split}
$$
</div>

注意最后一步的推导再次利用了独热编码的性质。

方便起见，本程序中计算交叉熵损失函数时，直接使用$l^{(i)}(y, p) = -\sum_{j=1}^q  y_j^{(i)} \log p_j^{(i)}$，后续推导结果则用于梯度计算。以下代码计算的是n个样本的交叉熵损失函数 $L = \sum_{i=1}^n l^{(i)}(y, p)$
```python
# Loss function (cross-entropy)
def loss(y, y_hat):
    return -np.sum(y * np.log(y_hat))
```

### 梯度下降法
梯度下降法是一种迭代算法，通过不断迭代来最小化损失函数。根据**链式求导**法则：

<div>
$$
\begin{split}
    & \frac{\partial L}{\partial W} = \frac{\partial L}{\partial o} \cdot \frac{\partial o}{\partial W} \\
    & \frac{\partial L}{\partial b} = \frac{\partial L}{\partial o} \cdot \frac{\partial o}{\partial b}
\end{split}
$$
</div>

先只考虑一个样本的损失函数。根据之前的推导中对 $l^{(i)}(y, p)$ 的简化，有
<div>
$$
\begin{split}
    \frac{\partial l^{(i)}}{\partial o} 
    &= \frac{\partial}{\partial o} \left( -\sum_{j=1}^q y_j o_j + \log \sum_{k=1}^q \exp(o_k) \right) \\
    &= -y + \frac{\exp(o)}{\sum_{k=1}^q \exp(o_k)} \\
    &= p - y
\end{split}
$$
</div>

可以发现单个样本损失函数对 $o$ 求导时刚好会出现 $\text{softmax}$ 函数，最终可以得到一个非常简洁的结果。

再让 $o = x W + b$ 分别对 $W$ 和 $b$ 求导，有

<div>
$$
\begin{split}
    \frac{\partial o}{\partial W} &= x \\
    \frac{\partial o}{\partial b} &= 1
\end{split}
$$
</div>

于是对于参数 $W$ 和 $b$，其梯度分别为
<div>
$$
\begin{split}
    \partial_{W} l^{(i)}(W, b) &= x (p - y) \\
    \partial_{b} l^{(i)}(W, b) &= (p - y)
\end{split}
$$
</div>

而对于n个样本，其梯度为单个样本梯度的累加，即
<div>
$$
\begin{split}
    \partial_{W} L^{(i)}(W, b) &= \sum_{i=1}^n x^{(i)} (p^{(i)} - y^{(i)}) \\
    \partial_{b} L^{(i)}(W, b) &= \sum_{i=1}^n (p^{(i)} - y^{(i)})
\end{split}
$$
</div>


这是个十分简洁优美的结果，程序实现上也很简洁
```python
# Gradient of the loss function with respect to w and b
def gradient(x, y, y_hat):
    return np.dot(x.T, y_hat - y), np.sum(y_hat - y, axis=0)
```

最终，通过梯度下降法，不断迭代更新参数 $W$ 和 $b$，使得损失函数最小化：
```python
for _ in range(epochs):
    loss_epoch = 0
    for x_batch, y_batch in get_batch(batch_size):
        y_hat = net(x_batch, w, b)
        w_grad, b_grad = gradient(x_batch, y_batch, y_hat)
        w -= (1 / x_batch.shape[0]) * learning_rate * w_grad
        b -= (1 / x_batch.shape[0]) * learning_rate * b_grad
        loss_epoch += loss(y_batch, y_hat)
    loss_lst.append(loss_epoch / batch_size)
```
其中有三个超参数需要调整：学习率 `learning_rate`、迭代次数 `epochs` 和批量大小 `batch_size`。