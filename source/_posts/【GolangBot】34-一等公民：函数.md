---
title: 【GolangBot】34-一等公民：函数
date: 2024-12-26 11:33:51
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---
欢迎来到 [Golang 教程系列](../golangbot/) 的第 34 个教程。

### 什么是一等函数？

**支持一等函数的语言允许将函数赋值给变量、作为参数传递给其他函数以及从其他函数返回。Go 支持一等函数。**

在本教程中，我们将讨论一等函数的语法和各种用例。

### 匿名函数

让我们从一个简单的示例开始，该示例将[函数](../【GolangBot】6-函数)赋值给[变量](../【GolangBot】3-变量)。

```go
package main

import (
	"fmt"
)

func main() {
	a := func() {
		fmt.Println("hello world first class function")
	}
	a()
	fmt.Printf("%T", a)
}
```

[在 Playground 中运行](https://play.golang.org/p/Xm_ihamhlEv)

在上面的程序中，我们在第 8 行将一个函数赋值给变量 `a`。这是将函数赋值给变量的语法。如果你仔细观察，赋值给 `a` 的函数没有名称。**这种函数称为匿名函数，因为它们没有名称。**

调用此函数的唯一方法是使用变量 `a`。我们在下一行中完成了这一点。`a()` 调用该函数，并打印 `hello world first class function`。在第 12 行，我们打印变量 `a` 的类型。这将打印 `func()`。

运行此程序将输出：

```fallback
hello world first class function
func()
```

也可以在不将匿名函数赋值给变量的情况下调用它。让我们在以下示例中看看如何做到这一点。

```go
package main

import (
	"fmt"
)

func main() {
	func() {
		fmt.Println("hello world first class function")
	}()
}
```

[在 Playground 中运行](https://play.golang.org/p/c0AjB3g8UEn)

在上面的程序中，第 8 行定义了一个匿名函数，紧接着在函数定义之后，我们在第 10 行使用 `()` 调用该函数。该程序将输出：

```fallback
hello world first class function
```

也可以像任何其他函数一样向匿名函数传递参数。

```go
package main

import (
	"fmt"
)

func main() {
	func(n string) {
		fmt.Println("Welcome", n)
	}("Gophers")
}
```

[在 Playground 中运行](https://play.golang.org/p/9ttJ5Wi4fj4)

在上面的程序中，第 10 行向匿名函数传递了一个字符串参数。运行此程序将打印：

```fallback
Welcome Gophers
```

### 用户定义的函数类型

就像我们定义自己的[结构体](../【GolangBot】16-结构体/#声明结构体)类型一样，也可以定义自己的函数类型。

```go
type add func(a int, b int) int
```

上面的代码片段创建了一个新的函数类型 `add`，它接受两个整数参数并返回一个整数。现在我们可以定义类型为 `add` 的变量。

让我们编写一个定义类型为 `add` 的变量的程序。

```go
package main

import (
	"fmt"
)

type add func(a int, b int) int

func main() {
	var a add = func(a int, b int) int {
		return a + b
	}
	s := a(5, 6)
	fmt.Println("Sum", s)
}
```

[在 Playground 中运行](https://play.golang.org/p/n3yPQ7hG7ip)

在上面的程序中，第 10 行我们定义了一个类型为 `add` 的变量 `a`，并为其分配了一个签名与 `add` 类型匹配的函数。我们在第 13 行调用该函数并将结果赋值给 `s`。该程序将打印：

```fallback
Sum 11
```

### 高阶函数

[维基百科](https://en.wikipedia.org/wiki/Higher-order_function)对高阶函数的定义是**至少执行以下操作之一的函数**：

- **接受一个或多个函数作为参数**
- **返回一个函数作为其结果**

让我们看一些上述两种情况的简单示例。

#### 将函数作为参数传递给其他函数

```go
package main

import (
	"fmt"
)

func simple(a func(a, b int) int) {
	fmt.Println(a(60, 7))
}

func main() {
	f := func(a, b int) int {
		return a + b
	}
	simple(f)
}
```

[在 Playground 中运行](https://play.golang.org/p/C0MNwz2TSGU)

在上面的示例中，第 7 行我们定义了一个函数 `simple`，它接受一个*接受两个整数参数并返回一个整数的函数*作为参数。在 `main` 函数中，第 12 行我们创建了一个匿名函数 `f`，其签名与 `simple` 函数的参数匹配。我们在下一行调用 `simple` 并将 `f` 作为参数传递给它。该程序输出 `67`。

#### 从其他函数返回函数

现在让我们重写上面的程序，并从 `simple` 函数返回一个函数。

```go
package main

import (
	"fmt"
)

func simple() func(a, b int) int {
	f := func(a, b int) int {
		return a + b
	}
	return f
}

func main() {
	s := simple()
	fmt.Println(s(60, 7))
}
```

[在 Playground 中运行](https://play.golang.org/p/82y2caejUy8)

在上面的程序中，第 7 行的 `simple` 函数返回一个接受两个 `int` 参数并返回一个 `int` 参数的函数。

在第 15 行调用 `simple` 函数。`simple` 函数的返回值赋值给 `s`。现在 `s` 包含了 `simple` 函数返回的函数。我们在第 16 行调用 `s` 并向其传递两个整数参数。该程序输出 `67`。

### 闭包

闭包是匿名函数的一种特殊情况。闭包是访问函数体外定义的变量的匿名函数。

一个示例将使事情更加清楚。

```go
package main

import (
	"fmt"
)

func main() {
	a := 5
	func() {
		fmt.Println("a =", a)
	}()
}
```

[在 Playground 中运行](https://play.golang.org/p/6QriMs-zbnf)

在上面的程序中，匿名函数访问了第 10 行中存在于其主体外部的变量 `a`。因此，这个匿名函数是一个闭包。

每个闭包都绑定到其自己的周围变量。让我们通过一个简单的示例来理解这意味着什么。

```go
package main

import (
	"fmt"
)

func appendStr() func(string) string {
	t := "Hello"
	c := func(b string) string {
		t = t + " " + b
		return t
	}
	return c
}

func main() {
	a := appendStr()
	b := appendStr()
	fmt.Println(a("World"))
	fmt.Println(b("Everyone"))

	fmt.Println(a("Gopher"))
	fmt.Println(b("!"))
}
```

[在 Playground 中运行](https://play.golang.org/p/134NiQGPOcS)

在上面的程序中，`appendStr` 函数返回一个闭包。这个闭包绑定到变量 `t`。让我们理解这意味着什么。

第 17 行和第 18 行声明的变量 `a` 和 `b` 是闭包，它们绑定到自己的 `t` 值。

我们首先使用参数 `World` 调用 `a`。现在 `a` 的 `t` 值变为 `Hello World`。

在第 20 行，我们使用参数 `Everyone` 调用 `b`。由于 `b` 绑定到自己的变量 `t`，`b` 的 `t` 值再次初始化为 `Hello`。因此，在此函数调用之后，`b` 的 `t` 值变为 `Hello Everyone`。程序的其余部分不言自明。

该程序将打印：

```fallback
Hello World
Hello Everyone
Hello World Gopher
Hello Everyone !
```

### 一等函数的实际用途

到目前为止，我们已经定义了什么是一等函数，并且我们已经看到了一些人为的示例来学习它们的工作原理。现在让我们编写一个具体的程序，展示一等函数的实际用途。

我们将创建一个程序，根据某些条件过滤学生的[切片](../【GolangBot】11-数组和切片)。让我们逐步解决这个问题。

首先，让我们定义 `student` 类型。

```go
type student struct {
	firstName string
	lastName  string
	grade     string
	country   string
}
```

下一步是编写 `filter` 函数。该函数接受一个学生切片和一个确定学生是否符合过滤条件的函数作为参数。一旦我们编写了这个函数，我们会更好地理解它。让我们继续编写它。

```go
func filter(s []student, f func(student) bool) []student {
	var r []student
	for _, v := range s {
		if f(v) == true {
			r = append(r, v)
		}
	}
	return r
}
```

在上面的函数中，`filter` 的第二个参数是一个函数，它接受一个 `student` 作为参数并返回一个 `bool`。该函数确定特定学生是否符合条件。我们在第 3 行遍历学生切片，并将每个学生作为参数传递给函数 `f`。如果返回 `true`，则表示该学生通过了过滤条件，并将其添加到切片 `r` 中。你可能对这个函数的实际用途感到有些困惑，但一旦我们完成程序，它就会变得清晰。我已经添加了 `main` 函数，并提供了完整的程序如下。

```go
package main

import (
	"fmt"
)

type student struct {
	firstName string
	lastName  string
	grade     string
	country   string
}

func filter(s []student, f func(student) bool) []student {
	var r []student
	for _, v := range s {
		if f(v) == true {
			r = append(r, v)
		}
	}
	return r
}

func main() {
	s1 := student{
		firstName: "Naveen",
		lastName:  "Ramanathan",
		grade:     "A",
		country:   "India",
	}
	s2 := student{
		firstName: "Samuel",
		lastName:  "Johnson",
		grade:     "B",
		country:   "USA",
	}
	s := []student{s1, s2}
	f := filter(s, func(s student) bool {
		if s.grade == "B" {
			return true
		}
		return false
	})
	fmt.Println(f)
}
```

[在 Playground 中运行](https://play.golang.org/p/YUL1CqSrvfc)

在 `main` 函数中，我们首先创建两个学生 `s1` 和 `s2`，并将它们添加到切片 `s` 中。现在假设我们想找出所有成绩为 `B` 的学生。我们在上面的程序中通过传递一个检查学生成绩是否为 `B` 的函数作为 `filter` 函数的参数来实现这一点。该程序将打印：

```fallback
[{Samuel Johnson B USA}]
```

假设我们想找出所有来自印度的学生。这可以通过更改 `filter` 函数的函数参数轻松实现。我提供了执行此操作的代码如下：

```go
c := filter(s, func(s student) bool {
    if s.country == "India" {
	    return true
    }
    return false
})
fmt.Println(c)
```

请将此代码添加到 `main` 函数中并检查输出。

让我们通过编写另一个程序来结束本节。该程序将对切片的每个元素执行相同的操作并返回结果。例如，如果我们想将切片中的所有整数乘以 5 并返回输出，可以使用一等函数轻松完成。这种对集合的每个元素进行操作的函数称为 `map` 函数。我提供了程序如下。它是不言自明的。

```go
package main

import (
	"fmt"
)

func iMap(s []int, f func(int) int) []int {
	var r []int
	for _, v := range s {
		r = append(r, f(v))
	}
	return r
}

func main() {
	a := []int{5, 6, 7, 8, 9}
	r := iMap(a, func(n int) int {
		return n * 5
	})
	fmt.Println(r)
}
```

[在 Playground 中运行](https://play.golang.org/p/cs37QwCQ_0H)

上面的程序将打印：

```fallback
[25 30 35 40 45]
```

以下是我们在本教程中学到的内容的快速回顾：

- 什么是一等函数？
- 匿名函数
- 用户定义的函数类型
- 高阶函数
  - 将函数作为参数传递给其他函数
  - 从其他函数返回函数
- 闭包
- 一等函数的实际用途


**下一个教程 - [反射](../【GolangBot】35-反射)**

