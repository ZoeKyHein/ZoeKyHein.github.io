---
title: 【GolangBot】15-指针
date: 2024-12-24 10:51:06
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---



欢迎来到[Golang系列教程](../golangbot/)的第15篇。
在本教程中，我们将学习Go中的指针，我们还将了解Go指针与其他语言（如C和C++）中的指针有何不同。
本教程包含以下部分：
- 什么是指针？
- 声明指针
- 指针的零值
- 使用`new`函数创建指针
- 解引用指针
- 将指针传递给函数
- 从函数返回指针
- 不要将数组作为参数传递给函数。使用切片代替。
- Go不支持指针运算

### 什么是指针？
指针是一个变量，它存储了另一个变量的内存地址。

![Pointers in Go](https://golangbot.com/content/images/2017/05/pointer-explained.png)


在上面的插图中，变量`b`的值为`156`，存储在内存地址`0x1040a124`。变量`a`持有`b`的地址。现在`a`指向`b`。现在`a`被称为指向`b`。
### 声明指针
**T** 是指针变量的类型，它指向一个值为类型**T**的变量。
让我们写一个程序来声明一个指针。


```go
package main

import (
	"fmt"
)

func main() {
	b := 255
	var a *int = &b
    fmt.Printf("Type of a is %T\n", a)
	fmt.Println("address of b is", a)
}
```


[Run in playground](https://play.golang.org/p/A4vmlgxAy8)

**&** 运算符用于获取变量的地址。在上面程序的第9行中，我们将`b`的地址分配给`a`，其类型为`*int`。现在`a`被称为指向`b`。当我们打印`a`的值时，将打印`b`的地址。此程序输出


```fallback
Type of a is *int
address of b is 0x1040a124
```

因为b的地址可能在内存中的任何位置，因此它的地址可能会有所不同。
### 指针的零值
指针的零值是`nil`。


```go
package main

import (
	"fmt"
)

func main() {
	a := 25
	var b *int
	if b == nil {
		fmt.Println("b is", b)
		b = &a
		fmt.Println("b after initialization is", b)
	}
}
```


[Run in playground](https://play.golang.org/p/yAeGhzgQE1)
b在上面的程序中被初始化为nil，然后它被赋值给了a的地址。此程序输出

```fallback
b is <nil>
b after initialisation is 0x1040a124
```

### 使用new函数创建指针

Go也提供了一个方便的函数`new`来创建指针。`new`函数接受一个类型作为参数，并返回一个指向新分配的零值的类型的指针。
下面的例子将会让你更好地理解指针。


```go
package main

import (
	"fmt"
)

func main() {
	size := new(int)
	fmt.Printf("Size value is %d, type is %T, address is %v\n", *size, size, size)
	*size = 85
	fmt.Println("New size value is", *size)
}
```


[Run in playground](https://play.golang.org/p/BNkfB3RZCOY)

在上面的程序中，在第8行，我们使用`new`函数创建一个类型为`int`的指针。这个函数将返回一个指向新分配的零值的类型为`int`的指针。`int`的零值为`0`。因此，`size`将为类型`*int`，并指向`0`，即`*size`将为`0`。
这个程序将打印


```fallback
Size value is 0, type is *int, address is 0x414020
New size value is 85
```

### 解引用指针

解引用指针意味着访问指针指向的变量的值。`*a`是解引用的语法。
让我们写一个程序来理解指针的解引用。

```go
package main
import (  
    "fmt"
)

func main() {  
    b := 255
    a := &b
    fmt.Println("address of b is", a)
    fmt.Println("value of b is", *a)
}
```


[Run in playground](https://play.golang.org/p/m5pNbgFwbM)

在上面程序的第10行，我们解引用指针`a`并打印它的值。正如预期的那样，它打印`b`的值。此程序的输出为

```fallback
address of b is 0x1040a124
value of b is 255
```

让我们再写一个程序来理解指针的解引用，这次我们使用指针修改`b`的值。

```go
package main

import (
	"fmt"
)

func main() {
	b := 255
	a := &b
	fmt.Println("address of b is", a)
	fmt.Println("value of b is", *a)
	*a++
	fmt.Println("new value of b is", b)
}
```


[Run in playground](https://play.golang.org/p/cdmvlpBNmb)

在上面程序的第 12 行，我们将 a 指向的值加 1，这会改变 b 的值，因为 a 指向的是 b。因此，b 的值变为 256。程序的输出是：

```fallback
address of b is 0x1040a124
value of b is 255
new value of b is 256
```

### 将指针传递给函数

```go
package main

import (
	"fmt"
)

func change(val *int) {
	*val = 55
}
func main() {
	a := 58
	fmt.Println("value of a before function call is",a)
	b := &a
	change(b)
	fmt.Println("value of a after function call is", a)
}
```


[Run in playground](https://play.golang.org/p/3n2nHRJJqn)

在上面程序的第 14 行，我们将保存了 a 地址的指针变量 b 传递给函数 change。在 change 函数内部，第 8 行通过解引用更改了 a 的值。程序的输出是：

```fallback
value of a before function call is 58
value of a after function call is 55
```

### 从函数返回指针
函数返回局部变量的指针是完全合法的。Go 编译器足够智能，会将该变量分配到堆上。


```go
package main

import (
	"fmt"
)

func hello() *int {
	i := 5
	return &i
}
func main() {
	d := hello()
	fmt.Println("Value of d", *d)
}
```

[Run in playground](https://play.golang.org/p/I6r-fRx2qML)


在上面程序的第 9 行，我们从函数 `hello` 返回局部变量 `i` 的地址。**在 C 和 C++ 等编程语言中，这段代码的行为是未定义的，因为变量 `i` 在函数 `hello` 返回后会超出作用域。但在 Go 中，编译器会进行逃逸分析，并将 `i` 分配到堆上，因为其地址超出了局部作用域。** 因此，这段程序可以正常运行，并将打印：

```fallback
Value of d 5
```

### 不要将数组作为参数传递给函数。使用切片代替。

假设我们想在函数内部修改一个数组，并希望在函数内部对该数组所做的更改对调用者可见。一种实现方法是将数组的指针作为参数传递给函数。

```go
package main

import (  
    "fmt"
)

func modify(arr *[3]int) {
	(*arr)[0] = 90
}

func main() {  
    a := [3]int{89, 90, 91}
    modify(&a)
    fmt.Println(a)
}
```


[Run in playground](https://play.golang.org/p/lOIznCbcvs)

在上面程序的第 13 行，我们将数组 `a` 的地址传递给了 `modify` 函数。在 `modify` 函数的第 8 行，我们通过解引用 `arr` 并将 `90` 分配给数组的第一个元素。该程序输出 `[90 90 91]`。

**`a[x]` 是 `(*a)[x]` 的简写。因此，上述程序中的 `(*arr)[0]` 可以替换为 `arr[0]`。** 让我们使用这种简写语法重写上述程序。

```go
package main

import (  
    "fmt"
)

func modify(arr *[3]int) {
	arr[0] = 90
}

func main() {  
    a := [3]int{89, 90, 91}
    modify(&a)
    fmt.Println(a)
}
```


[Run in playground](https://play.golang.org/p/k7YR0EUE1G)

该程序同样输出 `[90 90 91]`。

**尽管通过将数组的指针作为参数传递给函数并修改它的方式可以实现目标，但这并不是 Go 中实现此功能的惯用方式。我们有 [切片](../【GolangBot】11-数组和切片/) 来实现这一点。**

让我们使用 [切片](../【GolangBot】11-数组和切片/) 重写同样的程序。

```go
package main

import (
	"fmt"
)

func modify(sls []int) {
	sls[0] = 90
}

func main() {
	a := [3]int{89, 90, 91}
	modify(a[:])
	fmt.Println(a)
}
```


[Run in playground](https://play.golang.org/p/rRvbvuI67W)

在上面程序的第 13 行，我们将切片传递给 `modify` 函数。在 `modify` 函数内部，将切片的第一个元素修改为 `90`。该程序同样输出 `[90 90 91]`。
 **所以，忘掉传递数组指针的方法，直接使用切片吧 :)。这种代码更简洁，并且符合 Go 的惯用写法 :)。**

### Go 不支持指针运算

Go 不支持像 C 和 C++ 等其他语言中存在的指针运算功能。

```go
package main

func main() {
	b := [...]int{109, 110, 111}
	p := &b
	p++
}
```


[Run in playground](https://play.golang.org/p/WRaj4pkqRD)

上述程序会抛出编译错误：  
**main.go:6: invalid operation: p++ (non-numeric type \*[3]int)**  

我已经在 [GitHub](https://github.com/golangbot/pointers) 上创建了一个完整的程序，涵盖了我们讨论的所有内容。  


**下一篇教程 - [结构体](../【GolangBot】16-结构体/)**
