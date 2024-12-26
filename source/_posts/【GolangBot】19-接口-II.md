---
title: 【GolangBot】19-接口 II
date: 2024-12-24 16:52:49
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---


欢迎来到[Golang教程系列](../golangbot/)的第19篇教程。这是接口教程的第二部分。如果你错过了第一部分，可以从[这里](../【GolangBot】18-接口-I/)阅读

### 使用指针接收器vs值接收器来实现接口

我们在[第一部分](../【GolangBot】18-接口-I)中讨论的所有示例接口都是使用值接收器实现的。使用指针接收器也可以实现接口。在使用指针接收器实现接口时需要注意一个细节。让我们通过以下程序来理解这一点。

```go
package main

import "fmt"

type Describer interface {
	Describe()
}
type Person struct {
	name string
	age  int
}

func (p Person) Describe() { //implemented using value receiver
	fmt.Printf("%s is %d years old\n", p.name, p.age)
}

type Address struct {
	state   string
	country string
}

func (a *Address) Describe() { //implemented using pointer receiver
	fmt.Printf("State %s Country %s", a.state, a.country)
}

func main() {
	var d1 Describer
	p1 := Person{"Sam", 25}
	d1 = p1
	d1.Describe()
	p2 := Person{"James", 32}
	d1 = &p2
	d1.Describe()

	var d2 Describer
	a := Address{"Washington", "USA"}

	/* compilation error if the following line is
	   uncommented
	   cannot use a (type Address) as type Describer
	   in assignment: Address does not implement
	   Describer (Describe method has pointer
	   receiver)
	*/
	//d2 = a

	d2 = &a //This works since Describer interface
	//is implemented by Address pointer in line 22
	d2.Describe()

}
```

