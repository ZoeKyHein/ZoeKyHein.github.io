---
title: 【GolangBot】30-错误处理
date: 2024-12-26 11:00:32
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---
# 【GolangBot】30-错误处理
欢迎来到 [Golang 教程系列](../golangbot/) 的第 30 个教程。

### 什么是错误？

错误表示程序中发生的任何异常情况。假设我们尝试打开一个文件，而该文件在文件系统中不存在。这是一种异常情况，它被表示为一个错误。

在 Go 中，错误是普通的值。就像任何其他内置[类型](../【GolangBot】4-数据类型)（如 `int`、`float64` 等）一样，错误值可以存储在[变量](../【GolangBot】3-变量)中，作为参数传递给函数，从[函数](../【GolangBot】6-函数)中返回，等等。

错误使用内置的 `error` 类型表示。我们将在本教程的后面部分了解更多关于 `error` 类型的内容。

### 示例

让我们从一个尝试打开不存在的文件的示例程序开始。

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.Open("/test.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(f.Name(), "opened successfully")
}
```

[在 Playground 中运行](https://go.dev/play/p/yOhAviFM05)

在上面的程序的第 9 行，我们尝试打开路径为 `/test.txt` 的文件（显然在 Playground 中不存在）。`os` 包的 *[Open](https://pkg.go.dev/os#Open)* 函数的签名如下：

**func Open(name string) (\*File, error)**

如果文件成功打开，`Open` 函数将返回文件句柄，并且错误为 `nil`。如果在打开文件时发生错误，将返回一个非 `nil` 的错误。

如果一个[函数](../【GolangBot】6-函数)或[方法](../【GolangBot】17-方法)返回一个错误，按照惯例，它应该是函数返回的最后一个值。因此，`Open` 函数将 `error` 作为最后一个值返回。

**在 Go 中处理错误的惯用方法是将返回的错误与 `nil` 进行比较。`nil` 值表示没有发生错误，非 `nil` 值表示存在错误。** 在我们的例子中，我们在第 10 行检查错误是否为 `nil`。如果不是 `nil`，我们只需打印错误并从主函数返回。

运行该程序将输出：

```fallback
open /test.txt: No such file or directory
```

完美 😃。我们得到了一个错误，指出文件不存在。

### 错误类型的表示

让我们深入一点，看看内置的 `error` 类型是如何定义的。`error` 是一个[接口](../【GolangBot】18-接口-I)类型，其定义如下：

```go
type error interface {
    Error() string
}
```

它包含一个签名 `Error() string` 的方法。任何实现此接口的类型都可以用作错误。此方法提供了错误的描述。

当打印错误时，`fmt.Println` 函数内部调用 `Error() string` 方法来获取错误的描述。这就是上面的[示例程序](../【GolangBot】30-错误处理/#示例)中第 11 行打印错误描述的方式。

获取免费的 Golang 工具备忘单

### 从错误中提取更多信息的不同方式

既然我们知道 `error` 是一个接口类型，让我们看看如何从错误中提取更多信息。

在上面看到的[示例](../【GolangBot】30-错误处理/#示例)中，我们只是打印了错误的描述。如果我们想要导致错误的文件的实际路径怎么办？一种可能的方法是解析错误字符串。这是我们程序的输出：

```fallback
open /test.txt: No such file or directory
```

*我们可以解析此错误消息并获取导致错误的文件路径“/test.txt”，但这是一个不太优雅的方式。错误描述可能会在 Go 的新版本中随时更改，我们的代码将会崩溃。*

有没有更好的方法来获取文件名 🤔？答案是肯定的，可以通过 Go 标准库提供的不同方式来实现。让我们逐一看看。

#### 1. 将错误转换为底层类型并从结构体字段中检索更多信息

如果你仔细阅读 [Open](https://pkg.go.dev/os#Open) 函数的文档，你会发现它返回一个 `*PathError` 类型的错误。[PathError](https://pkg.go.dev/os#PathError) 是一个[结构体](../【GolangBot】16-结构体)类型，其标准库中的实现如下：

```go
type PathError struct {
    Op   string
    Path string
    Err  error
}
  
