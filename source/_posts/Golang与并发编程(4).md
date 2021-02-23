---
title: "#Golang# Golang与并发编程(4) Channel使用"
date: 2020-3-6 07:00:21
categories: Programming Language
tags:
- Golang
- Concurrency
thumbnail: /images/channel_thumbnail.png
---



本文主要讨论 **`channel` 使用**的相关内容。



---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---

## **前情提要**

> 为了**实现 `goroutine` 通信**，有两种常见并发模型：
>
> - **共享内存：**使用共享内存方式，`Go` 中 `sync` 库包提供了多种同步的机制。
> - **消息队列：**使用类似管道和消息队列的方式，各个并发单元数据相互独立，通过消息进行数据交换，`Go` 中 `channel` 类型模拟了这种同步的模式。
>
> 让我们再一次来复读 `Go` 社区的并发口号——**“不要通过共享内存来通信，而应该通过通信来共享内存”**。
>
> 本文将讨论 **`Go` 并发编程中的通信桥梁 `channel`的使用**，如有错漏，欢迎指出 ;P

## **概念引申**

### 管道 Pipe

<div align="center">
    <img src="/images/pipe.png" alt="Pipe">
</div>

- **管道 (Pipe)** 是操作系统中**进程间通信**的一种方式。管道的本质是一个存在于**内存或文件系统**的**缓冲有限的特殊文件**，进程以**先进先出**的方式从缓冲区存取数据，管道一端的进程顺序地将数据写入缓冲区，另一端的进程则顺序地读出数据。
- 匿名管道没有名字，是**半双工**的；匿名管道对于管道两端的进程而言是一个存在于**内存**的特殊文件；匿名管道只能用于**父子进程或兄弟进程**之间通信。
- 有名管道拥有名字，是**全双工**的；有名管道对于管道两端的进程而言是一个存在于**文件系统**的特殊文件；有名管道可以用于**本机任意两个进程**之间通信。
- 管道传送**无格式字节流**，这就要求管道的读出方和写入方必须事先约定好数据的格式。

### 消息队列 Message

<div align="center">
    <img src="/images/message.png" alt="Message">
</div>

- **消息队列 (Message)** 是操作系统中**进程间通信**的一种方式。管道的本质是一个存放在**内核**中的**长度不限的消息链表**，一般遵循**先进先出**规则，也可以**随机或按消息的类型**查询，并且允许一个或多个进程向它写入或读取消息。
- 消息队列由消息队列标识符标识，提供**有格式字节流**并且**具有类型**。

### 信道 Channel

<div align="center">
    <img src="/images/channel.png" alt="Channel">
</div>

- **信道 (Channel)** 是 `Golang` 中**协程间通信**的一种方式。信道的本质是一个线程安全的**先进先出队列**，可选择**无缓冲或有缓冲**，任意时刻同时只能有一个 `goroutine` 访问 `channel`。
- 信道可以**指定类型**。

## **`channel` 使用**

### 创建

```go
// 双向channel
ch := make(chan T, cap)

// 只读channel（但是没什么意义，双向channel可以赋值单向channel）
ch_or := make(<-chan T, cap)

// 只写channel（但是没什么意义，双向channel可以赋值单向channel）
ch_ow := make(chan<- T, cap)
```

- 引用类型，**使用 `make` 创建**，零值为 `nil`
- 可选择**数据类型**
- 可选择**缓冲容量**
  - **无缓冲**：`cap = 0`，读写**同步**
  - **有缓冲**：`cap > 0`，读写**异步**
- 可选择**通信方向**
  - **只读**：`<-chan`
  - **只写**：`chan<-`

### 读写

```go
data := <- ch
```

- 从 `channel` 中读取数据：

  - **无缓冲**：没有 `goroutine` 写入，`channel` 阻塞
  - **有缓冲**：`channel` 容量变空，`channel` 阻塞

```go
ch <- data
```

