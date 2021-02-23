---
title: "#Golang# Golang与并发编程(3) goroutine泄漏"
date: 2020-3-5 07:00:21
categories: Programming Language
tags:
- Golang
- Concurrency
thumbnail: /images/goroutine.png
---



介绍 **`goroutine` 泄漏**的相关内容，主要讨论以下**三个部分**：

- **`goroutine` 泄漏判断**
- **`goroutine` 泄漏分类**
- **`goroutine` 泄漏预防**

---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---

## **前情提要**

> **`goroutine` 泄漏**指的是因为编码中的陷阱使得 **`goroutine` 无法正常释放而造成的内存泄漏**，甚至导致内存溢出或最终程序崩溃。
>
> 本文将讨论 **`goroutine`泄漏的判断、分类和预防**，如有错漏，欢迎指出 ;P

## **`goroutine `泄漏判断**

### runtime.NumGoroutine

- **`runtime.NumGoroutine`** 可以返回正在运行中的 `goroutine` 数量**（包括 `main goroutine` 和 `normal goroutine`）**。

```go
import(
	"fmt"
    "runtime"
)

// ...

fmt.Println(runtime.NumGoroutine())

// ...
```

### runtime/pprof

- **`runtime/pprof`** 可以<u>（以写入的形式）</u>返回 `goroutine` 的**运行数量和堆栈信息**。

```go
import (  
    "os"  
    "runtime/pprof"  
)

// ...

pprof.Lookup("goroutine").WriteTo(os.Stdout, 1)

// ...
```

### http/net/pprof

- **`http/net/pprof`** 可以<u>（以访问的形式）</u>返回 `goroutine` 的**运行数量、堆栈信息和其它资源信息**。

```go
import (
    "net/http"  
    _ "net/http/pprof"  
)

// ...

http.ListenAndServe("localhost:6060", nil)

// ...

// 进入 http://localhost:6060/debug/pprof/goroutine?debug=1 查看
```

### gops

- **`gops`** 支持列出当前环境下的**进程信息**。

  - **在 `.go` 中：**

    ```go
    import (
        "log"
        "github.com/google/gops/agent"
    )
    
    // ...
    
    if err := agent.Start(); err != nil {  
        log.Fatalln(err)
    }
    
    // ...
    ```

  - **在 `terminal` 中：**

    ```bash
    $ gops # 查看进程信息
    
    $ gops stats PID # 查看状态信息
    
    $ gops stack PID # 查看堆栈信息
    ```

### leaktest

- **`leaktest`** 将泄漏检测过程加入到**自动化测试**中去。

```go
import (
    "github.com/fortytw2/leaktest"
)

// ...

defer leaktest.Check(t)
defer leaktest.CheckTimeout(t, time.Second)
defer leaktest.CheckContext(ctx, t)

// ...
```

## **`goroutine` 泄漏分类**

### 无退出的计算循环

- 对于没有函数调用，**纯循环计算的 `G`**，`runtime` **无法实行抢占**；
- 显然，如果**没有退出机制且程序常驻**的话，每次启动的 `goroutine` 都得不到释放，就会发生 `goroutine` 泄漏。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func test() {
	for {
		fmt.Println("Testing...") // 死循环无法抢占和回收
	}
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("NumGoroutine:", runtime.NumGoroutine()) // 输出为 2 ，发生泄漏
	}()

	go test()
}

```

### 不结束的I/O请求

- 对于 **I/O 请求**，`runtime` **无法实行抢占**；
- 如果 I/O 请求**一直处于等待期间**，该 `goroutine` 则无法释放，出现泄漏。

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"runtime"
	"time"
)

func test() {
	input := bufio.NewScanner(os.Stdin)
	if input.Scan() {
		text := input.Text() // I/O请求无法抢占和回收
		fmt.Println(text)
	}
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("NumGoroutine:", runtime.NumGoroutine()) // 输出为 2 ，发生泄漏
	}()

	go test()
}

```

### channel 引起的泄漏

#### 只发送不接收

- 上游生产速度远远大于下游消费速度，**阻塞的 `goroutine` 会一直在 `channel` 的发送等待队列**。
- 无缓冲 `channel` 没有接收就会阻塞，有缓冲 `channel` 缓冲满了就会阻塞。

```go
package main

import (
	"fmt"
	"math/rand"
	"runtime"
	"time"
)

func query() int {
	n := rand.Intn(100)
	return n
}

func queryAll() int {
	ch := make(chan int)
	go func() { ch <- query() }()
	go func() { ch <- query() }()
	go func() { ch <- query() }()
	return <-ch
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("NumGoroutine:", runtime.NumGoroutine()) // 输出为 3 ，发生泄漏
	}()

	queryAll()
}

```

