---
title: "#Golang# Golang与并发编程(2) goroutine调度"
date: 2020-3-4 07:00:21
categories: Programming Language
tags:
- Golang
- Concurrency
thumbnail: /images/goroutine.png
---



本文主要讨论 **`goroutine` 调度**的相关内容。



---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---

## **前情提要**

> `Go` 中的并发性是以 **`goroutine` (独立活动)**和 **`channel` (用于通信)**的形式实现的，这是因为 `Go` 所信奉的**“不要通过共享内存来通信，而应该通过通信来共享内存”**。当然，`Go` 也提供了对传统通信机制的支持，如 `Mutex` (互斥锁) 和 `WaitGroup` (信号量)。
>
> 本文将讨论 **`Go` 并发编程中的基本单位 `goroutine`的调度**，如有错漏，欢迎指出 ;P

## **`goroutine` 调度**

<br/>

> `goroutines` 仅存在于 `go runtime` 中而不存在于 `OS` 中，因此需要 **Go Runtime Scheduler** 来管理它们的生命周期。

### 线程模型

![MPG Model](/images/MPG_model.png)

> - **`M`**：`machine`，一个 `M` 代表一个**用户线程**
> - **`P`**：`processor`，一个 `P` 代表一个 `Go` 协程所需的**上下文环境**
> - **`G`**：`goroutine`，一个 `G` 代表一个 **`Go` 协程**
> - **`KSE`**：`kernel schedule entity`，一个 `KSE` 代表一个**内核线程**

#### + 共享任务队列（Global Queue）

<div align=center>
    <img src="/images/global_queue.png" alt="Global Queue"/>
</div>

- **流程：**
  - *Step1.* 先开一个**线程池**；
  - *Step2.* 每个线程**不断从 `Global Queue` 中取 `G` 执行**，亦即一个线程非同时刻可以执行多个协程；
  - *Step3.* **如果没有**系统调用、I/O 中断或者 channel 阻塞，`goroutine` 对应的线程**正常工作**；**如果有**系统调用、I/O中断或者 channel 阻塞，`goroutine` 对应的线程**将被阻塞**。
- **问题：**
  - 线程每次取 `G` 时需要**加锁**，否则出现竞态危害；
  - `goroutine` 本身可以被阻塞，`goroutine` 对应的**线程不应该被阻塞**。
- **解决：**
  - 引入**独有任务队列**，每个线程只读自己的任务队列，不存在资源竞争；
  - 发生系统调用或I/O中断的 `goroutine` 对应的线程要**记录 `goroutine` 的上下文**；
  - 发生系统调用或I/O中断的 `goroutine` 对应的线程**可以换出该 `goroutine`，然后执行新 `goroutine`**。

#### + 独有任务队列（Local Queue）

<div align=center>
    <img src="/images/local_queue.png" alt="Local Queue"/>
</div>

- **流程：**
  - *Step2.* 每个线程**不断从 `Local Queue` 中取 `G` 执行**，亦即一个线程非同时刻可以执行多个协程；
- **问题：**
  - 线程**既要负责调度又要负责执行**，性能下降。
- **解决：**
  - 引入一个**负责线程调度**的抽象层，也就是 **`M`**。

#### + 线程调度器（M）

<div align=center>
	<img src="/images/machine.png" alt="Machine"/>
</div>

- **流程：**
  - *Step1.* 线程池中每个线程对应一个 `M`，`M` 将**根据调度算法选择**合适的 `G` 交由内核线程执行。
- **问题：**
  - 到目前为止，`goroutine` 仍然是**协作式**的，这意味着一个 `G` 可以长时间占用 CPU，直到它完成任务或被阻塞。
- **解决：**
  - 引入一个**负责资源分配**的抽象层，也就是 **`P`**。

#### + 抢占式（P）

<div align=center>
	<img src="/images/processor.png" alt="Processor"/>
</div>

- **流程：**
  - *Step3.*当一个 `G` 占用一定的 CPU 时间，又没有被调度过，那么它**将被 `runtime` 抢占**。
- **问题：**
  - 保留了 `Global Queue` 承载 `Local Queue` 溢出的 `G`，此时如果多个 `P` 同时访问 `Global Queue`，还是要**加全局锁**。 
- **解决：**
  - 引入**全局调度器 `Sched`**。

#### + 全局调度器（Sched）

- **`P`** 主要负责 **`Local Queue`** 的调度，**`Sched`** 主要负责 **`Global Queue`** 的调度。

