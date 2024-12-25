---
title: 【GolangBot】21-Goroutines
date: 2024-12-17 08:53:27
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---

# 【GolangBot】21-Goroutines

欢迎来到[Golang教程系列](../golangbot/)的第21篇教程。

在[上一篇教程](../【GolangBot】20-并发介绍)中，我们讨论了并发以及它与并行的区别。在本教程中，我们将讨论Go如何使用Goroutines实现并发。

### 什么是Goroutines？

Goroutines是与其他[函数](../【GolangBot】6-函数)或[方法](../【GolangBot】17-方法)并发运行的函数或方法。Goroutines可以被认为是轻量级线程。与线程相比，创建Goroutine的成本很小。因此，Go应用程序通常会有数千个Goroutines并发运行。

### Goroutines相对于线程的优势

- 与线程相比，Goroutines非常轻量。它们的栈大小只有几kb，而且栈可以根据应用程序的需要增长和收缩，而线程的栈大小必须指定且是固定的。
- Goroutines被多路复用到更少数量的OS线程上。一个程序中可能只有一个线程却有数千个Goroutines。如果该线程中的任何Goroutine阻塞（比如等待用户输入），则会创建另一个OS线程，并将剩余的Goroutines移动到新的OS线程。这些都由运行时处理，我们作为程序员不需要关心这些复杂的细节，而是得到了一个处理并发的清晰API。
- Goroutines通过channels进行通信。channels的设计可以防止在使用Goroutines访问共享内存时发生竞态条件。channels可以被认为是Goroutines用来通信的管道。我们将在下一个教程中详细讨论channels。

### 如何启动Goroutine？

在函数或方法调用前加上关键字`go`，你就会有一个新的并发运行的Goroutine。

让我们创建一个Goroutine :)

```go
package main

import (
    "fmt"
)

func hello() {
    fmt.Println("Hello world goroutine")
}
func main() {
    go hello()
    fmt.Println("main function")
}
```

[在playground中运行程序](https://play.golang.org/p/zC78_fc1Hn)

在第11行，`go hello()`启动了一个新的Goroutine。现在`hello()`函数将与`main()`函数并发运行。main函数在它自己的Goroutine中运行，它被称为*主Goroutine*。

*运行这个程序，你会有一个惊喜！*

这个程序只输出文本`main function`。我们启动的Goroutine怎么了？我们需要理解goroutines的两个主要特性来理解为什么会这样。

- **当启动一个新的Goroutine时，goroutine调用会立即返回。与函数不同，控制不会等待Goroutine执行完成。控制会立即返回到Goroutine调用后的下一行代码，而且会忽略Goroutine的任何返回值。**
- **主Goroutine必须运行才能让其他Goroutines运行。如果主Goroutine终止，则程序将被终止，其他Goroutine也不会运行。**

我想现在你应该能理解为什么我们的Goroutine没有运行了。在第11行调用`go hello()`之后，控制立即返回到下一行代码，没有等待hello goroutine完成就打印了`main function`。然后主Goroutine终止了，因为没有其他要执行的代码，因此`hello` Goroutine没有机会运行。

让我们现在来修复这个问题。

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

[在playground中运行程序](https://play.golang.org/p/U9ZZuSql8-)

在上面程序的第13行，我们调用了*time*包的[Sleep](https://golang.org/pkg/time/#Sleep)方法，它会使执行该方法的goroutine休眠。在这个例子中，主goroutine被置于休眠状态1秒。现在调用`go hello()`有足够的时间在主Goroutine终止之前执行。这个程序首先打印`Hello world goroutine`，等待1秒，然后打印`main function`。

*在主Goroutine中使用sleep来等待其他Goroutines完成执行是一种hack方法，我们用它来理解Goroutines是如何工作的。channels可以用来阻塞主Goroutine直到所有其他Goroutines完成执行。我们将在下一个教程中讨论channels。*

### 启动多个Goroutines

让我们再写一个启动多个Goroutines的程序来更好地理解Goroutines。

```go
package main

import (  
    "fmt"
    "time"
)

func numbers() {  
    for i := 1; i <= 5; i++ {
        time.Sleep(250 * time.Millisecond)
        fmt.Printf("%d ", i)
    }
}
func alphabets() {  
    for i := 'a'; i <= 'e'; i++ {
        time.Sleep(400 * time.Millisecond)
        fmt.Printf("%c ", i)
    }
}
func main() {  
    go numbers()
    go alphabets()
    time.Sleep(3000 * time.Millisecond)
    fmt.Println("main terminated")
}
```

[在playground中运行](https://play.golang.org/p/oltn5nw0w3)

上面的程序在第21行和第22行启动了两个Goroutines。这两个Goroutines现在并发运行。`numbers` Goroutine最初休眠250毫秒然后打印`1`，然后再次休眠并打印`2`，这个循环一直持续到打印5。同样，`alphabets` Goroutine打印从`a`到`e`的字母，休眠时间为400毫秒。主Goroutine启动`numbers`和`alphabets` Goroutines，休眠3000毫秒然后终止。

这个程序输出

```fallback
1 a 2 3 b 4 c 5 d e main terminated
```

下面的图片描述了这个程序是如何工作的。请在新标签页中打开图片以获得更好的可视性 :)

![img](https://golangbot.com/content/images/2017/07/Goroutines-explained.png)

图片中蓝色部分代表*numbers Goroutine*，栗色部分代表*alphabets Goroutine*，绿色部分代表*main Goroutine*，最后的黑色部分合并了上面三个部分，展示了程序如何工作。每个框顶部的字符串如*0 ms, 250 ms*代表毫秒时间，输出显示在每个框的底部，如*1, 2, 3*等。蓝色框告诉我们`1`在`250 ms`后打印，`2`在`500 ms`后打印，以此类推。最后黑色框的底部有值`1 a 2 3 b 4 c 5 d e main terminated`，这也是程序的输出。这张图片很好理解，你能够理解程序是如何工作的。

**下一篇教程 - [Channels](../【GolangBot】22-Channels)**
