---
title: "#Linux&Shell# 速查手册：常用 Linux&Shell 命令"
date: 2020-5-14 07:00:21
categories: Quick Ref Guide
tags:
- Linux
- Shell
thumbnail: /images/linux.png
---





覆盖在学习和工作中的**常用 Linux 命令**：连接 / 网络 / 进程 / 文件

---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---

## 管理

### 列举

```bash
# Ubuntu
apt list --installed
# CentOS
yum list installed
# Alpine
apk info
```

### 更新

```bash
# Ubuntu
apt-get update
apt-get -y upgrade
# CentOS
yum update
# Alpine
apk update
apk upgrade
```

### 安装

```bash
# Ubuntu
apt-get install -y ${package}
# CentOS
yum install -y ${package}
# Alpine
apk add --upgrade ${package}
```

### 卸载

```bash
# Ubuntu
apt-get --purge remove ${package}
# CentOS
yum remove ${package}
# Alpine
apk del ${package}
```

## **连接**

### ssh

```bash
ssh -p ${port} ${username}@${host}
sshpass -p ${password} ssh -p ${port} ${username}@${host} 
```

### telnet

```bash
telnet ${host} ${port}
```

### ping

```bash
ping ${host}
```

### nslookup

```bash
nslookup ${domain}
```

### curl

```bash
curl \
-X ${method} \
-H "${header_key}: ${"header_val"}" \
-d "${json_data}" \
${api_url} \
```

### mysql

```bash
mysql -h ${host} -P ${port} -u ${username} -p 
```

## **网络**

### 查看 ip

```bash
ip a |grep eth
```

### 查看 dns

```bash
cat /etc/resolv.conf |grep nameserver
```

## **进程**

### 查询 proc

```bash
ps -ef |grep ${proc_name}
```

### 查询 port

```bash
lsof -i:${port}
```

### 从 proc 查 port / 从 port 查 proc

```bash
netstat -tunlp |grep ${proc_name}
netstat -tunlp |grep ${port}
```

### 杀死进程

```bash
ps -ef | grep ${including_keyword} | grep -v ${excluding_keyword}
kill -s 9 ${pid}

ps -ef | grep ${including_keyword} | grep -v ${excluding_keyword} | awk '{print $2}' | xargs kill -s 9

kill -s 9 `pgrep ${keyword}`

pkill -9 ${keyword}
```

## **文件**

### 从服务器下载到本机

```bash
scp -r -P ${port} ${username}@${host}:/path/to/remote /path/to/local # 包括文件
rsync -a -e ssh --exclude="${pattern}" ${username}@${host}:/path/to/remote /path/to/local # 排除文件
```

### 从本机上传到服务器

```bash
scp -r /path/to/local -P ${port} ${username}@${host}:/path/to/remote # 包括文件
rsync -a -e ssh --exclude="${pattern}" /path/to/local ${username}@${host}:/path/to/remote # 排除文件
```

### 解压

zip

```bash
unzip -d /path/to/unzip ${pkg_to_unzip}
```

tar

```bash
tar -xvf ${pkg_to_untar} # 解压 tar 包
tar -zxvf ${pkg_to_untar} # 解压 tar.gz 包
tar -jxvf ${pkg_to_untar} # 解压 tar.bz2 包
```

### 加压

zip

```bash
zip -r /path/to/unzip ${pkg_to_zip}
zip -d /path/to/zip ${file_to_delete} # 从压缩包中删除文件
zip -m /path/to/zip ${file_to_append} # 向压缩包中添加文件
```

tar

```bash
tar -cvf ${pkg_to_tar} # 加压 tar 包
tar -zcvf ${pkg_to_tar} # 加压 tar.gz 包
tar -zjvf ${pkg_to_tar} # 加压 tar.bz2 包
```

## **文本**

### 查看

json

```bash
cat ${file} | python -m json.tool
```

log

```bash
less ${log_file}
vim ${log_file}
tail -n ${number} ${log_file}
tail -f ${log_file}
```

### 编辑

复制

```bash
ctrl + Insert
```

粘贴

```bash
shift + Insert
```

移动

```bash
gg # 首行
G # 末行
shift + ^ # 行首
shift + $ # 行末
```

搜索

```bash
/pattern # 向前搜索
？pattern # 向后搜索
n # 上一个
N # 下一个
```

查看缩进和行尾

```bash
: set list
```

### 处理

分隔

```bash
awk -F:"${seperator}" '/${pattern}/${command}' ${file}
```

替换

```bash
sed "s/${pattern}/${substr}/g"
```

---