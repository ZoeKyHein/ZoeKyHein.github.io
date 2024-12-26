---
title: 【GolangBot】6-函数
date: 2024-12-18 14:48:21
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---



欢迎来到[Golang系列教程](../golangbot/)的第六篇。

### 什么是函数？

函数是执行特定任务的代码块。函数接受输入，对输入执行一些操作并产生输出。例如，一个函数可以把半径作为输入然后计算出面积和周长作为输出。

### 函数声明

下面是Go中声明一个函数的语法。

```go
func functionname(parametername datatype) returntype {
 //function body
}
```

函数声明以`func`关键字开始，后面跟着`functionname`。参数被指定在` (` 和 ` ) `之间，再往后是函数的`returntype`。指定参数的语法是，参数类型跟在参数名字后面。任何数量的参数都是可以的，比如：`(parameter1 datatype, parameter2 datatype)`。然后在`{` 和 `}`之间有一个代码块，这是函数体。

参数和返回值类型在一个函数中都是可选项。因此下面的函数声明也是符合规定的。

```go
func functionname() {
}
```

### 示例函数

我们来写一个函数，把产品单价和数量作为输入参数，返回这两个值的乘积作为总价。

```go
func calculateBill(price int, quantity int) int {
	var totalPrice = price * quantity
	return totalPrice
}
```

上面的函数有两个`int`类型的输入参数`price`和`quantity`，它返回`totalPrice`，这是`price`和`quantity`的乘积。返回值也是`int`类型。

**如果连着的几个参数是同一种类型，我们可以省去每次都写类型的麻烦，只在最后写一个类型就够了，即`price int, quantity int` 可以写成`price, quantity int`。**上面的函数因此可以重写为，

```go
func calculateBill(price, quantity int) int {
	var totalPrice = price * quantity
	return totalPrice
}
```

现在我们有一个函数就虚了，让我们从代码里的某些地方来调用它。调用函数的语法是`functionname(parameters)`。上面的函数可以用这个代码调用。

```go
calculateBill(10, 5)
```

这是我们用上面的函数打印总价的完整程序。

```go
package main

import (
	"fmt"
)

func calculateBill(price, quantity int) int {
	var totalPrice = price * quantity
	return totalPrice
}

func main() {
	price, quantity := 90, 6
	totalPrice := calculateBill(price, quantity)
	fmt.Println("Total price is", totalPrice)
}
```

[Run in playground](https://go.dev/play/p/07SsuF0MpDP)

上面的程序将会打印

```fallback
Total price is 540
```

### 多个返回值

一个函数返回多个值这种情况也是可能的。让我们写一个函数`rectProps`，它接受一个矩形的`length`(长)和`width`(宽)，返回矩形的`area`(面积)和`perimeter`(周长)。举行的面积是长和宽的乘积，周长是长宽之和的两倍。

```go
package main

import (
	"fmt"
)

func rectProps(length, width float64)(float64, float64) {
	var area = length * width
	var perimeter = (length + width) * 2
	return area, perimeter
}

func main() {
 	area, perimeter := rectProps(10.8, 5.6)
	fmt.Printf("Area %f Perimeter %f", area, perimeter)	
}
```

[Run in playground](https://go.dev/play/p/wlUbh-WyOKt)

如果一个函数返回多个值，那他们必须被再`(` 和 `)`之间被指定。`func rectProps(length, width float64)(float64, float64)`有两个float64类型的参数`length`和`width`，并且返回两个`float64`类型的值。上面的程序打印

```fallback
Area 60.480000 Perimeter 32.800000
```

### 为返回值命名

从函数中返回一个命名好的值也是可以的。如果返回值被命名，它可以被视为函数第一行的变量。

上面的rectProps可以用命名返回值重写为

```go
func rectProps(length, width float64)(area, perimeter float64) {  
    area = length * width
    perimeter = (length + width) * 2
    return //no explicit return value
}
```

**area**和**perimeter**在上面的函数中是被命名的返回值。请注意，函数的返回语句没有显式地返回任何值。因为`area`和`perimeter`在函数声明中被指定为返回值，所以当运行到返回语句的时候它们被自动返回了。

### 空白标识符

**_** 在Go中是空白标识符。它可以用于代替任何类型的任何值。让我们看看这个空白标识符的作用是什么。

函数`rectProps`返回矩形的面积和周长。如果我们只需要`area`并且想要抛掉`perimeter`该怎么办呢？这就是`_`的作用场景。

下面的程序只使用了`rectProps`函数返回的`area`。

```go
package main

import (
	"fmt"
)

func rectProps(length, width float64) (float64, float64) {
	var area = length * width
	var perimeter = (length + width) * 2
	return area, perimeter
}
func main() {
	area, _ := rectProps(10.8, 5.6) // perimeter is discarded
	fmt.Printf("Area %f ", area)
}
```

[Run in playground](https://go.dev/play/p/Tw6KYjo5zoI)

在第13行，我们只用了`area`，而`_`标识符用于抛掉`perimeter`。



**下一篇教程 - [包](../_GolangBot】7-包/)**
