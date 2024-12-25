---
title: 【GolangBot】22-Channels
date: 2024-12-25 08:53:39
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---

# 【GolangBot】22-Channels

欢迎来到 [Golang系列教程](../golangbot) 的第22篇教程。

在[上一篇教程](../【GolangBot】21-Goroutines)中，我们讨论了如何使用 Goroutines 在 Go 中实现并发。在本教程中，我们将讨论 channels 以及 Goroutines 如何通过 channels 进行通信。

### 什么是 channels

Channels 可以被视为 Goroutines 通过它进行通信的管道。就像水在管道中从一端流向另一端一样，数据可以通过 channels 从一端发送并从另一端接收。

### 声明 channels

每个 channel 都有一个与之关联的类型。这个类型就是该 channel 允许传输的数据类型。不允许使用该 channel 传输其他类型的数据。

*chan T* 是一个类型为 `T` 的 channel

Channel 的零值是 `nil`。`nil` channels 没有任何用处，因此必须使用 `make` 来定义 channel，这与 [maps](../【GolangBot】13-Maps) 和 [slices](../【GolangBot】11-数组和切片) 类似。

让我们编写一些声明 channel 的代码。

```go
package main

import "fmt"

func main() {
    var a chan int
    if a == nil {
        fmt.Println("channel a is nil, going to define it")
        a = make(chan int)
        fmt.Printf("Type of a is %T", a)
    }
}
```

[在 playground 中运行程序](https://play.golang.org/p/QDtf6mvymD)

第 6 行声明的 channel `a` 是 `nil`，因为 channel 的零值是 `nil`。因此 if 条件内的语句被执行，并定义了该 channel。在上面的程序中，`a` 是一个 int 类型的 channel。该程序将输出：

```fallback
channel a is nil, going to define it
Type of a is chan int
```

像往常一样，使用简短声明也是定义 channel 的一种有效且简洁的方式。

```fallback
a := make(chan int)
```

上面的代码行同样定义了一个 int 类型的 channel `a`。

### 通过 channel 发送和接收数据

从 channel 发送和接收数据的语法如下：

```fallback
data := <- a // 从 channel a 中读取数据
a <- data // 向 channel a 写入数据
```

箭头相对于 channel 的方向指定了数据是被发送还是被接收。

在第一行中，箭头指向远离 `a` 的方向，因此我们是从 channel `a` 读取数据并将值存储到变量 data 中。

在第二行中，箭头指向 `a`，因此我们是向 channel `a` 写入数据。

### 发送和接收默认是阻塞的

向 channel 发送和接收数据默认是阻塞的。这是什么意思？当数据被发送到一个 channel 时，发送语句会被阻塞，直到其他 Goroutine 从该 channel 中读取数据。同样，当从一个 channel 中读取数据时，读取操作会被阻塞，直到某个 Goroutine 向该 channel 写入数据。

channels 的这个特性使得 Goroutines 能够有效地通信，而无需使用在其他编程语言中常见的显式锁或条件变量。

如果现在这些还不太理解也没关系。接下来的章节将更清楚地说明 channels 是如何默认阻塞的。

### Channel 示例程序

理论说够了 :)。让我们编写一个程序来理解 Goroutines 是如何使用 channels 进行通信的。

我们实际上将重写我们在学习 [Goroutines](../GolangBot】21-Goroutines) 时编写的程序。

让我从上一个教程中引用这个程序。

```go
package main

import (  
    "fmt"
    "time"
)

func hello() {  
    fmt.Println("Hello world goroutine")
}
func main() {  
    go hello()
    time.Sleep(1 * time.Second)
    fmt.Println("main function")
}
```

