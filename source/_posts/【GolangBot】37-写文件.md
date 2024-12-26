---
title: 【GolangBot】37-写文件
date: 2024-12-26 14:20:14
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---
# 【GolangBot】37-写文件
欢迎来到 [Golang 教程系列](../golangbot/) 的第 37 个教程。

在本教程中，我们将学习如何使用 Go 将数据写入文件。我们还将学习如何并发地写入文件。

本教程包含以下部分：

- 将字符串写入文件
- 将字节写入文件
- 逐行将数据写入文件
- 追加到文件
- 并发写入文件

请在你的本地系统中运行本教程的所有程序，因为 playground 不支持文件操作。

### 将字符串写入文件

最常见的文件写入操作之一是将字符串写入文件。这非常简单。它包括以下步骤：

1. 创建文件
2. 将字符串写入文件

让我们直接看代码。

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.Create("test.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	l, err := f.WriteString("Hello World")
	if err != nil {
		fmt.Println(err)
        f.Close()
		return
	}
	fmt.Println(l, "bytes written successfully")
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
}
```

程序第 9 行的 `create` 函数创建了一个名为 *test.txt* 的文件。如果该文件已经存在，则 `create` 函数会截断该文件。此函数返回一个 [文件描述符](https://pkg.go.dev/os#File)。

在第 14 行，我们使用 `WriteString` 方法将字符串 **Hello World** 写入文件。此方法返回写入的字节数和可能的错误。

最后，我们在第 21 行关闭文件。

上述程序将打印：

```fallback
11 bytes written successfully
```

你可以在运行此程序的目录中找到一个名为 **test.txt** 的文件。如果你使用任何文本编辑器打开该文件，你会发现它包含文本 **Hello World**。

### 将字节写入文件

将字节写入文件与将字符串写入文件非常相似。我们将使用 [Write](https://pkg.go.dev/os#File.Write) 方法将字节写入文件。以下程序将字节切片写入文件。

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.Create("/home/naveen/bytes")
	if err != nil {
		fmt.Println(err)
		return
	}
	d2 := []byte{104, 101, 108, 108, 111, 32, 98, 121, 116, 101, 115}
	n2, err := f.Write(d2)
	if err != nil {
		fmt.Println(err)
        f.Close()
		return
	}
	fmt.Println(n2, "bytes written successfully")
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
}
```

在上述程序中，第 15 行我们使用 **Write** 方法将字节切片写入目录 `/home/naveen` 中名为 `bytes` 的文件。你可以将此目录更改为其他目录。程序的其余部分不言自明。该程序将打印 `11 bytes written successfully`，并创建一个名为 `bytes` 的文件。打开该文件，你可以看到它包含文本 `hello bytes`。

### 逐行将字符串写入文件

另一个常见的文件操作是需要逐行将字符串写入文件。在本节中，我们将编写一个程序来创建一个包含以下内容的文件。

```fallback
Welcome to the world of Go.
Go is a compiled language.
It is easy to learn Go.
```

让我们直接看代码。

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.Create("lines")
	if err != nil {
		fmt.Println(err)
                f.Close()
		return
	}
	d := []string{"Welcome to the world of Go1.", "Go is a compiled language.", "It is easy to learn Go."}

	for _, v := range d {
		fmt.Fprintln(f, v)
        if err != nil {
			fmt.Println(err)
			return
		}
	}
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("file written successfully")
}
```

在程序的第 9 行，我们创建了一个名为 **lines** 的新文件。在第 17 行，我们使用 for range 循环遍历数组，并使用 [Fprintln](https://golang.org/pkg/fmt/#Fprintln) 函数将行写入文件。**Fprintln** 函数以 `io.writer` 作为参数，并追加一个新行，这正是我们想要的。运行此程序将打印 `file written successfully`，并在当前目录中创建一个文件 `lines`。文件 `lines` 的内容如下：

```fallback
Welcome to the world of Go1.
Go is a compiled language.
It is easy to learn Go.
```

### 追加到文件

在本节中，我们将向上一节中创建的 `lines` 文件追加一行。我们将追加行 **File handling is easy** 到 `lines` 文件。

文件必须以追加和只写模式打开。这些标志作为参数传递给 [Open](https://pkg.go.dev/os#OpenFile) 函数。文件以追加模式打开后，我们将新行添加到文件中。

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.OpenFile("lines", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println(err)
		return
	}
	newLine := "File handling is easy."
	_, err = fmt.Fprintln(f, newLine)
	if err != nil {
		fmt.Println(err)
                f.Close()
		return
	}
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("file appended successfully")
}
```

