---
title: 【GolangBot】16-结构体
date: 2024-12-24 12:51:13
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---
# Golang结构体教程

欢迎来到我们的 [Golang 教程系列](../golangbot/)的第16篇教程。

### 什么是结构体？

结构体是一个用户定义的类型，表示一组字段的集合。在某些情况下，将数据组合成一个单元比将每个数据作为单独的值更有意义，这时就可以使用结构体。

例如，一个员工有firstName、lastName和age。将这三个属性组合到一个名为`Employee`的结构体中是有意义的。

### 声明结构体

```go
type Employee struct {
    firstName string
    lastName  string
    age       int
}
```

上面的代码声明了一个`Employee`结构体类型，包含`firstName`、`lastName`和`age`字段。上面的`Employee`结构体被称为**命名结构体**，因为它使用新的数据类型名称`Employee`创建了一个新的数据类型，可以用它来创建`Employee`结构体。

这个结构体也可以通过将属于同一类型的字段在一行中声明来使其更加紧凑，后面跟着类型名称。在上面的结构体中，`firstName`和`lastName`属于同一个类型`string`，因此结构体可以重写为：

```go
type Employee struct {
    firstName, lastName string
    age                int
}
```

*虽然上述语法节省了几行代码，但它没有使字段声明明确。请避免使用上述语法。*

### 创建命名结构体

让我们使用以下简单程序声明一个**命名结构体Employee**。

```go
package main

import (
    "fmt"
)

type Employee struct {
    firstName string
    lastName  string
    age       int
    salary    int
}

func main() {

    //creating struct specifying field names
    emp1 := Employee{
        firstName: "Sam",
        age:       25,
        salary:    500,
        lastName:  "Anderson",
    }

    //creating struct without specifying field names
    emp2 := Employee{"Thomas", "Paul", 29, 800}

    fmt.Println("Employee 1", emp1)
    fmt.Println("Employee 2", emp2)
}
```

在上面程序的第7行中，我们创建了一个命名结构体类型`Employee`。在程序的第17行中，通过指定每个字段名的值来定义`emp1`结构体。字段的顺序不需要与声明结构体类型时的字段名顺序相同。在这种情况下，我们改变了`lastName`的位置并将其移到末尾。这样做不会有任何问题。

**在程序的第25行中，定义`emp2`时省略了字段名。在这种情况下，必须保持字段的顺序与结构体声明中指定的顺序相同。请避免使用这种语法，因为很难弄清楚哪个值对应哪个字段。**我们在这里提到这种格式只是为了让大家了解这也是一种有效的语法 :)

上述程序打印输出：

```
Employee 1 {Sam Anderson 25 500}  
Employee 2 {Thomas Paul 29 800}
```

### 创建匿名结构体

可以在不创建新数据类型的情况下声明结构体。这种类型的结构体称为**匿名结构体**。

```go
package main

import (
    "fmt"
)

func main() {
    emp3 := struct {
        firstName string
        lastName  string
        age       int
        salary    int
    }{
        firstName: "Andreah",
        lastName:  "Nikola",
        age:       31,
        salary:    5000,
    }

    fmt.Println("Employee 3", emp3)
}
```

在上面程序的第8行中，定义了一个**匿名结构体变量**`emp3`。正如我们已经提到的，这个结构体之所以称为匿名，是因为它只创建了一个新的结构体变量`emp3`，而没有像命名结构体那样定义任何新的结构体类型。

此程序输出：

```
Employee 3 {Andreah Nikola 31 5000}
```

### 访问结构体的单个字段

使用点`.`运算符来访问结构体的单个字段。

```go
package main

import (
    "fmt"
)

type Employee struct {
    firstName string
    lastName  string
    age       int
    salary    int
}

func main() {
    emp6 := Employee{
        firstName: "Sam",
        lastName:  "Anderson",
        age:       55,
        salary:    6000,
    }
    fmt.Println("First Name:", emp6.firstName)
    fmt.Println("Last Name:", emp6.lastName)
    fmt.Println("Age:", emp6.age)
    fmt.Printf("Salary: $%d\n", emp6.salary)
    emp6.salary = 6500
    fmt.Printf("New Salary: $%d", emp6.salary)
}
```

