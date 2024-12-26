---
title: 【GolangBot】3-变量
date: 2024-12-17 08:47:51
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
---


这是我们[Golang系列教程](../golangbot/)的第三篇，这篇我们来处理Golang中的变量。

你可以阅读[第二部分](../【GolangBot】2-Hello-World/)来学习配置Go，运行Hello World程序。

### 什么是变量？

变量是给内存位置的名称，用来存储特定[类型](../【GolangBot】4-数据类型/)的值。Go中有多种语法来声明变量。我们来逐一查看。

### 声明单个变量

**var name type** 是声明单个变量的语法。

```go
package main

import "fmt"

func main() {
	var age int // variable declaration
	fmt.Println("My initial age is", age)
}
```

[Run in playground](https://go.dev/play/p/5sC9oz7j_lR)



`var age int`语法定义了一个名为`age`，类型为`int`的变量。我们还没有给这个变量分配任何值。如果一个没有被分配任何值，Go会自动用变量类型的零值来为它赋值。在这个例子中，age被分配值为0，0是`int`类型的零值。如果你运行这个程序，你会看到如下输出。

```fallback
My initial age is 0  
```

一个变量可以被分配它类型的任意值。在上面的程序中，`age`可以被分配任意的整型值。

```go
package main
 
import "fmt"
 
func main() {
	var age int // variable declaration
	fmt.Println("My initial age is", age)
	age = 29 //assignment
	fmt.Println("My age after first assignment is", age)
	age = 54 //assignment
	fmt.Println("My age after second assignment is", age)
}
```

[Run in playground](https://go.dev/play/p/6olrLC2kBTB)

上面的程序将会打印如下内容。

```fallback
My initial age is 0
My age after first assignment is 29
My age after second assignment is 54
```

### 声明一个带初始值的变量

当一个变量被声明时，也可以初始化一个值。下面是声明一个带初始值的变量的语法。

A variable can also be initialized with a value when it is declared. The following is the syntax to declare a variable with an initial value.

``` go
var name type = initialvalue
```

```go
package main

import "fmt"

func main() {
	var age int = 29 // variable declaration with initial value

	fmt.Println("My initial age is", age)
}
```

[Run in playground](https://go.dev/play/p/17Py5gyLc87)

在上面的程序中，`age`是一个`int`型变量，有初始值`29`。上面的程序会打印如下输出。

```fallback
My initial age is 29 
```

这证实了age已被初始化为值29.

### 类型推断

如果一个变量有一个初始值，Go能够自动根据初始值推断出该变量的类型。因此如果变量有一个初始值，变量声明中的`type`就可以去掉。

如果变量是用下面的语法声明

```fallback
var name = initialvalue
```

Go会自动从初始值推断出变量的类型。

在下面的例子中，我们可以看到第6行变量`age`的类型`int`已经被去掉了。因为变量有一个初始值`29`，Go可以推断出它是`int`类型。

```go
package main

import "fmt"

func main() {
	var age = 29 // type will be inferred
	fmt.Println("My initial age is", age)
}
```

[Run in playground](https://go.dev/play/p/3ybtHsu6uLk)

### 多变量声明

多个变量可以用一条语句进行声明。

**var name1, name2 type = initialvalue1, initialvalue2** 是多变量声明的语法。

```go
package main

import "fmt"

func main() {
	var price, quantity int = 5000, 100 //declaring multiple variables

	fmt.Println("price is", price, "quantity is", quantity)
}
```

[Run in playground](https://go.dev/play/p/ik6-iRcjggI)

如果变量有初始值，类型可以去掉。因为上面的程序中，变量有初始值，类型`int`可以去掉。

```go
package main

import "fmt"

func main() {
	var price, quantity = 5000, 100 //declaring multiple variables with type inference

	fmt.Println("price is", price, "quantity is", quantity)
}
```

[Run in playground](https://go.dev/play/p/cwD5TmYG2Nf)

上面的程序将会打印

```fallback
price is 5000 quantity is 100
```

现在也许你已经猜到了，如果初始值未指定`price`和`quanlity`的初始值，它们的初始值将是`0`。

```go
package main

import "fmt"

func main() {
	var price, quantity int
	fmt.Println("price is", price, "quantity is", quantity)
	price = 3000
	quantity = 500
	fmt.Println("new price is", price, "new quantity is", quantity)
}
```

[Run in playground](https://go.dev/play/p/W2zBg9dNzNA)

上面的程序将会打印

```fallback
price is 0 quantity is 0
new price is 3000 new quantity is 500
```

有一种情况，我们可能会想要在一条语句中定义多个变量，并且它们属于不同的数据类型。这么做的语法是

```fallback
var (
      name1 = initialvalue1
      name2 = initialvalue2
)
```

下面的程序使用了上面的语法来声明不同类型的变量。

```go
package main

import "fmt"

func main() {
	var (
		name   = "Naveen"
		age    = 38
		height int
	)
	fmt.Println("my name is", name)
	fmt.Println("my age is", age)
	fmt.Println("my height is", height)
}
```

[Run in playground](https://go.dev/play/p/UMeFO9KoV-U)

这里我们声明了一个`string`类型的变量`name`，`int`类型的`age`和`height`。(我们将会在[下一篇](../【GolangBot】4-数据类型/)讨论Golang中不同类型的变量)

运行上面的程序将会打印。

```fallback
my name is Naveen
my age is 38
my height is 0
```

### 简短声明

Go也提供了另一种简短的声明变量方式。这就是所谓的简短声明，它使用`:=`操作符。

**name := initialvalue** 是声明一个变量的简短语法。

下面的程序使用了简短语法声明了一个初始化为`10`的变量`count`。Go会自动推断出`count`是`int`类型，因为它被整型值`10`初始化。

```go
package main

import "fmt"

func main() {
	count := 10
	fmt.Println("Count =",count)
}
```

[Run in playground](https://go.dev/play/p/9X5FGM0QYxS)

上面的程序将会打印，

```fallback
Count = 10
```

使用简短声明语法声明多个变量也是可以的。

```go
package main

import "fmt"

func main() {
	name, age := "Naveen", 29 //short hand declaration

	fmt.Println("my name is", name)
	fmt.Println("my age is", age)
}
```

[Run in playground](https://go.dev/play/p/t_leq8orwxL)

上面的程序声明了两个变量`name`和`age`，它们分别是`string`和`int`类型。

如果你运行上面的程序，你将会看到下面的内容被打印。

```fallback
my name is Naveen
my age is 29
```

简短声明要求赋值左侧的所有变量都有初始值。下面的程序将会打印错误`assignment mismatch: 2 variables but 1 value`。这是因为**age没有被分配值**。

```go
package main

import "fmt"

func main() {
	name, age := "Naveen" //error

	fmt.Println("my name is", name, "age is", age)
}
```

[Run in playground](https://go.dev/play/p/kM93hDPatVP)

简短声明只能在**:=**左边至少有一个新声明的变量时才可以使用。看一下下面的程序，

```go
package main

import "fmt"

func main() {
	a, b := 20, 30 // declare variables a and b
	fmt.Println("a is", a, "b is", b)
	b, c := 40, 50 // b is already declared but c is new
	fmt.Println("b is", b, "c is", c)
	b, c = 80, 90 // assign new values to already declared variables b and c
	fmt.Println("changed b is", b, "c is", c)
}
```

[Run in playground](https://go.dev/play/p/V2aMZKS5Kpj)

在上面的程序中，第8行，**b**已经被声明过了，而**c**是新声明的，因此它可以正常运行并且输出。

```fallback
a is 20 b is 30
b is 40 c is 50
changed b is 80 c is 90
```

而如果我们运行下面的程序，

```go
package main

import "fmt"

func main() {
	a, b := 20, 30 //a and b declared
	fmt.Println("a is", a, "b is", b)
	a, b := 40, 50 //error, no new variables
}
```

[Run in playground](https://go.dev/play/p/SYyrrmPHMG_F)

它会打印错误`./prog.go:8:7: no new variables on left side of :=`。这是因为变量**a**和**b**都已经被定义过，并且第8行**:=**左边没有新变量了。

变量也可以分配值，这些值时在运行时计算得出的。请看下面的程序，

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	a, b := 145.8, 543.8
	c := math.Min(a, b)
	fmt.Println("Minimum value is", c)
}
```

[Run in playground](https://go.dev/play/p/SVQ2RGT7JuP)

在上面的程序中，[math](https://golang.org/pkg/math/)是一个[包](https://golangbot.com/go-packages/)，[Min](https://golang.org/pkg/math/#Min)是那个包中的一个[函数](https://golangbot.com/functions/)。现在不需要担心这个，我们将会在接下来的章节[包](../【GolangBot】7-包/)和[函数](../【GolangBot】6-函数/)中详细介绍它们。我们现在只需要知道，`c`的值时在运行过程中计算出来的，并且它时`a`和`b`之中的最小值。上面的程序将会打印，

```fallback
Minimum value is  145.8
```

因为Go是强类型的，声明为一种类型的变量不能赋值给另外一种类型。下面的程序将会打印错误`cannot use "Naveen" (untyped string constant) as int value in assignment`，因为`age`声明为`int`类型，我们正尝试为它分配一个`string`值。

```go
package main

func main() {
	age := 29      // age is int
	age = "Naveen" // error since we are trying to assign a string to a variable of type int
}
```

[Run in playground](https://go.dev/play/p/_L67J5Mxy8F)

**下一篇教程 - [类型](../【GolangBot】4-数据类型/)**
