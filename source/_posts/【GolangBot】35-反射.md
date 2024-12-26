---
title: 【GolangBot】35-反射
date: 2024-12-26 12:00:58
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---
欢迎来到 [Golang 教程系列](../golangbot/) 的第 35 个教程。

反射是 Go 中的高级主题之一。我将尽量使其简单易懂。

本教程包含以下部分：

- 什么是反射？
- 为什么需要检查变量并找到其类型？
- reflect 包
  - reflect.Type 和 reflect.Value
  - reflect.Kind
  - NumField() 和 Field() 方法
  - Int() 和 String() 方法
- 完整程序
- 是否应该使用反射？

让我们逐一讨论这些部分。

### 什么是反射？

反射是程序在运行时检查其变量和值并找到其类型的能力。你可能还不理解这意味着什么，但没关系。通过本教程的学习，你会对反射有一个清晰的理解，所以请继续阅读。

### 为什么需要检查变量并找到其类型？

学习反射时，第一个问题通常是：为什么我们需要在运行时检查[变量](../【GolangBot】3-变量)并找到其类型，而程序中的每个变量都是由我们定义的，我们在编译时就已经知道其类型了。嗯，大多数情况下确实如此，但并非总是如此。

让我解释一下我的意思。让我们编写一个简单的程序。

```go
package main

import (
	"fmt"
)

func main() {
	i := 10
	fmt.Printf("%d %T", i, i)
}
```