**emp6.firstName**在上面的程序中访问`emp6`结构体的`firstName`字段。在第25行中，我们修改了员工的工资。这个程序打印：

```
First Name: Sam
Last Name: Anderson
Age: 55
Salary: $6000
New Salary: $6500
```

### 结构体的零值

当定义一个结构体但没有显式地用任何值初始化时，结构体的字段会被默认赋予它们的零值。

```go
package main

import (
    "fmt"
)

type Employee struct {
    firstName string
    lastName  string
    age       int
    salary    int
}

func main() {
    var emp4 Employee //zero valued struct
    fmt.Println("First Name:", emp4.firstName)
    fmt.Println("Last Name:", emp4.lastName)
    fmt.Println("Age:", emp4.age)
    fmt.Println("Salary:", emp4.salary)
}
```

上面的程序定义了`emp4`但没有用任何值初始化它。因此`firstName`和`lastName`被赋予了string的零值，即空字符串`""`，而`age`和`salary`被赋予了int的零值，即0。这个程序打印：

```
First Name: 
Last Name: 
Age: 0
Salary: 0
```

也可以指定某些字段的值而忽略其他字段。在这种情况下，被忽略的字段会被赋予零值。

```go
package main

import (
    "fmt"
)

type Employee struct {
    firstName string
    lastName  string
    age       int
    salary    int
}

func main() {
    emp5 := Employee{
        firstName: "John",
        lastName:  "Paul",
    }
    fmt.Println("First Name:", emp5.firstName)
    fmt.Println("Last Name:", emp5.lastName)
    fmt.Println("Age:", emp5.age)
    fmt.Println("Salary:", emp5.salary)
}
```

在上面程序的第16行和第17行，初始化了`firstName`和`lastName`，而`age`和`salary`没有被初始化。因此`age`和`salary`被赋予了它们的零值。这个程序输出：

```
First Name: John
Last Name: Paul
Age: 0
Salary: 0
```

### 结构体指针

也可以创建指向结构体的指针。

```go
package main

import (
    "fmt"
)

type Employee struct {
    firstName string
    lastName  string
    age       int
    salary    int
}

func main() {
    emp8 := &Employee{
        firstName: "Sam",
        lastName:  "Anderson",
        age:       55,
        salary:    6000,
    }
    fmt.Println("First Name:", (*emp8).firstName)
    fmt.Println("Age:", (*emp8).age)
}
```

**emp8**在上面的程序中是指向`Employee`结构体的指针。`(*emp8).firstName`是访问`emp8`结构体的`firstName`字段的语法。这个程序打印：

```
First Name: Sam
Age: 55
```

**Go语言给我们提供了使用`emp8.firstName`而不是显式解引用`(*emp8).firstName`来访问`firstName`字段的选项。**

```go
package main

import (
    "fmt"
)

type Employee struct {
    firstName string
    lastName  string
    age       int
    salary    int
}

func main() {
    emp8 := &Employee{
        firstName: "Sam",
        lastName:  "Anderson",
        age:       55,
        salary:    6000,
    }
    fmt.Println("First Name:", emp8.firstName)
    fmt.Println("Age:", emp8.age)
}
```

我们在上面的程序中使用了`emp8.firstName`来访问`firstName`字段，这个程序也输出：

```
First Name: Sam
Age: 55
```

### 匿名字段

可以创建只包含类型而没有字段名的结构体字段。这种字段被称为匿名字段。

下面的代码片段创建了一个`Person`结构体，它有两个匿名字段`string`和`int`

```go
type Person struct {
    string
    int
}
```

**虽然匿名字段没有显式的名称，但默认情况下匿名字段的名称是其类型的名称。**例如在上面的Person结构体中，尽管这些字段是匿名的，但默认情况下它们采用字段类型的名称。所以Person结构体有2个名为`string`和`int`的字段。

