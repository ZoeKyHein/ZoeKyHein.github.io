---
title: 【GolangBot】12-可变参数函数
date: 2024-12-17 11:50:32
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---

欢迎来到[Golang系列教程](../GolangBot/)的第12篇。

### 什么是变参函数？

通常，[函数](../【GolangBot】6-函数)只接受固定数量的参数。*变参函数是接受可变数量参数的函数。如果函数定义的最后一个参数前面有省略号 **...**，则该函数可以接受任意数量的参数。*

**只有函数的最后一个参数可以是变参。我们将在本教程的下一部分了解为什么会这样。**

### 语法

### Syntax

```fallback
func hello(a int, b ...int) {
}
```

在上述函数中，参数 `b` 是变参，因为它前面加了省略号，它可以接受任意数量的参数。这个函数可以通过以下语法调用：

```fallback
hello(1, 2) //passing one argument "2" to b
hello(5, 6, 7, 8, 9) //passing arguments "6, 7, 8 and 9" to b
```

在上述代码中，第 1 行我们传递了一个参数 `2` 给 `b`，在下一行，我们将四个参数 `6, 7, 8, 9` 传递给 `b`。

也可以向变参函数传递零个参数。

```fallback
hello(1)
```

在上述代码中，我们向 `hello` 传递了零个参数给 `b`，这是完全允许的。

到现在为止，你应该已经明白为什么变参参数只能放在最后了。

让我们尝试将 `hello` 函数的第一个参数改为变参。

语法将如下所示：

```fallback
func hello(b ...int, a int) {
}
```

在上述函数中，无法为参数 `a` 传递参数，因为无论我们传递什么参数，它都会分配给第一个参数 `b`，因为它是变参。因此，变参参数只能在函数定义的最后出现。上述函数会因错误 `syntax error: cannot use ... with non-final parameter b` 编译失败。

### 示例和理解变参函数如何工作

让我们创建一个自己的变参函数。我们将编写一个简单的程序来判断一个整数是否存在于输入的整数列表中。

```go
package main

import (
	"fmt"
)

func find(num int, nums ...int) {
	fmt.Printf("type of nums is %T\n", nums)
	found := false
	for i, v := range nums {
		if v == num {
			fmt.Println(num, "found at index", i, "in", nums)
			found = true
		}
	}
	if !found {
		fmt.Println(num, "not found in ", nums)
	}
	fmt.Printf("\n")
}
func main() {
	find(89, 89, 90, 95)
	find(45, 56, 67, 45, 90, 109)
	find(78, 38, 56, 98)
    find(87)
}
```


