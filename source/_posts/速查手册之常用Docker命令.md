---
title: "#Docker# 速查手册：常用 Docker 命令"
date: 2020-5-28 07:00:21
categories: Quick Ref Guide
tags:
- Docker
thumbnail: /images/docker_qrg.png
---





覆盖在学习和工作中的**常用 Docker 命令**：镜像 / 容器 / 存储 / 网络

---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---

## **镜像**

### 查看

```bash
# 查看全部镜像详情
docker images
docker images -a
# 查看全部镜像标识
docker images -aq
# 查看指定镜像详情
docker inspect ${registry}:${tag}
docker inspect ${image_id} 
```

### 制作

```bash
docker build --network ${network_mode} -f /path/to/dockerfile -t ${registry}:${tag} /path/to/data
```

### 保存

```bash
# 具备元数据等
docker save -o ${output}.tar ${registry}:${tag}
docker save -o ${output}.tar ${image_id}
```

### 加载

```bash
# 具备元数据等
docker load -i ${input}.tar
```

### 修改

```bash
docker tag ${image_id} ${registry}:${tag}
```

### 删除

```bash
# 删除全部镜像
docker rmi -f $(docker images -aq)
# 删除指定镜像
docker rmi -f ${registry}:${tag}
docker rmi -f ${image_id}
```

### Dockerfile

```dockerfile
# Example: a sample python image
FROM ubuntu # basic image
MAINTAINER lottewong <lottewong21@gmail.com> # author info

# install sqlite3, python3, pip3 and other tools
RUN apt-get update && \
    apt-get install -y python3 \
                       python3-dev \
                       python3-pip \
                       openssl \
                       libssl-dev \
                       libffi-dev \
                       net-tools \
                       sqlite3 && \
    ln -s /usr/bin/python3 /usr/bin/python && \
    ln -s /usr/bin/pip3 /usr/bin/pip && \
    apt install -y vim lsof curl

# make data and log directories
RUN mkdir -p /data/test-proj/dependencies && \
    mkdir -p /var/log/test-proj
WORKDIR /data/test-proj # fix work directory

# install test-proj depending packages
COPY dependencies dependencies # copy files from host to container
RUN cd dependencies && \
    pip install -r requirements.txt && \
    tar -zvxf some-package.tar.gz && \
    rm -rf some-package.tar.gz && \
    cd /data/test-proj/dependencies/some-package && python setup.py install

# start init script (mount file when running container)
# CMD ["sh", "start.sh"]
# ENTRYPOINT ["sh", "start.sh"]

# RUN: 镜像层命令，应用于 docker build 时
# CMD: 容器层命令，应用于 docker run 时（当指定命令时会被忽略）
# ENTRYPOINT: 容器层命令，应用于 docker run 时（无论何种情况都被执行）

# Shell形式执行命令: <instruction> <command>
# Exec形式执行命令: <instruction> ["executable", "param1", "param2", "param3", ...]
```

## **容器**

### 查看

```bash
# 查看全部容器详情
docker ps
docker ps -a
# 查看全部容器标识
docker ps -aq
```

### 运行

```bash
# -p 可以映射多个端口
# -v 可以挂载多个目录
# -d 后台运行
# -it 交互运行
docker run --name=${container_name} -d -p ${host_port}:${ctnr_port} -v /path/to/host:/path/to/ctnr ${registry}:${tag} (${command})
docker run --name=${container_name} -it -p ${host_port}:${ctnr_port} -v /path/to/host:/path/to/ctnr ${image_id} (${command})

# --rm 运行后就删除
# Example: docker run --rm busybox nslookup baidu.com
```

### 执行

```bash
# Example: docker exec -it test /bin/bash
docker exec -it ${container_name} ${command}
docker exec -it ${container_id} ${command}
```

### 启动

```bash
docker start ${container_name}
docker start ${container_id}
```

### 暂停

```bash
docker stop ${container_name}
docker stop ${container_id}
```

### 重启

```bash
docker restart ${container_name}
docker restart ${container_id}
```

### 删除

```bash
# 删除全部容器
docker rm -f $(docker ps -aq)
# 删除指定容器
docker rm -f ${container_name}
docker rm -f ${container_id}
```

### 提交

```bash
docker commit -a "${author}" -m "${message}" ${container_name} ${registry}:${tag}
docker commit -a "${author}" -m "${message}" ${container_id} ${registry}:${tag}
```

### 导出

```bash
# 没有元数据等
docker export ${container_name} > ${export}.tar
docker export ${container_id} > ${export}.tar
```

### 导入

```bash
# 没有元数据等
cat ${import}.tar | docker import - ${registry}:${tag}
```

### 传输

```bash
# 从容器到主机
docker cp ${container}:/path/to/ctnr /path/to/host
# 从主机到容器
docker cp /path/to/host ${container}:/path/to/ctnr
```

### 日志

```bash
docker logs ${container_name}
docker logs ${container_id}
```

## **存储**

- *To be continued...*

## **网络**

- *To be continued...*

---