```go
package main

import (
    "fmt"
)

type Person struct {
    string
    int
}

func main() {
    p1 := Person{
        string: "naveen",
        int:    50,
    }
    fmt.Println(p1.string)
    fmt.Println(p1.int)
}
```

在上面程序的第17行和第18行中，我们使用它们的类型作为字段名来访问Person结构体的匿名字段，即`string`和`int`。上面程序的输出是：

```
naveen
50
```

### 嵌套结构体

结构体可以包含一个字段，这个字段本身就是一个结构体。这种结构体被称为嵌套结构体。

```go
package main

import (
    "fmt"
)

type Address struct {
    city  string
    state string
}

type Person struct {
    name    string
    age     int
    address Address
}

func main() {
    p := Person{
        name: "Naveen",
        age:  50,
        address: Address{
            city:  "Chicago",
            state: "Illinois",
        },
    }

    fmt.Println("Name:", p.name)
    fmt.Println("Age:", p.age)
    fmt.Println("City:", p.address.city)
    fmt.Println("State:", p.address.state)
}
```

上面程序中的`Person`结构体有一个字段`address`，这个字段本身就是一个结构体。这个程序打印：

```
Name: Naveen
Age: 50
City: Chicago
State: Illinois
```

### 提升字段

属于结构体中的匿名结构体字段的字段被称为提升字段，因为可以像它们属于拥有匿名结构体字段的结构体一样访问它们。我知道这个定义很复杂，所以让我们直接看一些代码来理解这一点 :)。

```go
type Address struct {
    city string
    state string
}
type Person struct {
    name string
    age  int
    Address
}
```

在上面的代码片段中，`Person`结构体有一个匿名字段`Address`，它是一个结构体。现在`Address`的字段`city`和`state`被称为提升字段，因为它们可以像直接在`Person`结构体中声明的一样被访问。

```go
package main

import (
    "fmt"
)

type Address struct {
    city  string
    state string
}
type Person struct {
    name string
    age  int
    Address
}

func main() {
    p := Person{
        name: "Naveen",
        age:  50,
        Address: Address{
            city:  "Chicago",
            state: "Illinois",
        },
    }

    fmt.Println("Name:", p.name)
    fmt.Println("Age:", p.age)
    fmt.Println("City:", p.city)   //city is promoted field
    fmt.Println("State:", p.state) //state is promoted field
}
```

在上面程序的第29行和第30行中，提升字段`city`和`state`使用语法`p.city`和`p.state`访问，就像它们是在结构体`p`中声明的一样。这个程序打印：

```
Name: Naveen
Age: 50
City: Chicago
State: Illinois
```

### 导出的结构体和字段

如果结构体类型以大写字母开头，那么它就是一个导出类型，可以从其他包中访问。类似地，如果结构体的字段以大写字母开头，它们也可以从其他包中访问。

让我们编写一个具有自定义包的程序来更好地理解这一点。

在你的`Documents`目录中创建一个名为`structs`的文件夹。你可以随意在任何地方创建它。我更喜欢我的`Documents`目录。

```
mkdir ~/Documents/structs
```

让我们创建一个名为`structs`的go模块。

```
cd ~/Documents/structs/
go mod init structs
```

在`structs`里面创建另一个目录`computer`。

```
mkdir computer
```

在`computer`目录中，创建一个文件`spec.go`，内容如下。

```go
package computer

type Spec struct { //exported struct
    Maker string //exported field
    Price int //exported field
    model string //unexported field

}
```