在程序的第 9 行，我们以追加和只写模式打开文件。文件成功打开后，我们在第 15 行向文件添加新行。该程序将打印 `file appended successfully`。运行此程序后，`lines` 文件的内容将是：

```fallback
Welcome to the world of Go1.
Go is a compiled language.
It is easy to learn Go.
File handling is easy.
```

### 并发写入文件

当多个 [goroutines](../【GolangBot】21-Goroutines) 并发写入文件时，我们将遇到 [竞态条件](../【GolangBot】25-互斥锁/#使用互斥锁解决竞态条件)。因此，必须使用通道来协调对文件的并发写入。

我们将编写一个程序，创建 100 个 goroutines。每个 goroutine 将并发生成一个随机数，从而总共生成 100 个随机数。这些随机数将被写入文件。我们将通过以下方法解决 [竞态条件](../【GolangBot】25-互斥锁/#使用互斥锁解决竞态条件) 问题。

1. 创建一个通道，用于读取和写入生成的随机数。
2. 创建 100 个生产者 goroutines。每个 goroutine 将生成一个随机数，并将随机数写入通道。
3. 创建一个消费者 goroutine，它将从通道中读取并将生成的随机数写入文件。因此，我们只有一个 goroutine 并发写入文件，从而避免竞态条件 :)
4. 完成后关闭文件。

让我们首先编写生成随机数的 `produce` 函数。

```fallback
func produce(data chan int, wg *sync.WaitGroup) {
	n := rand.Intn(999)
	data <- n
	wg.Done()
}
```

上述函数生成一个随机数并将其写入通道 `data`，然后在 [waitgroup](../【GolangBot】23-缓冲通道与工作池/#waitgroup) 上调用 `Done` 以通知它已完成其任务。

让我们现在转到写入文件的函数。

```fallback
func consume(data chan int, done chan bool) {
	f, err := os.Create("concurrent")
	if err != nil {
		fmt.Println(err)
		return
	}
	for d := range data {
		_, err = fmt.Fprintln(f, d)
		if err != nil {
			fmt.Println(err)
			f.Close()
			done <- false
			return
		}
	}
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		done <- false
		return
	}
	done <- true
}
```

`consume` 函数创建一个名为 `concurrent` 的文件。然后它从 `data` 通道读取随机数并将其写入文件。一旦它读取并写入了所有随机数，它就会向 `done` 通道写入 `true`，以通知它已完成其任务。

让我们编写 `main` 函数并完成此程序。我提供了整个程序如下。

```go
package main

import (
	"fmt"
	"math/rand"
	"os"
	"sync"
)

func produce(data chan int, wg *sync.WaitGroup) {
	n := rand.Intn(999)
	data <- n
	wg.Done()
}

func consume(data chan int, done chan bool) {
	f, err := os.Create("concurrent")
	if err != nil {
		fmt.Println(err)
		return
	}
	for d := range data {
		_, err = fmt.Fprintln(f, d)
		if err != nil {
			fmt.Println(err)
			f.Close()
			done <- false
			return
		}
	}
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		done <- false
		return
	}
	done <- true
}

func main() {
	data := make(chan int)
	done := make(chan bool)
	wg := sync.WaitGroup{}
	for i := 0; i < 100; i++ {
		wg.Add(1)
		go produce(data, &wg)
	}
	go consume(data, done)
	go func() {
		wg.Wait()
		close(data)
	}()
	d := <-done
	if d {
		fmt.Println("File written successfully")
	} else {
		fmt.Println("File writing failed")
	}
}
```

主函数在第 41 行创建 `data` 通道，从中读取随机数并写入。第 42 行的 `done` 通道由 `consume` goroutine 用于通知 `main` 它已完成其任务。第 43 行的 `wg` waitgroup 用于等待所有 100 个 goroutines 完成生成随机数。

第 44 行的 `for` 循环创建了 100 个 goroutines。第 49 行的 goroutine 调用在 waitgroup 上调用 `wait()`，以等待所有 100 个 goroutines 完成创建随机数。之后，它关闭通道。一旦通道关闭并且 `consume` goroutine 已完成将所有生成的随机数写入文件，它将在第 37 行向 `done` 通道写入 `true`，主 goroutine 被解除阻塞并打印 `File written successfully`。

现在你可以在任何文本编辑器中打开文件 **concurrent**，并查看生成的 100 个随机数 :)

本教程到此结束。希望你喜欢阅读。祝你有个美好的一天。

上一个教程 - [读取文件](../【GolangBot】36-读文件)
