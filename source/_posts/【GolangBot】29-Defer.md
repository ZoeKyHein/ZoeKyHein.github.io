---
title: 【GolangBot】29-Defer
date: 2024-12-26 10:40:21
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---
欢迎来到 [Golang 教程系列](../golangbot/) 的第 29 个教程。

### 什么是 Defer？

**Defer 语句用于在包含它的函数返回之前执行一个[函数](../【GolangBot】6-函数)调用。** 这个定义可能看起来有些复杂，但通过一个例子会很容易理解。

### 示例

```go
package main

import (
	"fmt"
	"time"
)

func totalTime(start time.Time) {
	fmt.Printf("Total time taken %f seconds", time.Since(start).Seconds())
}

func test() {
	start := time.Now()
	defer totalTime(start)
	time.Sleep(2 * time.Second)
	fmt.Println("Sleep complete")
}

func main() {
	test()
}
```

[在 Playground 中运行](https://go.dev/play/p/5RgaRZ1Cj_h)

上面的程序是一个简单的示例，展示了 `defer` 的用法。在这个程序中，`defer` 用于计算 `test()` 函数的执行时间。`test()` 函数的开始时间作为参数传递给 `defer totalTime(start)`（第 14 行）。这个 `defer` 调用会在 `test()` 返回之前执行。`totalTime` 函数使用 `time.Since` 计算 `start` 和当前时间的差值并打印（第 9 行）。为了模拟 `test()` 中的一些计算操作，我们在第 15 行添加了一个 2 秒的 `sleep`。

运行该程序会输出：

```fallback
Sleep complete
Total time taken 2.000000 seconds
```

输出结果与 2 秒的 `sleep` 相符。在 `test()` 函数返回之前，`totalTime` 被调用并打印了 `test()` 执行的总时间。

### 参数评估

延迟函数的参数在 `defer` 语句执行时评估，而不是在实际函数调用时评估。

让我们通过一个例子来理解这一点。

```go
package main

import (
	"fmt"
)

func displayValue(a int) {
	fmt.Println("value of a in deferred function", a)
}

func main() {
	a := 5
	defer displayValue(a)
	a = 10
	fmt.Println("value of a before deferred function call", a)
}
```

[在 Playground 中运行](https://go.dev/play/p/K5UMFy1FAxg)

在上面的程序中，`a` 的初始值为 `5`（第 11 行）。当 `defer` 语句在第 12 行执行时，`a` 的值为 `5`，因此这将是传递给 `displayValue` 函数的参数。我们在第 13 行将 `a` 的值更改为 `10`。下一行打印 `a` 的值。该程序输出：

```fallback
value of a before deferred function call 10
value of a in deferred function 5
```

从上面的输出可以看出，尽管 `a` 的值在 `defer` 语句执行后变为 `10`，但实际的延迟函数调用 `displayValue(a)` 仍然打印 `5`。

### 延迟方法

`defer` 不仅限于[函数](../【GolangBot】6-函数)，也可以用于延迟[方法](../【GolangBot】17-方法)调用。让我们编写一个小程序来测试这一点。

```go
package main

import (
	"fmt"
)

type person struct {
	firstName string
	lastName  string
}

func (p person) fullName() {
	fmt.Printf("%s %s", p.firstName, p.lastName)
}

func main() {
	p := person{
		firstName: "John",
		lastName:  "Smith",
	}
	defer p.fullName()
	fmt.Printf("Welcome ")
}
```

[在 Playground 中运行](https://go.dev/play/p/0rTbGqe0p4l)

在上面的程序中，我们在第 21 行延迟了一个方法调用。程序的其余部分不言自明。该程序输出：

```fallback
Welcome John Smith
```

### 多个 Defer 调用按栈顺序执行

当一个函数有多个 `defer` 调用时，它们会被推入一个栈中，并按后进先出（LIFO）的顺序执行。

我们将编写一个小程序，使用 `defer` 栈来逆序打印字符串。

```go
package main

import (
	"fmt"
)

func main() {
	str := "Gopher"
	fmt.Printf("Original String: %s\n", string(str))
	fmt.Printf("Reversed String: ")
	for _, v := range str {
		defer fmt.Printf("%c", v)
	}
}
```

[在 Playground 中运行](https://go.dev/play/p/E28utiyg7Oj)

在上面的程序中，第 11 行的 `for range` 循环遍历字符串，并在第 12 行调用 `defer fmt.Printf("%c", v)`。这些延迟调用将被添加到一个栈中。

![stack of defers](https://golangbot.com/content/defer/stack-of-defers.png)

上面的图像表示 `defer` 调用被添加到栈后的内容。[栈](https://en.wikipedia.org/wiki/Stack_(abstract_data_type))是一种后进先出的数据结构。最后推入栈的 `defer` 调用将首先被弹出并执行。在这种情况下，`defer fmt.Printf("%c", 'n')` 将首先执行，因此字符串将逆序打印。

该程序将输出：

```fallback
Original String: Gopher
Reversed String: rehpoG
```

### Defer 的实际用途

在本节中，我们将探讨 `defer` 的一些实际用途。

`defer` 用于在代码流中无论发生什么情况都应执行函数调用的地方。让我们通过一个使用 [WaitGroup](../【GolangBot】23-缓冲通道与工作池/#waitgroup) 的程序来理解这一点。我们首先编写一个不使用 `defer` 的程序，然后修改它以使用 `defer`，并理解 `defer` 的实用性。

```go
package main

import (
	"fmt"
	"sync"
)

type rect struct {
	length int
	width  int
}

func (r rect) area(wg *sync.WaitGroup) {
	if r.length < 0 {
		fmt.Printf("rect %v's length should be greater than zero\n", r)
		wg.Done()
		return
	}
	if r.width < 0 {
		fmt.Printf("rect %v's width should be greater than zero\n", r)
		wg.Done()
		return
	}
	area := r.length * r.width
	fmt.Printf("rect %v's area %d\n", r, area)
	wg.Done()
}

func main() {
	var wg sync.WaitGroup
	r1 := rect{-67, 89}
	r2 := rect{5, -67}
	r3 := rect{8, 9}
	rects := []rect{r1, r2, r3}
	for _, v := range rects {
		wg.Add(1)
		go v.area(&wg)
	}
	wg.Wait()
	fmt.Println("All go routines finished executing")
}
```

[在 Playground 中运行](https://go.dev/play/p/1Wg8k__JcUy)

在上面的程序中，我们在第 8 行创建了一个 `rect` 结构体，并在第 13 行定义了一个 `area` 方法，用于计算矩形的面积。该方法检查矩形的长度和宽度是否小于零。如果是，则打印相应的错误消息，否则打印矩形的面积。

`main` 函数创建了三个 `rect` 类型的变量 `r1`、`r2` 和 `r3`。它们在第 34 行被添加到 `rects` 切片中。然后使用 `for range` 循环遍历该切片，并在第 37 行将 `area` 方法作为并发 [Goroutine](../【GolangBot】21-Goroutines) 调用。[WaitGroup](../【GolangBot】23-缓冲通道与工作池/#waitgroup) `wg` 用于确保主函数等待所有 Goroutine 执行完毕。这个 WaitGroup 作为参数传递给 `area` 方法，`area` 方法在第 16、21 和 26 行调用 `wg.Done()`，以通知主函数 Goroutine 已完成其工作。*如果你仔细观察，你会发现这些调用发生在 `area` 方法返回之前。无论代码流采取什么路径，`wg.Done()` 都应在方法返回之前调用，因此这些调用可以有效地替换为单个 `defer` 调用。*

让我们使用 `defer` 重写上面的程序。

在下面的程序中，我们删除了上面程序中的三个 `wg.Done()` 调用，并在第 14 行替换为单个 `defer wg.Done()` 调用。这使得代码更简洁和易读。

```go
package main

import (
	"fmt"
	"sync"
)

type rect struct {
	length int
	width  int
}

func (r rect) area(wg *sync.WaitGroup) {
	defer wg.Done()
	if r.length < 0 {
		fmt.Printf("rect %v's length should be greater than zero\n", r)
		return
	}
	if r.width < 0 {
		fmt.Printf("rect %v's width should be greater than zero\n", r)
		return
	}
	area := r.length * r.width
	fmt.Printf("rect %v's area %d\n", r, area)
}

func main() {
	var wg sync.WaitGroup
	r1 := rect{-67, 89}
	r2 := rect{5, -67}
	r3 := rect{8, 9}
	rects := []rect{r1, r2, r3}
	for _, v := range rects {
		wg.Add(1)
		go v.area(&wg)
	}
	wg.Wait()
	fmt.Println("All go routines finished executing")
}
```

[在 Playground 中运行](https://go.dev/play/p/xU4rYMHCPZl)

该程序输出：

```fallback
rect {8 9}'s area 72
rect {-67 89}'s length should be greater than zero
rect {5 -67}'s width should be greater than zero
All go routines finished executing
```

在上面的程序中使用 `defer` 还有一个好处。假设我们使用一个新的 `if` 条件向 `area` 方法添加另一个返回路径。如果 `wg.Done()` 的调用没有延迟，我们必须小心确保在这个新的返回路径中调用 `wg.Done()`。但由于 `wg.Done()` 的调用被延迟了，我们不需要担心向该方法添加新的返回路径。


**下一个教程 - [错误处理](../【GolangBot】30-错误处理)**