func (e *PathError) Error() string { return e.Op + " " + e.Path + ": " + e.Err.Error() }
```

如果你有兴趣了解上述源代码的位置，可以在这里找到：https://cs.opensource.google/go/go/+/refs/tags/go1.19:src/io/fs/fs.go;l=250

从上面的代码中，你可以理解 `*PathError` 通过声明 `Error() string` 方法实现了 `error` 接口。此方法将操作、路径和实际错误连接起来并返回。因此我们得到了错误消息：

```fallback
open /test.txt: No such file or directory
```

`PathError` 结构体的 `Path` 字段包含导致错误的文件路径。

我们可以使用 `errors` 包中的 [As](https://pkg.go.dev/errors#As) 函数将错误转换为其底层类型。`As` 函数的描述提到了错误链。现在请忽略它。我们将在[单独的教程](../【GolangBot】32-错误包装)中了解错误链和包装的工作原理。`As` 的简单描述是，它尝试将错误转换为错误类型，并返回 `true` 或 `false`，指示转换是否成功。

一个程序会让事情更清楚。让我们修改上面编写的程序，并使用 `As` 函数打印路径。

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

func main() {
	f, err := os.Open("test.txt")
	if err != nil {
		var pErr *os.PathError
		if errors.As(err, &pErr) {
			fmt.Println("Failed to open file at path", pErr.Path)
			return
		}
		fmt.Println("Generic error", err)
		return
	}
	fmt.Println(f.Name(), "opened successfully")
}
```