#### 只接收不发送

- 下游消费速度远远大于上游生产速度，**阻塞的 `goroutine` 会一直在 `channel` 的接收等待队列**。
- 无缓冲 `channel` 仍然接收就会阻塞，有缓冲 `channel` 缓冲空了就会阻塞。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("NumGoroutine:", runtime.NumGoroutine()) // 输出为 2 ，发生泄漏
	}()

	ch := make(chan bool)
	go func() {
		ch <- true
	}()
}

```

#### 空 channel

- 向 `nil channel` 发送和接收数据都会导致阻塞，**只进行声明而不初始化 `channel` 容易出现该类泄漏**。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func test() {
	var ch chan bool
	<-ch
	// ch <- data
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("NumGoroutine:", runtime.NumGoroutine()) // 输出为 2 ，发生泄漏
	}()

	go test()
}

```

#### 空 select

- `select{}` 永远无法响应导致协程阻塞，**一般不会出现这种情况**。

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func test() {
	select {}
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("NumGoroutine:", runtime.NumGoroutine()) // 输出为 2 ，发生泄漏
	}()

	go test()
}

```

### sync 引起的泄漏

#### Mutex 忘记解锁

- 有一个 `goroutine` **加锁忘了解锁**，另一个 `goroutine` 竞争锁会失败，由此这个 `goroutine` 将一直地阻塞。

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"time"
)

func test() {
	var mutex sync.Mutex

	for i := 1; i <= 2; i++ {
		go func() {
			mutex.Lock()

			fmt.Println("goroutine index:", i)
		}()
	}
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("NumGoroutine:", runtime.NumGoroutine()) // 输出为 2 ，发生泄漏
	}()

	go test()
}

```

#### WaitGroup 计数错误

- `WaiteGroup` 的 **`Add` 和 `Done` 数量不对应**将引起 `Wait` 的等待退出条件永远无法满足，从而阻塞协程。

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"time"
)

func test() {
	var wg sync.WaitGroup

	wg.Add(2)

	go func() {
		wg.Done()
	}()

	wg.Wait()
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("NumGoroutine:", runtime.NumGoroutine()) // 输出为 2 ，发生泄漏
	}()

	go test()
}

```

#### Cond 没发信号

- `Wait` 方法阻塞等待条件变量满足条件，**如果没有 `Signal` 或者 `Broadcast`**，`Wait` 将会一直不能唤醒。

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"time"
)

func test() {
	cond := sync.NewCond(&sync.Mutex{})

	condition := false

	go func() {
		cond.L.Lock()

		condition = true

		cond.L.Unlock()
	}()

	cond.L.Lock()
	for !condition {
		cond.Wait()
	}
	cond.L.Unlock()
}

func main() {
	defer func() {
		time.Sleep(time.Second)
		fmt.Println("NumGoroutine:", runtime.NumGoroutine()) // 输出为 2 ，发生泄漏
	}()

	go test()
}

```

## **`goroutine `泄漏预防**

- **对于计算循环和 I/O 请求：**<u>检查代码</u>。

  - 确保每个计算循环都会退出；
  - 确保每个 I/O 请求都会关闭。

- **对于 `channel` 引起的泄漏：**<u>本质上是防止 `channel` 阻塞</u>。

  - 使用有缓冲的 `channel` ；
  - 用 `make` 来声明并初始化 `channel`；
  - 避免空的 `select`；
  - [优雅地关闭 `channel`]() 等。

- **对于 `sync` 引起的泄漏：**<u>本质上是防止 `sync` 阻塞</u>。

  - 对于**互斥锁**，**加锁解锁应该成对地出现**，这时候可以利用 `defer`：

    ```go
    // ...
    mutex.Lock()
    // ...
    defer mutex.Unlock()
    // ...
    ```

  - 对于**信号量**，**`Add(1)` 和 `Done()` 搭配使用**，而不是一开始就规定好任务计数：

    ```go
    // ...
    wg.Add(1)
    go func() {
    	wg.Done()
        // ...
    }()
    ```

  - 对于**条件变量**，**条件改变后应发送信号**，单播还是多播根据具体情况而定：

    ```go
    // ...
    condition = true
    cond.Signal() // or cond.Broadcast()
    // ...
    ```

## **参考链接**

- [Go语言圣经](https://books.studygolang.com/gopl-zh/)
- [如何防止 goroutine 泄露（一）](https://juejin.im/post/5d2f66a251882553086ab819#heading-2)
- [如何防止 goroutine 泄露（二）](https://juejin.im/post/5d3d76066fb9a07ee463aba0)
- [Goroutine 泄露](https://studygolang.com/articles/12495)

---

