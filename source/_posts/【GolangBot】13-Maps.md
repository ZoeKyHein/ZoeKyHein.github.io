---
title: 【GolangBot】13-Maps
date: 2024-12-23 15:50:54
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---

# 【GolangBot】13-Maps

欢迎来到[Golang系列教程](../GolangBot/)的第13篇。

### 什么是map？

Go 中的 map 是一种内建的数据类型，用于存储键值对。一个实际的使用场景是存储货币代码和相应的货币名称。

```fallback
USD - United States Dollar
EUR - Euro
INR - India Rupee
```

一个 map 非常适合上面的使用场景。货币代码可以作为键，货币名称可以作为值。map 类似于其他语言（如 Python）中的字典。

### **如何创建一个 map？**

可以通过将键和值的[数据类型](../【GolangBot】4-数据类型/)传递给 `make` 函数来创建一个 map。以下是创建新 map 的语法。

```fallback
make(map[type of key]type of value)
currencyCode := make(map[string]string)
```

上面的代码创建了一个名为 currencyCode 的 map，其键和值的类型均为 string。

```go
package main

import (
	"fmt"
)

func main() {
	currencyCode := make(map[string]string)
	fmt.Println(currencyCode)
}
```

[Run in Playground](https://go.dev/play/p/A31zIgnFiW5)

上述程序创建了一个名为 currencyCode 的 map，其键和值的类型均为 string。程序的输出将是：

```fallback
map[]
```

因为我们没有把任何元素添加进map，所以它是空的。

### 向 Map 中添加条目

向 map 中添加新条目的语法与 [数组](../【GolangBot】11-数组和切片/) 的操作类似。下面的程序向 `currencyCode` map 中添加了一些新的货币代码和对应的货币名称：

```go
package main

import (
	"fmt"
)

func main() {
	currencyCode := make(map[string]string)
	currencyCode["USD"] = "US Dollar"
	currencyCode["GBP"] = "Pound Sterling"
	currencyCode["EUR"] = "Euro"
	currencyCode["INR"] = "Indian Rupee"
	fmt.Println("currencyCode map contents:", currencyCode)
}
```

[Run in playground](https://go.dev/play/p/2M4ZVP1QaQ1)

我们添加了4个货币代码，分别是 `USD`、`GBP`、`EUR` 和`INR`，以及它们对应的名称。

```fallback
currencyCode map contents: map[EUR:Euro GBP:Pound Sterling INR:Indian Rupee USD:US Dollar]
```

**正如你可能从上述输出中注意到的，从 map 中检索值的顺序并不保证与添加到 map 中的顺序相同。**

在声明时，也可以直接对 map 进行初始化。

```go
package main

import (
	"fmt"
)

func main() {
	currencyCode := map[string]string {
		"USD": "US Dollar",
		"GBP": "Pound Sterling",
		"EUR": "Euro",
	}
	currencyCode["INR"] = "Indian Rupee"
	fmt.Println("currencyCode map contents:", currencyCode)
}
```

[Run in playground](https://go.dev/play/p/Rd6QoqZ7udE)

上述程序在声明时定义了 `currencyCode` map，并初始化了 3 个键值对。随后又添加了一个键为 `INR` 的新元素。程序的输出为：

```fallback
currencyCode map contents: map[EUR:Euro GBP:Pound Sterling INR:Indian Rupee USD:US Dollar]
```

键的类型并不局限于 [string](../【GolangBot】14-Strings/)。所有可以比较的类型（如布尔值、整数、浮点数、复数、字符串等）都可以用作键。甚至用户定义的类型（如 [structs](../【GolangBot】16-结构体/)）也可以作为键。

如果您想了解更多关于可比较类型的信息，请访问 [Go 官方文档](https://go.dev/ref/spec#Comparison_operators)。

### nil map panics

映射（map）的零值是 nil。如果尝试向一个 nil 映射中添加元素，会触发运行时的 [panic](../【GolangBot】33-Panic和Recover/)。因此，在向映射中添加元素之前，必须先对映射进行初始化。

```go
package main

func main() {
	var currencyCode map[string]string
	currencyCode["USD"] = "US Dollar"
}
```

[Run in playground](https://go.dev/play/p/UuJ8pmtOp5D)

在上面的程序中，`currencyCode` 是 `nil`，我们尝试向一个 `nil` 映射中添加新键值对。程序将会发生 panic，错误信息如下：

```
panic: assignment to entry in nil map
```

### 从映射中检索键的值

现在我们已经向映射中添加了一些元素，接下来学习如何检索它们。使用 `map[key]` 语法来检索映射中的元素。

```go
package main

import (
	"fmt"
)

func main() {
	currencyCode := map[string]string{
		"USD": "US Dollar",
		"GBP": "Pound Sterling",
		"EUR": "Euro",
	}
	currency := "USD"
	currencyName := currencyCode[currency]
	fmt.Println("Currency name for currency code", currency, "is", currencyName)
}
```

[Run in playground](https://go.dev/play/p/5eLMNMwQJbY)	

上面的程序非常简单。程序检索并打印了货币代码 `USD` 对应的货币名称。程序的输出是：

```fallback
Currency name for currency code USD is US Dollar
```

如果尝试访问不存在的元素，map 将返回该元素类型的零值。在 `currencyCode` map 的例子中，如果我们尝试访问一个不存在的元素，返回的将是该类型的零值，即空字符串 ""。

```go
package main

import (
	"fmt"
)

func main() {
	currencyCode := map[string]string{
		"USD": "US Dollar",
		"GBP": "Pound Sterling",
		"EUR": "Euro",
	}
	fmt.Println("Currency name for currency code INR is", currencyCode["INR"])
}
```

[Run in playground](https://go.dev/play/p/zikbC-C_DEP)

上述程序的输出是

```fallback
Currency name for currency code INR is 
```

在上述程序中，尝试获取 `INR` 的货币名称时，返回的是空字符串。这表明，当尝试检索一个不存在的键时，**map 不会抛出运行时错误**。

### 检查key是否存在

在上一节中，我们了解到，当一个键不存在时，map 会返回该类型的零值。然而，这在我们需要确认某个键是否实际存在于 map 中时并不起作用。

例如，我们想知道某个货币代码键是否存在于 `currencyCode` map 中。可以使用以下语法来检查某个键是否存在于 map 中：

```fallback
value, ok := map[key]
```

在上述代码中，当键存在时，`ok` 的值为 `true`，对应键的值会存储在变量 `value` 中。
如果键不存在，`ok` 的值为 `false`，而 `value` 则会返回类型的零值。

```go
package main

import (
	"fmt"
)

func main() {
	currencyCode := map[string]string{
		"USD": "US Dollar",
		"GBP": "Pound Sterling",
		"EUR": "Euro",
	}
	cyCode := "INR"
	if currencyName, ok := currencyCode[cyCode]; ok {
		fmt.Println("Currency name for currency code", cyCode, " is", currencyName)
		return
	}
	fmt.Println("Currency name for currency code", cyCode, "not found")
}
```

[Run in playground](https://go.dev/play/p/Jo3AUobSf9J)

在上述程序中，第 14 行的 `ok` 将为 `false`，因为键 `INR` 不存在。因此，程序将打印：

```fallback
Currency name for currency code INR not found
```

### 遍历 map 中的所有元素

可以使用 `for` 循环的 `range` 形式来遍历 map 中的所有元素。

```go
package main

import (
	"fmt"
)

func main() {
	currencyCode := map[string]string{
		"USD": "US Dollar",
		"GBP": "Pound Sterling",
		"EUR": "Euro",
	}
	for code, name := range currencyCode {
		fmt.Printf("Currency Name for currency code %s is %s\n", code, name)
	}
}
```

[Run in playground](https://go.dev/play/p/nCbYjtQxA2m)

上述程序将会输出，

```fallback
Currency Name for currency code GBP is Pound Sterling
Currency Name for currency code EUR is Euro
Currency Name for currency code USD is US Dollar
```

*一个需要注意的重要事实是，使用 `for range` 遍历 map 时，值的检索顺序在每次程序执行时都可能不同。它也不一定与元素添加到 map 的顺序相同。*

### 从 map 中删除元素

要从 `map` 中删除某个键，可以使用以下语法：
*[delete(map, key)](https://pkg.go.dev/builtin#delete)*。

`delete` 函数不会返回任何值。

```go
package main

import (
	"fmt"
)

func main() {
	currencyCode := map[string]string{
		"USD": "US Dollar",
		"GBP": "Pound Sterling",
		"EUR": "Euro",
	}
	fmt.Println("map before deletion", currencyCode)
	delete(currencyCode, "EUR")
	fmt.Println("map after deletion", currencyCode)
}
```

[Run in playground](https://go.dev/play/p/6R00ro5T6Q0)

上述程序删除键`EUR`，它打印

```fallback
map before deletion map[EUR:Euro GBP:Pound Sterling USD:US Dollar]
map after deletion map[GBP:Pound Sterling USD:US Dollar]
```

即使我们尝试删除一个 map 中不存在的键，也不会发生运行时错误。

### 结构体的 map

到目前为止，我们仅在 map 中存储了货币名称。如果还能存储货币符号，是不是更方便呢？这可以通过使用结构体的map来实现。
货币可以用一个包含货币名称和货币符号字段的结构体表示。然后可以将该结构体作为值存储在 map 中，并以货币代码作为键。
让我们通过编写一个程序来理解如何实现这一点。

```go
package main

import (
	"fmt"
)

type currency struct {
	name   string
	symbol string
}

func main() {
	curUSD := currency{
		name:   "US Dollar",
		symbol: "$",
	}
	curGBP := currency{
		name:   "Pound Sterling",
		symbol: "£",
	}
	curEUR := currency{
		name:   "Euro",
		symbol: "€",
	}

	currencyCode := map[string]currency{
		"USD": curUSD,
		"GBP": curGBP,
		"EUR": curEUR,
	}

	for cyCode, cyInfo := range currencyCode {
		fmt.Printf("Currency Code: %s, Name: %s, Symbol: %s\n", cyCode, cyInfo.name, cyInfo.symbol)
	}

}
```

[Run in playground](https://go.dev/play/p/o6ONzGDm6-m)

在上述程序中，`currency` 结构体包含 `name` 和 `symbol` 字段。我们创建了三个货币对象：`curUSD`、`curGBP` 和 `curEUR`。

在第 26 行，我们初始化了一个以 `string` 类型为键、`currency` 类型为值的 map，其中包含我们创建的三个货币。

第 32 行使用 `for range` 遍历 map，接下来的代码打印了货币的详细信息。程序将输出如下内容：

```fallback
Currency Code: USD, Name: US Dollar, Symbol: $
Currency Code: GBP, Name: Pound Sterling, Symbol: £
Currency Code: EUR, Name: Euro, Symbol: €
```

### map的长度

可以使用 [len](https://pkg.go.dev/builtin#len) 函数来确定 map 的长度。

```go
package main

import (
	"fmt"
)

func main() {
	currencyCode := map[string]string{
		"USD": "US Dollar",
		"GBP": "Pound Sterling",
		"EUR": "Euro",
	}
	fmt.Println("length is", len(currencyCode))

}
```

[Run in playground](https://go.dev/play/p/jGWltEW5X1c)

上面的程序中，`len(currencyCode)` 返回 map 的长度。程序会输出：

```fallback
length is 3
```

### Map 是引用类型

与 [切片](../【GolangBot】11-数组和切片/)类似，map 是引用类型。当一个 map 被赋值给一个新变量时，它们都会指向相同的底层数据结构。因此，在一个变量中所做的更改会反映在另一个变量中。

```go
package main

import (
	"fmt"
)

func main() {
	employeeSalary := map[string]int{
		"steve": 12000,
		"jamie": 15000,		
		"mike": 9000,
	}
	fmt.Println("Original employee salary", employeeSalary)
	modified := employeeSalary
	modified["mike"] = 18000
	fmt.Println("Employee salary changed", employeeSalary)

}
```

[Run in playground](https://go.dev/play/p/YUhmL6d0K5__x)

在上面程序的第 14 行，`employeeSalary` 被赋值给了 `modified`。接下来一行中，`mike` 的薪水在 `modified` map 中被更改为 `18000`。由于 map 是引用类型，`employeeSalary` 中 `mike` 的薪水也会变成 `18000`。

程序会输出：

```fallback
Original employee salary map[jamie:15000 mike:9000 steve:12000]
Employee salary changed map[jamie:15000 mike:18000 steve:12000]
```

类似地，当 map 被作为参数传递给函数时，对 map 的任何修改也会影响到调用者。

### Map 的相等性

Map 不能使用 `==` 运算符进行比较。`==` 运算符只能用于检查一个 map 是否是 `nil`。

```go
package main

func main() {  
    map1 := map[string]int{
        "one": 1,
        "two": 2,
    }

    map2 := map1

    if map1 == map2 {
    }
}
```

[Run in playground](https://go.dev/play/p/mkfkORUV5pi)

The above program will fail to compile with error

上面的程序将会编译错误

```fallback
invalid operation: map1 == map2 (map can only be compared to nil)
```

检查两个 map 是否相等的一种方法是逐个比较它们的元素。另一种方法是使用 [反射](../【GolangBot】35-反射/)。我鼓励你自己写一个程序来实现这个功能并使其工作 :)。


**下一篇教程 - [Strings](../【GolangBot】14-Strings/)**