#### + 非阻塞（netpoller）

- **阻塞的 IO 模型**为：`G1` 被阻塞后，`M1` 也被阻塞，但是 `M1` 的其它 `G#` 可以转移到空闲或新起的 `M#` 中执行。
- **非阻塞的 IO 模型**为：`G1` 被阻塞后，`M1` 不会阻塞，`M1` 的其它 `G#` 仍然可以执行。
- 此外，`netpoller` 还抽象了 **`epoll` 的多路复用**，`netpoller` 将对应的文件描述符注册到 `epoll` 实例中进行 `epoll_wait`，就绪的文件描述符回调通知给阻塞的 `G`，`G` 更新为就绪状态等待调度继续执行。

### 调度场景

![MPG Flowchart](/images/MPG_flowchart.png)

> 调度的本质就是 **`P` 将 `G` 合理地分配给某个 `M`** 的过程，`M` 只有和 `P` 绑定才能运行 `G`。

#### 负载均衡

- **从本地到全局**：`Local Queue` 满了，会将前一半的 `G` 和新创建的 `G` 放到 `Global Queue` 并打乱顺序；
- **从全局到本地**：`Local Queue` 空了，会从 `Glocal Queue` 随机选取 `n = min(len(GQ)/GOMAXPROCS + 1, len(GQ/2))` 个 `G`；
- **为了调度公平**：1/61 的几率在 `Global Queue` 中找 `G`，60/61 的几率在 `Local Queue` 找 `G`，否则 `Global Queue` 中的 `G` 有可能永远没办法被调度。

#### 任务窃取

- 出现空闲的`M`，此时它绑定的 `P` 的 `Local Queue` 为空，并且 `Global Queue` 也为空，则从其它 `P` 中窃取后一半的 `G`，这就是 **`work-stealing` 算法**。

  <img src="/images/work_stealing.png" alt="Work-stealing" align=center/>

#### 线程自旋

- 在创建 `G` 时，运行的 `G` 会尝试唤醒其他空闲的 `M` 执行，如果没有 `G`，`M` 将**自旋而不阻塞**，不停地找 `G`；
- 自旋的优点是**降低了 `M` 切换上下文的成本**，缺点是 **CPU 空转浪费资源**；
- 折中方案是：**最多有 `GOMAXPROCS` 个自旋的线程，多余的线程将休眠。**

#### 请求抢占

- 满足以下两个条件，`sysmon` 将会**发送抢占请求**，但是**能否抢占成功不被保障**：
  - `G` **进行系统调用**超过 **20 μs**
  - `G` **运行 (非循环)**超过 **10 ms**

#### network IO / channel 阻塞

- **当 network IO / channel 操作发生**，被阻塞的 `G` 放入等待队列，`M` 选择其它的 `G` ；
- **如果有**就绪的 `G`，`M` **正常执行**；**如果无**就绪的 `G`，`M` 将**解绑 `P` 并且休眠**；
- **当 network IO / channel 操作完成**，被阻塞的 `G` 从等待变成就绪，加入某个 `P` 中被新的 `M` 执行或加入到 `Global Queue`，休眠的 `M` 将等待新创建的 `G` 唤醒。
- 在这个过程中，调度实际上是**非阻塞的**——`M` 不会因为 `G` 阻塞而休眠。

#### system call 阻塞

- **当 system call 操作发生**，`G` 会阻塞，`M` 会解绑 `P` 并且休眠；
- **如果有**空闲的 `M`，该空闲的 `M` 将**绑定 `P`** ，**如果无**空闲的 `M`，将新建一个 `M` 来**绑定 `P`**；
- **当 system call 操作完成**，被阻塞的 `G` 从阻塞变成就绪，加入某个 `P` 中被新的 `M` 执行或加入到 `Global Queue`，休眠的 `M` 将等待新创建的 `G` 唤醒。
- 在这个过程中，调度实际上是**阻塞的**——`M` 会因为 `G` 阻塞而休眠

## **参考链接**

- [Go并发编程-Goroutine如何调度的?](https://www.iminho.me/wiki/blog-20.html)
- [Go调度器系列（3）图解调度原理](https://segmentfault.com/a/1190000018775901)
- [一文摸清 Go 的并发调度模型](https://studygolang.com/articles/20991)
- [也谈goroutine调度器](https://tonybai.com/2017/06/23/an-intro-about-goroutine-scheduler/)

---

