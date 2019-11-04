---
title: \#Machine Learning\# SCUT Machine Learning(3) 算法与工程
date: 2019-11-04 15:10:21
categories: Artificial Intelligence
tags:
- Machine Learning
thumbnail: /images/Artificial Intelligence.jpg

---



本文主要讨论**Machine Learning**的基本步骤，包括以下**两个部分**：

- **优化与调参学**
- **软件工程之戒**



<!-- more -->



## **Machine Learning**

### **优化与调参学**

#### **数据的预处理**

##### **归一化**

##### **标准化**

##### **中心化**

##### **正则化**

#### **参数的初始化**

##### **全零初始**

```python
self.theta = np.zeros((n, 1))  # 权重偏置
```

##### **全一初始**

```python
self.theta = np.ones((n, 1))  # 权重偏置
```

##### **随机初始**

```python
self.theta = np.random.random((n, 1))  # 权重偏置
```

##### **正态初始**

```python
self.theta = np.random.normal(size=(n, 1))  # 权重偏置
或
self.theta = np.random.randn(n, 1)  # 权重偏置
```

#### **梯度下降选择**

> 参考链接：[深度学习最全优化方法总结比较（SGD，Adagrad，Adadelta，Adam，Adamax，Nadam）](https://zhuanlan.zhihu.com/p/22252270)
>
> 经验之谈：
>
> - 对于稀疏数据，尽量使用学习率可自适应的优化方法，不用手动调节，而且最好采用默认值。
> - SGD通常训练时间更长，但是在好的初始化和学习率调度方案的情况下，结果更可靠。
> - 如果在意更快的收敛，并且需要训练较深较复杂的网络时，推荐使用学习率自适应的优化方法。
> - Adadelta，RMSprop，Adam是比较相近的算法，在相似的情况下表现差不多。
> - 在想使用带动量的RMSprop，或者Adam的地方，大多可以使用Nadam取得更好的效果。

##### **单样本**

- 梯度下降速度最快，迭代计算时间最长

```python
# 单样本梯度下降
def sgd(self, X_array, y_array):
    for X_item, y_item in zip(X_array, y_array):
        X_single = X_item.reshape(1, -1)
        y_single = y_item.reshape(1, -1)
        self.update(X_single, y_single)
```

##### **小批量**

- 梯度下降速度适中，迭代计算时间适中

```python
# 小批量梯度下降
def mbgd(self, X_array, y_array, batch_size=64):
    for idx in range(0, X_array.shape[0], batch_size):
        X_batch = X_array[idx:idx+batch_size]
        y_batch = y_array[idx:idx+batch_size]
        self.update(X_batch, y_batch)
```

##### **全样本**

- 梯度下降速度最慢，迭代计算时间最短

```python
# 全样本梯度下降
def fbgd(self, X_array, y_array):
    self.update(X_array, y_array)
```

##### **Adagrad**

- 自定义学习率，后期更新很慢

```python
# Adagrad
def adagrad(self, X_array, y_array):
    sigma = 1e-7  # 平滑项
    for X_item, y_item in zip(X_array, y_array):
        X_single = X_item.reshape(1, -1)
        y_single = y_item.reshape(1, -1)

        grad = X_single.T.dot(self.model(X_single) - y_single) / 1
        self.grad_sum = self.grad_sum + np.sum(grad * grad)
        dTheta = -self.eta * (grad / math.sqrt(self.grad_sum + sigma))

        self.theta = self.theta + dTheta
```

##### **RMSprop**

- 进化Adagrad，变种Adadelta

```python
# RMSprop
def adadelta(self, X_array, y_array):
    sigma = 1e-7  # 平滑项
    gama = 0.5  # 冲顶项

    for X_item, y_item in zip(X_array, y_array):
        X_single = X_item.reshape(1, -1)
        y_single = y_item.reshape(1, -1)

        grad = X_single.T.dot(self.model(X_single) - y_single) / 1
        self.grad_sum = gama * self.grad_sum + (1 - gama) * np.sum(grad * grad)
        dTheta = -self.eta * (grad / math.sqrt(self.grad_sum + sigma))

```

##### **Adadelta**

- 自定义学习率，后期更新变快

```python
# Adadelta
def adadelta(self, X_array, y_array):
    sigma = 1e-7  # 平滑项
    gama = 0.9  # 冲顶项

    for X_item, y_item in zip(X_array, y_array):
        X_single = X_item.reshape(1, -1)
        y_single = y_item.reshape(1, -1)

        grad = X_single.T.dot(self.model(X_single) - y_single) / 1
        self.grad_sum = gama * self.grad_sum + (1 - gama) * np.sum(grad * grad)
        dTheta = -self.eta * (grad / math.sqrt(self.grad_sum + sigma))

        self.theta = self.theta + dTheta

```

##### **Adamax**

- 简单范围的Adam，学习率有确定范围，参数比较平稳

```python
# Adamax
def adamax(self, X_array, y_array):
    sigma = 1e-7  # 平滑项
    u = 0.5  # 冲顶项
    v = 0.5  # 冲顶项

    for X_item, y_item in zip(X_array, y_array):
        X_single = X_item.reshape(1, -1)
        y_single = y_item.reshape(1, -1)

        grad = X_single.T.dot(self.model(X_single) - y_single) / 1
        self.grad = u * self.grad + (1 - u) * grad
        self.grad_sum = np.maximum(v * self.grad_sum, np.sum(grad))
        m_t_hat = self.grad / (1 - u)
        n_t_hat = self.grad_sum / (1 - v)
        dTheta = - self.eta / (n_t_hat + sigma) * m_t_hat

        self.theta = self.theta + dTheta

```

##### **Adam**

- 带动量项的RMSprop，学习率有确定范围，参数比较平稳

```python
# Adam
def adam(self, X_array, y_array):
    sigma = 1e-7  # 平滑项
    u = 0.5  # 冲顶项
    v = 0.5  # 冲顶项

    for X_item, y_item in zip(X_array, y_array):
        X_single = X_item.reshape(1, -1)
        y_single = y_item.reshape(1, -1)

        grad = X_single.T.dot(self.model(X_single) - y_single) / 1
        self.grad = u * self.grad + (1 - u) * grad
        self.grad_sum = v * self.grad_sum + (1 - v) * np.sum(grad * grad)
        m_t_hat = self.grad / (1 - u)
        n_t_hat = self.grad_sum / (1 - v)
        dTheta = - self.eta * (m_t_hat / (math.sqrt(n_t_hat) + sigma))

        self.theta = self.theta + dTheta

```

### **软件工程之戒**

#### **模块**

#### **函数**