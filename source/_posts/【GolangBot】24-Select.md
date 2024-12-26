---
title: 【GolangBot】24-Select
date: 2024-12-25 17:54:32
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---


欢迎来到 [Golang系列教程](../golangbot/) 的第24篇教程。

### 什么是 *select*？

`select` 语句用于在多个发送/接收通道操作中进行选择。select 语句会阻塞，直到其中一个发送/接收操作就绪。如果多个操作同时就绪，则会随机选择其中一个。其语法类似于 `switch`，只是每个 case 语句都将是一个通道操作。让我们直接通过一些代码来更好地理解。

### 示例

```go
package main

import (
	"fmt"
	"time"
)

func server1(ch chan string) {
	time.Sleep(6 * time.Second)
	ch <- "from server1"
}
func server2(ch chan string) {
	time.Sleep(3 * time.Second)
	ch <- "from server2"

}
func main() {
	output1 := make(chan string)
	output2 := make(chan string)
	go server1(output1)
	go server2(output2)
	select {
	case s1 := <-output1:
		fmt.Println(s1)
	case s2 := <-output2:
		fmt.Println(s2)
	}
}
```

[在 playground 中运行](https://play.golang.org/p/3_yaJSoSpG)

在上面的程序中，第8行的 `server1` 函数休眠6秒，然后将文本 *from server1* 写入通道 `ch`。第12行的 `server2` 函数休眠3秒，然后将 *from server2* 写入通道 `ch`。

main 函数在第20行和第21行分别调用 `server1` 和 `server2` Goroutine。

在第22行，控制流到达 `select` 语句。select 语句会阻塞，直到其中一个 case 就绪。在我们的程序中，server1 Goroutine 在6秒后写入 `output1` 通道，而 server2 在3秒后写入 `output2` 通道。因此，select 语句将阻塞3秒，等待 server2 Goroutine 写入 `output2` 通道。3秒后，程序打印：

```fallback
from server2
```

然后终止。

### select 的实际用途

将上述程序中的函数命名为 `server1` 和 `server2` 的原因是为了说明 *select* 的实际用途。

假设我们有一个关键任务应用程序，需要尽快向用户返回输出。该应用程序的数据库已复制并存储在世界各地的不同服务器中。假设函数 `server1` 和 `server2` 实际上是在与两个这样的服务器通信。每个服务器的响应时间取决于各自的负载和网络延迟。我们向两个服务器发送请求，然后使用 `select` 语句在相应的通道上等待响应。select 会选择最先响应的服务器，而忽略另一个响应。这样，我们就可以向多个服务器发送相同的请求，并向用户返回最快的响应 :)。

### 默认分支

select 语句中的默认分支在没有其他 case 就绪时执行。这通常用于防止 select 语句阻塞。

```go
package main

import (
	"fmt"
	"time"
)

func process(ch chan string) {
	time.Sleep(10500 * time.Millisecond)
	ch <- "process successful"
}

func main() {
	ch := make(chan string)
	go process(ch)
	for {
		time.Sleep(1000 * time.Millisecond)
		select {
		case v := <-ch:
			fmt.Println("received value: ", v)
			return
		default:
			fmt.Println("no value received")
		}
	}

}
```

