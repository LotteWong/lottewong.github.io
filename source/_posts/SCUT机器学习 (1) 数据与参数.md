---
title: "#Machine Learning# SCUT Machine Learning(1) 数据与参数"
date: 2019-11-04 15:00:21
categories: Artificial Intelligence
tags:
- Machine Learning
thumbnail: /images/Artificial Intelligence.jpg
---



本文主要讨论**Machine Learning**的基本步骤，包括以下**两个部分**：

- **读取处理数据**
- **初始化各参数**



<!-- more -->



## **Machine Learning**

### **读取处理数据**

#### **本地数据**

##### **合并更新参数**

``` python
local_file = '$local_dataset_path'

def load_data(path):
    # 读取本地原始数据集
    raw_data = datasets.load_svmlight_file(path)  # 返回scipy.matrix
    或
    raw_data = datasets.load_files(path)  # 返回numpy.ndarray
    
    # 分开处理特征和标签
    X, y = np.c_[raw_data[0].A, np.ones((raw_data[0].shape[0], 1))], raw_data[1].reshape(-1, 1)
    或
    X, y = np.c_[raw_data[0], np.ones((raw_data[0].shape[0], 1))], raw_data[1].reshape(-1, 1)

    # 切分训练集和验证集
    return model_selection.train_test_split(X, y, test_size=0.3, random_state=0)
```

##### **分开更新参数**

```python
local_file = '$local_dataset_path'

def load_data(path):
    # 读取本地原始数据集
    raw_data = datasets.load_svmlight_file(path)  # 返回scipy.matrix
    或
    raw_data = datasets.load_files(path)  # 返回numpy.ndarray"""
   	
    # 分开处理特征和标签
    X, y = raw_data[0].A, raw_data[1].reshape(-1, 1)
    或
    X, y = raw_data[0], raw_data[1].reshape(-1, 1)
	
    # 切分训练集和验证集
    return model_selection.train_test_split(X, y, test_size=0.3, random_state=0)
```

#### **网络数据**

##### **合并更新参数**

```python
# pass
```

##### **分开更新参数**

```python
# pass
```

#### **随机抽样**

```python
# 随机抽样
def random_select(self, X, y, sample_size=1024):
    m = X.shape[0]  # 标签个数
    i_list = random.sample([idx for idx in range(n)], batch_size)  # 下标序列

    X_list = []
    y_list = []
    for i in i_list:
        X_list.append(X[i])
        y_list.append(y[i])
        X_array = np.array(X_list)
        y_array = np.array(y_list)

        return X_array, y_array
```

### **初始化各参数**

#### **不抽样**

##### **合并更新参数**

```python
# 加载参数
def __init__(self, eta, epo, batch_size, n):
    self.eta = eta  # 学习速率
    self.epo = epo  # 迭代次数
    self.batch_size = batch_size  # 批处理量
    self.theta = np.zeros((n, 1))  # 权重偏置
```

##### **分开更新参数**

```python
# 加载参数
def __init__(self, eta, epo, batch_size, n):
    self.eta = eta  # 学习速率
    self.epo = epo  # 迭代次数
    self.batch_size = batch_size  # 批处理量
    self.W = np.zeros((n, 1))  # 权重参数
    self.b = 0  # 偏置参数
```

#### **要抽样**

##### **合并更新参数**

```python
# 加载参数
def __init__(self, eta, epo, sample_size, batch_size, n):
    self.eta = eta  # 学习速率
    self.epo = epo  # 迭代次数
    self.sample_size = sample_size  # 抽样大小
    self.batch_size = batch_size  # 批处理量
    self.theta = np.zeros((n, 1))  # 权重偏置
```

##### **分开更新参数**

```python
# 加载参数
def __init__(self, eta, epo, sample_size, batch_size, n):
    self.eta = eta  # 学习速率
    self.epo = epo  # 迭代次数
    self.sample_size = sample_size  # 抽样大小
    self.batch_size = batch_size  # 批处理量
    self.W = np.zeros((n, 1))  # 权重参数
    self.b = 0  # 偏置参数
```
