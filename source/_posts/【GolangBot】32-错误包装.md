---
title: 【GolangBot】32-错误包装
date: 2024-12-26 11:20:01
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---


欢迎来到 [Golang 教程系列](../golangbot/) 的第 32 个教程。

在本教程中，我们将学习 Go 中的错误包装以及为什么我们需要错误包装。让我们开始吧。

### 什么是错误包装？

错误包装是将一个错误封装到另一个错误中的过程。假设我们有一个访问数据库并尝试从数据库中获取记录的 Web 服务器。如果数据库调用返回错误，我们可以决定是否包装此错误或从 Web 服务发送我们自己的自定义错误。让我们编写一个小程序来理解这一点。

```go
package main

import (
	"errors"
	"fmt"
)

var noRows = errors.New("no rows found")

func getRecord() error {
	return noRows
}

func webService() error {
	if err := getRecord(); err != nil {
		return fmt.Errorf("Error %s when calling DB", err)
	}
	return nil
}

func main() {
	if err := webService(); err != nil {
		fmt.Printf("Error: %s when calling webservice\n", err)
		return
	}
	fmt.Println("webservice call successful")
}
```

[在 Playground 中运行](https://go.dev/play/p/0kVGzdt47GW)

在上面的程序中，在第 16 行，我们发送了调用 `getRecord` 函数时发生的错误的字符串描述。虽然这看起来像是错误包装，但实际上并不是 :)。让我们在下一节中了解如何包装错误。

### 错误包装和 Is 函数

[errors](https://pkg.go.dev/errors) 包中的 [Is](https://pkg.go.dev/errors#Is) 函数报告错误链中的任何错误是否与目标匹配。在我们的例子中，我们在第 11 行从 `getRecord` 函数返回 `noRows` 错误。在第 16 行从 `webService` 函数返回此错误的字符串格式。让我们修改此程序的 `main` 函数，并使用 `Is` 函数检查错误链中的任何错误是否与 `noRows` 错误匹配。

```go
func main() {
	if err := webService(); err != nil {
		if errors.Is(err, noRows) {
			fmt.Printf("The searched record cannot be found. Error returned from DB is %s", err)
			return
		}
		fmt.Println("unknown error when searching record")
		return
	}
	fmt.Println("webservice call successful")
}
```

在上面的 `main` 函数中，在第 3 行，`Is` 函数将检查 `err` 持有的错误链中的任何错误是否包含 `noRows` 错误。当前状态的代码将不起作用，并且上述 `main` 函数第 3 行的 `if` 条件将失败。为了使其工作，我们需要在从 `webService` 函数返回时包装 `noRows` 错误。一种方法是在返回错误时使用 `%w` 格式说明符而不是 `%s`。因此，如果我们将返回错误的行修改为：

```go
return fmt.Errorf("Error %w when calling DB", err)
```

这意味着新返回的错误包装了原始的 `noRows`，并且上述 `main` 函数第 3 行的 `if` 条件将成功。带有错误包装的完整程序如下。

```go
package main

import (
	"errors"
	"fmt"
)

var noRows = errors.New("no rows found")

func getRecord() error {
	return noRows
}

func webService() error {
	if err := getRecord(); err != nil {
		return fmt.Errorf("Error %w when calling DB", err)
	}
	return nil
}

func main() {
	if err := webService(); err != nil {
		if errors.Is(err, noRows) {
			fmt.Printf("The searched record cannot be found. Error returned from DB is %s", err)
			return
		}
		fmt.Println("unknown error when searching record")
		return
	}
	fmt.Println("webservice call successful")
}
```

[在 Playground 中运行](https://go.dev/play/p/t0h3WtJ5fu5)

运行此程序时，它将打印：

```fallback
The searched record cannot be found. Error returned from DB is Error no rows found when calling DB
```

### As 函数

[errors](https://pkg.go.dev/errors) 包中的 [As](https://pkg.go.dev/errors#As) 函数将尝试将作为输入传递的错误转换为目标错误类型。如果错误链中的任何错误与目标匹配，它将成功。如果成功，它将返回 true，并将目标设置为错误链中与目标匹配的第一个错误。一个程序将使事情更容易理解 :)

```go
package main

import (
	"errors"
	"fmt"
)

type DBError struct {
	desc string
}

func (dbError DBError) Error() string {
	return dbError.desc
}

func getRecord() error {
	return DBError{
		desc: "no rows found",
	}
}

func webService() error {
	if err := getRecord(); err != nil {
		return fmt.Errorf("Error %w when calling DB", err)
	}
	return nil
}

func main() {
	if err := webService(); err != nil {
		var dbError DBError
		if errors.As(err, &dbError) {
			fmt.Printf("The searched record cannot be found. Error returned from DB is %s", dbError)
			return
		}
		fmt.Println("unknown error when searching record")
		return
	}
	fmt.Println("webservice call successful")
}
```

[在 Playground 中运行](https://go.dev/play/p/I268pAa4NyR)

在上面的程序中，我们修改了第 16 行的 `getRecord` 函数，以返回类型为 `DBError` 的[自定义错误](../【GolangBot】31-自定义错误)。

在 `main` 函数的第 32 行，我们尝试将从 `webService()` 函数调用返回的错误转换为 `DBError` 类型。第 32 行的 `if` 语句将成功，因为我们在第 24 行从 `webService()` 函数返回错误时包装了 `DBError`。运行此程序将打印：

```fallback
The searched record cannot be found. Error returned from DB is no rows found
```

### 我们应该包装错误吗？

这个问题的答案是，视情况而定。如果我们包装错误，我们将其暴露给我们的库/函数的调用者。我们通常不希望包装包含函数内部实现细节的错误。另一个需要记住的重要事情是，如果我们返回一个包装的错误，然后决定删除错误包装，使用我们库的代码将开始失败。因此，包装的错误应被视为 API 的一部分，如果我们决定修改返回的错误，则应进行适当的版本更改。

**下一个教程 - [Panic 和 Recover](../【GolangBot】33-Panic和Recover)**