[在 playground 中运行](https://play.golang.org/p/8xS5r9g1Uy)

在上面的程序中，第8行的 `process` 函数休眠10500毫秒（10.5秒），然后将 `process successful` 写入 `ch` 通道。这个函数在程序的第15行被并发调用。

在并发调用 process Goroutine 后，main Goroutine 中启动了一个无限 for 循环。无限循环在每次迭代开始时休眠1000毫秒（1秒），然后执行 select 操作。在前10500毫秒期间，select 语句的第一个 case（即 `case v := <-ch:`）不会就绪，因为 process Goroutine 只会在10500毫秒后才向 ch 通道写入。因此，在此期间将执行 `default` 分支，程序将打印10次 `no value received`。

10.5秒后，process Goroutine 在第10行将 `process successful` 写入 ch。现在 select 语句的第一个 case 将被执行，程序将打印 `received value: process successful`，然后终止。这个程序的输出将是：

```fallback
no value received
no value received
no value received
no value received
no value received
no value received
no value received
no value received
no value received
no value received
received value:  process successful
```

### 死锁和默认分支

```go
package main

func main() {
	ch := make(chan string)
	select {
	case <-ch:
	}
}
```

[在 playground 中运行](https://play.golang.org/p/za0GZ4o7HH)

在上面的程序中，我们在第4行创建了一个通道 `ch`。我们尝试在第6行的 select 中从这个通道读取。由于没有其他 Goroutine 向这个通道写入，select 语句将永远阻塞，因此会导致死锁。这个程序将在运行时出现 panic，并显示以下消息：

```fallback
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
	/tmp/sandbox627739431/prog.go:6 +0x4d
```

如果存在默认分支，则不会发生这种死锁，因为当没有其他 case 就绪时将执行默认分支。下面是添加了默认分支的程序重写版本。

```go
package main

import "fmt"

func main() {
	ch := make(chan string)
	select {
	case <-ch:
	default:
		fmt.Println("default case executed")
	}
}
```

[在 playground 中运行](https://play.golang.org/p/Pxsh_KlFUw)

上面的程序将打印：

```fallback
default case executed
```

同样，即使 select 只有 `nil` 通道，默认分支也会被执行。

```go
package main

import "fmt"

func main() {
	var ch chan string
	select {
	case v := <-ch:
		fmt.Println("received value", v)
	default:
		fmt.Println("default case executed")

	}
}
```

[在 playground 中运行](https://play.golang.org/p/IKmGpN61m1)

在上面的程序中，`ch` 是 `nil`，我们试图在第8行的 select 中从 `ch` 读取。如果没有 `default` 分支，`select` 将永远阻塞并导致死锁。由于我们在 select 中有默认分支，它将被执行，程序将打印：

```fallback
default case executed
```

### 随机选择

当 `select` 语句中有多个 case 就绪时，将随机执行其中一个。

```go
package main

import (
	"fmt"
	"time"
)

func server1(ch chan string) {
	ch <- "from server1"
}
func server2(ch chan string) {
	ch <- "from server2"

}
func main() {
	output1 := make(chan string)
	output2 := make(chan string)
	go server1(output1)
	go server2(output2)
	time.Sleep(1 * time.Second)
	select {
	case s1 := <-output1:
		fmt.Println(s1)
	case s2 := <-output2:
		fmt.Println(s2)
	}
}
```

[在 playground 中运行](https://play.golang.org/p/vJ6VhVl9YY)

在上面的程序中，server1 和 server2 goroutine 分别在第18行和第19行被调用。然后 main 程序在第20行休眠1秒。当控制流到达第21行的 select 语句时，server1 已经将 `from server1` 写入 `output1` 通道，server2 已经将 `from server2` 写入 `output2` 通道，因此 select 语句的两个 case 都准备就绪可以执行。如果多次运行这个程序，输出将在 `from server1` 和 `from server2` 之间随机变化，具体取决于随机选择的 case。

请在本地系统中运行此程序以获得这种随机性。如果在 playground 中运行此程序，它将打印相同的输出，因为 playground 是确定性的。

### 注意事项 - 空 select

```go
package main

func main() {
	select {}
}
```

[在 playground 中运行](https://play.golang.org/p/u8hErIxgxs)

你认为上面程序的输出会是什么？

我们知道 select 语句将阻塞，直到其中一个 case 被执行。在这种情况下，select 语句没有任何 case，因此它将永远阻塞，导致死锁。这个程序将出现 panic，输出如下：

```fallback
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [select (no cases)]:
main.main()
	/tmp/sandbox246983342/prog.go:4 +0x25
```


**下一篇教程 - [Mutex](https://golangbot.com/mutex/)**
