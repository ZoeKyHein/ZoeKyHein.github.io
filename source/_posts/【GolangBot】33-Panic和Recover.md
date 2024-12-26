---
title: 【GolangBot】33-Panic和Recover
date: 2024-12-26 11:30:19
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: false
---
# 【GolangBot】33-Panic和Recover

欢迎来到 [Golang 教程系列](../golangbot/) 的第 33 个教程。

### 什么是 Panic？

在 Go 程序中处理异常情况的惯用方法是使用[错误](../【GolangBot】30-错误处理)。对于大多数程序中的异常情况，错误处理已经足够。

**但在某些情况下，程序在异常情况发生后无法继续执行。在这种情况下，我们使用 `panic` 来提前终止程序。当一个[函数](../【GolangBot】6-函数)遇到 panic 时，它的执行会停止，任何[defer](../【GolangBot】29-Defer)的函数会被执行，然后控制权返回给它的调用者。这个过程会一直持续，直到当前 [goroutine](../【GolangBot】21-Goroutines) 的所有函数都返回，此时程序会打印 panic 消息，接着是堆栈跟踪，然后终止。** 当我们编写一个示例程序时，这个概念会更加清晰。

**可以使用 `recover` 来恢复对 panic 程序的控制**，我们将在本教程的后面部分讨论这一点。

panic 和 recover 可以看作是其他语言（如 Java）中的 try-catch-finally 习惯用法，只不过它们在 Go 中很少使用。

### 什么时候应该使用 Panic？

一个重要的因素是，你应该尽量避免使用 panic 和 recover，尽可能使用[错误处理](../【GolangBot】30-错误处理)。只有在程序无法继续执行的情况下，才应该使用 panic 和 recover 机制。

panic 有两个有效的使用场景：

1. **不可恢复的错误，程序无法继续执行。** 一个例子是 Web 服务器无法绑定到所需的端口。在这种情况下，panic 是合理的，因为如果端口绑定本身失败，就没有其他事情可做了。
2. **程序员错误。** 假设我们有一个[方法](../【GolangBot】17-方法)接受一个指针作为参数，而有人使用 `nil` 参数调用此方法。在这种情况下，我们可以 panic，因为使用 `nil` 参数调用期望有效指针的方法是程序员的错误。

### Panic 示例

内置 `panic` 函数的签名如下：

```go
func panic(interface{})
```

传递给 panic 函数的参数将在程序终止时打印出来。当我们编写示例程序时，这一点会更加清楚。让我们马上编写一个示例程序。

我们将从一个展示 panic 如何工作的简单示例开始。

```go
package main

import (
	"fmt"
)

func fullName(firstName *string, lastName *string) {
	if firstName == nil {
		panic("runtime error: first name cannot be nil")
	}
	if lastName == nil {
		panic("runtime error: last name cannot be nil")
	}
	fmt.Printf("%s %s\n", *firstName, *lastName)
	fmt.Println("returned normally from fullName")
}

func main() {
	firstName := "Elon"
	fullName(&firstName, nil)
	fmt.Println("returned normally from main")
}
```