[在playground中运行](https://play.golang.org/p/IzspYiAQ82)

在上面的程序中，`Person`结构体在第13行使用值接收器实现了`Describer`接口。

正如我们在[方法](../【GolangBot】17-方法/#方法中的值接收者vs函数中的值参数)的讨论中已经了解到的，具有值接收器的方法既接受指针接收器也接受值接收器。*对于任何值或可以解引用的值，调用值方法都是合法的。*

*p1*是`Person`类型的值，它在第29行被赋值给`d1`。`Person`实现了`Describer`接口，因此第30行将打印`Sam is 25 years old`。

同样，在第32行`d1`被赋值为`&p2`，因此第33行将打印`James is 32 years old`。很棒:)。

`Address`结构体在第22行使用指针接收器实现了`Describer`接口。

如果取消注释上面程序的第45行，我们将得到编译错误**main.go:42: cannot use a (type Address) as type Describer in assignment: Address does not implement Describer (Describe method has pointer receiver)**。这是因为，`Describer`接口在第22行使用Address指针接收器实现，而我们试图赋值`a`是一个值类型，它没有实现`Describer`接口。这肯定会让你感到惊讶，因为我们之前学到[方法](../【GolangBot】17-方法/#方法中的指针接收者vs函数中的指针参数)中带有指针接收器的方法将同时接受指针和值接收器。那为什么第45行的代码不能工作呢？

**原因是只有在已经是指针或者可以获取地址的情况下，才可以合法地调用指针值方法。存储在接口中的具体值是不可寻址的，因此编译器无法自动获取第45行中`a`的地址，所以这段代码失败了。**

第47行可以工作，因为我们将`a`的地址`&a`赋值给`d2`。

程序的其余部分是不言自明的。这个程序将打印：

```fallback
Sam is 25 years old
James is 32 years old
State Washington Country USA
```

### 实现多个接口

一个类型可以实现多个接口。让我们看看下面的程序是如何实现的。

```go
package main

import (
	"fmt"
)

type SalaryCalculator interface {
	DisplaySalary()
}

type LeaveCalculator interface {
	CalculateLeavesLeft() int
}

type Employee struct {
	firstName string
	lastName string
	basicPay int
	pf int
	totalLeaves int
	leavesTaken int
}

func (e Employee) DisplaySalary() {
	fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {
	return e.totalLeaves - e.leavesTaken
}

func main() {
	e := Employee {
		firstName: "Naveen",
		lastName: "Ramanathan",
		basicPay: 5000,
		pf: 200,
		totalLeaves: 30,
		leavesTaken: 5,
	}
	var s SalaryCalculator = e
	s.DisplaySalary()
	var l LeaveCalculator = e
	fmt.Println("\nLeaves left =", l.CalculateLeavesLeft())
}
```

[在playground中运行](https://play.golang.org/p/DJxS5zxBcV)

上面的程序在第7行和第11行分别声明了两个接口`SalaryCalculator`和`LeaveCalculator`。

`Employee`结构体在第15行定义，它在第24行提供了`SalaryCalculator`接口的`DisplaySalary`方法实现，在第28行提供了`LeaveCalculator`接口的`CalculateLeavesLeft`方法实现。现在`Employee`同时实现了`SalaryCalculator`和`LeaveCalculator`接口。

在第41行，我们将`e`赋值给`SalaryCalculator`接口类型的变量，在第43行，我们将相同的变量`e`赋值给`LeaveCalculator`类型的变量。这是可能的，因为类型为`Employee`的`e`同时实现了`SalaryCalculator`和`LeaveCalculator`接口。

这个程序输出：

```fallback
Naveen Ramanathan has salary $5200
Leaves left = 25
```

### 嵌入接口

虽然go不提供继承，但可以通过嵌入其他接口来创建新的接口。

让我们看看如何实现这一点。

```go
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    DisplaySalary()
}

type LeaveCalculator interface {  
    CalculateLeavesLeft() int
}

type EmployeeOperations interface {
	SalaryCalculator
	LeaveCalculator
}

type Employee struct {  
    firstName string
    lastName string
    basicPay int
    pf int
    totalLeaves int
    leavesTaken int
}

func (e Employee) DisplaySalary() {  
    fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {  
    return e.totalLeaves - e.leavesTaken
}

func main() {  
    e := Employee {
        firstName: "Naveen",
        lastName: "Ramanathan",
        basicPay: 5000,
        pf: 200,
        totalLeaves: 30,
        leavesTaken: 5,
    }
    var empOp EmployeeOperations = e
    empOp.DisplaySalary()
    fmt.Println("\nLeaves left =", empOp.CalculateLeavesLeft())
}
```

[在playground中运行](https://play.golang.org/p/Hia7D-WbZp)

上面程序第15行的*EmployeeOperations*接口是通过嵌入*SalaryCalculator*和*LeaveCalculator*接口创建的。

如果一个类型为*SalaryCalculator*和*LeaveCalculator*接口中的方法提供了方法定义，那么就说这个类型实现了`EmployeeOperations`接口。

`Employee`结构体实现了`EmployeeOperations`接口，因为它分别在第29行和第33行为`DisplaySalary`和`CalculateLeavesLeft`方法提供了定义。

在第46行，`Employee`类型的`e`被赋值给`EmployeeOperations`类型的`empOp`。在接下来的两行中，在`empOp`上调用了`DisplaySalary()`和`CalculateLeavesLeft()`方法。

这个程序将输出：

```fallback
Naveen Ramanathan has salary $5200
Leaves left = 25
```

### 接口的零值

接口的零值是nil。一个nil接口的底层值和具体类型都是nil。

```go
package main

import "fmt"

type Describer interface {
	Describe()
}

func main() {
	var d1 Describer
	if d1 == nil {
		fmt.Printf("d1 is nil and has type %T value %v\n", d1, d1)
	}
}
```

[在playground中运行](https://play.golang.org/p/vwYHC6Y78H)

上面程序中的*d1*是`nil`，这个程序将输出：

```fallback
d1 is nil and has type <nil> value <nil>
```

如果我们试图在`nil`接口上调用方法，程序将会panic，因为`nil`接口既没有底层值也没有具体类型。

```go
package main

type Describer interface {
	Describe()
}

func main() {
	var d1 Describer
	d1.Describe()
}
```

[在playground中运行](https://play.golang.org/p/rM-rY0uGTI)

由于上面程序中的`d1`是`nil`，这个程序将会panic，运行时错误为**panic: runtime error: invalid memory address or nil pointer dereference [signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 pc=0xc8527]"**


**下一篇教程 - [并发简介](../【GolangBot】20-并发介绍/)**

