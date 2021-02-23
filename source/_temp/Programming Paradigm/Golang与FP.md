---
title: "#Programming Paradigm# Let's Go: POP in Golang"
date: 2019-12-12 07:00:21
categories: Programming Paradigm
tags:
- Golang
thumbnail: /images/golang.png
---



对比**适用于 Linux 的 Windows 子系统（Windows Subsystem for Linux, WSL）**的两个版本 - **WSL 1** 和 **WSL 2** ，主要分为以下**六个部分**：

- 前情提要
- 相关简介
- **性能对比**
- **架构对比**
- 几点总结
- 参考链接

---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---



## **前情提要**

<br/>

*前阵网上冲浪围观 M$ Build 2019 的 WSL Session ，感受了一波  Microsoft ❤ Open Source，遂决定根据讲座和文档内容当一次复读机讨论一下 WSL ( 啊我的塑料英语\_(:з)∠)\_。*

## **相关简介**

<br/>

> **The Windows Subsystem for Linux** lets developers run a GNU/Linux environment -- including most command-line tools, utilities, and applications -- directly on Windows, unmodified, without the overhead of a virtual machine.

- **适用于 Linux 的 Windows 子系统（Windows Subsystem for Linux, WSL）**可以简单地理解为在 Windows 上提供了运行 Linux 的平台。由于是 Subsystem ，与裸机装 Linux Distribution 相比，存在功能限制和性能打折的情况。

> You can:
>
> ......
>
> - **Invoke Windows applications** using a Unix-like command-line shell.
>
> - **Invoke GNU/Linux applications** on Windows.
>
> ......

- 对于懒得开虚拟机or装双系统 + ECS渣渣级配置 + 非尊贵苹果用户而言， WSL 还是很有吸引力的 (虽然也很多坑。除了**对 Linux 本身有需求**外，上面提到的 **Windows 和 Linux 互相invoke** 也很有意思， WSL 因此大大降低了原本配置两个文件系统的繁琐程度。

## **性能对比**

<br/>

**WSL 2** 相比 **WSL 1**，主要**优化了访问文件系统的速度**以及**提供了更完整的系统调用接口**。

### File system performance

![File system performance](/images/file_system_performance.png)

### Full system call compatibility

![Full system call compatibility](/images/full_system_call_compatibility.png)

## **架构对比**

<br/>

### WSL 1

![WSL 1 Architecture](/images/wsl1_architecture.png)

- **WSL 1** 的大体思路是，在一台 Windows 主机上安装 Linux 发行版本，依赖**中间层的驱动器**完成 Linux Namespace 和 Windows Kernel 之间的**通信捕获和指令翻译**（比如 `path` 和 `flag` ），作用范围包括但不局限于系统调用、文件系统、权限管理和网络配置等。
- 这样的设计存在一些**缺陷**：

  1. 某些翻译无可避免地需要付出时空代价，甚至由于两种内核的设计思想存在冲突而无法进行。
  2. 由于 Linux Kernel 更新得非常快，单靠 M$ Team 自己实现翻译，开发进度远远滞后于实际生产需求。

【更多详见 👉 [WSL从入门到...(2) Dive into WSL 1](https://lottewong.github.io/2019/12/13/WSL%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0...(2)/)】

### WSL 2

![WSL 2 Architecture](/images/wsl2_architecture.png)

- **WSL 2** 的大体思路是，**开启 Hyper-V 功能**，Windows Hyper-V Container（包含Windows Kernel和Windows Usermode）**持续运行**，Linux Hyper-V Container（包含Linux Kernel和Linux Usermode）**随用随开**，两者**通过 Socket 通信**。

- 这样的设计存在一些**缺陷**：

  1. 需要处理器提供虚拟化选项，一些 arm64 架构的芯片不支持该功能。
  2. Windows 开Hyper-V后将无法同时运行 VMware 和 Virtual Box 等工具，因为它们都要求独占 Hypervisor 才能够运行。
  3. 虚拟化伴随一系列一致性问题，比如权限管理和网络配置等等。

【更多详见 👉 [WSL从入门到...(3) Dive into WSL 2](https://lottewong.github.io/2019/12/13/WSL%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0...(3)/)】

#### Linux Kernel

- 用 Linux Kernel 是为了**解决 WSL 1 的遗留问题**（<u>"Uses real Linux kernel for improved performance and perfect compatibility"</u>），翻译不好搞就“拿来主义”。 M$ 甚至为 WSL 2 定制了 Linux Kernel  (没想到吧.jpg

  ![Shipping a Linux Kernel with Windows](/images/linux_kernel.png) 

#### Virtualization

- 要**使两种内核能够共存**最直接的思路就是使用虚拟化技术，而 WSL 2 采用了一种有别于传统虚拟机和新兴容器的新(?)方法 - Lightweight utility VM，其特点是集成度高（关联 Windows 服务）且只在运行时启动（否则被销毁回收）。

  ![Lightweight utility VM](/images/virtualization.png)
  1. 相比传统虚拟机，内存占用更小、启动速度更快以及可以同时运行更多实例。
  
  2. 相比新兴容器，基于 Hyper-V 面向 server 场景，本质还是 VM ，隔离得更彻底。
  
  ![Hyper-V Containers](/images/hyper_v_container.png)

## **几点总结**

<br/>

根据PM的说法， **WSL 1** 和 **WSL 2** 会并行维护，既适用于**桌面版**也可以运行在**服务器**。下面简单回顾两者的**区别**：

|                       | WSL 1                 | WSL 2                  |
| --------------------- | --------------------- | ---------------------- |
| 核心技术              | Pico provider drivers | Lightweight utility VM |
| 同一实例（Container） | ✓                     | ✗                      |
| 额外支持（虚拟化）    | ✗                     | ✓                      |
| 文件系统              | 访问慢                | 访问快                 |
| 系统调用              | 缺少                  | 完整                   |
| 权限管理              | 相同                  | 不同                   |
| 网络配置              | 相同                  | 不同                   |

<br/>

后续两节将分别再深入介绍 **WSL 1 和 WSL 2 的实现细节**。那么我先占坑逃了（（（

## **参考链接**

- [The new Windows subsystem for Linux architecture: a deep dive](https://mybuild.techcommunity.microsoft.com/sessions/77003)
- [Windows Subsystem for Linux Documentation](https://docs.microsoft.com/en-us/windows/wsl/about)

---

