---
title: numpy实现softmax回归
toc: true
date: 2024-05-06 22:20:05
updated: 2024-05-06 22:20:05
cover:
thumbnail:
categories:
tags:
---

先挖坑，后面补
<!-- more -->

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

    def get_batch():
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
        for x_batch, y_batch in get_batch():
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
<!-- ![alt text](../../public/img/2024/loss.png) -->