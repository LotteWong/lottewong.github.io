---
title: \#Configuration\# 如何令本地代码在服务器端启动：以GitHub作中转
date: 2019-06-03 07:00:21
categories: Configuration
tags:
- Git
- GitHub
- SSH
- Linux
thumbnail: /images/architecture.png
---





利用**GitHub**作为中转，实现本地端和服务器的双向互通，主要分为以下**五个部分**：

- 前期的准备
- **从远程克隆**
- 本地端修改
- **推送到远程**
- 服务器操作



<!-- more -->



## **前期的准备**

### **Step 1: 环境需要**

- [Git](https://git-scm.com/downloads)
- [GitHub](https://github.com/)
- <u>SSH客户端</u>
  - Windows用户：[PuTTY](https://www.putty.org/)
  - Unix用户：<u>cmd</u>

### **Step 2: Git和GitHub配置**

#### 配置Git

- [安装Git](https://www.liaoxuefeng.com/wiki/896043488029600/896067074338496)

#### 配置GitHub

- [添加远程库](https://www.liaoxuefeng.com/wiki/896043488029600/898732864121440)

### **Step 3: SSH配置**

#### Windows用户

- [使用 PuTTY 远程登录管理服务器](https://cnzhx.net/blog/putty-basic-usage/)

#### Unix用户

- [如何使用SSH登录远程服务器](https://blog.csdn.net/u011054333/article/details/52443061)

## **从远程克隆**

### **Step 1: 在GitHub上Fork对应repo**

- 选择特定repo，如：https://github.com/LotteWong/Bone-Metastasis-Neural-Network

- 点击`Fork`按钮，即可在自己的GitHub看到对应的repo

  ![Fork](/images/Fork.png)

### **Step 2: git clone到本地目录**

- 在自己的GitHub打开对应repo，复制`SSH key`

  ![SSH key](/images/SSH-key.png)

- 切到想要下载到的目录，使用`git clone`命令

  ![git clone](/images/git-clone.png)

## **本地端修改**

### **Step 1: 本地edit**

- 在本地修改文件

### **Step 2: 本地add**

- [学习`git add`命令](https://www.liaoxuefeng.com/wiki/896043488029600/896827951938304)

### **Step 3: 本地commit**

- [学习`git commit`命令](https://www.liaoxuefeng.com/wiki/896043488029600/896827951938304)

## **推送到远程**

### **Step 1: bash查看远程服务器**

![git remote](/images/git-remote.png)

### **Step 2: push到GitHub**

![git push](/images/git-push.png)

## **服务器操作**

### **Step 1: SSH登录远程服务器**

- 按照**前期的准备**中的教程登录服务器

  ![SSH login](/images/SSH-login.png)

### **Step 2: pull到服务器**

- `pwd`：查看当前路径

- `ll`: 查看当前路径下的目录和文件

- `cd $path`：切换特定路径

- `git clone`：**若目录中还没有repo**，按上述步骤做过的，在服务器进行`git clone`*【PS：因为之前已经`git clone`过，所以服务器上已经有`Bone-Metastasis-Neural-Network`目录】*

- `git pull`：**若目录中已经有repo**，使用`git pull`命令将GitHub的更新拉到服务器

  ![git pull](/images/git-pull.png)

### **Step 3: 运行py文件**

- 切到py文件所在路径

- `python $filename`*【PS：现在`Network.py`是有bug的，所以没run起来】*

  ![run](/images/run.png)

## **总结**

**每次需要更新文件重复下述步骤：**

- Step 1: **在本地**修改文件，`git add`并`git commit`
- Step 2: **在本地**`git push`到origin
- Step 3: **在服务器**从origin中`git clone` / `git pull`
- Step 4: **在服务器**切到py文件所在路径`python $filename`启动

## **附录**

1. [Git教程](https://www.liaoxuefeng.com/wiki/896043488029600)

2. [GitHub教程](http://rogerdudler.github.io/git-guide/index.zh.html)

3. [Git练习repo](https://github.com/LotteWong/LearnGit)

4. [Git命令速查](https://www.jianshu.com/p/091518780e2a)

5. [Linux命令速查](https://www.jianshu.com/p/77f47f4a6706)

6. <u>使用GPU加速</u>

   1. **使用tensorflow的代码：**

      ```python
      # limit GPU memory
      tf_config = tf.ConfigProto() 
      tf_config.gpu_options.allow_growth = True
      with tf.Session(config=tf_config) as sess:
      ```

   2. **使用keras的代码：**

      ```python
      # limit GPU memory
      import keras.backend.tensorflow_backend as KTF
      config = tf.ConfigProto()  
      config.gpu_options.allow_growth=True   
      session = tf.Session(config=config)
      KTF.set_session(session)
      ```