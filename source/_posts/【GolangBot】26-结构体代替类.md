---
title: 【GolangBot】26-结构体代替类
date: 2024-12-26 09:16:42
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---

欢迎来到 [Golang 教程系列](../golangbot/) 的第 26 个教程。

### Go 是面向对象的吗？

Go 不是纯粹的面向对象编程语言。以下摘录自 Go 的 [FAQ](https://go.dev/doc/faq#Is_Go_an_object-oriented_language)，回答了 Go 是否是面向对象的问题。

```fallback
Yes and no. Although Go has types and methods and allows an object-oriented style of programming, 
there is no type hierarchy. The concept of “interface” in Go provides a different approach that we 
believe is easy to use and in some ways more general. There are also ways to embed types in other 
types to provide something analogous—but not identical—to subclassing. Moreover, methods in Go are 
more general than in C++ or Java: they can be defined for any sort of data, even built-in types 
such as plain, "unboxed" integers. They are not restricted to structs (classes).

是，也不是。尽管 Go 具有类型和方法，并支持面向对象的编程风格，但它没有类型层次结构。Go 中的“接口”概念提供了一种不同的方式，我们认为这种方式易于使用，并且在某些方面更加通用。Go 还可以通过将一种类型嵌入到另一种类型中，提供类似（但不完全相同于）子类化的功能。此外，Go 中的方法比 C++ 或 Java 更加通用：它们可以为任何类型的数据定义，甚至包括内置类型，例如普通的“非装箱”整数。它们并不限于 struct（类）。
```

在接下来的教程中，我们将讨论如何使用 Go 实现面向对象编程的概念。其中一些实现与其他面向对象语言（如 Java）相比有很大的不同。

### 使用结构体代替类

Go 不提供类，但它提供 [结构体](../【GolangBot】16-结构体)。可以在结构体上添加 [方法](../【GolangBot】17-方法)。这提供了将数据和方法捆绑在一起的行为，类似于类。

为了更好地理解，让我们直接从一个例子开始。

在这个例子中，我们将创建一个自定义的 [包](../GolangBot】7-包)，因为它有助于更好地理解结构体如何有效地替代类。

在 `~/Documents/` 中创建一个子文件夹，并将其命名为 `oop`。

让我们初始化一个名为 `oop` 的 Go 模块。在 `oop` 目录中键入以下命令以创建一个名为 `oop` 的 go mod。

```fallback
go mod init oop
```

在 `oop` 中创建一个子文件夹 `employee`。在 `employee` 文件夹中，创建一个名为 `employee.go` 的文件。

文件夹结构如下：

```fallback
├── Documents
│   └── oop
│       ├── employee
│       │   └── employee.go
│       └── go.mod
```

请将 `employee.go` 的内容替换为以下内容：

```go
package employee

import (
	"fmt"
)

type Employee struct {  
    FirstName   string
    LastName    string
    TotalLeaves int
    LeavesTaken int
}

func (e Employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining\n", e.FirstName, e.LastName, (e.TotalLeaves - e.LeavesTaken))
}
```

在上面的程序中，第 1 行指定此文件属于 `employee` 包。第 7 行声明了一个 `Employee` 结构体。第 14 行向 `Employee` 结构体添加了一个名为 `LeavesRemaining` 的方法。该方法计算并显示员工剩余的假期天数。现在我们有了一个结构体和一个操作结构体的方法，类似于类。

在 `oop` 文件夹中创建一个名为 `main.go` 的文件。

现在文件夹结构如下：

```fallback
├── Documents
│   └── oop
│       ├── employee
│       │   └── employee.go
│       ├── go.mod
│       └── main.go
```

`main.go` 的内容如下：

```go
package main

import "oop/employee"

func main() {
	e := employee.Employee {
		FirstName: "Sam",
		LastName: "Adolf",
		TotalLeaves: 30,
		LeavesTaken: 20,
	}
	e.LeavesRemaining()
}
```

我们在第 3 行导入了 `employee` 包。第 12 行在 `main()` 中调用了 `Employee` 结构体的 `LeavesRemaining()` 方法。

由于此程序包含自定义包，因此无法在 playground 上运行。如果您在本地运行此程序，通过执行 `go install` 命令，然后执行 `oop`，程序将输出：

```fallback
Sam Adolf has 10 leaves remaining
```

如果您不确定如何运行此程序，请访问 [/hello-world-gomod/](../【GolangBot】2-Hello-World) 了解更多信息。

### 使用 New() 函数代替构造函数

我们上面编写的程序看起来不错，但其中存在一个微妙的问题。让我们看看当我们用零值定义 employee 结构体时会发生什么。将 `main.go` 的内容替换为以下代码：

```go
package main

import "oop/employee"

func main() {
	var e employee.Employee
	e.LeavesRemaining()
}
```

我们唯一做的更改是在第 6 行创建了一个零值的 `Employee`。此程序将输出：

```fallback
  has 0 leaves remaining
```

如您所见，使用 `Employee` 的零值创建的变量是不可用的。它没有有效的名字、姓氏，也没有有效的假期详细信息。

在其他 OOP 语言（如 Java）中，可以通过使用构造函数来解决此问题。可以通过使用参数化构造函数来创建有效的对象。

Go 不支持构造函数。如果类型的零值不可用，程序员的工作是取消导出该类型以防止从其他包访问，并提供一个名为 `NewT(parameters)` 的 [函数](../【GolangBot】6-函数)，该函数使用所需的值初始化类型 `T`。在 Go 中，命名一个创建类型 `T` 的值的函数为 `NewT(parameters)` 是一种约定。这将充当构造函数。如果包只定义了一个类型，那么在 Go 中约定将此函数命名为 `New(parameters)` 而不是 `NewT(parameters)`。

让我们对我们编写的程序进行更改，以便每次创建 employee 时，它都是可用的。

第一步是取消导出 `Employee` 结构体并创建一个 `New()` 函数，该函数将创建一个新的 `Employee`。将 `employee.go` 中的代码替换为以下内容：

```go
package employee

import (  
    "fmt"
)

type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {
	e := employee {firstName, lastName, totalLeave, leavesTaken}
	return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining\n", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

我们在这里做了一些重要的更改。我们将第 7 行的 `Employee` 结构体的首字母 `e` 改为小写，即将 `type Employee struct` 更改为 `type employee struct`。通过这样做，我们成功地取消了 `employee` 结构体的导出，并防止了其他包的访问。除非有特定的需要导出字段，否则最好将所有未导出结构体的字段也取消导出。由于我们不需要在 `employee` 包之外的任何地方访问 `employee` 结构体的字段，因此我们也取消了所有字段的导出。

我们在 `LeavesRemaining()` 方法中相应地更改了字段名称。

现在，由于 `employee` 是未导出的，因此无法从其他包创建类型为 `Employee` 的值。因此，我们在第 14 行提供了一个导出的 `New` 函数，该函数接受所需的参数作为输入并返回一个新创建的 employee。

此程序仍然需要进行更改才能使其工作，但让我们运行此程序以了解到目前为止更改的效果。如果运行此程序，它将失败并出现以下编译错误：

```fallback
# oop
./main.go:6:8: undefined: employee.Employee
```

这是因为我们在 `employee` 包中有一个未导出的 `employee`，并且无法从 `main` 包访问它。因此，编译器会抛出错误，指出此类型在 `main.go` 中未定义。完美。这正是我们想要的。现在，其他包将无法创建零值的 `employee`。我们成功地防止了创建不可用的 employee 结构体值。现在创建 employee 的唯一方法是使用 `New` 函数。

将 `main.go` 的内容替换为以下内容：

```go
package main  

import "oop/employee"

func main() {  
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

此文件的唯一更改是在第 6 行。我们通过将所需的参数传递给 `New` 函数来创建一个新的 employee。

以下是进行所需更改后两个文件的内容。

employee.go

```go
package employee

import (  
    "fmt"
)

type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {  
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining\n", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

main.go

```go
package main  

import "oop/employee"

func main() {  
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

运行此程序将输出：

```fallback
Sam Adolf has 10 leaves remaining
```

因此，您可以理解，尽管 Go 不支持类，但结构体可以有效地替代类，并且签名 `New(parameters)` 的方法可以替代构造函数。

这就是 Go 中的类和构造函数。

**下一个教程 - [组合代替继承](../【GolangBot】27-组合而不是继承.md)**