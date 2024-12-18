---
title: 【GolangBot】5-常量
date: 2024-12-18 09:48:12
tags: 
    - GoLang
    - 教程
categories:
    - [学习心得, GoLangBot]
published: true
---


欢迎来到我们Golang系列教程的第五篇。

### 什么是常量？

Go中的常量用于定义固定的值，例如：

```fallback
95 
"I love Go" 
67.89 
```

等等。常量通常用来表示一个值，这个值在应用的整个生命周期中都不会改变。

### 声明一个常量

关键字`const`在Go中用来声明一个常量。我们用一个例子来看看如何声明一个常量。

```go
package main

import (
	"fmt"
)

func main() {
	const a = 50
	fmt.Println(a)
}
```

[Run in playground](https://go.dev/play/p/mv3B-q3h0zh)


在上面的代码中，`a`是一个常量，它被赋值为`50`。

### 定义一组常量

Go中也有语法用于使用一条语句声明一组常量。下面是用这个语法来声明一组常量的例子。

```go
package main

import (
	"fmt"
)

func main() {
	const (
		retryLimit = 4
		httpMethod = "GET"
	)

	fmt.Println(retryLimit)
	fmt.Println(httpMethod)
}
```

[Run in playground](https://go.dev/play/p/gSPJC0y49Sm)

在上面的程序中，我们已经定义了2个常量`retryLimit`和`httpMethod`。上面的程序将会打印，

```fallback
4
GET
```

常量，顾名思义，不能被再赋值为其他值。在上面的程序中，我们尝试为`a`赋值另一个值`89`。因为`a`是一个常量，所以这是不允许的。这个程序将会运行失败，报错`cannot assign to a (neither addressable nor a map index expression)`。

```go
package main

func main() {  
    const a = 55 //allowed
    a = 89 //reassignment not allowed
}
```

[Run in playground](https://go.dev/play/p/5ggPss1iSsl)

**常量的值应该在编译时就已经知道了。**因此，它不能被赋值为[函数调用](../【GolangBot】6-函数/)的值，因为函数调用发生在运行时。

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	var a = math.Sqrt(4) //allowed
	fmt.Println(a)
	const b = math.Sqrt(4) //not allowed
	fmt.Println(b)
}
```

[Run in playground](https://go.dev/play/p/fFLfoN0L3Nf)

在上面的程序中，`a`是一个[变量](../【GolangBot】3-变量/)，因此它可以被赋值为函数`math.Sqrt(4)`的结果（我们将在[单独的教程](../【GolangBot】6-函数/)中讨论函数）。

`b`是一个常量，并且`b`的值需要在编译时就知道。函数`math.Sqrt(4)`仅在运行时求值，因此`const b = math.Sqrt(4)`编译失败，出现错误，

```fallback
./prog.go:11:12: math.Sqrt(4) (value of type float64) is not constant
```

### 字符串常量、类型常量和无类型常量

在Go中任何被双引号包起来的值都是一个字符串常量。例如，像`"Hello World"`,`"Sam"`在Go中都是常量。

一个字符串常量属于什么类型？答案是**无类型。**

**一个像"Hello World"的字符串没有任何类型。**

```go
const hello = "Hello World"
```

在上面这行代码中，常量`hello`没有任何类型。

Go是强类型语言。所有变量都需要显式的类型。下面的程序中，为变量`name`赋值无类型的常量`n`会发生什么？

```go
package main

import (
	"fmt"
)

func main() {
	const n = "Sam"
	var name = n
	fmt.Printf("type %T value %v", name, name)

}
```

[Run in playground](https://go.dev/play/p/CFqbC8yfe6C)

**答案是无类型常量有一个与之关联的默认类型，只有在某行代码需要时才会提供默认类型。在第9行的语句`var name = n`中，`name`需要一个类型，它从`string`常量`n`的默认类型中获取**

有没有一种办法可以创建一个**类型常量**？答案是有的。下面的代码创建了一个类型常量。

```go
const name string = "Hello World"
```

*name* 在上面的代码中是一个`string`类型的常量。

Go is a strongly typed language. Mixing types during the assignment is not allowed. Let’s see what this means with the help of a program.

Go是强类型语言。赋值过程中是不允许混合类型的。我们用一个程序来看看这是啥意思。

```go
package main

import "fmt"

func main() {
	var defaultName = "Sam" //allowed
	type myString string
	var customName myString = "Sam" //allowed
	customName = defaultName        //not allowed
	fmt.Println(customName)
}
```

[Run in playground](https://go.dev/play/p/sJ1rCtzebkP)

在上面的代码中，我们首先创建了一个变量`defaultName`，并将常量`Sam`赋值给他。**常量`Sam`的默认类型是`string`，所以在赋值之后，`defaultName`的类型是`string`。**

在下一行，我们创建了一个新的类型`myString`，这是`string`类型的别称。

然后我们创建一个`myString`类型的变量`customName`，并且将常量`Sam`赋值给它。因为常量`Sam`是无类型的，它可以被赋值给任意`string`类型的变量。因此这个赋值操作是允许的，`customName`得到类型`myString`。

现在我们有一个`string`类型的变量`defalutName`，另一个`myString`类型的变量`customName`。尽管我们知道`myString`是`string`的别名，Go的强类型规则不允许将一种类型的值赋值给另外一种类型。**因此赋值操作`customName = defaultName`是不被允许的，编译操作会抛出一个错误：``./prog.go:9:15: cannot use defaultName (variable of type string) as myString value in assignment``**

为了让上面的程序正常工作，`defalutName`必须被转换为`myString`类型。这在下面的程序中的第9行进行操作。

```go
package main

import "fmt"

func main() {
	var defaultName = "Sam" //allowed
	type myString string
	var customName myString = "Sam"    //allowed
	customName = myString(defaultName) //allowed
	fmt.Println(customName)
}
```

[Run in playground](https://go.dev/play/p/EObC2BTq4KW)

上面的程序将会打印`Sam`。

### 布尔型常量

布尔型常量和字符串常量没有什么区别。它们是两个无类型的常量`true`和`false`。字符串常量的规则对于布尔型常量也同样适用，所以我们就不再赘述了。下面是一个简单的程序来解释布尔型常量。

```go
package main

func main() {
	const trueConst = true
	type myBool bool
	var defaultBool = trueConst //allowed
	var customBool myBool = trueConst //allowed
	defaultBool = customBool //not allowed
}
```

[Run in playground](https://go.dev/play/p/YWZ80x94Q1D)

The above program is self-explanatory.

上面的程序是不言自明的。

### 数字常量

数字常量包括整数、浮点数和复数。数字常量有一些细节需要注意。

让我们举一些例子来说明问题。

```go
package main

import (
	"fmt"
)

func main() {
	const c = 5
	var intVar int = c
	var int32Var int32 = c
	var float64Var float64 = c
	var complex64Var complex64 = c
	fmt.Println("intVar", intVar, "\nint32Var", int32Var, "\nfloat64Var", float64Var, "\ncomplex64Var", complex64Var)
}
```

[Run in playground](https://go.dev/play/p/9MzdS-nmA1Z)

在上面的程序中，常量`c`是`untyped`类型并且有一个值`5`。**你也许想知道c的默认类型是什么，以及如果它确实有默认类型，我们又该如何将其赋值给其他不同类型的变量。**答案就在`c`的语法当中。下面的程序将会让这个问题更加清晰。

```go
package main

import (
	"fmt"
)

func main() {
	var i = 5
	var f = 5.6
	var c = 5 + 6i
	fmt.Printf("i's type is %T, f's type is %T, c's type is %T", i, f, c)
}
```

[Run in playground](https://go.dev/play/p/XVEGb9aUsxs)

在上面的程序中，每个变量的类型都被数字常量的语法所定义。根据语法，**5**是一个整数，**5.6**是一个浮点数，**5 + 6i**是一个复数。上面的程序运行会打印

```fallback
i's type is int, f's type is float64, c's type is complex128
```

知道了这些，我们来试着理解下面的程序会如何工作。

```go
package main

import (
	"fmt"
)

func main() {
	const c = 5
	var intVar int = c
	var int32Var int32 = c
	var float64Var float64 = c
	var complex64Var complex64 = c
	fmt.Println("intVar", intVar, "\nint32Var", int32Var, "\nfloat64Var", float64Var, "\ncomplex64Var", complex64Var)
}
```

[Run in playground](https://go.dev/play/p/9MzdS-nmA1Z)

在上面的程序中，`c`的值是`5`并且`c`的语法是通用语法。它可以表示浮点数、整数，甚至是无虚部复数。因此它可以赋值给任意兼容类型。可以认为，这些类型的常量的默认类型是由它们所使用的上下文生成的。`var intVar int = c`需要`c`是`int`类型，所以它变成了一个`int`常量。`var complex64Var complex64 = c`需要`c`是复数类型，因此它变成一个复数常量。以此类推:)。

### 数字表达式

数字常量可以在表达式中自由混合和匹配，只有当它们赋值给变量或者用于需要类型的代码中才需要类型。

```go
package main

import (
	"fmt"
)

func main() {
	var a = 5.9 / 8
	fmt.Printf("a's type is %T and value is %v", a, a)
}
```

[Run in playground](https://go.dev/play/p/Nsak9scUAWg)

在上面的程序中，`5.9`语法上是一个浮点数，`8`语法上是一个整数。不过，由于两者都是数字常量，因此`5.9/8`是被允许的。相除后的结果是`0.7375`是一个浮点数`float`。程序的输出是

```fallback
a's type is float64 and value is 0.7375
```

**下一篇教程 - [函数](../_posts/【GolangBot】6-函数/)**
