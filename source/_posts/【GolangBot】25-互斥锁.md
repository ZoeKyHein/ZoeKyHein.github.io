---
title: 【GolangBot】25-互斥锁
date: 2024-12-6 08:54:50
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---
# GolangBot】25-互斥锁

欢迎来到 [Golang系列教程](../golangbot/) 的第25篇教程。

在本教程中，我们将学习互斥锁（mutexes）。我们还将学习如何使用互斥锁和[通道](../【GolangBot】22-Channels)来解决竞态条件。

### 临界区

在开始学习互斥锁之前，理解并发编程中[临界区](https://en.wikipedia.org/wiki/Critical_section)的概念很重要。当程序并发运行时，修改共享资源的代码部分不应该同时被多个 [Goroutines](../【GolangBot】22-Channels) 访问。这段修改共享资源的代码被称为临界区。例如，假设我们有一段将变量 x 加 1 的代码。

```fallback
x = x + 1
```

只要上面这段代码被单个 Goroutine 访问，就不会有任何问题。

让我们看看当有多个 Goroutine 并发运行时，为什么这段代码会失败。为了简单起见，让我们假设有 2 个 Goroutine 并发运行上面的代码。

在内部，系统将按以下步骤执行上述代码（还有更多涉及寄存器、加法工作原理等技术细节，但为了本教程的目的，让我们假设这是三个步骤）：

1. 获取 x 的当前值
2. 计算 x + 1
3. 将步骤 2 中计算的值赋给 x

当这三个步骤只由一个 Goroutine 执行时，一切正常。

让我们讨论当 2 个 Goroutine 并发运行这段代码时会发生什么。下图描述了当两个 Goroutine 并发访问代码 `x = x + 1` 时可能发生的一种情况。

![critical-section](https://golangbot.com/content/images/2017/08/cs5.png)

我们假设 x 的初始值为 0。*Goroutine 1* 获取 x 的初始值，计算 x + 1，但在它能将计算的值赋给 x 之前，系统上下文切换到了 `Goroutine 2`。现在 `Goroutine 2` 获取 `x` 的初始值（仍然是 `0`），计算 `x + 1`。之后，系统再次上下文切换到 *Goroutine 1*。现在 *Goroutine 1* 将其计算的值 *1* 赋给 *x*，因此 x 变成了 1。然后 *Goroutine 2* 再次开始执行，将其计算的值（同样是 `1`）赋给 `x`，因此在两个 Goroutine 执行后 `x` 的值是 `1`。

现在让我们看看可能发生的另一种情况。

![critical-section](https://golangbot.com/content/images/2017/08/cs-6.png)

在上面的情况中，`Goroutine 1` 开始执行并完成其所有三个步骤，因此 x 的值变成 `1`。然后 `Goroutine 2` 开始执行。现在 x 的值是 1，当 `Goroutine 2` 完成执行时，x 的值是 `2`。

从这两种情况可以看出，x 的最终值是 `1` 或 `2`，这取决于上下文切换是如何发生的。这种程序输出依赖于 Goroutine 执行顺序的不良情况称为**[竞态条件](https://en.wikipedia.org/wiki/Race_condition)**。

在上述情况下，如果在任何时候只允许一个 Goroutine 访问代码的临界区，就可以避免竞态条件。这可以通过使用互斥锁来实现。

### 互斥锁

互斥锁用于提供一种锁定机制，以确保在任何时候只有一个 Goroutine 在运行代码的临界区，从而防止竞态条件的发生。

互斥锁在 [sync](https://golang.org/pkg/sync/) 包中提供。[Mutex](https://tip.golang.org/pkg/sync/#Mutex) 上定义了两个方法，即 [Lock](https://tip.golang.org/pkg/sync/#Mutex.Lock) 和 [Unlock](https://tip.golang.org/pkg/sync/#Mutex.Unlock)。位于 `Lock` 和 `Unlock` 调用之间的任何代码将只能由一个 Goroutine 执行，从而避免竞态条件。

```fallback
mutex.Lock()
x = x + 1
mutex.Unlock()
```

在上面的代码中，`x = x + 1` 在任何时候都只能由一个 Goroutine 执行，从而防止竞态条件。

**如果一个 Goroutine 已经持有锁，而新的 Goroutine 试图获取锁，新的 Goroutine 将被阻塞，直到互斥锁被解锁。**

### 存在竞态条件的程序

在本节中，我们将编写一个存在竞态条件的程序，在接下来的章节中，我们将修复这个竞态条件。

```go
package main
import (
    "fmt"
    "sync"
)
var x = 0
func increment(wg *sync.WaitGroup) {
    x = x + 1
    wg.Done()
}
func main() {
    var w sync.WaitGroup
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

在上面的程序中，`increment` 函数将 x 的值加 1，然后在 [WaitGroup](../【GolangBot】23-缓冲通道与工作池/#waitgroup) 上调用 `Done()` 来通知其完成。

我们在程序中生成了 1000 个 `increment` Goroutine。所有这些 Goroutine 都并发运行，当多个 Goroutine 试图同时访问 x 的值时，就会出现竞态条件。

*请在本地运行此程序，因为 playground 是确定性的，竞态条件不会在 playground 中出现。* 在本地机器上多次运行此程序，你会看到由于竞态条件，每次的输出都会不同。我遇到的一些输出是 `final value of x 941`、`final value of x 928`、`final value of x 922` 等。

### 使用互斥锁解决竞态条件

在上面的程序中，我们生成了 1000 个 Goroutine。如果每个都将 x 的值加 1，x 的最终期望值应该是 1000。在本节中，我们将使用互斥锁修复上面程序中的竞态条件。

```go
package main
import (
    "fmt"
    "sync"
)
var x = 0
func increment(wg *sync.WaitGroup, m *sync.Mutex) {
    m.Lock()
    x = x + 1
    m.Unlock()
    wg.Done()    
}
func main() {
    var w sync.WaitGroup
    var m sync.Mutex
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, &m)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

[在 playground 中运行](https://play.golang.org/p/VX9dwGhR62)

[Mutex](https://golang.org/pkg/sync/#Mutex) 是一个结构体类型，我们创建了一个零值变量 `m`，类型为 Mutex。在上面的程序中，我们修改了 `increment` 函数，使得递增 x 的代码 `x = x + 1` 位于 `m.Lock()` 和 `m.Unlock()` 之间。现在这段代码没有任何竞态条件，因为在任何时候只允许一个 Goroutine 执行这段代码。

现在运行这个程序，它将输出

```fallback
final value of x 1000
```

重要的是要传递互斥锁的地址。如果互斥锁通过值传递而不是传递地址，每个 Goroutine 将有其自己的互斥锁副本，竞态条件仍然会发生。

### 使用通道解决竞态条件

我们也可以使用通道来解决竞态条件。让我们看看如何实现。

```go
package main
import (
    "fmt"
    "sync"
)
var x = 0
func increment(wg *sync.WaitGroup, ch chan bool) {
    ch <- true
    x = x + 1
    <- ch
    wg.Done()    
}
func main() {
    var w sync.WaitGroup
    ch := make(chan bool, 1)
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, ch)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

[在 playground 中运行](https://play.golang.org/p/M1fPEK9lYz)

在上面的程序中，我们创建了一个容量为 `1` 的[缓冲通道](../【GolangBot】23-缓冲通道与工作池)，并将其传递给 `increment` Goroutine。这个缓冲通道用于确保只有一个 Goroutine 访问增加 x 的临界区代码。这是通过在增加 x 之前向缓冲通道传递 `true` 来实现的。由于缓冲通道的容量为 `1`，所有其他试图向该通道写入的 Goroutine 都会被阻塞，直到在增加 x 后从该通道读取值。这有效地只允许一个 Goroutine 访问临界区。

这个程序也会打印

```fallback
final value of x 1000
```

### 互斥锁 vs 通道

我们已经使用互斥锁和通道解决了竞态条件问题。那么我们如何决定在什么时候使用什么呢？答案在于你试图解决的问题。如果你要解决的问题更适合使用互斥锁，那就使用互斥锁。如果需要的话，不要犹豫使用互斥锁。如果问题似乎更适合使用通道，那就使用它 :)。

大多数 Go 新手试图使用通道来解决每个并发问题，因为这是语言的一个很酷的特性。这是错误的。语言给了我们使用互斥锁或通道的选择，选择任何一个都没有错。

一般来说，当 Goroutine 需要相互通信时使用通道，当只有一个 Goroutine 需要访问代码的临界区时使用互斥锁。

对于我们上面解决的问题，我更倾向于使用互斥锁，因为这个问题不需要 goroutine 之间的任何通信。因此互斥锁会是一个自然的选择。

我的建议是为问题选择合适的工具，而不是试图让问题适应工具 :)


**下一篇教程 - [Structs Instead of Classes](../【GolangBot】26-结构体代替类.md)**
