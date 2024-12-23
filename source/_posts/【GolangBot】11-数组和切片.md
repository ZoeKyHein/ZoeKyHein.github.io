---
title: 【GolangBot】11-数组和切片
date: 2024-12-23 10:49:39
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---

# 【GolangBot】11-数组和切片

欢迎来到[Golang系列教程](../golangbot/)的第11篇。我们将学习Go中的数组和切片。

### 数组

数组是一组属于同一类型的元素的集合。例如，整数集合 5、8、9、79、76 构成一个数组。在 Go 语言中，不允许混合不同类型的值，例如一个同时包含字符串和整数的数组是非法的。

##### 声明

数组属于类型 `[n]T`。其中，`n` 表示数组中的元素数量，`T` 表示每个元素的类型。元素数量 `n` 也是类型的一部分（我们稍后会对此进行更详细的讨论）。

声明数组有多种方式，让我们逐一来看。

```go
package main

import (
	"fmt"
)


func main() {
	var a [3]int //int array with length 3
	fmt.Println(a)
}
```

[Run in playground](https://play.golang.org/p/Zvgh82u0ej)

**`var a [3]int`** 声明了一个长度为 3 的整数数组。**数组中的所有元素会自动赋值为该数组类型的零值**。在这个例子中，`a` 是一个整数数组，因此 `a` 的所有元素都被赋值为 `0`，即整数类型的零值。运行上述程序将会打印

```fallback
[0 0 0]
```

数组的索引从 `0` 开始，到 `length - 1` 结束。让我们为上述数组分配一些值。

```go
package main

import (
	"fmt"
)


func main() {
	var a [3]int //int array with length 3
	a[0] = 12 // array index starts at 0
	a[1] = 78
	a[2] = 50
	fmt.Println(a)
}
```

[Run in playground](https://play.golang.org/p/WF0Uj8sv39)

`a[0]` 为数组的第一个元素赋值。运行程序将会打印：

```fallback
[12 78 50]
```

让我们使用**简写声明**创建相同的数组。

```go
package main 

import (
	"fmt"
)

func main() {
	a := [3]int{12, 78, 50} // short hand declaration to create array
	fmt.Println(a)
}
```

[Run in playground](https://play.golang.org/p/NKOV04zgI6)

上面的程序将打印相同的输出。

```fallback
[12 78 50]
```

在使用简写声明时，并不一定需要为数组中的所有元素都赋值。

```go
package main

import (
	"fmt"
)

func main() {
	a := [3]int{12} 
	fmt.Println(a)
}
```

[Run in playground](https://play.golang.org/p/AdPH0kXRly)

在上面的程序中，第 8 行 `a := [3]int{12}` 声明了一个长度为 3 的数组，但只提供了一个值 `12`。剩余的 2 个元素会自动赋值为 `0`。该程序将打印：

```fallback
[12 0 0]
```

你甚至可以在声明中忽略数组的长度，并用 `...` 替代，让编译器自动推导数组的长度。以下程序就是这样做的：

```go
package main

import (
	"fmt"
)

func main() {
	a := [...]int{12, 78, 50} // ... makes the compiler determine the length
	fmt.Println(a)
}
```

[Run in playground](https://play.golang.org/p/_fVmr6KGDh)

**数组的大小是类型的一部分。** 因此，`[5]int` 和 `[25]int` 是不同的类型。由于这个原因，数组的大小不能被修改。不要担心这个限制，因为存在 `切片`（slices）来解决这个问题。

```go
package main

func main() {
	a := [3]int{5, 78, 8}
	var b [5]int
	b = a //not possible since [3]int and [5]int are distinct types
}
```

[Run in playground](https://play.golang.org/p/kBdot3pXSB)

在上面程序的第 6 行，我们尝试将一个类型为 `[3]int` 的变量赋值给一个类型为 `[5]int` 的变量，这是不允许的，因此编译器会打印以下错误：

```fallback
./prog.go:6:7: cannot use a (type [3]int) as type [5]int in assignment
```

##### 数组是值类型。

Go 中的数组是值类型，而不是引用类型。这意味着当数组被赋值给一个新变量时，会将原数组的副本赋值给新变量。如果对新变量进行更改，原数组不会受到影响。

```go
package main

import "fmt"

func main() {
	a := [...]string{"USA", "China", "India", "Germany", "France"}
	b := a // a copy of a is assigned to b
	b[0] = "Singapore"
	fmt.Println("a is ", a)
	fmt.Println("b is ", b)	
}
```

[Run in playground](https://play.golang.org/p/-ncGk1mqPd)

在上面的程序中，第 7 行将 `a` 的副本赋值给 `b`。在第 8 行，`b` 的第一个元素被更改为 `Singapore`。由于数组是值类型，这一更改不会影响原始数组 `a`。程序将打印：

```fallback
a is [USA China India Germany France]
b is [Singapore China India Germany France]
```

同样，当数组作为参数传递给函数时，它们是按值传递的，原始数组不会被改变。

```go
package main

import "fmt"

func changeLocal(num [5]int) {
	num[0] = 55
	fmt.Println("inside function ", num)

}
func main() {
	num := [...]int{5, 6, 7, 8, 8}
	fmt.Println("before passing to function ", num)
	changeLocal(num) //num is passed by value
	fmt.Println("after passing to function ", num)
}
```

[Run in playground](https://play.golang.org/p/e3U75Q8eUZ)

在上面的程序中，第 13 行，数组 `num` 实际上是按值传递给函数 `changeLocal` 的，因此由于函数调用，数组 `num` 不会被更改。程序将打印：

```fallback
before passing to function  [5 6 7 8 8]
inside function  [55 6 7 8 8]
after passing to function  [5 6 7 8 8]
```

##### 数组的长度

数组的长度可以通过将数组作为参数传递给 `len` 函数来获得。

```go
package main

import "fmt"

func main() {
	a := [...]float64{67.7, 89.8, 21, 78}
	fmt.Println("length of a is",len(a))
	
}
```

[Run in playground](https://play.golang.org/p/UrIeNlS0RN)

上面程序的输出是：

```fallback
length of a is 4
```

##### 使用 `range` 遍历数组

可以使用 `for` 循环遍历数组的元素。

```go
package main

import "fmt"

func main() {
	a := [...]float64{67.7, 89.8, 21, 78}
	for i := 0; i < len(a); i++ { //looping from 0 to the length of the array
		fmt.Printf("%d th element of a is %.2f\n", i, a[i])
	}
}
```

[Run in playground](https://play.golang.org/p/80ejSTACO6)

上述程序使用 `for` 循环遍历数组的元素，从索引 `0` 开始，到数组的长度减一为止。这个程序能够正常运行，并将打印：

```fallback
0 th element of a is 67.70
1 th element of a is 89.80
2 th element of a is 21.00
3 th element of a is 78.00
```

Go 提供了一种更简洁的方式来遍历数组，即使用 `for` 循环的 **range** 形式。`range` 会返回数组元素的索引和值。让我们使用 `range` 重写上述代码，并计算数组所有元素的和。

```go
package main

import "fmt"

func main() {
	a := [...]float64{67.7, 89.8, 21, 78}
	sum := float64(0)
	for i, v := range a {//range returns both the index and value
		fmt.Printf("%d the element of a is %.2f\n", i, v)
		sum += v
	}
	fmt.Println("\nsum of all elements of a",sum)
}
```

[Run in playground](https://play.golang.org/p/Ji6FRon36m)

第 8 行 `for i, v := range a` 是上述程序中 `for` 循环的 range 形式。它将返回数组 `a` 中每个元素的索引和该索引处的值。我们打印这些值，并计算数组 `a` 中所有元素的和。该程序的 **输出** 是：

```fallback
the element of a is 67.70
the element of a is 89.80
the element of a is 21.00
the element of a is 78.00

sum of all elements of a 256.5
```

如果你只需要值并且想忽略索引，可以通过将索引替换为 `_` 空白标识符来实现。

```fallback
for _, v := range a { //ignores index
}
```

上述 `for` 循环忽略了索引。同样，值也可以被忽略。

##### 多维数组

到目前为止，我们创建的数组都是一维数组。实际上，可以创建多维数组。

```go
package main

import (
	"fmt"
)

func printarray(a [3][2]string) {
	for _, v1 := range a {
		for _, v2 := range v1 {
			fmt.Printf("%s ", v2)
		}
		fmt.Printf("\n")
	}
}

func main() {
	a := [3][2]string{
		{"lion", "tiger"},
		{"cat", "dog"},
		{"pigeon", "peacock"}, //this comma is necessary. The compiler will complain if you omit this comma
	}
	printarray(a)
	var b [3][2]string
	b[0][0] = "apple"
	b[0][1] = "samsung"
	b[1][0] = "microsoft"
	b[1][1] = "google"
	b[2][0] = "AT&T"
	b[2][1] = "T-Mobile"
	fmt.Printf("\n")
	printarray(b)
}
```

[Run in playground](https://play.golang.org/p/InchXI4yY8)

在上面的程序中，第 17 行使用简写语法声明了一个二维字符串数组 `a`。第 20 行末尾的逗号是必须的。这是因为词法分析器（lexer）会根据简单的规则自动插入分号。如果你有兴趣了解更多关于为什么需要分号的原因，可以阅读 [Effective Go](https://golang.org/doc/effective_go.html#semicolons)。

在第 23 行声明了另一个二维数组 `b`，并逐个为每个索引添加字符串。这是初始化二维数组的另一种方式。

第 7 行的 `printarray` 函数使用了两个 `for range` 循环来打印二维数组的内容。上述程序将打印：

```fallback
lion tiger 
cat dog 
pigeon peacock 

apple samsung 
microsoft google 
AT&T T-Mobile 
```

数组的内容就到这里。虽然数组看起来足够灵活，但它们有一个限制，即长度是固定的。无法增加数组的长度。在这里，**切片**（slices）发挥了作用。事实上，在 Go 中，切片比传统的数组更常见。

### 切片

切片是数组之上的一种方便、灵活且强大的封装。切片本身并不拥有任何数据，它们只是对现有数组的引用。

##### 创建切片

元素类型为 T 的切片表示为 `[]T`。

```go
package main

import (
	"fmt"
)

func main() {
	a := [5]int{76, 77, 78, 79, 80}
	var b []int = a[1:4] //creates a slice from a[1] to a[3]
	fmt.Println(b)
}
```

[Run in playground](https://play.golang.org/p/Za6w5eubBB)

语法 `a[start:end]` 创建一个从数组 `a` 中索引 `start` 到索引 `end - 1` 的切片。所以在上述程序的第 9 行，`a[1:4]` 创建了数组 `a` 从索引 1 到索引 3 的切片表示。因此，切片 `b` 的值是 `[77 78 79]`。

让我们看看另一种创建切片的方式。

```go
package main

import (
	"fmt"
)

func main() {
	c := []int{6, 7, 8} //creates an array and returns a slice reference
	fmt.Println(c)
}
```

[Run in playground](https://go.dev/play/p/anQIndv7Sm6)

在上述程序的第 8 行，`c := []int{6, 7, 8}` 创建了一个包含 3 个整数的数组，并返回一个切片引用，该引用存储在 `c` 中。

##### 修改切片

切片本身不拥有任何数据，它只是对底层数组的一个表示。对切片所做的任何修改都会反映在底层数组中。

```go
package main

import (  
    "fmt"
)

func main() {  
    darr := [...]int{57, 89, 90, 82, 100, 78, 67, 69, 59}
    dslice := darr[2:5]
    fmt.Println("array before",darr)
    for i := range dslice {
        dslice[i]++
    }
    fmt.Println("array after",darr) 
}
```

[Run in playground](https://play.golang.org/p/6FinudNf1k)

在上述程序的第 9 行，我们从数组的索引 2、3、4 创建了 `dslice`。`for` 循环将这些索引中的值增加了 1。当我们在 `for` 循环后打印数组时，可以看到切片的更改反映在了数组中。程序的输出是：

```fallback
array before [57 89 90 82 100 78 67 69 59]
array after [57 89 91 83 101 78 67 69 59]
```

当多个切片共享相同的底层数组时，每个切片所做的更改都会反映在该数组中。

```go
package main

import (
	"fmt"
)

func main() {
	numa := [3]int{78, 79 ,80}
	nums1 := numa[:] //creates a slice which contains all elements of the array
	nums2 := numa[:]
	fmt.Println("array before change 1",numa)
	nums1[0] = 100
	fmt.Println("array after modification to slice nums1", numa)
	nums2[1] = 101
	fmt.Println("array after modification to slice nums2", numa)
}
```

[Run in playground](https://play.golang.org/p/mdNi4cs854)

在第 9 行，`numa[:]` 中缺少起始值和结束值。默认情况下，起始值为 `0`，结束值为 `len(numa)`。因此，两个切片 `nums1` 和 `nums2` 共享相同的数组。程序的输出是：

```fallback
array before change 1 [78 79 80]
array after modification to slice nums1 [100 79 80]
array after modification to slice nums2 [100 101 80]
```

从输出可以看出，当切片共享相同的数组时，对切片所做的修改会反映在数组中。

##### 切片的长度和容量

切片的长度是切片中元素的数量。**切片的容量是从切片创建的索引开始，底层数组中元素的数量。**

让我们编写一些代码来更好地理解这一点。

```go
package main

import (
	"fmt"
)

func main() {
	fruitarray := [...]string{"apple", "orange", "grape", "mango", "water melon", "pine apple", "chikoo"}
	fruitslice := fruitarray[1:3]
	fmt.Printf("length of slice %d capacity %d", len(fruitslice), cap(fruitslice)) //length of fruitslice is 2 and capacity is 6
}
```

[Run in playground](https://play.golang.org/p/a1WOcdv827)

在上述程序中，`fruitslice` 是从 `fruitarray` 的索引 1 和 2 创建的。因此，`fruitslice` 的长度是 2。

`fruitarray` 的长度是 7，`fruitslice` 是从 `fruitarray` 的索引 1 创建的。因此，`fruitslice` 的容量是从索引 1 开始的 `fruitarray` 中的元素数量，即从 `orange` 开始，容量为 6。因此，`fruitslice` 的容量是 6。该程序（[程序链接](https://play.golang.org/p/a1WOcdv827)）打印出 **slice 的长度 2 容量 6**。

切片可以重新切片，直到其容量为止。超出此范围将导致程序抛出运行时错误。

```go
package main

import (  
    "fmt"
)

func main() {  
    fruitarray := [...]string{"apple", "orange", "grape", "mango", "water melon", "pine apple", "chikoo"}
    fruitslice := fruitarray[1:3]
    fmt.Printf("length of slice %d capacity %d\n", len(fruitslice), cap(fruitslice)) //length of is 2 and capacity is 6
    fruitslice = fruitslice[:cap(fruitslice)] //re-slicing fruitslice till its capacity
    fmt.Println("After re-slicing length is",len(fruitslice), "and capacity is",cap(fruitslice))
}
```

[Run in playground](https://go.dev/play/p/GI1gkZReeya)

在上述程序的第 11 行，`fruitslice` 被重新切片，直到其容量为止。该程序的输出是：

```fallback
length of slice 2 capacity 6
After re-slicing length is 6 and capacity is 6
```

##### 使用 `make` 创建切片

`func make([]T, len, cap) []T` 可以通过传递类型、长度和容量来创建一个切片。容量参数是可选的，默认为长度。`make` 函数创建一个数组并返回一个切片引用。

```go
package main

import (
	"fmt"
)

func main() {
	i := make([]int, 5, 5)
	fmt.Println(i)
}
```

[Run in playground](https://play.golang.org/p/M4OqxzerxN)

当使用 `make` 创建切片时，默认情况下切片中的值会被初始化为零。上述程序的输出将是 `[0 0 0 0 0]`。

##### 向切片追加元素

正如我们所知道的，数组的长度是固定的，不能增加。而切片是动态的，可以使用 `append` 函数向切片中追加新元素。`append` 函数的定义是 `func append(s []T, x ...T) []T`。

**x ...T** 在函数定义中表示该函数接受可变数量的参数。这样的函数被称为 [变参函数](../【GolangBot】12-可变参数函数)。

不过，有一个问题可能困扰你。如果切片是由数组支持的，而数组本身是固定长度的，那么切片是如何具有动态长度的呢？实际上，发生的情况是，当向切片中追加新元素时，首先会创建一个新的数组。现有数组的元素会被复制到这个新数组中，并且返回一个指向新数组的切片引用。新切片的容量将是旧切片容量的两倍。很酷，对吧？下面的程序将使这一点更加清晰。

```go
package main

import (
	"fmt"
)

func main() {
	cars := []string{"Ferrari", "Honda", "Ford"}
	fmt.Println("cars:", cars, "has old length", len(cars), "and capacity", cap(cars)) //capacity of cars is 3
	cars = append(cars, "Toyota")
	fmt.Println("cars:", cars, "has new length", len(cars), "and capacity", cap(cars)) //capacity of cars is doubled to 6
}
```

[Run in playground](https://play.golang.org/p/VUSXCOs1CF)

在上述程序中，`cars` 的初始容量为 3。在第 10 行，我们向 `cars` 中追加了一个新元素，并将 `append(cars, "Toyota")` 返回的切片重新赋值给 `cars`。现在，`cars` 的容量翻倍，变成了 6。该程序的输出是：

```fallback
cars: [Ferrari Honda Ford] has old length 3 and capacity 3
cars: [Ferrari Honda Ford Toyota] has new length 4 and capacity 6
```

切片类型的零值是 `nil`。一个 `nil` 切片的长度和容量都是 0。可以使用 `append` 函数向一个 `nil` 切片追加值。

```go
package main

import (
	"fmt"
)

func main() {
	var names []string //zero value of a slice is nil
	if names == nil {
		fmt.Println("slice is nil going to append")
		names = append(names, "John", "Sebastian", "Vinay")
		fmt.Println("names contents:",names)
	}
}
```

[Run in playground](https://play.golang.org/p/x_-4XAJHbM)

在上述程序中，`names` 是 `nil`，我们向 `names` 中追加了 3 个字符串。该程序的输出是：

```fallback
slice is nil going to append
names contents: [John Sebastian Vinay]
```

也可以使用 `...` 操作符将一个切片追加到另一个切片。你可以在 [变参函数](../【GolangBot】12-可变参数函数) 教程中了解更多关于这个操作符的信息。

```go
package main

import (
	"fmt"
)

func main() {
	veggies := []string{"potatoes","tomatoes","brinjal"}
	fruits := []string{"oranges","apples"}
	food := append(veggies, fruits...)
	fmt.Println("food:",food)
}
```

[Run in playground](https://play.golang.org/p/UnHOH_u6HS)

在上述程序的第 10 行，`food` 是通过将 `fruits` 追加到 `veggies` 上创建的。程序的输出是 `food: [potatoes tomatoes brinjal oranges apples]`。

##### 将切片传递给函数

切片可以被认为是通过一个结构类型在内部表示的。它的结构如下所示：

```fallback
type slice struct {
    Length        int
    Capacity      int
    ZerothElement *byte
}
```

一个切片包含长度、容量以及指向数组第零元素的指针。当切片传递给函数时，尽管是按值传递，但指针变量仍然指向相同的底层数组。因此，当切片作为参数传递给函数时，函数内部所做的更改在函数外部也可见。让我们编写一个程序来验证这一点。

```go
package main

import (
	"fmt"
)

func subtactOne(numbers []int) {
	for i := range numbers {
		numbers[i] -= 2
	}

}
func main() {
	nos := []int{8, 7, 6}
	fmt.Println("slice before function call", nos)
	subtactOne(nos)                               //function modifies the slice
	fmt.Println("slice after function call", nos) //modifications are visible outside
}
```

[Run in playground](https://play.golang.org/p/IzqDihNifq)

在上述程序的第 17 行，函数调用将切片的每个元素都减少了 2。当函数调用之后打印切片时，这些更改是可见的。如果你还记得，这与数组不同，因为数组在函数内部所做的更改在函数外部是不可见的。该[程序](https://play.golang.org/p/bWUb6R-1bS)的输出是：

```fallback
slice before function call [8 7 6]
slice after function call [6 5 4]
```

##### 多维切片

与数组类似，切片也可以具有多个维度。

```go
package main

import (
	"fmt"
)


func main() {
 	pls := [][]string {
			{"C", "C++"},
			{"JavaScript"},
			{"Go", "Rust"},
			}
	for _, v1 := range pls {
		for _, v2 := range v1 {
			fmt.Printf("%s ", v2)
		}
		fmt.Printf("\n")
	}
}
```

[Run in playground](https://play.golang.org/p/--p1AvNGwN)

程序的输出是，

```fallback
C C++ 
JavaScript 
Go Rust 
```

##### 内存优化

切片持有对底层数组的引用。只要切片存在于内存中，数组就无法被垃圾回收。当涉及到内存管理时，这可能会成为一个问题。假设我们有一个非常大的数组，并且只关心处理其中的一小部分。接下来，我们从该数组创建一个切片并开始处理该切片。这里需要注意的一个重要问题是，由于切片引用了数组，数组仍然会保留在内存中。

解决这个问题的一种方法是使用 [copy](https://golang.org/pkg/builtin/#copy) 函数 `func copy(dst, src []T) int` 来创建该切片的副本。通过这种方式，我们可以使用新的切片，并且原始数组可以被垃圾回收。

```go
package main

import (
	"fmt"
)

func countries() []string { 
    countries := []string{"USA", "Singapore", "Germany", "India", "Australia"}
	neededCountries := countries[:len(countries)-2]
	countriesCpy := make([]string, len(neededCountries))
	copy(countriesCpy, neededCountries) //copies neededCountries to countriesCpy
	return countriesCpy
}
func main() {
	countriesNeeded := countries()
	fmt.Println(countriesNeeded)
}
```

[Run in playground](https://play.golang.org/p/35ayYBhcDE)

在上述程序的第 9 行，`neededCountries := countries[:len(countries)-2]` 创建了一个切片，去除了 `countries` 的最后两个元素。第 11 行将 `neededCountries` 复制到 `countriesCpy` 中，并在下一行将其返回。现在，`countries` 数组可以被垃圾回收，因为 `neededCountries` 不再被引用。

我已经将我们讨论过的所有概念汇总到一个程序中。你可以从 [GitHub](https://github.com/golangbot/arraysandslices) 下载它。



**下一篇教程 - [变参函数](../【GolangBot】12-可变参数函数)**