[Run in playground](https://play.golang.org/p/7occymiS6s)

在上述程序中，第 7 行的 `func find(num int, nums ...int)` 接受可变数量的参数给 `nums` 参数。在 `find` 函数内部，`nums` 的类型是 `[]int`，即整数切片。

**变参函数的工作方式是将可变数量的参数转换为变参类型的切片。例如，在上述程序的第 22 行，传递给 `find` 函数的可变数量的参数是 89、90、95。`find` 函数期望一个变参 `int` 类型的参数。因此，编译器会将这三个参数转换为类型为 `int` 的切片 `[]int{89, 90, 95}`，然后传递给 `find` 函数。**

在第 10 行，`for` 循环遍历 `nums` 切片，并打印 `num` 的位置，如果它存在于切片中。如果不存在，则打印该数字未找到。

上述程序的输出是：

```fallback
type of nums is []int
89 found at index 0 in [89 90 95]

type of nums is []int
45 found at index 2 in [56 67 45 90 109]

type of nums is []int
78 not found in  [38 56 98]

type of nums is []int
87 not found in  []
```

在上述程序的第 25 行，`find` 函数调用只传递了一个参数。我们没有向变参 `nums ...int` 传递任何参数。如前所述，这完全合法，在这种情况下，`nums` 将是一个 `nil` 切片，长度和容量都为 0。

### 切片参数与变参参数

现在你一定会有一个问题。在上一节中，我们学到变参传递给函数时，实际上会被转换成切片。那么，当我们可以使用切片实现相同的功能时，为什么还需要变参函数呢？我已经用切片重写了上面的程序。

```go
package main

import (  
    "fmt"
)

func find(num int, nums []int) {  
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {  
    find(89, []int{89, 90, 95})
    find(45, []int{56, 67, 45, 90, 109})
    find(78, []int{38, 56, 98})
    find(87, []int{})
}
```


[Run in playground](https://play.golang.org/p/rG-XRL3yycJ)

以下是使用变参参数而非切片的优点：

1. 每次函数调用时无需创建切片。如果你查看上面的程序，我们在第 22、23、24 和 25 行每次调用时都创建了新的切片。使用变参函数时可以避免这种额外的切片创建。
2. 在上述程序的第 25 行，我们仅仅为了满足 `find` 函数的签名创建了一个空切片。这在使用变参函数时完全不需要。使用变参函数时，这行代码可以简化为 `find(87)`。
3. 我个人觉得使用变参函数的程序比使用切片的程序更具可读性 :)

### `append` 是一个变参函数

你是否曾经想过，标准库中的 [append](https://golang.org/pkg/builtin/#append) 函数是如何接受任意数量的参数来追加值到 [切片](https://golangbot.com/arrays-and-slices/) 的？这是因为它是一个变参函数。

```fallback
func append(slice []Type, elems ...Type) []Type
```

上面是 `append` 函数的定义。在这个定义中，`elems` 是一个变参参数。因此，`append` 可以接受任意数量的参数。

### 将切片传递给变参函数

让我们通过以下示例来传递一个切片给变参函数，并查看会发生什么。

```go
package main

import (  
    "fmt"
)

func find(num int, nums ...int) {  
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {  
    nums := []int{89, 90, 95}
    find(89, nums)
}
```


[Run in playground](https://play.golang.org/p/A-DNilpH2L)

在第 23 行，我们将一个切片传递给一个期望变参参数的函数。

这将不起作用。上述程序会因编译错误 `./prog.go:23:10: cannot use nums (type []int) as type int in argument to find` 而失败。

为什么这样不行呢？其实非常简单。`find` 函数的签名如下所示：

```fallback
func find(num int, nums ...int)
```

根据变参函数的定义，`nums ...int` 意味着它将接受可变数量的 `int` 类型的参数。

在上面的程序的第 23 行，`nums` 是一个 `[]int` 切片，它被传递给 `find` 函数，而 `find` 函数期望的是变参的 `int` 类型参数。如我们之前讨论的，变参将被转换为 `int` 类型的切片，因为 `find` 期望的是变参的 `int` 类型参数。在这种情况下，`nums` 已经是一个 `[]int` 切片，编译器会尝试创建一个新的 `[]int`，即编译器尝试执行以下操作：

```fallback
find(89, []int{nums})
```

这会失败，因为 `nums` 是一个 `[]int`，而不是一个 `int`。

那么，有没有办法将切片传递给变参函数呢？答案是有的。

**有一种语法糖可以用来将切片传递给变参函数。你需要在切片后面加上省略号 `...`。这样做时，切片会直接传递给函数，而不会创建一个新的切片。**

在上述程序中，如果你将第 23 行的 `find(89, nums)` 替换为 `find(89, nums...)`，程序将会编译并打印以下输出。

```fallback
type of nums is []int
89 found at index 0 in [89 90 95]
```

这里是完整的程序供你参考。

```go
package main

import (  
    "fmt"
)

func find(num int, nums ...int) {  
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {  
    nums := []int{89, 90, 95}
    find(89, nums...)
}
```


[Run in playground](https://play.golang.org/p/IvzwhzhFsT)

### 注意事项

在变参函数内部修改切片时，一定要确保你知道自己在做什么。

让我们看一个简单的例子。

```go
package main

import (
	"fmt"
)

func change(s ...string) {
	s[0] = "Go"
}

func main() {
	welcome := []string{"hello", "world"}
	change(welcome...)
	fmt.Println(welcome)
}
```


[Run in playground](https://play.golang.org/p/R0GsuW7rdd)

你认为上述程序的输出是什么？如果你认为输出是 `[Go world]`，恭喜你！你已经理解了变参函数和切片的概念。如果你没有猜对，没关系，让我来解释一下我们是如何得到这个输出的。

在程序的第 13 行，我们使用了语法糖 `...`，将切片作为变参传递给 `change` 函数。

如我们之前讨论的，如果使用了 `...`，切片本身会作为参数传递，而不会创建一个新的切片。因此，`welcome` 会作为参数传递给 `change` 函数。

在 `change` 函数内部，切片的第一个元素被修改为 `Go`。因此，程序的输出是：

```fallback
[Go world]
```

这里有一个程序，帮助你更好地理解变参函数。

```go
package main

import (
	"fmt"
)

func change(s ...string) {
	s[0] = "Go"
	s = append(s, "playground")
	fmt.Println(s)
}

func main() {
	welcome := []string{"hello", "world"}
	change(welcome...)
	fmt.Println(welcome)
}
```


[Run in playground](https://play.golang.org/p/WdbFIkdLoe)

我把上面的程序作为一个练习留给你，让你自己弄明白它是如何工作的 :）。

**下一篇教程 - [Maps](../【GolangBot】13-Maps)**
