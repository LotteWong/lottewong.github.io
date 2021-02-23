---
title: "#Docker# LinuxOne上的Docker初体验"
date: 2019-06-18 07:00:21
categories: Cloud Computing
tags:
- Docker
thumbnail: /images/docker.png
---



在**LinuxOne**上利用**Docker**部署应用与服务，主要分为以下**五个部分**：

- 环境准备 Environment
- **Docker原理 Theory**
- **Docker使用 Usage**
- **Docker实战 Practice**
- 注意事项 Notices

---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---



## **环境准备 Environment** 

### 申请Github账号并配置好本地Git

- [廖雪峰Git教程](https://www.liaoxuefeng.com/wiki/896043488029600)

### 申请IBM ID账号并开通开发者账号

> *该步骤主要提供接口权限*

1. Register IBM ID （统一邮箱）

2. Create an API Developer Portal account （统一邮箱）

3. Apps

   - Create new App

   - Configure the App: 
     - Title: $TITLE
   - Sumbit
     - Client ID
     - Client Secret

4. API Products

   - Use banking API
     - Subscribe Default Plan
     - Select Previous App
   - Subscribe

### 申请IBM LinuxOne账号

> *该步骤主要提供部署环境*

#### Virtual Machine

1. Login
2. Virtual Services

   - Manage Instances
   - Create Instances
   - Configure the Instance
     - Type: General purpose VM
     - Instance Name: $INSTANCE_NAME
     - Instance Description: $INSTANCE_DESCRITION
     - Image: RHEL7.6
     - SSH Key Pairs: Create → Save → Select → Create

   - Check the Instance
     - Status: Active
     - Linux User: linux1
     - IP Address: 148.100.xxx.xxx

#### Private Cloud

1. Login

2. Catalog
   - openmplbank

3. Configure
   - Input Release name
   - Select Target Namespace

4. Install

5. View Helm Release
   - Deployment
     - AVAILABLE = 1
   - Launch

### 安装Node.js环境

1. 安装Node.js: 

   [Download | Node.js](https://link.zhihu.com/?target=https%3A//nodejs.org/en/download/)

2. 检查是否安装成功:

    ```bash
    $ node -v
    $ npm -v
    ```

### 安装SSH登录工具

- Windows：可选[PuTTY](https://www.zybuluo.com/abelsu7/note/1476987?tdsourcetag=s_pctim_aiomsg)或[Xshell](https://lottewong.github.io/2019/05/17/_files/xshell连接linux1.doc)或[WSL](https://docs.microsoft.com/en-us/windows/wsl/faq)
- Linux：`ssh -i /path/to/key/keyname.pem linuxusername@serveripaddress`

## **Docker原理 Theory**

### 工作流程

![Flowchart](/images/docker_flowchart.png)

### 名词辨析

| 概念                   | 含义                                                         |
| :--------------------- | :----------------------------------------------------------- |
| Docker 镜像(Images)    | Docker镜像是用于创建Docker容器的模板。                       |
| Docker 容器(Container) | Docker容器是独立运行的一个或一组应用。                       |
| Docker 客户端(Client)  | Docker客户端通过命令行或者其他工具使用Docker API，与Docker的守护进程通信。 |
| Docker 主机(Host)      | Docker主机是一个物理或者虚拟的机器用于执行Docker守护进程和容器。 |

## **Docker使用 Usage**

### 安装

1. 安装docker

    ```bash
    # 下载Docker归档包

    wget ftp://ftp.unicamp.br/pub/linuxpatch/s390x/redhat/rhel7.3/docker-17.05.0-ce-rhel7.3-20170523.tar.gz

    # 解压Docker归档包

    tar -xzvf docker-17.05.0-ce-rhel7.3-20170523.tar.gz

    # 迁移Docker归档包
    # !!! 这里直接cp到/usr/bin就好，因为/usr/local/bin不在PATH环境变量里 !!!

    cp docker-17.05.0-ce-rhel7.3-20170523/docker* /usr/bin/
    ```

2. 安装docker-compose

    ```bash
    # 查看python-setuptools

    yum info python-setuptools

    # 安装python-setuptools

    yum install -y python-setuptools

    # 安装pip

    easy_install pip

    # 网速过慢的话先禁用掉 IPv6

    echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6

    # 升级backports.ssl_match_hostname

    pip install backports.ssl_match_hostname --upgrade --ignore-installed

    # 先安装依赖，不然会报错

    yum install python-devel libffi-devel

    # 安装docker-compose

    pip install docker-compose==1.13.0

    # 查看docker-compose安装情况和版本信息

    docker-compose version
    ```

### 命令

- 后台启动daemon进程

  ```bash
  # -g 设置Docker Daemon运行时的根目录
  # & 放在命令后面表示设置此进程为后台进程
  
  docker daemon -g /local/docker/lib &
  ```

- 命令查看docker信息

  ```bash
  # 查看当前机器docker版本老旧
  
  docker version
  
  # 检查后台有无docker进程运行
  
  ps aux | grep docker
  ```

- docker镜像处理

  ```bash
  # 查看所有镜像
  
  docker images
  
  # 拉取远程镜像
  
  docker image pull repository:tag
  
  # 构建本地镜像
  
  docker build -t "repository:tag" ./
  ```

- docker容器处理

  ```bash
  # 查看所有容器
  
  docker ps
  
  # 创建运行容器
  
  docker run image
  
  # 停止运行容器
  
  docker stop container
  ```

- docker服务处理

  ```bash
  # 查看所有服务
  
  docker-compose ps
  
  # 创建运行服务
  
  docker-compose up
  
  # 停止运行服务
  
  docker-compose down
  ```

- docker build

  ```bash
  # -t 在新容器内指定一个伪终端或终端
  
  docker build -t "repository:tag" ./
  ```

- docker run

  ```bash
  # -d 开启daemon模式
  # -i 允许你对容器内的标准输入 (STDIN) 进行交互
  # -p 指定端口映射规则
  
  docker run -d -i -p ipadress1:port1/protocal:ipadress2:port2/protocal repository:tag
  ```

- docker-compose up

  ```bash
  # -d 开启daemon模式
  # 端口映射和镜像来源都写在了.yml配置文件
  
  docker-compose up -d
  ```

- docker exec

  ```bash
  # -i 允许你对容器内的标准输入 (STDIN) 进行交互
  # -t 在新容器内指定一个伪终端或终端
  
  docker exec –it container bash
  ```

## **Docker实战 Practice**

### 切换权限和路径，配置用户习惯

```bash
# 切为根用户，否则没有权限

sudo su

# 切到家目录，否则难找文件

cd ~

# RHEL 7.6已经自带安装了VIM 7.4，启动命令是vi，习惯用vim命令的同学可以先设置一下别名[当前生效]

alias vim='vi'

# RHEL 7.6已经自带安装了VIM 7.4，启动命令是vi，习惯用vim命令的同学可以先设置一下别名[永久生效]
# !!! 可以将alias vim='vi'加到~/.bashrc中 !!!
source ~/.bashrc
```

### 安装并运行 WebSphere Liberty**（练习使用docker run）**

```bash
# 手动拉取websphere-liberty镜像到本地

docker image pull s390x/websphere-liberty:webProfile7

# 后台运行容器，并指定端口映射规则

docker run -d -p 80:9080 -p 443:9443 s390x/websphere-liberty:webProfile7

# 浏览器访问http://[LinuxOne Host IP]，即可看到WebSphere Liberty的界面
```

### 安装并运行 WordPress**（练习使用docker-compose up）**

```bash
# 创建docker-compose.yml

vim docker-compose.yml

# 编辑docker-compose.yml

version: '2'
services:
  wordpress:
    image: s390x/wordpress
  ports:
    - 8080:80 # 将本地 8080 端口映射到容器的 80 端口
  environment:
    WORDPRESS_DB_PASSWORD: example
  mysql:
    image: brunswickheads/mariadb-5.5-s390x
  environment:
    MYSQL_ROOT_PASSWORD: example

:wq

# 查看docker-compose.yml

cat docker-compose.yml

# 创建wordpress目录方便整理

mkdir wordpress
mv docker-compose.yml wordpress/
cd wordpress/

# 根据docker-compose.yml中定义的服务启动容器

docker-compose up -d

# 创建完成后，查看相关容器的状态

docker-compose ps

# 浏览器访问http://[Your LinuxONE IP Address]:8080，即可看到 WordPress 的页面
```

### 安装并运行 Todo App**（熟悉MEAN Stack + Docker架构）**

```bash
# 为方便管理文件，切换到家目录

cd ~

# 从Github拉取源码到本地使用

git clone https://github.com/IBM/Cloud-Native-Workloads-on-LinuxONE

# 迁移源码文件夹到家目录

cp -r Cloud-Native-Workloads-on-LinuxONE/files/mean-docker ./

# 安装显示目录树的插件包

yum install -y tree

# 显示mean-docker的目录树

tree mean-docker

mean-docker
├── docker-compose.yml # docker-compose 配置文件
├── express-server
│   ├── app
│   │   ├── models
│   │   │   └── todo.js
│   │   └── routes.js
│   ├── config
│   │   └── database.js
│   ├── Dockerfile # docker image 生成文件
│   ├── license
│   ├── package.json
│   ├── public
│   │   ├── index.html # 前端文件
│   │   └── js
│   │       ├── controllers
│   │       │   └── main.js # 后端文件
│   │       ├── core.js
│   │       └── services
│   │           └── todos.js # 数据库文件
│   ├── README.md
│   └── server.js
└── README.md # 说明文档
8 directories, 14 files

# 修改Angular.js成国内镜像源

vim mean-docker/express-server/public/index.html
src="//cdn.bootcss.com/angular.js/1.2.16/angular.min.js"

# 查看Dockerfile的内容

cd express-server/
ls
vim Dockerfile

# 编辑Dockerfile的内容

# Expose the port the app runs in
EXPOSE 8081
...
...
# Express listening port
ENV PORT 8081

:wq

# 重新构建镜像

cd mean-docker
docker-compose down  # 停止正在运行的容器
docker-compose build # 先重新构建镜像
docker-compose up    # 再基于新镜像重新启动容器

# 查看docker-compose.yml的内容

cd mean-docker/
ls
vim docker-compose.yml

# 编辑docker-compose.yml的内容
# 因为之前本地的8080端口被 WordPress 占用了，所以这里我们使用8081端口

...
...
ports:
- "8081:8081" # 本地 8081 端口映射到 express 容器的 8081 端口
...
...

:wq

# 启动指定服务

docker-compose up -d

# 使用docker-compose ps命令查看启动的容器

docker-compose ps

# 浏览器访问http://[ip of machine]:8081，即可看到你的 TODO-List App
```

### Todo App前端插入数据、后端处理数据、数据库查数据**（熟悉MEAN Stack + Docker前后端数据库交互）**

```bash
......

# 在运行的容器中执行命令

docker exec –it meandocker_database_1 bash

# 进入MongoDB

> mongo

# 查看数据库

> show dbs

# 指定数据库

> use docker-mean

# 枚举数据表

> show tables

# 查看元祖集

> db.todos.find()

# 指定元祖项

> db.todos.find({"key": "value"})
```

### 本地部署金融微服务**（熟悉Localhost → Micro-services模式）**

```bash
# Fork ICp-banking-microservices 到自己账号下，将你Fork的项目git clone至本地

# Github配置过SSH
git clone git@github.com:LotteWong/ICp-banking-microservices.git

# Github未配置SSH
git clone https://github.com/YOUR_USERNAME/ICp-banking-microservices

# 在banking-application/public/js/bankingAPI.js中填入你的Client ID和Client Secret

# 进入ICp-banking-microservices/banking-application目录，安装npm依赖

npm install

# 如果出现npm代理设置错误，重新设置代理即可

npm config set registry "http://registry.npmjs.org/"

# 进入ICp-banking-microservices/banking-application目录，启动应用

node app.js

# 浏览器访问http://localhost:3000，即可访问应用

# 随便选择一个customer ID测试，若有JSON格式的数据返回，则说明API可用。如果出错可自排查，可能是ID和Secret不匹配（前往开发者页面的应用程序页面中验证ID和Secret）或者浏览器不支持网速较慢之类（更换浏览器更换网络源）
```

### 远程部署金融微服务**（熟悉Docker → Micro-services模式）**

```bash
# 在非 LinuxOne 的本机将项目推送至 Github 远程仓库
# !!! 实际上不应该把Client ID和Client Secret这种密钥类型的数据推到 Github 上，这里为了方便实验暂时这么做，以后切勿模仿。 !!!

git add public/js/bankingAPI.js
git commit -m "Update of bankingAPI.js"
git push origin master

# 先登录你的 LinuxONE 主机实例，为方便管理文件，切换到家目录

cd ~

# 将你 Fork 后又更新的代码拉取到本地

# Github配置过SSH
git clone git@github.com:LotteWong/ICp-banking-microservices.git

# Github未配置SSH
git clone https://github.com/YOUR_USERNAME/ICp-banking-microservices

# 构建 Docker 镜像

docker build -t "respository:tag" ./

# 查看 Docker 镜像

docker images

# 启动 Docker 容器

docker run -p 3000:3000 respository:tag

# 查看 Docker 容器

docker ps

# 浏览器访问http://[LinuxOne Host IP]:3000，即可访问应用

# 随便选择一个customer ID测试，若有JSON格式的数据返回，则说明API可用。如果出错可自排查，可能是ID和Secret不匹配（前往开发者页面的应用程序页面中验证ID和Secret）或者浏览器不支持网速较慢之类（更换浏览器更换网络源）
```

### 云端部署金融微服务**（熟悉Cloud → Micro-services模式）**

```bash
# 到 ICP 中部署好的应用，点击启动，浏览器会自动跳转到分配的端口

# 之后就和此前的实验一样了，只不过你的应用是部署在 ICP 上，由 Kubernetes 自动维护可用的 Pod 数量
```

## **注意事项 Notices**

- **端口映射**就是将主机的IP地址的一个端口映射到局域网中一台机器，当用户访问这个IP的这个端口时，服务器自动将请求映射到对应局域网分机。
- `.pem`为通用证书格式，`ppk`为PuTTY下面的专有格式。两者都为**SSH Key Pairs**格式，内含公钥和密钥。
- **镜像是类，容器是对象，服务是对象集。**`Dockerfile`用于构建镜像；`docker-compose.yml`用于组织镜像；`docker run`用于启动容器；`docker-compose up`用于启动服务。
- 使用Docker需要非常注意**卷的管理**，如果采用默认匿名的方式而不指定卷的位置，服务器的容量很快就会被每次重新生成的同一镜像给爆掉。
- **MEAN Stack**包括`MongoDB（数据库）`、`Express.js（路由）`、`AngularJS（前端）`和`Node.js（后端）`。本次实验最终项目友链 👉 [SCUT Online Bank Application](https://github.com/LotteWong/SCUT_Online_Bank_Application)。

---

