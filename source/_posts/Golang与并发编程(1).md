---
title: "#Golang# Golang与并发编程(1) goroutine使用"
date: 2020-3-3 07:00:21
categories: Programming Language
tags:
- Golang
- Concurrency
thumbnail: /images/goroutine.png
---



本文主要讨论 **`goroutine` 使用**的相关内容。



---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---

## **前情提要**

> `Go` 中的并发性是以 **`goroutine` (独立活动)**和 **`channel` (用于通信)**的形式实现的，这是因为 `Go` 所信奉的**“不要通过共享内存来通信，而应该通过通信来共享内存”**。当然，`Go` 也提供了对传统通信机制的支持，如 `Mutex` (互斥锁) 和 `WaitGroup` (信号量)。
>
> 本文将讨论 **`Go` 并发编程中的基本单位 `goroutine`的使用**，如有错漏，欢迎指出 ;P

## **概念辨析**

### 串行、并发与并行

#### 串行

> CPU**按顺序执行**完一个线程后，才能开始另一个线程<u>*（显然不是同时执行）*</u>。

#### 并发

> CPU**在各个任务间切换**，同时刻一个CPU只能运行一个线程<u>*（看起来的同时执行）*</u>。

#### 并行

> 同时刻**多个任务在多个CPU**上运行<u>*（事实上的同时执行）*</u>。

### 进程、线程与协程

#### 进程

> - 进程是操作系统**资源分配的最小单位**。
> - 进程**具有独立的地址空间**，通信需要通过**IPC**。
> - 一个系统内的多个进程可以**并发执行**。

#### 线程

> - **线程**是操作系统**调度执行的最小单位**。
> - 线程**共享进程的地址空间**，通信直接**共享内存**。
> - 一个进程内的多个线程可以**并发执行**。

#### 协程

> - 协程运行在**用户态**，而进程和线程都运行在内核态。
> - 协程是函数间切换不是线程间切换，**占用内存和切换开销更小**。
> - 一个线程内的多个协程只可**串行执行**。

*PS：[Linux虚拟地址空间布局](https://www.cnblogs.com/clover-toeic/p/3754433.html)*

### 用户线程与内核线程

- **内核可见：**用户线程对于内核来说是透明的，用户线程数量几乎不受限；内核线程对于内核来说是可见的，内核线程数量是有限制的。
- **调度实体：**用户线程由应用程序或运行时调度器调度，用户线程切换只发生在用户态；内核线程由内核调度，内核线程切换需要陷入内核态。
- **资源竞争：**用户线程在进程内竞争资源，用户线程阻塞那么对应进程阻塞，其它线程同时阻塞；内核线程在系统内竞争资源，内核线程阻塞只会阻塞该个线程，其它线程不被影响。
- **并行支持：**使用用户线程，CPU 的执行单位为进程，要并行就要多进程；使用内核线程，CPU 的执行单位为线程，要并行只需多线程。

*PS：[线程实现模型](https://bingjian-zhu.github.io/2019/09/11/%E7%BA%BF%E7%A8%8B%E5%AE%9E%E7%8E%B0%E6%A8%A1%E5%9E%8B/)*

### `goroutine` 与 `coroutine`

- **`goroutine`** 的运行机制属于**抢占式任务处理**，应用程序对CPU的控制**由 `Go Runtime Scheduler` 管理**；**`coroutine`** 的运行机制属于**协作式任务处理**，应用程序对CPU的控制**由应用程序决定**。
- **`goroutine`** 可以**并行执行**，可能发生在**多线程**环境下；**`coroutine`** 始终**顺序执行**，总是发生在**单线程**环境下。
- **`goroutine`** 使用 **`channel`** 通信；**`coroutine`** 使用 **`yield` 和 `resume`** 通信。

## **`goroutine` 使用**

### main goroutine

- `Golang` 的入口函数 `main()` 本身就是个 `goroutine`：

  ```go
  func main() {
      // ...
  }
  ```

### normal goroutine

- 创建一个普通的 `goroutine`，只需要在函数前加关键字 `go`：

  ```go
  func sample() {
      // ...
  }
  
  func main() {
      // 命名goroutine
      go sample()
      
      // 匿名goroutine
      go func() {
          // ...
      }
  }
  ```

*PS：除了 main 可以打断 goroutine 执行外，各 goroutine 间是独立的。由于 goroutine 执行顺序无法确定，代码的逻辑要独立于调用的顺序。*

## **要点总结**

- `goroutine` 是在语言层面就提供的，只需要关键字 `go`，使用非常方便。

## **参考链接**

- [并发简略-概述](https://github.com/overnote/over-golang/blob/master/02-%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/00-1-%E5%B9%B6%E5%8F%91%E7%AE%80%E7%95%A5-%E6%A6%82%E8%BF%B0.md)
- [并发简略-对比并发模型](https://github.com/overnote/over-golang/blob/master/02-%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/00-6-%E5%B9%B6%E5%8F%91%E7%AE%80%E7%95%A5-%E5%AF%B9%E6%AF%94%E5%B9%B6%E5%8F%91%E6%A8%A1%E5%9E%8B.md)
- [Go语言圣经](https://books.studygolang.com/gopl-zh/)

---