- 向 `channel` 里写入数据：
  - **无缓冲**：没有 `goroutine` 读取，`channel` 阻塞
  - **有缓冲**：`channel` 容量变满，`channel` 阻塞

### 关闭

```go
// 关闭 channel
// close 手动关闭或等待自动 GC
close(ch)

// 尝试写入已关闭的 channel
// 导致 panic 异常
ch <- d

// 返回数据或零值
// 还有剩余数据则返回数据
// 没有剩余数据则返回零值
d := <-ch

// 返回数据和布尔值
// true 表示成功从 channel 接收到值
// false 表示 channel 已经被关闭并且里面没有值可接收
d, ok := <-ch

// 尝试重复关闭 channel
// 导致 panic 异常
close(ch)
```

- 关闭 `channel` 可以通过显式的**代码关闭**或隐式的**垃圾回收**
- 对关闭后的 `channel` 进行**写入操作**：
  - 导致 **`panic`** 异常
- 对关闭后的 `channel` 进行**读取操作**：
  - **存在已经发送成功的数据**：返回数据
  - **不存在已经发送成功的数据**：返回零值
- 对关闭后的 `channel` 进行**重复关闭**：
  - 导致 **`panic`** 异常
- `close` 常常与 **`select`、`range` 和 `defer`** 一起使用

*PS：`Golang` 并不支持无限容量的 `channel`，原因是如果生产速率远远大于消费速率，那么 `channel` 内的数据不断累积将会爆掉内存。*

## **应用实例**

### 单生产者-单消费者模型

```go
package main

import (
	"fmt"
	"time"
)

const BUFLEN = 5

// 生产者
func producer(ch chan<- int) {
	d := 1
	for {
		ch <- d
		fmt.Println("Produce:", d)
		d++
		time.Sleep(1 * time.Second) // 生产者速率不宜过快
	}
}

// 消费者
func consumer(ch <-chan int) {
	for {
		d := <-ch
		fmt.Println("Consume:", d)
		time.Sleep(2 * time.Second) // 消费者速率不宜过快
	}
}

func main() {
	// 生产消费的缓冲区
	ch := make(chan int, BUFLEN)

	// 启动生产者协程
	go producer(ch)

	// 启动消费者协程
	go consumer(ch)

	// 防止主函数的退出
	for {

	}
}

```

### 多生产者-多消费者模型

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

const BUFLEN = 5

var cond *sync.Cond = sync.NewCond(&sync.Mutex{})

// 生产者
func producer(ch chan<- int) {
	for {
		cond.L.Lock()
		for len(ch) == BUFLEN {
			cond.Wait()
		}
		data := rand.Intn(1000)
		ch <- data
		fmt.Println("Produce:", data)
		cond.L.Unlock()

		cond.Signal()
		time.Sleep(2 * time.Second) // 生产者速率不宜过快
	}
}

// 消费者
func consumer(ch <-chan int) {
	for {
		cond.L.Lock()
		for len(ch) == 0 {
			cond.Wait()
		}
		data := <-ch
		fmt.Println("Consume:", data)
		cond.L.Unlock()

		cond.Signal()
		time.Sleep(1 * time.Second) // 消费者速率不宜过快
	}
}

func main() {
	rand.Seed(time.Now().UnixNano())

	// 生产消费的缓冲区
	ch := make(chan int, BUFLEN)

	// 启动 10 个生产者协程
	for i := 0; i < 10; i++ {
		go producer(ch)
	}

	// 启动 10 个消费者协程
	for i := 0; i < 10; i++ {
		go consumer(ch)
	}

	// 防止主函数的退出
	for {

	}
}

```

## **参考链接**

- [Go语言圣经](https://books.studygolang.com/gopl-zh/)
- [channel](https://github.com/overnote/over-golang/blob/master/02-%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/02-channel.md)
- [channel的应用](https://github.com/overnote/over-golang/blob/master/02-%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/04-channel%E7%9A%84%E5%BA%94%E7%94%A8.md)

---

