---
title: 【GolangBot】4-数据类型
date: 2024-12-17 08:48:02
tags: 
    - GoLang
    - 教程
categories:
    - [学习心得, GoLangBot]
---

# 【GolangBot】4-数据类型

这是我们[Golang系列教程](https://golangbot.com/learn-golang-series/)的第四篇.

请阅读[第三部分：变量](../【GolangBot】3-变量/)来学习变量。

下面是Go中可用的基本类型。

*   bool
*   Numeric Types
    *   int8, int16, int32, int64, int
    *   uint8, uint16, uint32, uint64, uint
    *   float32, float64
    *   complex64, complex128
    *   byte
    *   rune
*   string

## bool

`bool`类型表示布尔值。它要么是`true`要么是`false`。

``` go
package main
  
import "fmt"

func main() {  
    a := true
    b := false
    fmt.Println("a:", a, "b:", b)
    c := a && b
    fmt.Println("c:", c)
    d := a || b
    fmt.Println("d:", d)
}
```

[Run in playground](https://go.dev/play/p/v_W3HQ0MdY)

在上面的程序中，a被分配了`true`,b被分配了`false`。

`&&`是一个布尔操作符，当两个操作数都为true的时候才返回true。

c被分配了值`a && b`。在这个示例中，`c`是`false`，因为`a`和`b`不都为`true`。

`||`操作符当`a`或`b`有一个为`true`的时候返回true。在这个示例中，`d`为`true`，因为`a`是true。这个程序我们将会得到下面的输出。

```fallback
a: true b: false
c: false
d: true
```

## 带符号整数

下面是在go中可用的带符号整数数据类型。

| 数据类型 | 描述                                                         | 大小                                     | 范围                                                         |
| -------- | ------------------------------------------------------------ | ---------------------------------------- | ------------------------------------------------------------ |
| int8     | 8 bit 带符号整数                                             | 8 bits                                   | -128 to 127                                                  |
| int16    | 16 bit 带符号整数                                            | 16 bits                                  | -32768 to 32767                                              |
| int32    | 32 bit 带符号整数                                            | 32 bits                                  | -2147483648 to 2147483647                                    |
| int64    | 64 bit 带符号整数                                            | 64 bits                                  | -9223372036854775808 to 9223372036854775807                  |
| int      | 表示32bit或64bit整数，取决于底层架构。通常应该用`int`表示整数，除非有需要使用特定大小的整数。 | 32位系统中 32 bits ，64位系统中64 bits . | 在32位系统中：-2147483648 to 2147483647  <br />在64位系统中：<br />-9223372036854775808 to 9223372036854775807 |

```go
package main

import "fmt"

func main() {
	var a int = 89
	b := 95
	fmt.Println("value of a is", a, "and b is", b)
}
```

[Run in playground](https://go.dev/play/p/Up5vcjHyunS)

上面的程序将会打印

```fallback
value of a is 89 and b is 95
```

在上面的程序中，`a`是`int`类型，`b`的类型是根据分配给它的值(95)推断出来的。如上所述，int的尺寸是*32位系统中 32 bits ，64位系统中64 bits*。让我们继续验证这个说法。

一个变量的类型可以使用`Printf`函数中的`%T`格式指定符来打印出来。Go有一个[unsafe](https://pkg.go.dev/unsafe)包，其中有一个[Sizeof](https://pkg.go.dev/unsafe#Sizeof)函数，它可以返回以字节为单位的变量大小。*unsafe*包应该谨慎使用，因为它可能存在可移植性问题。但出于教程目的，我们可以使用它。

下面的程序输出了变量`a`和`b`的类型和大小。`%T`是一个格式指定符用来打印类型，`%d`用来打印大小。

```go
package main

import (
	"fmt"

	"unsafe"
)

func main() {

	var a = 89
	b := 95
	fmt.Println("value of a is", a, "and b is", b)
	fmt.Printf("type of a is %T, size of a is %d bytes", a, unsafe.Sizeof(a))   //type and size of a
	fmt.Printf("\ntype of b is %T, size of b is %d bytes", b, unsafe.Sizeof(b)) //type and size of b
}
```

[Run in playground](https://go.dev/play/p/fAIGqDZGm7C)

上面的程序将会打印如下输出。

```fallback
value of a is 89 and b is 95
type of a is int, size of a is 8 bytes
type of b is int, size of b is 8 bytes
```

我们可以从上面的输出中推断出`a`和`b`都是`int`类型，并且它们有8字节大小(64bits)。当你在32位系统运行上述程序时，结果会有所不同。在32位系统，a和b占4字节(32bits)。

## 无符号整数

顾名思义，无符号整数只能用于存储正整数。以下是Go中可用的无符号整数数据类型。

| 数据类型 | 描述                                    | 大小                                | 范围                                                         |
| -------- | --------------------------------------- | ----------------------------------- | ------------------------------------------------------------ |
| uint8    | 8 bit 无符号整数                        | 8 bits                              | 0 to 255                                                     |
| uint16   | 16 bit 无符号整数                       | 16 bits                             | 0 to 65535                                                   |
| uint32   | 32 bit 无符号整数                       | 32 bits                             | 0 to 4294967295                                              |
| uint64   | 64 bit 无符号整数                       | 64 bits                             | 0 to 18446744073709551615                                    |
| uint     | 取决于底层架构，32 or 64 bit 无符号整数 | 32位系统上32 bits;64位系统上64 bits | 0 to 4294967295 in 32 bit systems and 0 to 18446744073709551615 in 64 bit systems |

无符号整数用于不需要使用负数的场景。

在下面的程序中，变量`a`和`b`是`uint`类型。

```go
package main

import "fmt"

func main() {
	var a uint = 60
	var b uint = 30
	c := a * b
	fmt.Println("c =", c)
	fmt.Printf("Data type of variable c is %T", c)
}
```

[Run in playground](https://go.dev/play/p/5CKH28Wl_k6)

上面的程序打印结果

```fallback
1c = 1800
2Data type of variable c is uint
```

因为`a`和`b`都是`uint`类型，所以推断`c`的类型也是`uint`。

## 浮点数类型

| 数据类型 | Description   |
| -------- | ------------- |
| float32  | 32 bit 浮点数 |
| float64  | 64 bit 浮点数 |

下面是一个简单的程序，用来说明整数和浮点数类型。

```go
package main

import (
	"fmt"
)

func main() {
	a, b := 5.67, 8.97
	fmt.Printf("type of a %T b %T\n", a, b)
	sum := a + b
	diff := a - b
	fmt.Printf("sum of %f and %f is %f, diff is %f\n", a, b, sum, diff)

	no1, no2 := 56, 89
	fmt.Printf("type of no1 %T no2 %T\n", no1, no2)
	fmt.Printf("sum of %d and %d is %d, diff is %d", no1, no2, no1+no2, no1-no2)
}
```

[Run in playground](https://go.dev/play/p/ST637417mye)

`a`和`b`的类型是从分配给它们的值推断出来的。在这个例子里面，`a`和`b`是`float64`类型。`float64`是浮点值的默认类型。我们把`a`和`b`相加，然后把结果分配给变量`sum`。我们用`a`减去`b`，然后把结果分配给变量`diff`。把`sum`和`diff`打印出来。相似的计算在`no1`和`no2`中进行。上面的程序将会打印,

```fallback
type of a float64 b float64
sum of 5.670000 and 8.970000 is 14.640000, diff is -3.300000
type of no1 int no2 int
sum of 56 and 89 is 145, diff is -33
```

## 复数类型

| 数据类型   | 描述                              |
| ---------- | --------------------------------- |
| complex64  | 具有float32类型的实部和虚部的复数 |
| complex128 | 具有float64类型的实部和虚部的复数 |

标准库函数**[complex](https://pkg.go.dev/builtin#complex)**用于构建具有实部和虚部的复数。complex函数具有如下定义

```fallback
func complex(r, i FloatType) ComplexType
```

它将实部和虚部作为参数并返回复数类型。*实部和虚部必须是相同的数据类型，即float32或float64。如果实部和虚部都是float32，函数将返回complex64类型的复数值。如果实部和虚部都是float64，则此函数返回complex128类型的复数值。*

复数类型也可以使用短语法创建。

```fallback
c := 6 + 7i
```

让我们写一个小程序来理解复数。

```go
package main

import (
	"fmt"
)

func main() {
	c1 := complex(5, 7)
	c2 := 8 + 27i
	cadd := c1 + c2
	fmt.Println("sum:", cadd)
	cmul := c1 * c2
	fmt.Println("product:", cmul)
}
```

[Run in playground](https://go.dev/play/p/-d_Y_medfSb)

在上面的程序中，c1和c2是两个复数。c1有5作为实部，7作为虚部。c2有8作为实部，27作为虚部。`cadd`被指定为c1和c2的和，`cmul`被指定为c1和c2的乘积。这个程序将会输出

```fallback
sum: (13+34i)
product: (-149+191i)
```

## 其他数字类型

**byte** 是uint8的别名。
**rune** 是int32的别名。

我们将会在学习[strings](../【GolangBot】14-Strings/)的时候详细讨论bytes和runes。

## string类型

Strings在Go中是bytes的集合。如果这个定义没有任何意义也没有关系。现在，我们可以假定字符串是字符的集合。我们将在单独的[strings教程](../【GolangBot】14-Strings/)中详细了解字符串。

让我们用字符串写一个程序。

```go
package main

import (
	"fmt"
)

func main() {
	first := "Naveen"
	last := "Ramanathan"
	name := first +" "+ last
	fmt.Println("My name is",name)
}
```

[Run in playground](https://go.dev/play/p/kYnedsD6OmN)

在上面的程序中，`first`被指定为字符串`Naveen`，`last`被指定为字符串`Ramanathan`。可以使用`+`运算符连接字符串。`name`的值为`first`，后加空格，最后加`last`。上述程序将打印

```fallback
1My name is Naveen Ramanathan
```

作为输出结果。

还可以对字符串执行更多操作。我们将在单独的教程中讨论这些内容。

## 类型转换

Go对于显式输入非常严格。没有自动类型提升或者转换。让我们用一个例子来看一下这是什么意思。

```go
package main

import (
	"fmt"
)

func main() {
	a := 80      //int
	b := 91.8    //float64
	sum := a + b //int + float64 not allowed
	fmt.Println(sum)
}
```

[Run in playground](https://go.dev/play/p/Fo537OgUC2B)

上面的代码在C语言中是完全合法的，但在`Go`当中，这个程序无法编译。`a`是一个`int`类型的变量，`b`是一个`float64`类型的变量。我们尝试将两个不同类型的数字相加，这是不允许的。当你运行这个程序，你将会看到如下的编译错误

```fallback
1./prog.go:10:9: invalid operation: a + b (mismatched types int and float64)
```

To fix the error, both `a` and `b` must be of the same type. Let’s convert `b` to `int`. *T(v) is the syntax to convert a value v to type T*

为了修复这个错误，`a`和`b`必须是相同类型。让我们把`b`转换为`int`。*T(v) 是将值v转换为T类型的语法*

```go
package main

import (
	"fmt"
)

func main() {
	a := 80           //int
	b := 91.8         //float64
	sum := a + int(b) //int + float64 not allowed
	fmt.Println(sum)
}
```

[Run in playground](https://go.dev/play/p/Yc00DWzrYl8)

因为`b`从`float`被转换为了`int`，它的浮点值将会被截断，因此我们将会看到`171`作为输出。

赋值的情况也是如此。奖一种类型的变量赋值给另一种类型的变量时，需要进行显式类型转换。下面的程序对此进行了解释。

```go
package main

import (
	"fmt"
)

func main() {
	i := 10
	var j float64 = float64(i) //this statement will not work without explicit conversion
	fmt.Println("j =", j)
}
```

[Run in playground](https://go.dev/play/p/Dia3qOrRi_j)

In line no. 9, `i` is converted to `float64` and then assigned to `j`. When you try to assign `i` to `j` without any type conversion, the compiler will throw an error.

在第9行,`i`被转换为了`float64`，然后赋值给了`j`。但你尝试在不进行类型转换的情况下，将`i`赋值给`j`时，编译器将会抛出一个错误。

**下一篇教程 - [常量](../【GolangBot】5-常量/)**
