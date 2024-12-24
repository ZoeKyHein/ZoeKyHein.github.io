---
title: 【GolangBot】18-接口 I
date: 2024-12-24 14:52:41
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---

欢迎来到 [Go 语言教程系列](../golangbot/) 的第 18 讲。这是接口教程两个部分中的第一部分。

### 什么是接口？

**在 Go 中，接口是一组方法签名的集合。当一个类型为接口中的所有方法提供定义时，就称该类型实现了这个接口。** 这与面向对象编程世界很相似。接口指定了一个类型应该有哪些[方法](https://golangbot.com/methods/)，而类型决定如何实现这些方法。

例如 *WashingMachine* 可以是一个带有[方法](../【GolangBot】17-方法/)签名 *Cleaning()* 和 *Drying()* 的接口。任何提供了 *Cleaning()* 和 *Drying()* 方法定义的类型都被认为实现了 *WashingMachine* 接口。

### 声明和实现接口

让我们直接通过一个创建和实现接口的程序来学习。

```go
package main

import (
    "fmt"
)

//interface definition
type VowelsFinder interface {
    FindVowels() []rune
}

type MyString string

//MyString implements VowelsFinder
func (ms MyString) FindVowels() []rune {
    var vowels []rune
    for _, rune := range ms {
        if rune == 'a' || rune == 'e' || rune == 'i' || rune == 'o' || rune == 'u' {
            vowels = append(vowels, rune)
        }
    }
    return vowels
}

func main() {
    name := MyString("Sam Anderson")
    var v VowelsFinder
    v = name // possible since MyString implements VowelsFinder
    fmt.Printf("Vowels are %c", v.FindVowels())

}
```

[在 playground 中运行](https://play.golang.org/p/F-T3S_wNNB)

上述程序第 8 行创建了一个名为 `VowelsFinder` 的接口类型，该接口有一个方法 `FindVowels() []rune`。

接下来一行创建了一个 `MyString` 类型。

**在第 15 行，我们为接收者类型 `MyString` 添加了方法 `FindVowels() []rune`。现在可以说 `MyString` 实现了接口 `VowelsFinder`。** 这与其他语言（如 Java）有很大不同，在 Java 中，一个类必须使用 `implements` 关键字显式声明它实现了一个接口。**在 Go 中不需要这样做，如果一个类型包含了接口声明的所有方法，那么这个类型就隐式地实现了该接口。**

在第 28 行，我们将 `MyString` 类型的 `name` 赋值给 VowelsFinder 类型的 v。这是可能的，因为 `MyString` 实现了 `VowelsFinder` 接口。下一行的 `v.FindVowels()` 调用了 `MyString` 类型的 FindVowels 方法，并打印出字符串 `Sam Anderson` 中的所有元音字母。该程序输出：

```
Vowels are [a e o]
```

恭喜！你已经创建并实现了你的第一个接口。

### 接口的实际应用

上面的例子教会了我们如何创建和实现接口，但它并没有真正展示接口的实际用途。在上面的程序中，如果我们使用 `name.FindVowels()` 而不是 `v.FindVowels()`，程序也能工作，接口就没有什么用了。

现在让我们看看接口的一个实际应用。

我们将编写一个简单的程序，根据员工的个人薪资计算公司的总支出。为了简单起见，我们假设所有费用都以美元计算。

```go
package main

import (
    "fmt"
)

type SalaryCalculator interface {
    CalculateSalary() int
}

type Permanent struct {
    empId    int
    basicpay int
    pf       int
}

type Contract struct {
    empId    int
    basicpay int
}

//salary of permanent employee is the sum of basic pay and pf
func (p Permanent) CalculateSalary() int {
    return p.basicpay + p.pf
}

//salary of contract employee is the basic pay alone
func (c Contract) CalculateSalary() int {
    return c.basicpay
}

/*
total expense is calculated by iterating through the SalaryCalculator slice and summing
the salaries of the individual employees
*/
func totalExpense(s []SalaryCalculator) {
    expense := 0
    for _, v := range s {
        expense = expense + v.CalculateSalary()
    }
    fmt.Printf("Total Expense Per Month $%d", expense)
}

func main() {
    pemp1 := Permanent{
        empId:    1,
        basicpay: 5000,
        pf:       20,
    }
    pemp2 := Permanent{
        empId:    2,
        basicpay: 6000,
        pf:       30,
    }
    cemp1 := Contract{
        empId:    3,
        basicpay: 3000,
    }
    employees := []SalaryCalculator{pemp1, pemp2, cemp1}
    totalExpense(employees)
}
```

[在 playground 中运行](https://play.golang.org/p/3DZQH_Xh_Pl)

上述程序第 7 行声明了一个带有单个方法 `CalculateSalary() int` 的 `SalaryCalculator` 接口。

公司有两种类型的员工，`Permanent` 和 `Contract`，分别由第 11 行和第 17 行的[结构体](../【GolangBot】16-结构体/)定义。永久员工的薪资是 `basicpay` 和 `pf` 的总和，而合同员工的薪资只有 `basicpay`。这在第 23 行和第 28 行的相应 `CalculateSalary` 方法中得到了体现。通过声明这个方法，`Permanent` 和 `Contract` 结构体现在都实现了 `SalaryCalculator` 接口。

第 36 行声明的 `totalExpense` [函数](../【GolangBot】6-函数/)展现了接口的优雅之处。这个方法接受一个 SalaryCalculator 接口的[切片](../【GolangBot】11-数组和切片/) `[]SalaryCalculator` 作为参数。在第 59 行，我们向 `totalExpense` 函数传递了一个包含 `Permanent` 和 `Contract` 类型的切片。`totalExpense` 函数通过调用相应类型的 `CalculateSalary` 方法来计算支出。这在第 39 行完成。

程序输出：

```
Total Expense Per Month $14050
```

这样做的最大优势是 `totalExpense` 可以扩展到任何新的员工类型，而无需任何代码更改。假设公司添加了一个具有不同薪资结构的新员工类型 `Freelancer`。这个 `Freelancer` 可以直接传入到 `totalExpense` 的切片参数中，而不需要对 `totalExpense` 函数进行任何代码更改。这个方法将按预期工作，因为 `Freelancer` 也将实现 `SalaryCalculator` 接口 :)。

让我们修改这个程序并添加新的 `Freelancer` 员工。自由职业者的薪资是每小时工资和工作总小时数的乘积。

```go
package main

import (
    "fmt"
)

type SalaryCalculator interface {
    CalculateSalary() int
}

type Permanent struct {
    empId    int
    basicpay int
    pf       int
}

type Contract struct {
    empId    int
    basicpay int
}

type Freelancer struct {
    empId       int
    ratePerHour int
    totalHours  int
}

//salary of permanent employee is sum of basic pay and pf
func (p Permanent) CalculateSalary() int {
    return p.basicpay + p.pf
}

//salary of contract employee is the basic pay alone
func (c Contract) CalculateSalary() int {
    return c.basicpay
}

//salary of freelancer
func (f Freelancer) CalculateSalary() int {
    return f.ratePerHour * f.totalHours
}

/*
total expense is calculated by iterating through the SalaryCalculator slice and summing
the salaries of the individual employees
*/
func totalExpense(s []SalaryCalculator) {
    expense := 0
    for _, v := range s {
        expense = expense + v.CalculateSalary()
    }
    fmt.Printf("Total Expense Per Month $%d", expense)
}

func main() {
    pemp1 := Permanent{
        empId:    1,
        basicpay: 5000,
        pf:       20,
    }
    pemp2 := Permanent{
        empId:    2,
        basicpay: 6000,
        pf:       30,
    }
    cemp1 := Contract{
        empId:    3,
        basicpay: 3000,
    }
    freelancer1 := Freelancer{
        empId:       4,
        ratePerHour: 70,
        totalHours:  120,
    }
    freelancer2 := Freelancer{
        empId:       5,
        ratePerHour: 100,
        totalHours:  100,
    }
    employees := []SalaryCalculator{pemp1, pemp2, cemp1, freelancer1, freelancer2}
    totalExpense(employees)
}
```

[在 playground 中运行](https://play.golang.org/p/J48P5g8ArLn)

我们在第 22 行添加了 `Freelancer` 结构体，并在第 39 行声明了 `CalculateSalary` 方法。由于 `Freelancer` 结构体也实现了 `SalaryCalculator` 接口，因此不需要在 `totalExpense` 方法中进行任何代码更改。我们在 `main` 方法中添加了几个 `Freelancer` 员工。这个程序打印：

```fallback
Total Expense Per Month $32450
```

### 接口的内部表示

接口在内部可以被认为是由一个元组 `(type, value)` 表示的。`type` 是接口的底层具体类型，而 `value` 持有具体类型的值。

让我们通过一个程序来更好地理解这一点。

```go
package main

import (
    "fmt"
)

type Worker interface {
    Work()
}

type Person struct {
    name string
}

func (p Person) Work() {
    fmt.Println(p.name, "is working")
}

func describe(w Worker) {
    fmt.Printf("Interface type %T value %v\n", w, w)
}

func main() {
    p := Person{
        name: "Naveen",
    }
    var w Worker = p
    describe(w)
    w.Work()
}
```

[在 playground 中运行](https://play.golang.org/p/kweC7_oELzE)

*Worker* 接口有一个方法 `Work()`，而 *Person* 结构体类型实现了该接口。在第 27 行，我们将 `Person` 类型的[变量](../【GolangBot】3-变量/) `p` 赋值给 `Worker` 类型的 `w`。现在 `w` 的具体类型是 `Person`，它包含一个 `name` 字段为 `Naveen` 的 `Person`。第 17 行的 `describe` 函数打印了接口的值和具体类型。这个程序输出：

```fallback
Interface type main.Person value {Naveen}
Naveen is working
```

我们将在接下来的章节中讨论如何提取接口的底层值。

### 空接口

**没有任何方法的接口称为空接口。它表示为 `interface{}`。** 由于空接口没有方法，所有类型都实现了空接口。

```go
package main

import (
    "fmt"
)

func describe(i interface{}) {
    fmt.Printf("Type = %T, value = %v\n", i, i)
}

func main() {
    s := "Hello World"
    describe(s)
    i := 55
    describe(i)
    strt := struct {
        name string
    }{
        name: "Naveen R",
    }
    describe(strt)
}
```

[在 playground 中运行](https://play.golang.org/p/Fm5KescoJb)

在上面的程序中，第 7 行的 `describe(i interface{})` 函数接受一个空接口作为参数，因此可以传递任何类型。

我们分别在第 13、15 和 21 行向 `describe` 函数传递了 `string`、`int` 和 `struct` 类型。这个程序打印：

```fallback
Type = string, value = Hello World
Type = int, value = 55
Type = struct { name string }, value = {Naveen R}
```

### 类型断言

类型断言用于提取接口的底层值。

**i.(T)** 是用来获取接口 `i` 的底层值的语法，其中 `T` 是接口的具体类型。

通过一个程序来演示类型断言。

```go
package main

import (
	"fmt"
)

func assert(i interface{}) {
	s := i.(int) //get the underlying int value from i
	fmt.Println(s)
}
func main() {
	var s interface{} = 56
	assert(s)
}
```

[Run in playground](https://play.golang.org/p/YstKXEeSBL)

如果在上面的程序中，`i` 的具体类型不是 `int`，程序会发生什么呢？让我们来看看。

如果我们尝试断言 `i` 为一个不同的类型（例如 `float64` 或 `string`），而 `i` 实际上并不是该类型，Go 会抛出一个运行时错误，称为 **panic**。这种情况称为类型断言失败。

我们可以修改程序，让它在类型不匹配时发生错误。

```go
package main

import (
	"fmt"
)

func assert(i interface{}) {
	s := i.(int) 
	fmt.Println(s)
}
func main() {
	var s interface{} = "Steven Paul"
	assert(s)
}
```

[Run in playground](https://play.golang.org/p/88KflSceHK)

在上述程序中，我们将具体类型为 `string` 的 `s` 传递给 `assert` 函数，该函数尝试从中提取 `int` 值。此程序会因 `panic: interface conversion: interface {} is string, not int` 错误而崩溃。

为了解决这个问题，我们可以使用以下语法：

```fallback
v, ok := i.(T)
```

如果 `i` 的具体类型是 `T`，那么 `v` 将具有 `i` 的底层值，`ok` 将为 `true`。

如果 `i` 的具体类型不是 `T`，那么 `ok` 将为 `false`，`v` 将具有类型 `T` 的零值，并且 **程序不会崩溃**。

```go
package main

import (
	"fmt"
)

func assert(i interface{}) {
	v, ok := i.(int)
	fmt.Println(v, ok)
}
func main() {
	var s interface{} = 56
	assert(s)
	var i interface{} = "Steven Paul"
	assert(i)
}
```

[Run in playground](https://play.golang.org/p/0sB-KlVw8A)

当 `Steven Paul` 被传递给 `assert` 函数时，由于 `i` 的具体类型不是 `int`，所以 `ok` 将为 `false`，`v` 将具有 `int` 类型的零值 0。该程序将打印：

```fallback
56 true
0 false
```

### 类型switch

**类型switch用于将接口的具体类型与多个类型进行比较，这些类型通过不同的 case 语句指定。它类似于 [switch case](../【GolangBot】10-switch语句/)，唯一的区别是 case 中指定的是类型而不是值。**

类型开关的语法与类型断言类似。在类型断言中，语法是 `i.(T)`，而在类型开关中，类型 `T` 应该用关键字 `type` 替换。让我们看看下面的程序如何工作。

```go
package main

import (
	"fmt"
)

func findType(i interface{}) {
	switch i.(type) {
	case string:
		fmt.Printf("I am a string and my value is %s\n", i.(string))
	case int:
		fmt.Printf("I am an int and my value is %d\n", i.(int))
	default:
		fmt.Printf("Unknown type\n")
	}
}
func main() {
	findType("Naveen")
	findType(77)
	findType(89.98)
}
```

[Run in playground](https://play.golang.org/p/XYPDwOvoCh)

在上面的程序中，**第8行**中的 `switch i.(type)` 指定了一个类型开关。每个 `case` 语句将接口 `i` 的具体类型与特定类型进行比较。如果某个 `case` 匹配，则打印相应的语句。该程序输出：

```fallback
I am a string and my value is Naveen
I am an int and my value is 77
Unknown type
```

**第20行的\*89.98\*是`float64`类型，不匹配任何`case`，因此最后一行打印`Unknown type`。**

**也可以将类型与接口进行比较。如果我们有一个类型，并且这个类型实现了某个接口，那么可以将该类型与它实现的接口进行比较。**

让我们编写一个程序来进一步说明。

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

func (p Person) Describe() {
	fmt.Printf("%s is %d years old", p.name, p.age)
}

func findType(i interface{}) {
	switch v := i.(type) {
	case Describer:
		v.Describe()
	default:
		fmt.Printf("unknown type\n")
	}
}

func main() {
	findType("Naveen")
	p := Person{
		name: "Naveen R",
		age:  25,
	}
	findType(p)
}
```

[Run in playground](https://play.golang.org/p/o6aHzIz4wC)

在上述程序中，`Person` 结构体实现了 `Describer` 接口。在第19行的 `case` 语句中，`v` 被与 `Describer` 接口类型进行比较。由于 `p` 实现了 `Describer` 接口，因此此 `case` 被满足并调用了 `Describe()` 方法。

该程序输出：

```fallback
unknown type
Naveen R is 25 years old
```

**Next tutorial - [Interfaces - II](https://golangbot.com/interfaces-part-2/)**
