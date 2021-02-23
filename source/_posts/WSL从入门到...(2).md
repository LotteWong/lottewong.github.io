---
title: "#WSL# WSL从入门到...(2) Dive into WSL 1"
date: 2019-12-13 07:00:21
categories: DevOps
tags:
- WSL
thumbnail: /images/wsl.jpg
---



对 **WSL 1 的一些实现细节**进行补充说明，主要分为以下**五个部分**：

- 前情提要
- 背景
- **架构与功能**
- 总结
- 参考链接

---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---



## **前情提要**

<br/>

*前面已经简要介绍了 WSL 1 和 WSL 2 的区别，本节将聚焦于 WSL 1的实现细节。点开官方博客开始收获惊(~~da~~)喜(~~keng~~)，才疏学浅写错也不要打我 (预先感谢各位勘误 逃*

【更多信息 👉 [WSL从入门到...(1) WSL 1 vs WSL 2](https://lottewong.github.io/2019/12/12/WSL%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0...(1)/)】

## **背景**

<br/>

> Early subsystems were implemented as **user mode modules** that **issued appropriate NT system calls based on the API** they presented to applications for that subsystem. All applications were PE/COFF executables, a set of libraries and services to implement the subsystem API and NTDLL to perform the NT system call. When a user mode application got launched, the loader invoked the right subsystem to satisfy the application dependencies based on the executable header.

- **早期的子系统**采用的方法是，将子系统视为上层的一系列进程组，通过 NTDLL 提供的 API 实现用户模式和内核模式通信，其中 `loader` 对 `application` 和 `subsystem` 进行管理。这里运行的仍然是 Win32 binaries，需要移植和维护。

> Later versions of subsystems replaced the POSIX layer to provide **the Subsystem for Unix-based Applications (SUA)**. The primary role of SUA was to encourage applications to **get ported to Windows without significant rewrites**. This was achieved by implementing the POSIX user mode APIs using NT constructs.
>

- **后来的子系统**采用的方法是，在 POSIX 层做优化，移植而不是重写。这里运行的仍然是 Win32 binaries，需要移植和维护。

> WSL is a collection of components that enables native Linux ELF64 binaries to run on Windows. It contains **both user mode and kernel mode components**. It is primarily comprised of:
>
> 1. **User mode session manager service** that handles the Linux instance life cycle
> 2. **Pico provider drivers** (lxss.sys, lxcore.sys) that emulate a Linux kernel by translating Linux syscalls
> 3. **Pico processes** that host the unmodified user mode Linux (e.g. /bin/bash)

- **WSL** 采用的方法是，构造一整套包含用户模式和内核模式组件的系统，其主要由用户模式会话管理服务（User mode session manager service）、Pico 驱动（Pico provider drivers）和 Pico 进程 （Pico processes）三个部分组成。这里运行的仍然是 Linux binaries，无需移植和维护。

## **架构**

<br/>

![WSL 1 Architecture](/images/wsl1_component.png)

**工作流大概是：**Windows 的 Bash 键入指令，交由 LXSS Manager Service 启动对应的 Linux 实例并进入对应的 Pico 进程，Linux 的 Bash 通过中间层的驱动器与 Windows Kernel 进行翻译交流（<u>"The lxss.sys and lxcore.sys drivers translate the Linux system calls into NT APIs and emulate the Linux kernel."</u>）。

![Deepu Thomas and Seth Juarez discuss the underlying architecture that enables the Windows Subsystem for Linux](/images/wsl1_lecture.png)

### LXSS Manager Service

- **LXSS Manager Service 负责 Windows 的 Bash（Win32 Process）和 Linux 的 Bash（Pico Process）之间的通信，主要在初始时工作。**作用范围包括但不局限于：同步 Linux 实例的安装和卸载、每次只允许一个进程启动 Linux Binary 并在 pending 时阻塞后续启动进程等等。

### Pico Drivers

- **Pico Drivers 负责 Linux Instance（User Mode）和 Windows Kernel（Kernel Mode）之间的通信，主要在运行时工作。**作用范围包括但不局限于：翻译 Linux system calls 为 Windows NT APIs 可以理解的形式，模拟 Linux 内核并对 Windows 内核进行操作等等。

### Pico Processes

- **将可执行的 ELF binaries 加载进到 Pico Processes 的地址空间，并在 Linux 层上运行。**其中 Linux 实例是一个特殊的为 Pico Processes 服务的数据结构，可以包含并追踪所有的 Linux 进程、线程和运行状态。它在 Win32 Process 首次启动 Linux Binary 时创建，在 Win32 Process 最后的客户端关闭连接时被销毁。从整体上看，整个 WSL 1 只有一个 Linux 实例，它隔离开了本机原有的 Windows 和所有新建的 Linux ；从内部来看，Linux 实例内的每个 Pico Process 都被单独隔离，这点和容器很类似。

## **功能**

<br/>

由于 Windows Kernel 和 Linux Kernel **基本上不兼容**，需要**额外处理很多中间转换**，一般包括系统调用、文件系统、权限管理和网络配置等，下面介绍最基本和最重要的两个方面：

### System Calls

- syscall 是内核提供的一项服务，可以在用户模式下调用。**Windows Kernel 和 Linux Kernel 都暴露了非常多的 syscall，但两者对此的设计模式并不同导致不兼容。**

- **解决这种冲突的方法是引入 Pico Drivers（`lxss.sys` and `lxcore.sys`）。**当一个 Linux Kernel syscall 被调用，该请求将被转发给 `lxcore.sys`，`lxcore.sys` 将 Linux Kernel syscall 翻译成等价的 Windows Kernel syscall，最后传到 Windows Kernel。特别地，如果在两种内核之间有某个 syscall 不存在映射关系（亦即不能互相翻译），`lxss.sys` 还被要求处理这种异常并提供相应的服务。

  1. *以 Linux 的 `fork()` 为例：*

		> As an example, the Linux fork() syscall has no direct equivalent call documented for Windows. When a fork system call is made to the Windows Subsystem for Linux, `lxcore.sys` does some of the initial work to prepare for copying the process. It then calls internal Windows NT kernel APIs to create the process with the correct semantics, and completes copying additional data for the new process.

  2. *以 Linux 的 `chmod` 为例：*

		> Linux Instance 产生附加 metadata，只由 Linux 文件系统能理解， Windows 文件系统并不能理解，但是 Windows Kernel 仍会接收该 metadata。但是 NTFS 对这个 metadata 无动于衷，而是 `lxss.sys` 处理该 metadata，最终反映到 ext4 中。由此可见，WSL 1 非常大的工作量都要花在应对不兼容的指令集。

  *PS: 尽管 Linux 的内核更新得非常快，用户模式的接口却是相对固定的，所以可以认为<u>"WSL 1 abstracts the Linux kernel via their interface"</u>*

### File System

- 由于 Windows 和 Linux 的文件系统也不相同（Windows 的文件系统是 NTFS，Linux 的文件系统是 ext4），不能直接互相操作。**WSL 的文件系统设计需要满足两点：**

  1. 完整支持 Linux 文件系统
  2. 可访问和操作 Windows 中的磁盘和文件

- **为实现以上的目标 WSL 引入了两套文件系统（VolFs and DriveFs）。** VolFs 文件系统主要提供了 Linux 文件系统特性的支持，比如 Linux 文件目录（/etc, /bin, /usr, etc.）、权限管理和符号链接等等。DriveDs 文件系统主要用于与 Windows 做交互，比如 Windows 文件目录（/mnt/c, /mnt/d, etc.）、启动 Windows 的可执行文件等等。

  *PS: 需要注意的是，两套文件系统之间是如何进行交互的并未在原博客详细讨论，感兴趣的小伙伴可自行阅读续篇 [WSL File System Support](https://blogs.msdn.microsoft.com/wsl/2016/06/15/wsl-file-system-support/)* 

## **总结**

<br/>

**简单的理解：**WSL 1 在 Windows Kernel 之上创建 Linux Instance，Win32 Process 通过 LXSS Manager Service 管理 Pico Process，用户模式的 Linux Instance 和内核模式的 Windows Kernel 之间通过 Pico Drivers 交流，依靠中间层驱动器翻译可以解决 System Calls 不兼容的问题，设计两套文件系统能够满足同时使用 Windows 和 Linux 的需要。

## **参考链接**

- [Windows Subsystem for Linux Overview](https://blogs.msdn.microsoft.com/wsl/2016/04/22/windows-subsystem-for-linux-overview/)
- [Project Drawbridge](http://research.microsoft.com/en-us/projects/drawbridge/)

---

