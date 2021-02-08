---
title: "#DevOps# 实习日常记录之流水线篇(0) 综述"
date: 2020-6-2 07:00:21
categories: DevOps
tags:
- CI/CD
thumbnail: /images/CI_CD.png

---



本文主要记录工作中常用的 Linux 工具与命令



---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---

## 授权操作

- 解决 Jet Brains IDE 无权限的报错：`sudo chown -R $username /var/run/`

## 文件操作

### 移动

- `mv old_path new_path`

### 复制

- 复制单文件：`cp file file.bak`
- 复制文件夹：`cp -r dir dir.bak`

### 更名

- `mv old_path new_path`

### 上传

- 单文件上传服务器：`scp -P $port local_path/file_name username@hostname_or_hostip:origin_path/file_name`
- 文件夹上传服务器：`scp -P $port -r local_path/dir_name username@hostname_or_hostip:origin_path/dir_name`

### 下载

- 服务器单文件下载：`scp -P $port username@hostname_or_hostip:origin_path/file_name local_path/file_name`
- 服务器文件夹下载：`scp -P $port username@hostname_or_hostip:origin_path/dir_name local_path/dir_name`

## 进程管理

- 查看状态：`supervisor status`
- 服务重启：`supervisor restart all`

## 快捷命令

- `Tab`：补全
- `Ctrl + Insert`：复制
- `Shift + Intert`：粘贴
- `/`：查找
- `?`：查找
- `n`：上一个
- `N`：下一个
- `w`：保存
- `q`：退出
- `yy`：复制整行
- `dd`：删除整行
- `:0`：文件首
- `:$`：文件尾
- `0`：行首
- `$`：行尾
- `gg`：文件首
- `GG`：文件尾

## 连接到远程服务器

- telnet连接：`telnet hostname_or_hostip port`
- ssh连接：`ssh username@hostname_or_hostip -p port`

## 连接到远程数据库

- `mysql -h $hostname_or_hostip -P $port db_name -u $username -p`

## 查看进程与端口

### 查看特定进程

- `ps -ef |grep $pname`

### 查看端口占用

- `lsof -i:$port`
- `netstat -tunlp |grep $port`

### 查看进程的端口

- 找到进程号：`ps -ef |grep $pname`

- 找到端口号：`netstat -tunlp |grep $pid`

## 查看日志文件

### tail -f

- `tail -f $filename`
- 适用于来查看动态变化

### less

- `less filename`
- 适用于来查看静态变化

### vi / vim

- `vi / vim filename`
- 适用于来编辑 all 文本

### json.tool

- `cat filename |python -m json.tool`
- 适用于来格式 json 文本