[在 Playground 中运行](https://go.dev/play/p/znhBV-QC7Nk)

在上面的程序中，我们首先在第 11 行检查错误是否为 `nil`，然后在第 13 行使用 `As` 函数将 `err` 转换为 `*os.PathError`。如果转换成功，`As` 将返回 `true`。然后我们在第 14 行使用 `pErr.Path` 打印路径。

如果你想知道为什么 `pErr` 是一个指针，原因是 `error` 接口由 `PathError` 的指针实现，因此 `pErr` 是一个指针。以下代码显示 `*PathError` 实现了 `error` 接口。

```go
func (e *PathError) Error() string { return e.Op + " " + e.Path + ": " + e.Err.Error() }
```

`As` 函数要求第二个参数是指向实现 `error` 的类型的指针。因此我们传递 `&pErr`。

该程序输出：

```fallback
Failed to open file at path test.txt
```

如果底层错误不是 `*os.PathError` 类型，控制将到达第 17 行，并打印一个通用错误消息。

太棒了 😃。我们成功地使用 `As` 函数从错误中获取了文件路径。

#### 2. 使用方法检索更多信息

从错误中获取更多信息的第二种方法是找出底层类型，并通过调用[结构体](../【GolangBot】16-结构体)类型的[方法](../【GolangBot】17-方法)来获取更多信息。

让我们通过一个例子更好地理解这一点。

标准库中的 *[DNSError](https://golang.org/pkg/net/#DNSError)* 结构体类型定义如下：

```go
type DNSError struct {
    ...
}
  
func (e *DNSError) Error() string {
    ...
}
func (e *DNSError) Timeout() bool { 
    ... 
}
func (e *DNSError) Temporary() bool { 
    ... 
}
```

`DNSError` 结构体有两个方法 `Timeout() bool` 和 `Temporary() bool`，它们返回一个布尔值，指示错误是否是由于超时或是否是临时错误。

让我们编写一个程序，将错误转换为 `*DNSError` 类型，并调用上述方法来确定错误是临时的还是由于超时。

```go
package main

import (
	"errors"
	"fmt"
	"net"
)

func main() {
	addr, err := net.LookupHost("golangbot123.com")
	if err != nil {
		var dnsErr *net.DNSError
		if errors.As(err, &dnsErr) {
			if dnsErr.Timeout() {
				fmt.Println("operation timed out")
				return
			}
			if dnsErr.Temporary() {
				fmt.Println("temporary error")
				return
			}
			fmt.Println("Generic DNS error", err)
			return
		}
		fmt.Println("Generic error", err)
		return
	}
	fmt.Println(addr)
}
```

*注意：DNS 查找在 Playground 中不起作用。请在本地机器上运行此程序。*

在上面的程序中，我们在第 9 行尝试获取无效域名 `golangbot123.com` 的 IP 地址。在第 13 行，我们使用 `As` 函数获取错误的底层值，并将其转换为 `*net.DNSError`。然后我们在第 14 行和第 18 行分别检查错误是否是由于超时或临时错误。

在我们的例子中，错误既不是*临时*的，也不是由于*超时*，因此程序将打印：

```fallback
Generic DNS error lookup golangbot123.com: no such host
```

如果错误是临时的或由于超时，则相应的 `if` 语句将执行，我们可以适当地处理它。

#### 3. 直接比较

从错误中获取更多详细信息的第三种方法是直接与 `error` 类型的变量进行比较。让我们通过一个例子来理解这一点。

`filepath` 包的 [Glob](https://golang.org/pkg/path/filepath/#Glob) 函数用于返回与模式匹配的所有文件的名称。当模式格式错误时，此函数返回错误 `ErrBadPattern`。

*ErrBadPattern* 在 `filepath` 包中定义为一个全局变量。

```go
var ErrBadPattern = errors.New("syntax error in pattern")
```

`errors.New()` 用于创建一个新的错误。我们将在[下一个教程](../【GolangBot】32-错误包装)中详细讨论这一点。

当模式格式错误时，`Glob` 函数返回 *ErrBadPattern*。

让我们编写一个小程序来检查此错误。

```go
package main

import (
	"errors"
	"fmt"
	"path/filepath"
)

func main() {
	files, err := filepath.Glob("[")
	if err != nil {
		if errors.Is(err, filepath.ErrBadPattern) {
			fmt.Println("Bad pattern error:", err)
			return
		}
		fmt.Println("Generic error:", err)
		return
	}
	fmt.Println("matched files", files)
}
```

[在 Playground 中运行](https://go.dev/play/p/_JLFsjIPBuw)

在上面的程序中，我们搜索模式为 `[` 的文件，这是一个格式错误的模式。我们检查错误是否为 `nil`。为了获取有关错误的更多信息，我们使用 [Is](https://pkg.go.dev/errors#Is) 函数在第 11 行直接将其与 `filepath.ErrBadPattern` 进行比较。与 `As` 类似，`Is` 函数在错误链上工作。我们将在[下一个教程](https://golangbot.com/custom-errors/)中了解更多信息。在本教程中，可以将 `Is` 函数视为在两个错误相同时返回 `true`。

由于错误是由于格式错误的模式引起的，`Is` 在第 12 行返回 `true`。该程序将打印：

```fallback
Bad pattern error: syntax error in pattern
```

标准库使用上述任何一种方式来提供有关错误的更多信息。我们将在下一个教程中使用这些方式来创建我们自己的[自定义错误](https://golangbot.com/custom-errors/)。

获取免费的 Golang 工具备忘单

### 不要忽略错误

永远不要忽略错误。忽略错误是在自找麻烦。让我重写[示例](https://go.dev/play/p/_JLFsjIPBuw)，该示例列出了与模式匹配的所有文件的名称，同时忽略错误。

```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	files, _ := filepath.Glob("[")
	fmt.Println("matched files", files)
}
```

[在 Playground 中运行](https://play.golang.org/p/2k8r_Qg_lc)

我们从[上一个示例](https://play.golang.org/p/KqhxmlLVFT6)中知道模式是无效的。我在第 9 行使用 `_` 空白标识符忽略了 `Glob` 函数返回的错误。我只是在第 10 行打印匹配的文件。该程序将打印：

```fallback
matched files []
```

由于我们忽略了错误，输出看起来好像没有文件匹配模式，但实际上模式本身是格式错误的。所以永远不要忽略错误。

本教程到此结束。

在本教程中，我们讨论了如何处理程序中发生的错误，以及如何检查错误以从中获取更多信息。以下是我们在本教程中讨论的内容的快速回顾：

- 什么是错误？
- 错误的表示
- 从错误中提取更多信息的不同方式
- 不要忽略错误

在[下一个教程](../【GolangBot】31-自定义错误)中，我们将创建自己的自定义错误，并为我们的自定义错误添加上下文。

**下一个教程 - [自定义错误](../【GolangBot】31-自定义错误)**