上述代码片段创建了一个 [包](https://golangbot.com/go-packages/) `computer`，该包包含一个已导出的结构体类型 `Spec`，其中包含两个已导出的字段 `Maker` 和 `Price`，以及一个未导出的字段 `model`。接下来，让我们从主包中导入这个包，并使用 `Spec` 结构体。

在 `structs` 目录下创建一个名为 `main.go` 的文件，并在 `main.go` 中编写以下程序：

```go
package main

import (
    "structs/computer"
    "fmt"
)

func main() {
	spec := computer.Spec {
            Maker: "apple",
            Price: 50000,
        }
	fmt.Println("Maker:", spec.Maker)
    fmt.Println("Price:", spec.Price)
}
```

`structs`文件夹应该有如下的结构，

```fallback
├── structs
│   ├── computer
│   │   └── spec.go
│   ├── go.mod
│   └── main.```
```

在上述程序的第4行，我们导入了 `computer` 包。在第13行和第14行，我们访问了结构体 `Spec` 的两个已导出的字段 `Maker` 和 `Price`。可以通过执行命令 `go install` 后跟 `structs` 命令来运行此程序。

```fallback
go install
structs
```

运行上面的程序将会打印，

```fallback
Maker: apple
Price: 50000
```

如果我们尝试访问未导出的字段 model，编译器会报错。将 main.go 的内容替换为以下代码。

```go
package main

import (
    "structs/computer"
    "fmt"
)

func main() {
	spec := computer.Spec {
            Maker: "apple",
            Price: 50000,
            model: "Mac Mini",
        }
	fmt.Println("Maker:", spec.Maker)
    fmt.Println("Price:", spec.Price)
}
```


在上面的程序中，第12行我们尝试访问未导出的字段 `model`。运行此程序将导致编译错误。

```fallback
# structs
./main.go:12:13: unknown field 'model' in struct literal of type computer.Spec
```

由于 `model` 字段是未导出的，因此无法从其他包中访问。

### 结构体的相等性

**结构体是值类型，只有当其每个字段都是可比较的时，结构体才是可比较的。** 如果两个结构体变量的对应字段相等，则认为这两个结构体变量是相等的。

```go
package main

import (
	"fmt"
)

type name struct {
	firstName string
	lastName  string
}

func main() {
	name1 := name{
		firstName: "Steve",
		lastName:  "Jobs",
	}
	name2 := name{
		firstName: "Steve",
		lastName:  "Jobs",
	}
	if name1 == name2 {
		fmt.Println("name1 and name2 are equal")
	} else {
		fmt.Println("name1 and name2 are not equal")
	}

	name3 := name{
		firstName: "Steve",
		lastName:  "Jobs",
	}
	name4 := name{
		firstName: "Steve",
	}

	if name3 == name4 {
		fmt.Println("name3 and name4 are equal")
	} else {
		fmt.Println("name3 and name4 are not equal")
	}
}
```


[Run in playground](https://play.golang.org/p/ntDT8ZuOVK8)

在上面的程序中，`name` 结构体类型包含两个 `string` 字段。由于字符串是可比较的，因此可以比较两个 `name` 类型的结构体变量。

在上面的程序中，`name1` 和 `name2` 是相等的，而 `name3` 和 `name4` 是不相等的。该程序的输出为：

```fallback
name1 and name2 are equal
name3 and name4 are not equal
```

**结构体变量在其包含不可比较的字段时不可比较**（感谢 [alasijia](https://www.reddit.com/r/golang/comments/6cht1j/a_complete_guide_to_structs_in_go/dhvf7hd/) 在 Reddit 上指出这一点）。

```go
package main

import (
	"fmt"
)

type image struct {
	data map[int]int
}

func main() {
	image1 := image{
		data: map[int]int{
			0: 155,
		}}
	image2 := image{
		data: map[int]int{
			0: 155,
		}}
	if image1 == image2 {
		fmt.Println("image1 and image2 are equal")
	}
}
```


[Run in playground](https://play.golang.org/p/NUfoyGdOgu4)

在上面的程序中，`image` 结构体类型包含一个字段 `data`，它是 `map` 类型。由于 [maps](https://golangbot.com/maps/) 是不可比较的，因此 `image1` 和 `image2` 不能进行比较。如果运行此程序，将会遇到编译错误。

```fallback
./prog.go:20:12: invalid operation: image1 == image2 (struct containing map[int]int cannot be compared)
```

**下一篇教程 - [方法](../【GolangBot】17-方法/)**