[在 Playground 中运行](https://play.golang.org/p/1oZzPCCG2Qw)

在上面的程序中，`i` 的类型在编译时是已知的，我们在下一行打印它。这里没有什么神奇的地方。

现在让我们理解为什么需要在运行时知道变量的类型。假设我们想编写一个简单的[函数](../【GolangBot】6-函数)，它将接受一个[结构体](../【GolangBot】16-结构体)作为参数，并使用它创建一个 SQL 插入查询。

考虑以下程序：

```go
package main

import (
	"fmt"
)

type order struct {
	ordId      int
	customerId int
}

func main() {
	o := order{
		ordId:      1234,
		customerId: 567,
	}
	fmt.Println(o)
}
```

[在 Playground 中运行](https://play.golang.org/p/0wLQAVErHuH)

我们需要编写一个函数，它将上述程序中的结构体 `o` 作为参数，并返回以下 SQL 插入查询：

```fallback
insert into order values(1234, 567)
```

这个函数很容易编写。让我们现在就来编写它。

```go
package main

import (
	"fmt"
)

type order struct {
	ordId      int
	customerId int
}

func createQuery(o order) string {
	i := fmt.Sprintf("insert into order values(%d, %d)", o.ordId, o.customerId)
	return i
}

func main() {
	o := order{
		ordId:      1234,
		customerId: 567,
	}
	fmt.Println(createQuery(o))
}
```

[在 Playground 中运行](https://play.golang.org/p/jhz4VHKIlQ5)

第 12 行的 `createQuery` 函数通过使用 `o` 的 `ordId` 和 `customerId` 字段创建插入查询。该程序将输出：

```fallback
insert into order values(1234, 567)
```

现在让我们将查询生成器提升到一个新的水平。如果我们想泛化我们的查询生成器，使其适用于任何结构体，该怎么办？让我通过一个程序来解释我的意思。

```go
package main

type order struct {
	ordId      int
	customerId int
}

type employee struct {
	name string
	id int
	address string
	salary int
	country string
}

func createQuery(q interface{}) string {
}

func main() {
	
}
```

我们的目标是完成上述程序中第 16 行的 `createQuery` 函数，使其接受任何结构体作为参数，并根据结构体字段创建插入查询。

例如，如果我们传递以下结构体：

```go
o := order {
	ordId: 1234,
	customerId: 567
}
```

我们的 `createQuery` 函数应返回：

```fallback
insert into order values (1234, 567)
```

同样，如果我们传递：

```go
e := employee {
	name: "Naveen",
	id: 565,
	address: "Science Park Road, Singapore",
	salary: 90000,
	country: "Singapore",
}
```

它应返回：

```fallback
insert into employee values("Naveen", 565, "Science Park Road, Singapore", 90000, "Singapore")
```

由于 `createQuery` 函数应适用于任何结构体，因此它接受一个[*interface{}*](../【GolangBot】18-接口-I/#空接口)作为参数。为简单起见，我们只处理包含 `string` 和 `int` 类型字段的结构体，但这可以扩展到任何类型。

`createQuery` 函数应适用于任何结构体。编写此函数的唯一方法是在运行时检查传递给它的结构体参数的类型，找到其字段，然后创建查询。这就是反射的用武之地。在教程的后续步骤中，我们将学习如何使用 `reflect` 包实现这一点。

### reflect 包

[reflect](https://golang.org/pkg/reflect/) 包在 Go 中实现了运行时反射。reflect 包有助于识别[*interface{}*](../【GolangBot】18-接口-I/#空接口)变量的具体类型和值。这正是我们所需要的。`createQuery` 函数接受一个 `interface{}` 参数，并且需要根据 `interface{}` 参数的具体类型和值创建查询。这正是 reflect 包的作用。

在编写我们的通用查询生成器程序之前，我们需要先了解 reflect 包中的一些类型和方法。让我们逐一看看它们。

#### reflect.Type 和 reflect.Value

`interface{}` 的具体类型由 [reflect.Type](https://golang.org/pkg/reflect/#Type) 表示，其基础值由 [reflect.Value](https://golang.org/pkg/reflect/#Value) 表示。有两个函数 [reflect.TypeOf()](https://golang.org/pkg/reflect/#TypeOf) 和 [reflect.ValueOf()](https://golang.org/pkg/reflect/#ValueOf)，它们分别返回 `reflect.Type` 和 `reflect.Value`。这两种类型是创建我们的查询生成器的基础。让我们编写一个简单的示例来理解这两种类型。

```go
package main

import (
	"fmt"
	"reflect"
)

type order struct {
	ordId      int
	customerId int
}

func createQuery(q interface{}) {
	t := reflect.TypeOf(q)
	v := reflect.ValueOf(q)
	fmt.Println("Type ", t)
	fmt.Println("Value ", v)
}

func main() {
	o := order{
		ordId:      456,
		customerId: 56,
	}
	createQuery(o)
}
```

[在 Playground 中运行](https://play.golang.org/p/81BS-bEfbCg)

在上面的程序中，第 13 行的 `createQuery` 函数接受一个 `interface{}` 作为参数。第 14 行的 [reflect.TypeOf](https://golang.org/pkg/reflect/#TypeOf) 函数接受一个 `interface{}` 作为参数，并返回包含传递给它的 `interface{}` 参数的具体类型的 [reflect.Type](https://golang.org/pkg/reflect/#Type)。同样，第 15 行的 [reflect.ValueOf](https://golang.org/pkg/reflect/#ValueOf) 函数接受一个 `interface{}` 作为参数，并返回包含传递给它的 `interface{}` 参数的基础值的 [reflect.Value](https://golang.org/pkg/reflect/#Value)。

上面的程序打印：

```fallback
Type  main.order
Value  {456 56}
```

从输出中可以看出，程序打印了 `interface{}` 的具体类型和值。

#### reflect.Kind

反射包中还有一个重要的类型叫做 [Kind](https://golang.org/pkg/reflect/#Kind)。

反射包中的 `Kind` 和 `Type` 类型可能看起来很相似，但它们有一个区别，下面的程序将清楚地展示这一点。

```go
package main

import (
	"fmt"
	"reflect"
)

type order struct {
	ordId      int
	customerId int
}

func createQuery(q interface{}) {
	t := reflect.TypeOf(q)
	k := t.Kind()
	fmt.Println("Type ", t)
	fmt.Println("Kind ", k)
}

func main() {
	o := order{
		ordId:      456,
		customerId: 56,
	}
	createQuery(o)
}
```

[在 Playground 中运行](https://play.golang.org/p/Xw3JIzCm54T)

上面的程序输出：

```fallback
Type  main.order
Kind  struct
```

我想你现在应该清楚了两者之间的区别。`Type` 表示 `interface{}` 的实际类型，在本例中是 **main.Order**，而 `Kind` 表示该类型的特定种类。在本例中，它是一个 **struct**。

#### NumField() 和 Field() 方法

[NumField()](https://golang.org/pkg/reflect/#Value.NumField) 方法返回结构体中的字段数，[Field(i int)](https://golang.org/pkg/reflect/#Value.Field) 方法返回第 `i` 个字段的 `reflect.Value`。

```go
package main

import (
	"fmt"
	"reflect"
)

type order struct {
	ordId      int
	customerId int
}

func createQuery(q interface{}) {
	if reflect.ValueOf(q).Kind() == reflect.Struct {
		v := reflect.ValueOf(q)
		fmt.Println("Number of fields", v.NumField())
		for i := 0; i < v.NumField(); i++ {
			fmt.Printf("Field:%d type:%T value:%v\n", i, v.Field(i), v.Field(i))
		}
	}
}

func main() {
	o := order{
		ordId:      456,
		customerId: 56,
	}
	createQuery(o)
}
```

[在 Playground 中运行](https://play.golang.org/p/FBHfJfuTaEe)

在上面的程序中，第 14 行我们首先检查 `q` 的 `Kind` 是否为 `struct`，因为 `NumField` 方法仅适用于结构体。程序的其余部分不言自明。该程序输出：

```fallback
Number of fields 2
Field:0 type:reflect.Value value:456
Field:1 type:reflect.Value value:56
```

#### Int() 和 String() 方法

[Int](https://golang.org/pkg/reflect/#Value.Int) 和 [String](https://golang.org/pkg/reflect/#Value.String) 方法帮助将 `reflect.Value` 提取为 `int64` 和 `string`。

```go
package main

import (
	"fmt"
	"reflect"
)

func main() {
	a := 56
	x := reflect.ValueOf(a).Int()
	fmt.Printf("type:%T value:%v\n", x, x)
	b := "Naveen"
	y := reflect.ValueOf(b).String()
	fmt.Printf("type:%T value:%v\n", y, y)
}
```

[在 Playground 中运行](https://play.golang.org/p/UIllrLVoGwI)

在上面的程序中，第 10 行我们将 `reflect.Value` 提取为 `int64`，第 13 行我们将其提取为 `string`。该程序打印：

```fallback
type:int64 value:56
type:string value:Naveen
```

### 完整程序

现在我们已经掌握了足够的知识来完成我们的查询生成器，让我们继续完成它。

```go
package main

import (
	"fmt"
	"reflect"
)

type order struct {
	ordId      int
	customerId int
}

type employee struct {
	name    string
	id      int
	address string
	salary  int
	country string
}

func createQuery(q interface{}) {
	if reflect.ValueOf(q).Kind() == reflect.Struct {
		t := reflect.TypeOf(q).Name()
		query := fmt.Sprintf("insert into %s values(", t)
		v := reflect.ValueOf(q)
		for i := 0; i < v.NumField(); i++ {
			switch v.Field(i).Kind() {
			case reflect.Int:
				if i == 0 {
					query = fmt.Sprintf("%s%d", query, v.Field(i).Int())
				} else {
					query = fmt.Sprintf("%s, %d", query, v.Field(i).Int())
				}
			case reflect.String:
				if i == 0 {
					query = fmt.Sprintf("%s\"%s\"", query, v.Field(i).String())
				} else {
					query = fmt.Sprintf("%s, \"%s\"", query, v.Field(i).String())
				}
			default:
				fmt.Println("Unsupported type")
				return
			}
		}
		query = fmt.Sprintf("%s)", query)
		fmt.Println(query)
		return
	}
	fmt.Println("unsupported type")
}

func main() {
	o := order{
		ordId:      456,
		customerId: 56,
	}
	createQuery(o)

	e := employee{
		name:    "Naveen",
		id:      565,
		address: "Coimbatore",
		salary:  90000,
		country: "India",
	}
	createQuery(e)
	i := 90
	createQuery(i)
}
```

[在 Playground 中运行](https://play.golang.org/p/82Bi4RU5c7W)

在第 22 行，我们首先检查传递的参数是否为 `struct`。在第 23 行，我们使用 `Name()` 方法从其 `reflect.Type` 中获取结构体的名称。在下一行，我们使用 `t` 并开始创建查询。

第 28 行的 [case](../【GolangBot】10-switch语句) 语句检查当前字段是否为 `reflect.Int`，如果是，我们使用 `Int()` 方法将该字段的值提取为 `int64`。[if else](../【GolangBot】8-if-else语句) 语句用于处理边缘情况。请添加日志以理解为什么需要它。类似的逻辑用于在第 34 行提取 `string`。

我们还添加了检查，以防止在将不受支持的类型传递给 `createQuery` 函数时程序崩溃。程序的其余部分不言自明。我建议在适当的位置添加日志并检查其输出，以更好地理解该程序。

该程序打印：

```fallback
insert into order values(456, 56)
insert into employee values("Naveen", 565, "Coimbatore", 90000, "India")
unsupported type
```

我将把将字段名称添加到输出查询的练习留给读者。请尝试更改程序以打印以下格式的查询：

```fallback
insert into order(ordId, customerId) values(456, 56)
```

### 是否应该使用反射？

在展示了反射的实际用途之后，现在出现了一个真正的问题。你应该使用反射吗？我想引用 [Rob Pike](https://en.wikipedia.org/wiki/Rob_Pike) 关于使用反射的谚语来回答这个问题。

*清晰胜于聪明。反射从来都不清晰。*

反射是 Go 中一个非常强大和高级的概念，应该谨慎使用。使用反射编写清晰且可维护的代码非常困难。应尽可能避免使用它，仅在绝对必要时使用。


**下一个教程 - [读取文件](../【GolangBot】36-读文件)**