[在 playground 中运行程序](https://play.golang.org/p/U9ZZuSql8-)

这是上一个教程中的程序。我们在这里使用 sleep 来让 main Goroutine 等待 hello Goroutine 完成。如果这对你来说没有意义，我建议阅读关于 [Goroutines](../GolangBot】21-Goroutines) 的教程。

我们将使用 channels 重写上面的程序。

```go
package main

import (
    "fmt"
)

func hello(done chan bool) {
    fmt.Println("Hello world goroutine")
    done <- true
}
func main() {
    done := make(chan bool)
    go hello(done)
    <-done
    fmt.Println("main function")
}
```

[在 playground 中运行程序](https://play.golang.org/p/I8goKv6ZMF)

在上面的程序中，我们在第 12 行创建了一个 `done` bool channel，并将其作为参数传递给 `hello` Goroutine。在第 14 行，我们正在从 `done` channel 接收数据。这行代码是阻塞的，这意味着在某个 Goroutine 向 `done` channel 写入数据之前，控制不会移动到下一行代码。因此，这就消除了在原始程序中使用 `time.Sleep` 来防止 main Goroutine 退出的需求。

代码行 `<-done` 从 done channel 接收数据，但不使用或存储该数据到任何变量中。这是完全合法的。

现在我们的 `main` Goroutine 被阻塞，等待 done channel 上的数据。`hello` Goroutine 将这个 channel 作为参数接收，打印 `Hello world goroutine`，然后向 `done` channel 写入数据。当这个写入完成时，main Goroutine 从 done channel 接收数据，它被解除阻塞，然后打印文本 *main function*。

这个程序输出：

```fallback
Hello world goroutine
main function
```

让我们通过在 `hello` Goroutine 中引入一个 sleep 来修改这个程序，以更好地理解这个阻塞概念。

```go
package main

import (
    "fmt"
    "time"
)

func hello(done chan bool) {
    fmt.Println("hello go routine is going to sleep")
    time.Sleep(4 * time.Second)
    fmt.Println("hello go routine awake and going to write to done")
    done <- true
}
func main() {
    done := make(chan bool)
    fmt.Println("Main going to call hello go goroutine")
    go hello(done)
    <-done
    fmt.Println("Main received data")
}
```

[在 playground 中运行](https://play.golang.org/p/EejiO-yjUQ)

在上面的程序中，我们在第 10 行为 `hello` 函数添加了 4 秒钟的睡眠时间。

这个程序首先会打印 `Main going to call hello go goroutine`。然后 hello Goroutine 将启动并打印 `hello go routine is going to sleep`。打印完这个后，`hello` Goroutine 将睡眠 4 秒钟，在此期间 `main` Goroutine 将被阻塞，因为它在第 18 行 `<-done` 等待来自 done channel 的数据。4 秒后，将打印 `hello go routine awake and going to write to done`，然后是 `Main received data`。

### Channel 的另一个示例

让我们再写一个程序来更好地理解 channels。这个程序将打印一个数字的各个数字的平方和与立方和。

例如，如果输入是 123，那么这个程序将计算输出为：

平方和 = (1 * 1) + (2 * 2) + (3 * 3)  
立方和 = (1 * 1 * 1) + (2 * 2 * 2) + (3 * 3 * 3)  
输出 = 平方和 + 立方和 = 50

我们将这样组织程序：在一个单独的 Goroutine 中计算平方和，在另一个 Goroutine 中计算立方和，最后的求和在 main Goroutine 中进行。

```go
package main

import (  
    "fmt"
)

func calcSquares(number int, squareop chan int) {  
    sum := 0
    for number != 0 {
        digit := number % 10
        sum += digit * digit
        number /= 10
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {  
    sum := 0 
    for number != 0 {
        digit := number % 10
        sum += digit * digit * digit
        number /= 10
    }
    cubeop <- sum
} 

func main() {  
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares + cubes)
}
```

[在 playground 中运行程序](https://play.golang.org/p/4RKr7_YO_B)

第 7 行的 `calcSquares` 函数计算数字各个位的平方和，并将结果发送到 `squareop` channel。类似地，第 17 行的 `calcCubes` 函数计算数字各个位的立方和，并将结果发送到 `cubeop` channel。

这两个函数在第 31 和 32 行作为单独的 Goroutines 运行，每个都传递了一个用于写入的 channel 作为参数。main Goroutine 在第 33 行等待这两个 channels 的数据。一旦从两个 channels 接收到数据，它们就被存储在 `squares` 和 `cubes` 变量中，最终输出被计算并打印。这个程序将打印：

```fallback
Final output 1536
```

### 死锁

使用 channels 时要考虑的一个重要因素是死锁。如果一个 Goroutine 正在向一个 channel 发送数据，就期望其他一些 Goroutine 应该接收数据。如果这种情况没有发生，程序将在运行时出现 `Deadlock` panic。

同样，如果一个 Goroutine 正在等待从一个 channel 接收数据，那么就期望其他一些 Goroutine 会向该 channel 写入数据，否则程序将出现panic。

```go
package main


func main() {
    ch := make(chan int)
    ch <- 5
}
```

[在 playground 中运行程序](https://play.golang.org/p/q1O5sNx4aW)

在上面的程序中，创建了一个 channel `ch`，我们在第 6 行 `ch <- 5` 向该 channel 发送 5。在这个程序中，没有其他 Goroutine 从 channel `ch` 接收数据。因此这个程序会出现以下运行时错误的panic：

```fallback
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
    /tmp/sandbox046150166/prog.go:6 +0x50
```

---

下面是我自己补充的内容，非原作者内容：

关于死锁，我自己补充一点，考虑清楚以下几个问题就基本能捋顺了。首先搞清楚一个概念，在触发死锁panic时，提示为`fatal error: all goroutines are asleep - deadlock!`。也就是说，当包括main Goroutine在内的所有Goroutine都阻塞的时候，程序才会触发死锁。那么来看下面的程序，

```go
package main

import (
	"fmt"
	"time"
)

func hello(done chan bool) {
	fmt.Println("Hello world goroutine")
	done <- true
}
func main() {
	done := make(chan bool, 0)
	go hello(done)
	time.Sleep(10 * time.Second)
	fmt.Println("main function")
} 
```

> 使用 channels 时要考虑的一个重要因素是死锁。如果一个 Goroutine 正在向一个 channel 发送数据，就期望其他一些 Goroutine 应该接收数据。如果这种情况没有发生，程序将在运行时出现 `Deadlock` panic。

基于作者的观点，我抛出一个疑问：为什么`done`这个channel只在`hello`中发送，没有被读取，程序却不会死锁呢？原因在于，`done <- true`，done只发送数据，却没有接收，因此`hello`函数进入阻塞，而这个时候，main这个Goroutine还在正常进行，而当`main function`被打印后，main Goroutine结束，`hello`也就跟着一起结束了。这时阻塞是1/2。

而将程序作如下修改：

```go
package main

import (
	"fmt"
	"time"
)

func hello(done chan bool) {
	fmt.Println("Hello world goroutine")
	done <- true
}
func main() {
	done := make(chan bool, 0)
	go hello(done)
	time.Sleep(10 * time.Second)
	fmt.Println("main function")
	select {}
} 
```

17行，我们添加了一个`select{}`，这会使得main Goroutine进入无尽的阻塞，这个时候的阻塞率是2/2。因此会触发死锁。

观点结束，继续原文

---

### 单向 channels

到目前为止，我们讨论的所有 channels 都是双向的，也就是说数据可以在它们上面发送和接收。也可以创建单向 channels，即只能发送或接收数据的 channels。

```go
package main

import "fmt"

func sendData(sendch chan<- int) {
    sendch <- 10
}

func main() {
    sendch := make(chan<- int)
    go sendData(sendch)
    fmt.Println(<-sendch)
}
```

[在 playground 中运行程序](https://play.golang.org/p/PRKHxM-iRK)

在上面的程序中，我们在第 10 行创建了一个只发送的 channel `sendch`。`chan<- int` 表示一个只发送的 channel，因为箭头指向 `chan`。我们试图在第 12 行从一个只发送的 channel 接收数据。这是不允许的，当程序运行时，编译器会抱怨说：

*./prog.go:12:14: invalid operation: <-sendch (receive from send-only type chan<- int)*

**一切都很好，但是如果不能从只发送的 channel 读取，那写入它有什么意义呢！**

**这就是 channel 转换发挥作用的地方。可以将双向 channel 转换为只发送或只接收 channel，但反过来是不可能的。**

```go
package main

import "fmt"

func sendData(sendch chan<- int) {
    sendch <- 10
}

func main() {
    chnl := make(chan int)
    go sendData(chnl)
    fmt.Println(<-chnl)
}
```

[在 playground 中运行程序](https://play.golang.org/p/aqi_rJ1U8j)

在上面程序的第 10 行，创建了一个双向 channel `chnl`。在第 11 行将其作为参数传递给 `sendData` Goroutine。`sendData` 函数在第 5 行的参数 `sendch chan<- int` 中将此 channel 转换为只发送的 channel。所以现在该 channel 在 `sendData` Goroutine 中是只发送的，但在 main Goroutine 中是双向的。这个程序将输出 `10`。

### 关闭 channels 和在 channels 上使用 for range 循环

发送者可以关闭 channel 以通知接收者不会再向该 channel 发送任何数据。

接收者可以在从 channel 接收数据时使用额外的变量来检查 channel 是否已关闭。

```fallback
v, ok := <- ch
```

在上面的语句中，如果值是通过向 channel 成功发送操作而接收的，则 `ok` 为 true。如果 `ok` 为 false，则意味着我们正在从一个已关闭的 channel 读取。从已关闭的 channel 读取的值将是该 channel 类型的零值。例如，如果 channel 是一个 `int` channel，那么从已关闭的 channel 接收到的值将是 `0`。

```go
package main

import (
    "fmt"
)

func producer(chnl chan int) {
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}
func main() {
    ch := make(chan int)
    go producer(ch)
    for {
        v, ok := <-ch
        if ok == false {
            break
        }
        fmt.Println("Received ", v, ok)
    }
}
```

[在 playground 中运行程序](https://play.golang.org/p/XWmUKDA2Ri)

在上面的程序中，`producer` Goroutine 向 `chnl` channel 写入 0 到 9，然后关闭该 channel。main 函数在第 16 行有一个无限 `for` 循环，该循环使用变量 `ok` 在第 18 行检查 channel 是否已关闭。如果 `ok` 为 false，则表示 channel 已关闭，因此循环被打破。否则，打印接收到的值和 `ok` 的值。此程序打印：

```fallback
Received  0 true
Received  1 true
Received  2 true
Received  3 true
Received  4 true
Received  5 true
Received  6 true
Received  7 true
Received  8 true
Received  9 true
```

for 循环的 **for range** 形式可以用于从 channel 接收值，直到它关闭为止。

让我们使用 for range 循环重写上面的程序。

```go
package main

import (
    "fmt"
)

func producer(chnl chan int) {
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}
func main() {
    ch := make(chan int)
    go producer(ch)
    for v := range ch {
        fmt.Println("Received ",v)
    }
}
```

[在 playground 中运行程序](https://play.golang.org/p/JJ3Ida1r_6)

第 16 行的 `for range` 循环从 `ch` channel 接收数据，直到它关闭为止。一旦 `ch` 关闭，循环会自动退出。该程序输出：

```fallback
Received  0
Received  1
Received  2
Received  3
Received  4
Received  5
Received  6
Received  7
Received  8
Received  9
```

[另一个 channel 示例](../【GolangBot】22-Channels/#channel-的另一个示例)部分的程序可以使用 for range 循环重写，以获得更好的代码重用性。

如果仔细查看程序，你会注意到在 `calcSquares` 函数和 `calcCubes` 函数中都重复了寻找数字各个位的代码。我们将把该代码移到它自己的函数中，并并发调用它。

```go
package main

import (
    "fmt"
)

func digits(number int, dchnl chan int) {
    for number != 0 {
        digit := number % 10
        dchnl <- digit
        number /= 10
    }
    close(dchnl)
}
func calcSquares(number int, squareop chan int) {
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit * digit
    }
    cubeop <- sum
}

func main() {
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares+cubes)
}
```

[在 playground 中运行程序](https://play.golang.org/p/oL86W9Ui03)

上面程序中的 `digits` 函数现在包含了从数字中获取各个位的逻辑，并被 `calcSquares` 和 `calcCubes` 函数并发调用。一旦数字中没有更多的位，channel 在第 13 行被关闭。`calcSquares` 和 `calcCubes` Goroutines 使用 `for range` 循环监听它们各自的 channels，直到它关闭。程序的其余部分相同。这个程序也将打印：

```fallback
Final output 1536
```

这就是本教程的结尾了。channels 中还有一些概念，如[缓冲 channels](../【GolangBot】23-缓冲通道与工作池)、[工作池](../【GolangBot】23-缓冲通道与工作池)和[select](../【GolangBot】24-Select)。我们将在单独的教程中讨论它们。感谢阅读。


**下一篇教程 - [缓冲 Channels 和工作池](../【GolangBot】23-缓冲通道与工作池)**
