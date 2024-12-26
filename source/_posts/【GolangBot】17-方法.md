---
title: 【GolangBot】17-方法
date: 2024-12-24 12:52:41
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---


欢迎来到我们的 [Golang 教程系列](../golangbot/)的第17篇教程。

### 简介

方法实际上就是一个带有特殊接收者类型的函数，这个接收者类型位于`func`关键字和方法名之间。接收者可以是结构体类型或非结构体类型。

方法声明的语法如下所示：

```go
func (t Type) methodName(parameter list) {
}
```

上面的代码片段创建了一个名为`methodName`的方法，其接收者类型为`Type`。`t`被称为接收者，可以在方法内部访问它。

### 方法示例

让我们编写一个简单的程序，在结构体类型上创建一个方法并调用它。

```go
package main

import (
    "fmt"
)

type Employee struct {
    name     string
    salary   int
    currency string
}

/*
 displaySalary() method has Employee as the receiver type
*/
func (e Employee) displaySalary() {
    fmt.Printf("Salary of %s is %s%d", e.name, e.currency, e.salary)
}

func main() {
    emp1 := Employee {
        name:     "Sam Adolf",
        salary:   5000,
        currency: "$",
    }
    emp1.displaySalary() //Calling displaySalary() method of Employee type
}
```

[Run program in playground](https://play.golang.org/p/rRsI_sWAOZ)

在上面程序的第16行中，我们在`Employee`结构体类型上创建了一个方法`displaySalary`。`displaySalary()`方法在其内部可以访问接收者`e`。在第17行中，我们使用接收者`e`打印员工的姓名、货币和工资。

在第26行中，我们使用语法`emp1.displaySalary()`调用了该方法。

这个程序打印输出：`Salary of Sam Adolf is $5000`。

### 方法与函数的比较

上面的程序可以只使用函数而不使用方法重写。

```go
package main

import (
    "fmt"
)

type Employee struct {
    name     string
    salary   int
    currency string
}

/*
 displaySalary() method converted to function with Employee as parameter
*/
func displaySalary(e Employee) {
    fmt.Printf("Salary of %s is %s%d", e.name, e.currency, e.salary)
}

func main() {
    emp1 := Employee{
        name:     "Sam Adolf",
        salary:   5000,
        currency: "$",
    }
    displaySalary(emp1)
}
```

[Run program in playground](https://play.golang.org/p/dFwObgCUU0)

在上面的程序中，`displaySalary`方法被转换为一个函数，并且`Employee`结构体作为参数传递给它。这个程序也产生完全相同的输出：`Salary of Sam Adolf is $5000`。

那么既然我们可以用函数写出相同的程序，为什么还要有方法呢？这里有几个原因。让我们一个一个来看。

- [Go不是一个纯面向对象的编程语言](https://go.dev/doc/faq#Is_Go_an_object-oriented_language)，它不支持类。因此，在类型上定义方法是实现类似类的行为的一种方式。方法允许对与类型相关的行为进行逻辑分组，类似于类。在上面的示例程序中，所有与`Employee`类型相关的行为都可以通过使用`Employee`接收者类型创建方法来分组。例如，我们可以添加诸如`calculatePension`、`calculateLeaves`等方法。
- 可以在不同的类型上定义相同名称的方法，而具有相同名称的函数是不允许的。假设我们有一个`Square`和`Circle`结构体。可以在`Square`和`Circle`上都定义一个名为`Area`的方法。这在下面的程序中演示：

```go
package main

import (
    "fmt"
    "math"
)

type Rectangle struct {
    length int
    width  int
}

type Circle struct {
    radius float64
}

func (r Rectangle) Area() int {
    return r.length * r.width
}

func (c Circle) Area() float64 {
    return math.Pi * c.radius * c.radius
}

func main() {
    r := Rectangle{
        length: 10,
        width:  5,
    }
    fmt.Printf("Area of rectangle %d\n", r.Area())
    c := Circle{
        radius: 12,
    }
    fmt.Printf("Area of circle %f", c.Area())
}
```

[Run program in playground](https://play.golang.org/p/0hDM3E3LiP)

这个程序打印输出：

```
Area of rectangle 50
Area of circle 452.389342
```

方法的这个特性用于实现接口。我们将在下一篇介绍[接口](../【GolangBot】18-接口-I/)的教程中详细讨论这一点。

### 指针接收者与值接收者

到目前为止，我们只看到了带有值接收者的方法。创建带有指针接收者的方法也是可能的。值接收者和指针接收者的区别在于，在带有指针接收者的方法内部所做的更改对调用者是可见的，而在值接收者中则不是这样。让我们通过一个程序来理解这一点。

```go
package main

import (
    "fmt"
)

type Employee struct {
    name string
    age  int
}

/*
Method with value receiver
*/
func (e Employee) changeName(newName string) {
    e.name = newName
}

/*
Method with pointer receiver
*/
func (e *Employee) changeAge(newAge int) {
    e.age = newAge
}

func main() {
    e := Employee{
        name: "Mark Andrew",
        age:  50,
    }
    fmt.Printf("Employee name before change: %s", e.name)
    e.changeName("Michael Andrew")
    fmt.Printf("\nEmployee name after change: %s", e.name)

    fmt.Printf("\n\nEmployee age before change: %d", e.age)
    (&e).changeAge(51)
    fmt.Printf("\nEmployee age after change: %d", e.age)
}
```

[Run program in playground](https://play.golang.org/p/tTO100HmUX)

在上面的程序中，`changeName`方法有一个值接收者`(e Employee)`，而`changeAge`方法有一个指针接收者`(e *Employee)`。在`changeName`内部对Employee结构体的`name`字段所做的更改对调用者不可见，因此在调用方法`e.changeName("Michael Andrew")`之前和之后，程序打印相同的名称。由于`changeAge`方法有一个指针接收者`(e *Employee)`，在调用方法`(&e).changeAge(51)`之后对`age`字段所做的更改对调用者是可见的。这个程序打印输出：

```
Employee name before change: Mark Andrew
Employee name after change: Mark Andrew

Employee age before change: 50
Employee age after change: 51
```

在上面程序的第36行中，我们使用`(&e).changeAge(51)`来调用`changeAge`方法。由于`changeAge`有一个指针接收者，我们使用了`(&e)`来调用该方法。这其实不是必需的，语言给了我们直接使用`e.changeAge(51)`的选项。`e.changeAge(51)`会被语言解释为`(&e).changeAge(51)`。

以下程序重写为使用`e.changeAge(51)`而不是`(&e).changeAge(51)`，它打印相同的输出：

```go
package main

import (
    "fmt"
)

type Employee struct {
    name string
    age  int
}

/*
Method with value receiver
*/
func (e Employee) changeName(newName string) {
    e.name = newName
}

/*
Method with pointer receiver
*/
func (e *Employee) changeAge(newAge int) {
    e.age = newAge
}

func main() {
    e := Employee{
        name: "Mark Andrew",
        age:  50,
    }
    fmt.Printf("Employee name before change: %s", e.name)
    e.changeName("Michael Andrew")
    fmt.Printf("\nEmployee name after change: %s", e.name)

    fmt.Printf("\n\nEmployee age before change: %d", e.age)
    e.changeAge(51)
    fmt.Printf("\nEmployee age after change: %d", e.age)
}
```

[Run program in playground](https://play.golang.org/p/nnXBsR3Uc8)

### 什么时候使用指针接收者，什么时候使用值接收者

通常，当在方法内部对接收者所做的更改需要对调用者可见时，可以使用指针接收者。

在复制数据结构代价较高的地方也可以使用指针接收者。考虑一个有很多字段的结构体。在方法中使用这个结构体作为值接收者将需要复制整个结构体，这将是昂贵的。在这种情况下，如果使用指针接收者，结构体将不会被复制，只会在方法中使用指向它的指针。

在所有其他情况下，可以使用值接收者。

### 匿名结构体字段的方法

属于结构体的匿名字段的方法可以像属于定义匿名字段的结构体一样被调用。

```go
package main

import (
    "fmt"
)

type address struct {
    city  string
    state string
}

func (a address) fullAddress() {
    fmt.Printf("Full address: %s, %s", a.city, a.state)
}

type person struct {
    firstName string
    lastName  string
    address
}

func main() {
    p := person{
        firstName: "Elon",
        lastName:  "Musk",
        address: address {
            city:  "Los Angeles",
            state: "California",
        },
    }

    p.fullAddress() //accessing fullAddress method of address struct
}
```

[Run program in playground](https://play.golang.org/p/vURnImw4_9)

在上面程序的第32行中，我们使用`p.fullAddress()`调用了`address`结构体的`fullAddress()`方法。不需要显式的方向`p.address.fullAddress()`。这个程序打印：

```
Full address: Los Angeles, California
```

### 方法中的值接收者vs函数中的值参数

这个主题让很多Go新手感到困惑。我会尽量让它变得清晰 😀。

当函数有一个值参数时，它只接受值参数。

当方法有一个值接收者时，它会同时接受指针和值接收者。

让我们通过一个例子来理解这一点。

```go
package main

import (
    "fmt"
)

type rectangle struct {
    length int
    width  int
}

func area(r rectangle) {
    fmt.Printf("Area Function result: %d\n", (r.length * r.width))
}

func (r rectangle) area() {
    fmt.Printf("Area Method result: %d\n", (r.length * r.width))
}

func main() {
    r := rectangle{
        length: 10,
        width:  5,
    }
    area(r)
    r.area()

    p := &r
    /*
       compilation error, cannot use p (type *rectangle) as type rectangle 
       in argument to area    
    */
    //area(p)

    p.area()//calling value receiver with a pointer
}
```

[Run program in playground](https://play.golang.org/p/gLyHMd2iie)

第12行的函数`func area(r rectangle)`接受一个值参数，第16行的方法`func (r rectangle) area()`接受一个值接收者。

在第25行，我们用值参数调用area函数`area(r)`，这是可以的。类似地，我们使用值接收者调用area方法`r.area()`，这也是可以的。

我们在第28行创建了一个指向`r`的指针`p`。如果我们试图将这个指针传递给只接受值的area函数，编译器会报错。我在第33行注释了这个操作。如果你取消这行的注释，编译器会抛出错误**compilation error, cannot use p (type \*rectangle) as type rectangle in argument to area**。这符合预期的行为。

现在来看棘手的部分，代码第35行`p.area()`使用指针接收者`p`调用只接受值接收者的方法`area`。这是完全有效的。原因是`p.area()`这行代码会被Go解释为`(*p).area()`，因为`area`有一个值接收者。这是为了方便而做的处理。

这个程序将输出：

```
Area Function result: 50
Area Method result: 50
Area Method result: 50
```

### 方法中的指针接收者vs函数中的指针参数

与值参数类似，带有指针参数的函数只接受指针，而带有指针接收者的方法将同时接受指针和值接收者。

```go
package main

import (
	"fmt"
)

type rectangle struct {
	length int
	width  int
}

func perimeter(r *rectangle) {
	fmt.Println("perimeter function output:", 2*(r.length+r.width))

}

func (r *rectangle) perimeter() {
	fmt.Println("perimeter method output:", 2*(r.length+r.width))
}

func main() {
	r := rectangle{
		length: 10,
		width:  5,
	}
	p := &r //pointer to r
	perimeter(p)
	p.perimeter()

	/*
		cannot use r (type rectangle) as type *rectangle in argument to perimeter
	*/
	//perimeter(r)

	r.perimeter()//calling pointer receiver with a value

}

```

[Run program in playground](https://play.golang.org/p/Xy5wW9YZMJ)

上面程序的第12行定义了一个接受指针参数的函数`perimeter`，第17行定义了一个有指针接收者的方法。

在第27行，我们用指针参数调用perimeter函数，在第28行，我们在指针接收者上调用perimeter方法。这些都是正确的。

在注释掉的第33行，我们试图用值参数`r`调用perimeter函数。这是不允许的，因为带有指针参数的函数不接受值参数。如果取消这行的注释并运行程序，编译会失败并显示错误**main.go:33: cannot use r (type rectangle) as type \*rectangle in argument to perimeter**。

在第35行，我们用值接收者`r`调用指针接收者方法`perimeter`。这是允许的，代码`r.perimeter()`会被语言解释为`(&r).perimeter()`，这是为了方便而做的处理。程序将会输出

```fallback
perimeter function output: 30
perimeter method output: 30
perimeter method output: 30
```

### 非结构体接收者的方法

到目前为止，我们只在结构体类型上定义了方法。实际上，也可以在非结构体类型上定义方法，但有一个注意事项：**为了在某个类型上定义方法，该类型的定义和方法的定义必须位于同一个包中。** 到目前为止，我们定义的所有结构体和结构体上的方法都位于同一个 `main` 包中，因此它们都能正常工作。

```go
package main

func (a int) add(b int) {
}

func main() {

}
```

[Run program in playground](https://play.golang.org/p/ybXLf5o_lA)

在上面的程序中，第三行我们尝试为内建类型 `int` 添加一个名为 `add` 的方法。这是不允许的，因为方法 `add` 的定义和类型 `int` 的定义不在同一个包中。这个程序会抛出编译错误 **cannot define new methods on non-local type int**。

使其正常工作的方式是为内建类型 `int` 创建一个类型别名，然后使用这个类型别名作为接收者定义方法。

```go
package main

import "fmt"

type myInt int

func (a myInt) add(b myInt) myInt {
	return a + b
}

func main() {
	num1 := myInt(5)
	num2 := myInt(10)
	sum := num1.add(num2)
	fmt.Println("Sum is", sum)
}
```

[Run program in playground](https://play.golang.org/p/sTe7i1qAng)

在上述程序的第五行，我们为 `int` 创建了一个类型别名 `myInt`。在第七行，我们定义了一个方法 `add`，其接收者是 `myInt`。

该程序将输出 `Sum is 15`。

我已在 [github](https://github.com/golangbot/methods) 上创建了一个包含我们讨论过的所有概念的程序。



**下一篇教程 - [接口 - I](../【GolangBot】18-接口-I)**
