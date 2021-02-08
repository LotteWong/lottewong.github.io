---
title: "#WSL# WSL从入门到...(3) Dive into WSL 2"
date: 2019-12-14 07:00:21
categories: DevOps
tags:
- WSL
thumbnail: /images/wsl.jpg
---



对 **WSL 2 的一些实现细节**进行补充说明，主要分为以下**五个部分**：

- 前情提要
- 背景
- **架构与答疑**
- 总结
- 参考链接

---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---



## **前情提要**

<br/>

*前面已经简要介绍了 WSL 1 和 WSL 2 的区别，本节将聚焦于 WSL 2的实现细节。点开官方博客开始收获惊(~~da~~)喜(~~keng~~)，才疏学浅写错也不要打我 (预先感谢各位勘误 逃*

【更多信息 👉 [WSL从入门到...(1) WSL 1 vs WSL 2](https://lottewong.github.io/2019/12/12/WSL%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0...(1)/)】

## **背景**

### 功能

> ...... still provides **the same user experience** as in WSL 1 (the current widely available version).

### 性能

> Its primary goals are to **increase file system performance**, as well as **adding full system call compatibility**.

### 兼容

> Individual Linux distros can be run either as a WSL 1 distro, or as a WSL 2 distro, can be **upgraded or downgraded at any time**, and you can run WSL 1 and WSL 2 distros **side by side**. 

## **架构**

<br/>

![WSL 2 Architecture](/images/wsl2_component.png)

**工作流大概是：**

1. 启动 LXSS Manager Service **跟踪 Linux Distribution 的安装、运行和卸载**。
2. 启动 Host compute service 和 Host Network Service **创建和初始化 Lightweight utility VM 实例**。
3. 被封装好的（<u>"shipping in box"</u>）基于 Linux Kernel 的**虚拟机运行**。
4. LXSS Manager Service **映射、加载和管理 Linux Distribution 的文件系统**。
5. 以上准备工作完成后，`wsl.exe` 和 `bash` 之间将**通过 `socket` 传递指令**来完成作业。

*PS: Lightweight utility VM 只会在需要时创建运行，如果 `bash` 或 `wsl.exe` 退出，一段时间后将会被回收。*

### 访问 Windows 文件

![Accessing Windows Files with WSL 2](/images/accessing_windows_files_with_wsl2.png)

- **Windows** 主机启动一个文件**服务器**，此时 **Linux** 作为**客户端**发送请求，两者**使用 9P 协议**进行通信。

- **效果看起来像**把 Windows 的 `C` 盘挂载到了 Linux 的 `/mnt` 目录下。访问 Linux 的 `/mnt/c` 就是在 访问 Window 的 `C:\` 。

*PS: 特别地，如果在 `bash` 内启动 `cmd.exe` ，由于 Windows 和 Linux 的可执行文件格式不同（前者是 PE 或 PE32+ ，后者是 ELF32 或 ELF64），`.exe` 文件无法直接运行。WSL 2 的处理方法实际上是：Linux 下的 `bash` 向 `wsl.exe` 发送 Interop command，Windows 下的 `wsl.exe` 解析后启动 `.exe` 文件。*

### 访问 Linux 文件

![Accessing Linux Files with WSL 2](/images/accessing_linux_files_with_wsl2.png)

- **Linux** 虚拟机启动一个文件**服务器**，此时 **Windows** 作为**客户端**发送请求，两者**使用 9P 协议**进行通信。
- **效果看起来像**在 Windows 上开一般的虚拟机，对应的目录（在 `Network` 内）可以被文件资源管理器轻松地访问。

*PS: 从上述中可见，WSL 2 访问文件的中心观点 —— “两个系统都做各自可以做 / 擅长做的事情，遇事不决就通过`socket`发指令吱一声，而不会选择简单粗暴地翻译。”*

## **答疑**

<br/>

> <u>Q1</u>: Microsoft 的 Linux Team 根据 WSL 2 **针对 Linux Kernel 做了哪些调整**？
>
> <u>A1</u>: 首先，和 WSL 1 一样，WSL 2 **本身并不提供 Linux 发行版本**，用户需自行**前往 Windows Store 下载**或**脚本安装自定义的发行版本**；其次，WSL 2 的 Linux Kernel **基于稳定的 [version 4.19](https://www.kernel.org/)**，并在**启动时间、内存占用和需支持设备数等方面**都进行了相应的优化；最后， WSL 2 将**通过 Windows 更新来推送新服务**，并由 Microsoft 及其它专业的商业伙伴**提供安全监控和保障**。

> <u>Q2</u>: 可以**在虚拟机中使用 WSL 2** 吗？
>
> <u>A2</u>: 可以，但要确保**开启 `nested virtualization` 选项**（在  Windows PowerShell （以管理员身份运行）中输入 `Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true` 命令）。

> <u>Q3</u>: WSL 1 中的**配置文件 `wsl.conf`** 在 WSL 2 中仍可以使用吗？
>
> <u>A3</u>: 可以，比如自动挂载磁盘、开启或关闭 interop command和更改挂载目录等操作都将**继续支持**，更多 WSL 配置项可见 [Distro Management](https://docs.microsoft.com/en-us/windows/wsl/wsl-config) 。

> <u>Q4</u>: WSL 2 将支持 Docker 技术，**WSL 2 的 Docker 会有什么特别的限制吗**？
>
> <u>A4</u>: 如果启动的是 Windows 的 Docker ，将按照 Window 的 Docker 工作；如果启动的是 Linux 的 Docker ，将按照 Linux 的 Docker 工作。尽管 Windows 的 Docker 和 Linux 的 Docker 实现方式有所出入（由于篇幅有限，将不展开说明），但**原则上 WSL 2 不存在对 Docker 的特别限制**。

> <u>Q5</u>: 有没有针对 WSL 2 的**硬件支持**，比如访问并加速GPU并之类？
>
> <u>A5</u>: **暂未完善**，但持续开发中（但 WSL 1 支持串行端口和USB设备的访问，有需要可以先凑合用）。

> <u>Q6</u>: 有没有统一 Windows 文件和 Linux 文件**权限管理**的解决方案？
>
> <u>A6</u>: 在 **WSL 1** 中，因为是同一主机不存在系统边界（映射相对容易），权限管理是通过**中间层的驱动器提供额外服务**实现的；在 WSL 2 中，因为是不同主机而存在系统边界（映射相对困难），WSL 2 采取的策略是**保持各自系统原有的访问权限**，通过**补充一些上层协议（比如 SSH）**进行辅助管理，新的 Release 版本已经支持 "[Sharing SSH keys between Windows and WSL 2](https://devblogs.microsoft.com/commandline/sharing-ssh-keys-between-windows-and-wsl-2/)"。

> <u>Q7</u>: 有没有 WSL 2 对**网络配置**的支持？
>
> <u>A7</u>: 在 **WSL 1** 中，因为是同一主机，网络出口的 IP 地址是相同的，**相关操作也接近原生** ；在 **WSL 2** 中，因为是不同主机，网络出口的 IP 地址是不同的，**WSL 2 计划实现的 Feature 包括**：① 在提供不同的IP 地址情景下，加速网络访问；② 将客机映射到主机，共享同一 IP 地址；③ 尽快支持全部 Linux 下的网络应用。

## **总结**

<br/>

**简单的理解：**通过使用虚拟化技术，使得 Hypervisor 上同时运行 Windows 实例和 Linux 实例，Windows 实例占主导地位并且持续地运行，Linux 实例非常轻量级并且只有在用到时才会启动。采用原生的 Linux Kernel 保证了系统调用的完整性，9P 协议的注入又很好地解决了两个不同文件系统之间的交互问题。

## **参考链接**

- [The new Windows subsystem for Linux architecture: a deep dive](https://mybuild.techcommunity.microsoft.com/sessions/77003)
- [About WSL 2](https://docs.microsoft.com/en-us/windows/wsl/wsl2-about)
- [Shipping a Linux Kernel with Windows](https://devblogs.microsoft.com/commandline/shipping-a-linux-kernel-with-windows/)
- [WSL 2 Frequently Asked Questions](https://docs.microsoft.com/en-us/windows/wsl/wsl2-faq)

---

