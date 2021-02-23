---
title: "#WSL# WSL从入门到...(4) Quick Start on WSL"
date: 2019-12-15 07:00:21
categories: DevOps
tags:
- WSL
thumbnail: /images/wsl.jpg
---



**安装**、**配置**并**使用** WSL ，主要分为以下**五个部分**：

- 前情提要
- **安装发行版本**
- **编写配置脚本**
- **使用相关命令**
- 参考链接

---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---



## **前情提要**

<br/>

*终于结束漫长的听力翻译和阅读理解，开始真正搞机（期待地搓手手.gif。本文将解锁 WSL 1 初体验（发现 Windows build 版本好像快落后了一个世纪，WSL 2 装不了惹。*

## **安装发行版本**

<br/>

<u>配置相关</u>：

- Windows 10 家庭中文版 1903 
- 64 位操作系统，基于 x64 的处理器
- WSL 1
- Ubuntu 16.04 LTS
- Alpine WSL

### Step 1: 开启支持 WSL 选项

- **方法一：控制面板设置**

  1. “控制面板” -> “程序” -> “启用或关闭 Windows 功能” -> “适用于 Linux 的 Windows 子系统”

  2. 重启电脑

- **方法二：输入命令设置**

  1. “Windows PowerShell（以管理员身份运行）” -> `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`

  2. 重启电脑

### Step 2: 安装 Linux 发行版本

