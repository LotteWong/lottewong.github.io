---
title: \#Machine Learning\# Artificial Neural Network(2) Multi-layer Perception(MLP)
date: 2019-07-13 11:13:21
categories: Machine Learning
tags:
- ANN
thumbnail: /images/Artificial Neural Network.png
---



本文主要讨论**多层感知机**的概念和应用，包括以下**七个部分**：

- 神经网络结构
- 神经网络流程
- **神经网络原理**
- **神经网络问题**
- **阐述多层感知机例子**
- **实现多层感知机源码**
- 本文参考链接



<!-- more -->



## 结构

### 输入层

### 隐含层

- 只要隐含的节点足够多，就可以拟合任意函数。
- 隐含层越多，越容易拟合更复杂的函数。
- 隐含层的两个属性：每个隐含层的节点数、隐含层的层数。层数越多，每一层需要的节点数就会越少。

- ![multi-layer perception](/images/multi-layer perception.png)

### 输出层

## 流程

*我们的任务就是找到权值和偏置这些参数的值，使得输出的东西让我们满意，达到我们的要求。*

### 前向过程 -> 预测

### 反向过程 -> 训练

## 原理

*通过合理地改变权值和偏置一点点的方式来让这个神经网络最后的结果向我们预期的结果进军，激活函数的选择对于神经网络的训练来说，也是很重要的。*

![delta change](/images/delta change.png)

## 问题

### 过拟合

- Q：模型拟合训练数据，泛化性较差，测试数据效果不好。
- A：**Dropout**，在神经网络的层中，随机丢弃一些点，并且加强剩余的点，学习含有噪声的数据，增强模型的泛化性。

### 局部最优

- Q：神经网络不是一个凸优化问题，存在大量的局部最优点，然而，神经网络的局部最优正好可以达到比较好的效果，全局最优反而容易拟合。
- A：**Adagrad**，减少计算参数的复杂度，学习速率先快后慢，恰好达到局部最优。

### 梯度弥散

- Q：神经网络的的激活函数，将信息上一层传递至下一层的变换，使模型学习到更加高阶的特征。[Sigmoid函数](https://www.jianshu.com/p/4528da002c43)，在反向传播中，梯度会急剧减少；Softplus是单侧抑制，即负值转换为正值，但没有稀疏激活性，即没有0状态。真正的神经元具有稀疏性，根据输入信号，选择响应或是屏蔽。
- A：**ReLU**，y=max(0,x)，将负值抑制为0，即非激活状态，正值反馈。

## 例子

**以MNIST手写体为例子：**

- 读取数据集（28×28的灰度图）
- 展开为一维（含有784个单元）
- 对于输入层而言，我们可以“设定”784个单元，分别接受每一个像素的值
- 对于输出层而言，我们可以只设定一个神经元，用来输出这个数字是几，我们也可以设定10个神经元，分别代表0到9，然后输出这个那个数字更加有可能
- 从上面看，设计输入层和输出层还是比较轻松的，但是设计隐藏层是比较难的，后面再讨论
- 训练神经网络
  - 确定损失函数
  - 随机梯度下降
- 测试神经网络

## 源码

**以MNIST手写体为例子：**

```python
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data

"""读取数据"""

mnist = input_data.read_data_sets("MNIST_data", one_hot=True)  # 读取数据集
sess = tf.InteractiveSession()  # 创建交互会话

"""创建模型"""

in_units = 28 * 28  # 输入层有784个单元
h1_units = 300  # 隐含层1有300个单元
out_units = 10  # 输出层有10个单元

# 初始化隐藏层W1
# 使用truncated_normal加噪，防止正态分布的完全对称
W1 = tf.Variable(tf.truncated_normal([in_units, h1_units], stddev=0.1))
# 初始化隐藏层b1
# 矩阵用大写字母，向量用小写字母
b1 = tf.Variable(tf.zeros[h1_units])

# 初始化输出层W2
W2 = tf.Variable(tf.zeros([h1_units, out_units]))
# 初始化输出层b2
b2 = tf.Variable(tf.zeros([out_units]))

# 回归函数：y = wx + b
# 激活函数：ReLU
x = tf.placeholder(tf.float32, [None, in_units])  # 输入层
h1 = tf.nn.relu(tf.matmul(x, W1) + b1)  # 隐藏层
keep_prob = tf.placeholder(tf.float32)  # dropout的保留比率
h1_dropout = tf.nn.dropout(h1, keep_prob)  # 隐藏层的dropout
y = tf.nn.softmax(tf.matmul(h1, W2) + b2)  # 输出层

# 损失函数：标准的交叉熵
y_ = tf.placeholder(tf.float32, [None, out_units])  # 待训标签
cross_entroy = tf.reduce_mean(tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))  # 交叉熵
train_step = tf.train.AdagradDAOptimizer(0.3)  # 训练步长

# 初始化全部变量
tf.global_variables_initializer().run()

"""训练模型"""

# 循环训练模型3000次，每次随机采样100个数据
# Feed三类数据：数据x，标签y_，dropout的保留比率keep_prob
for i in range(3000):
    batch_xs, batch_ys = mnist.train.next_batch(100)  # 随机采样
    train_step.run({x: batch_xs, y_: batch_ys, keep_prob: 0.75})  # 训练数据

"""测试模型"""

# argmax选择最大数的索引，全部数据求平均
# keep_prob设置为1，不丢弃任何信息
correct_pred = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))
accuracy = tf.reduce_mean(tf.cast(correct_pred, tf.float32))

# 计算并输出准确率结果
print(accuracy.eval({x: mnist.test.image, y_: mnist.test.labels, keep_prob: 1.0}))
```

## 参考链接

- [深度学习笔记二：多层感知机（MLP）与神经网络结构](https://www.jianshu.com/p/a088a9d8307f)
- [MNIST数据集下载](https://blog.csdn.net/lykymy/article/details/91128833)
- [MNIST数据集介绍及读取](https://www.jianshu.com/p/d282bce1a999)
- [人工智能 - 多层感知机 MLP [3]](https://www.jianshu.com/p/ac5c1d83dc71)