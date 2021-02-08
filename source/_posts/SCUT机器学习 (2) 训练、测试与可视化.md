---
title: "#Machine Learning# SCUT Machine Learning(2) 训练、测试与可视化"
date: 2019-11-04 15:05:21
categories: Artificial Intelligence
tags:
- Machine Learning
thumbnail: /images/Artificial Intelligence.jpg
---



本文主要讨论**Machine Learning**的基本步骤，包括以下**三个部分**：

- **训练的三部曲**
- **测试的验证法**
- **数据的可视化**



<!-- more -->



## **Machine Learning**

### **训练的三部曲**

#### **Step 1: Model**

#### **Step 2: Goodness of Function**

#### **Step 3: Best Function**

##### **合并更新参数**

```python
# 更新参数：梯度下降
def update(self, X, y):
    m = X.shape[0]  # 标签个数
    dTheta = X.T.dot(self.model(X) - y) / m

    self.theta = self.theta - self.eta * dTheta
```

##### **分开更新参数**

```python
# 更新参数：梯度下降
def update(self, X, y):
    m = X.shape[0]  # 标签个数
    dZ = self.model(X) - y
    dW = X.T.dot(dZ) / m
    db = dZ.sum(axis=0, keepdims=True) / m

    self.W = self.W - self.eta * dW
    self.b = self.b - self.eta * db
```

### **测试的验证法**

#### **损失值**

#### **准确率**

##### **回归问题**

```python
# pass
```

##### **分类问题**

```python
# 数据的准确率
def accuracy(self, X, y, threshold):
    y_pred = self.predict(X_test, threshold)
    y_true = y

    right = 0
    total = y_pred.shape[0]
    for i in range(total):
        if y_pred[i] == y_true[i]:
            right += 1
            
	acc = right / total

	return str(acc * 100)
```

### **数据的可视化**

#### **损失值**

```python
# 数据的可视化
def visualize(self, iter_list, loss_list):
    plt.figure('program title')
    plt.title('chart title')
    ax = plt.gca()
    ax.set_xlabel('epoch')
    ax.set_ylabel('loss')
    ax.plot(iter_list, loss_list, color='r', linewidth=1, alpha=0.6)
    plt.show()
```