- **方法一：Microsoft Store 安装**
  1. 打开应用商店 
  2. 选择发行版本
     - [Ubuntu 16.04 LTS](https://www.microsoft.com/store/apps/9pjn388hp8c9)
     - [Ubuntu 18.04 LTS](https://www.microsoft.com/store/apps/9N9TNGVNDL3Q)
     - [OpenSUSE Leap 15](https://www.microsoft.com/store/apps/9n1tb6fpvj8c)
     - [OpenSUSE Leap 42](https://www.microsoft.com/store/apps/9njvjts82tjx)
     - [SUSE Linux Enterprise Server 12](https://www.microsoft.com/store/apps/9p32mwbh6cns)
     - [SUSE Linux Enterprise Server 15](https://www.microsoft.com/store/apps/9pmw35d7fnlx)
     - [Kali Linux](https://www.microsoft.com/store/apps/9PKR34TNCV07)
     - [Debian GNU/Linux](https://www.microsoft.com/store/apps/9MSVKQC78PK6)
     - [Fedora Remix for WSL](https://www.microsoft.com/store/apps/9n6gdm4k2hnc)
     - [Pengwin](https://www.microsoft.com/store/apps/9NV1GV1PXZ6P)
     - [Pengwin Enterprise](https://www.microsoft.com/store/apps/9N8LP0X93VCP)
     - [Alpine WSL](https://www.microsoft.com/store/apps/9p804crf0395)
  3. 安装发行版本
- **方法二：Command-Line/Script 安装**
  - [Manually download Windows Subsystem for Linux distro packages](https://docs.microsoft.com/en-us/windows/wsl/install-manual)

### Step 3: 配置 Linux 发行版本

1. 启动已安装好的 Linux 发行版本
   - 点击启动：
     - 菜单栏单击运行发行版本
     - 磁贴板单击运行发行版本
   - 命令启动：
     - `bash.exe` ：默认目录为 Windows 用户目录
     - `distro.exe`：默认目录为 Linux 用户目录
     - `wsl.exe`：允许 Windows 和 Linux 的命令混用
2. 等待 Linux 发行版本解压缩和初始化
3. 创建 Linux 下的新用户及其密码
   - 管理员用户之一
   - 每次进入的默认用户
   - 与 Windows 用户名无关
4. 定期更新 Linux 下的 Package
   - `sudo apt update && sudo apt upgrade`

*PS: 常见报错解决页 👉  [WSL troubleshooting page](https://docs.microsoft.com/en-us/windows/wsl/troubleshooting)*

## **编写配置脚本**

<br/>

WSL 支持使用 `wsl.conf` 来初始化自动挂载和网络配置两大功能。`wsl.conf` 位于 `/etc`目录，如果文件已存在，WSL 会自动读取适配；如果文件不存在，可以自行在目录内新建；如果文件出错了，WSL 会忽略配置文件。

### 自动挂载

<u>Section Label</u>: `[automount]`

| key        | value                          | default      | notes                                                        |
| :--------- | :----------------------------- | :----------- | :----------------------------------------------------------- |
| enabled    | boolean                        | true         | `true` causes fixed drives (i.e `C:/` or `D:/`) to be automatically mounted with DrvFs under `/mnt`. `false` means drives won’t be mounted automatically, but you could still mount them manually or via `fstab`. |
| mountFsTab | boolean                        | true         | `true` sets `/etc/fstab` to be processed on WSL start. /etc/fstab is a file where you can declare other filesystems, like an SMB share. Thus, you can mount these filesystems automatically in WSL on start up. |
| root       | String                         | `/mnt/`      | Sets the directory where fixed drives will be automatically mounted. For example, if you have a directory in WSL at `/windir/`and you specify that as the root, you would expect to see your fixed drives mounted at `/windir/c` |
| options    | comma-separated list of values | empty string | This value is appended to the default DrvFs mount options string. **Only DrvFs-specific options can be specified.** Options that the mount binary would normally parse into a flag are not supported. If you want to explicitly specify those options, you must include every drive for which you want to do so in /etc/fstab. |

### 网络配置

<u>Section label</u>: `[network]`

| key                | value   | default | notes                                                        |
| :----------------- | :------ | :------ | :----------------------------------------------------------- |
| generateHosts      | boolean | `true`  | `true` sets WSL to generate `/etc/hosts`. The `hosts` file contains a static map of hostnames corresponding IP address. |
| generateResolvConf | boolean | `true`  | `true` set WSL to generate `/etc/resolv.conf`. The `resolv.conf` contains a DNS list that are capable of resolving a given hostname to its IP address. |

### 交互操作

<u>Section label</u>: `[interop]`

These options are available in Insider Build 17713 and later.

| key               | value   | default | notes                                                        |
| :---------------- | :------ | :------ | :----------------------------------------------------------- |
| enabled           | boolean | `true`  | Setting this key will determine whether WSL will support launching Windows processes. |
| appendWindowsPath | boolean | `true`  | Setting this key will determine whether WSL will add Windows path elements to the $PATH environment variable. |

### 具体示例

```sh
# Enable extra metadata options by default
[automount]
enabled = true
root = /windir/
options = "metadata,umask=22,fmask=11"
mountFsTab = false

# Enable DNS – even though these are turned on by default, we’ll specify here just to be explicit.
[network]
generateHosts = true
generateResolvConf = true
```

## **使用相关命令**

### 查看版本列表

1. `wsl -l`, `wsl --list`：可用的 Linux 发行版
2. `wsl --list --all`： 未可用和已可用的 Linux 发行版
3. `wsl --list --running`：运行的 Linux 发行版

### 设置默认版本

- `wsl -s <DistributionName>`, `wsl --setdefault <DistributionName>`

### 重新安装版本

1. `wsl --unregister <DistributionName>`
2. 重复“安装发行版本”的“Step 3: 配置 Linux 发行版本”

### 指定参数登录

1. `wsl -d `, `wsl --distribution `：指定版本登录
2. `wsl -u `, `wsl --user`：指定用户登录

### 交互操作选择

1. `$ echo 0 > /proc/sys/fs/binfmt_misc/WSLInterop`：停用交互操作
2. `$ echo 1 > /proc/sys/fs/binfmt_misc/WSLInterop`：开启交互操作

### Windows命令行运行Linux工具

- `wsl <LinuxCommand>`

  ```powershell
  C:\temp> wsl ls -la | findstr "foo"
  -rwxrwxrwx 1 root root     14 Sep 27 14:26 foo.bat
  
  C:\temp> dir | wsl grep foo
  09/27/2016  02:26 PM                14 foo.bat
  ```

### Linux命令行运行Windows工具

- `<WindowsCommand>`

  ```sh
  $ cmd.exe /C dir
  <- contents of C:\ ->
  
  $ PING.EXE www.microsoft.com
  Pinging e1863.dspb.akamaiedge.net [2600:1409:a:5a2::747] with 32 bytes of data:
  Reply from 2600:1409:a:5a2::747: time=2ms
  ```

*PS: 以上适用于 "<u>Windows 10 Version 1903 and later</u>", "<u>Versions Earlier than Windows 10 Version 1903</u> "的对应目录请查看文末的参考链接。Windows Insiders Builds 17063 起支持 Windows 和 Linux 共享环境变量；Fall Creators Update 起 Windows `Path` 将加入 Linux `$PATH` 。*

## **参考链接**

- [Windows Subsystem for Linux Documentation](https://docs.microsoft.com/en-us/windows/wsl/about)

---