[在 Playground 中运行](https://play.golang.org/p/xQJYRSCu8S)

上面的程序是一个打印人名的全名的简单程序。第 7 行的 `fullName` 函数打印一个人的全名。该函数分别在第 8 行和第 11 行检查 `firstName` 和 `lastName` 指针是否为 `nil`。如果是 `nil`，函数会调用 `panic` 并传递相应的消息。此消息将在程序终止时打印。

运行此程序将打印以下输出：

```fallback
panic: runtime error: last name cannot be nil

goroutine 1 [running]:
main.fullName(0xc00006af58, 0x0)
	/tmp/sandbox210590465/prog.go:12 +0x193
main.main()
	/tmp/sandbox210590465/prog.go:20 +0x4d
```

让我们分析这个输出，以了解 panic 的工作原理以及程序 panic 时如何打印堆栈跟踪。

在第 19 行，我们将 `Elon` 赋值给 `firstName`。我们在第 20 行调用 `fullName` 函数，并将 `lastName` 设置为 `nil`。因此，第 11 行的条件将满足，程序将 panic。当遇到 panic 时，程序执行终止，传递给 panic 函数的参数会被打印，然后是堆栈跟踪。由于程序在第 12 行的 panic 函数调用后终止，第 13、14 和 15 行的代码将不会执行。

该程序首先打印传递给 `panic` 函数的消息：

```fallback
panic: runtime error: last name cannot be nil
```

然后打印堆栈跟踪。

程序在第 12 行的 `fullName` 函数中 panic，因此：

```fallback
goroutine 1 [running]:
main.fullName(0xc00006af58, 0x0)
	/tmp/sandbox210590465/prog.go:12 +0x193
```

将首先打印。然后打印堆栈中的下一个项目。在我们的例子中，第 20 行调用 `fullName` 的地方是堆栈跟踪中的下一个项目。因此，接下来会打印它。

```fallback
main.main()
	/tmp/sandbox210590465/prog.go:20 +0x4d
```

现在我们已经到达了导致 panic 的顶层函数，并且没有更多的层级，因此没有更多内容可打印。

### 另一个示例

panic 也可能由运行时发生的错误引起，例如尝试访问切片中不存在的索引。

让我们编写一个由于切片越界访问而导致 panic 的示例。

```go
package main

import (
	"fmt"
)

func slicePanic() {
	n := []int{5, 7, 4}
	fmt.Println(n[4])
	fmt.Println("normally returned from a")
}

func main() {
	slicePanic()
	fmt.Println("normally returned from main")
}
```

[在 Playground 中运行](https://play.golang.org/p/__PAabvchxt)

在上面的程序中，第 9 行我们尝试访问 `n[4]`，这是[切片](../【GolangBot】11-数组和切片)中的无效索引。该程序将 panic，并输出以下内容：

```fallback
panic: runtime error: index out of range [4] with length 3

goroutine 1 [running]:
main.slicePanic()
	/tmp/sandbox942516049/prog.go:9 +0x1d
main.main()
	/tmp/sandbox942516049/prog.go:13 +0x22
```

### Panic 期间的 Defer 调用

让我们回顾一下 panic 的作用。**当一个函数遇到 panic 时，它的执行会停止，任何延迟的函数会被执行，然后控制权返回给它的调用者。这个过程会一直持续，直到当前 goroutine 的所有函数都返回，此时程序会打印 panic 消息，接着是堆栈跟踪，然后终止。**

在上面的示例中，我们没有延迟任何函数调用。如果存在延迟的函数调用，它会被执行，然后控制权返回给它的调用者。

让我们稍微修改上面的示例并使用 defer 语句。

```go
package main

import (
	"fmt"
)

func fullName(firstName *string, lastName *string) {
	defer fmt.Println("deferred call in fullName")
	if firstName == nil {
		panic("runtime error: first name cannot be nil")
	}
	if lastName == nil {
		panic("runtime error: last name cannot be nil")
	}
	fmt.Printf("%s %s\n", *firstName, *lastName)
	fmt.Println("returned normally from fullName")
}

func main() {
	defer fmt.Println("deferred call in main")
	firstName := "Elon"
	fullName(&firstName, nil)
	fmt.Println("returned normally from main")
}
```

[在 Playground 中运行](https://play.golang.org/p/oUFnu-uTmC)

唯一的更改是在第 8 行和第 20 行添加了延迟函数调用。

该程序打印：

```fallback
deferred call in fullName
deferred call in main
panic: runtime error: last name cannot be nil

goroutine 1 [running]:
main.fullName(0xc00006af28, 0x0)
	/tmp/sandbox451943841/prog.go:13 +0x23f
main.main()
	/tmp/sandbox451943841/prog.go:22 +0xc6
```

当程序在第 13 行 panic 时，首先执行任何延迟的函数调用，然后控制权返回给调用者，调用者的延迟调用被执行，依此类推，直到到达顶层调用者。

在我们的例子中，`fullName` 函数第 8 行的 `defer` 语句首先执行。这会打印以下消息：

```fallback
deferred call in fullName
```

然后控制权返回到 `main` 函数，其延迟调用被执行，因此打印：

```fallback
deferred call in main
```

现在控制权已经到达顶层函数，因此程序会打印 panic 消息，接着是堆栈跟踪，然后终止。

### 从 Panic 中恢复

*recover* 是一个内置函数，用于恢复对 panic 程序的控制。

recover 函数的签名如下：

```go
func recover() interface{}
```

recover 只有在延迟函数内部调用时才有用。在延迟函数内部执行 recover 调用会通过恢复正常执行来停止 panic 序列，并检索传递给 panic 函数调用的错误消息。如果在延迟函数外部调用 recover，它将不会停止 panic 序列。

让我们修改我们的程序并使用 recover 在 panic 后恢复正常执行。

```go
package main

import (
	"fmt"
)

func recoverFullName() {
	if r := recover(); r != nil {
		fmt.Println("recovered from ", r)
	}
}

func fullName(firstName *string, lastName *string) {
	defer recoverFullName()
	if firstName == nil {
		panic("runtime error: first name cannot be nil")
	}
	if lastName == nil {
		panic("runtime error: last name cannot be nil")
	}
	fmt.Printf("%s %s\n", *firstName, *lastName)
	fmt.Println("returned normally from fullName")
}

func main() {
	defer fmt.Println("deferred call in main")
	firstName := "Elon"
	fullName(&firstName, nil)
	fmt.Println("returned normally from main")
}
```

[在 Playground 中运行](https://play.golang.org/p/enCM-dd5DUr)

第 7 行的 `recoverFullName()` 函数调用 `recover()`，它返回传递给 `panic` 函数调用的值。在这里，我们只是打印 recover 返回的值。`recoverFullName()` 在第 14 行的 `fullName` 函数中被延迟。

当 `fullName` panic 时，延迟函数 `recoverName()` 将被调用，它使用 `recover()` 来停止 panic 序列。

该程序将打印：

```fallback
recovered from  runtime error: last name cannot be nil
returned normally from main
deferred call in main
```

当程序在第 19 行 panic 时，延迟的 `recoverFullName` 函数被调用，它反过来调用 `recover()` 来恢复对 panic 序列的控制。第 8 行的 `recover()` 调用返回传递给 `panic()` 的参数，因此打印：

```fallback
recovered from  runtime error: last name cannot be nil
```

在执行 `recover()` 后，panic 停止，控制权返回给调用者，在本例中是 `main` 函数。由于 panic 已恢复，程序从 `main` 的第 29 行继续正常执行 😃。它打印 `returned normally from main`，然后是 `deferred call in main`。

让我们再看一个示例，我们从由于访问切片的无效索引引起的 panic 中恢复。

```go
package main

import (
	"fmt"
)

func recoverInvalidAccess() {
	if r := recover(); r != nil {
		fmt.Println("Recovered", r)
	}
}

func invalidSliceAccess() {
	defer recoverInvalidAccess()
	n := []int{5, 7, 4}
	fmt.Println(n[4])
	fmt.Println("normally returned from a")
}

func main() {
	invalidSliceAccess()
	fmt.Println("normally returned from main")
}
```

[在 Playground 中运行](https://play.golang.org/p/Bth98An9Ah0)

运行上面的程序将输出：

```fallback
Recovered runtime error: index out of range [4] with length 3
normally returned from main
```

从输出中可以看出，我们已经从 panic 中恢复。

### 恢复后获取堆栈跟踪

如果我们从 panic 中恢复，我们会丢失有关 panic 的堆栈跟踪。即使在上面的程序恢复后，我们也丢失了堆栈跟踪。

有一种方法可以使用 Debug [包](../【GolangBot】7-包)的 [PrintStack](https://golang.org/pkg/runtime/debug/#PrintStack) 函数打印堆栈跟踪。

```go
package main

import (
	"fmt"
	"runtime/debug"
)

func recoverFullName() {
	if r := recover(); r != nil {
		fmt.Println("recovered from ", r)
		debug.PrintStack()
	}
}

func fullName(firstName *string, lastName *string) {
	defer recoverFullName()
	if firstName == nil {
		panic("runtime error: first name cannot be nil")
	}
	if lastName == nil {
		panic("runtime error: last name cannot be nil")
	}
	fmt.Printf("%s %s\n", *firstName, *lastName)
	fmt.Println("returned normally from fullName")
}

func main() {
	defer fmt.Println("deferred call in main")
	firstName := "Elon"
	fullName(&firstName, nil)
	fmt.Println("returned normally from main")
}
```

[在 Playground 中运行](https://play.golang.org/p/9VxcG4Gt-MU)

在上面的程序中，我们在第 11 行使用 `debug.PrintStack()` 来打印堆栈跟踪。

该程序将打印：

```fallback
recovered from  runtime error: last name cannot be nil
goroutine 1 [running]:
runtime/debug.Stack(0x37, 0x0, 0x0)
	/usr/local/go-faketime/src/runtime/debug/stack.go:24 +0x9d
runtime/debug.PrintStack()
	/usr/local/go-faketime/src/runtime/debug/stack.go:16 +0x22
main.recoverFullName()
	/tmp/sandbox771195810/prog.go:11 +0xb4
panic(0x4a1b60, 0x4dc300)
	/usr/local/go-faketime/src/runtime/panic.go:969 +0x166
main.fullName(0xc0000a2f28, 0x0)
	/tmp/sandbox771195810/prog.go:21 +0x1cb
main.main()
	/tmp/sandbox771195810/prog.go:30 +0xc6
returned normally from main
deferred call in main
```

从输出中可以看出，panic 已恢复，并打印了 `recovered from runtime error: last name cannot be nil`。接着打印堆栈跟踪。然后：

```fallback
returned normally from main
deferred call in main
```

在 panic 恢复后打印。

### Panic、Recover 和 Goroutines

recover 只有在从发生 panic 的同一个 [goroutine](../【GolangBot】21-Goroutines) 中调用时才有效。**无法从不同 goroutine 中发生的 panic 中恢复。** 让我们通过一个示例来理解这一点。

```go
package main

import (
	"fmt"
)

func recovery() {
	if r := recover(); r != nil {
		fmt.Println("recovered:", r)
	}
}

func sum(a int, b int) {
	defer recovery()
	fmt.Printf("%d + %d = %d\n", a, b, a+b)
	done := make(chan bool)
	go divide(a, b, done)
	<-done
}

func divide(a int, b int, done chan bool) {
	fmt.Printf("%d / %d = %d", a, b, a/b)
	done <- true
}

func main() {
	sum(5, 0)
	fmt.Println("normally returned from main")
}
```

[在 Playground 中运行](https://play.golang.org/p/yCPL_4lqbzk)

在上面的程序中，`divide()` 函数将在第 22 行 panic，因为 `b` 为零，无法将数字除以零。`sum()` 函数调用了一个延迟函数 `recovery()`，用于从 panic 中恢复。`divide()` 函数在第 17 行作为一个单独的 goroutine 被调用。我们在第 18 行等待 `done` [通道](../GolangBot】22-Channels)以确保 `divide()` 完成执行。

你认为程序的输出会是什么？panic 会被恢复吗？答案是否定的。panic 不会被恢复。这是因为 `recovery` 函数存在于不同的 goroutine 中，而 panic 发生在 `divide()` 函数中的不同 goroutine 中。因此，无法恢复。

运行此程序将打印：

```fallback
5 + 0 = 5
panic: runtime error: integer divide by zero

goroutine 18 [running]:
main.divide(0x5, 0x0, 0xc0000a2000)
	/tmp/sandbox877118715/prog.go:22 +0x167
created by main.sum
	/tmp/sandbox877118715/prog.go:17 +0x1a9
```

从输出中可以看出，恢复没有发生。

如果 `divide()` 函数在同一个 goroutine 中被调用，我们将能够从 panic 中恢复。

如果将程序的第 17 行从：

```go
go divide(a, b, done)
```

更改为：

```go
divide(a, b, done)
```

恢复将会发生，因为 panic 发生在同一个 goroutine 中。如果程序使用上述更改运行，它将打印：

```fallback
5 + 0 = 5
recovered: runtime error: integer divide by zero
normally returned from main
```

本教程到此结束。

以下是我们在本教程中学到的内容的快速回顾：

- 什么是 Panic？
- 什么时候应该使用 Panic？
- Panic 使用示例
- Panic 期间的 Defer 调用
- 从 Panic 中恢复
- 恢复后获取堆栈跟踪
- Panic、Recover 和 Goroutines


**下一个教程 - [一等公民：函数](../【GolangBot】34-一等公民：函数)**
