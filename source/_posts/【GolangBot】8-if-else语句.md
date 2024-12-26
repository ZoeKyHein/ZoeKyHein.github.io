---
title: 【GolangBot】8-if else语句
date: 2024-12-19 17:48:50
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---


欢迎来到[Golang系列教程](../golangbot/)的第八篇。

**`if`语句有个条件，如果这个条件值为`true`，它就会执行一段代码块。如果条件为`false`，就会执行另一段代码块。**在这个教程中，我们会看一下使用`if`语句的不同语法。

### If 语句语法

下面是`if`语句的语法。

```fallback
if condition {
}
```

如果`condition`运算为`true`，在`{` 和 `}`之间的代码块将被执行。

与C语言括号`{}`是可选的不同，即使`{}`之间只有一行语句，括号也是强制的。	

### 示例

我们来写一个简单的示例判断一个数字是奇数还是偶数。

```go
package main

import (
	"fmt"
)

func main() {
	num := 10
	if num%2 == 0 { //checks if number is even
		fmt.Println("The number", num, "is even")
		return
	}
	fmt.Println("The number", num, "is odd")
}
```

[Run in Playground](https://go.dev/play/p/RRxkgK07ul4)

在上面的程序中，第9行的条件`num%2`检测`num`除以`2`的余数是不是0。因为在这个例子中是`0`，文本`The number 10 is even`被打印，然后程序退出。

### If else 语句

`if`语句有一个可选结构，`else`语句，如果`if`语句的值是`false`，它就会执行。

```fallback
if condition {
} else {
}
```

在上面的片段中，如果`condition`的值是`false`，那么在`else {` 和 `}`之间的代码就会被执行。

让我们用上`if else`语句重写程序，重新检测数值的奇偶。

```go
package main

import (
	"fmt"
)

func main() {
	num := 11
	if num%2 == 0 { //checks if number is even
		fmt.Println("The number", num, "is even")
	} else {
		fmt.Println("The number", num, "is odd")
	}
}
```

[Run in playground](https://go.dev/play/p/Ku3HivlA5N4)

在上面的代码中，与我们在[上一节](../【GolangBot】8-if-else语句/#示例)中直接在条件为true时返回不同，我们创建了一个else语句，该语句会在条件为false时执行。这里，因为11是奇数，所以if条件为false，else语句内的代码会被执行。上述程序将输出：

```fallback
The number 11 is odd
```

### If … else if … else 语句

`if`语句还可以包含可选的`else if`和`else`组件。其语法如下：

```fallback
if condition1 {
...
} else if condition2 {
...
} else {
...
}
```

条件从上到下进行判断。

在上述语句中，如果`condition1`为`true`，那么`if condition1 {`和紧接着的结束括号`}`之间的代码块将被执行。

如果`condition1`为`false`且`condition2`为`true`，则`else if condition2 {`和下一个结束括号`}`之间的代码块将被执行。

如果`condition1`和`condition2`都为`false`，则`else`语句中的代码块，即`else {`和`}`之间的代码，将会被执行。

可以包含任意数量的`else if`语句。

**一般而言，无论是`if`还是`else if`，只要条件满足`true`，相应的代码块就会被执行。如果所有条件都为`false`，则执行`else`块中的代码。**

让我们用`else if`来编写一个公交车票价计算程序。程序需满足以下要求：

- 如果乘客的年龄小于5岁，则票免费。
- 如果乘客的年龄在5到22岁之间，则票价为10美元。
- 如果乘客的年龄超过22岁，则票价为15美元。

```go
package main

import (
	"fmt"
)

func main() {
	age := 10
	ticketPrice := 0
	if age < 5 {
		ticketPrice = 0
	} else if age >= 5 && age <= 22 {
		ticketPrice = 10
	} else {
		ticketPrice = 15
	}
	fmt.Printf("Ticket price is $%d", ticketPrice)
}
```

[Run in playground](https://go.dev/play/p/CPaydvwiegY)

在上述程序中，乘客的年龄被设为`10`。第12行的条件为`true`，因此程序将打印：

```fallback
Ticket price is $10
```

请尝试更改乘客年龄以测试`if else`语句中不同块是否按预期执行。

### If语句中的赋值

`if`语句还有一种变体，包含一个可选的[简短赋值](../【GolangBot】3-变量/#简短声明)语句，在条件判断之前执行。语法如下：

```fallback
if assignment-statement; condition {
}
```

在上述语法中，`assignment-statement`在条件判断前被执行。

让我们使用这种简短语法重写公交车票价程序：

```go
package main

import (
	"fmt"
)

func main() {
	ticketPrice := 0
	if age := 10; age < 5 {
		ticketPrice = 0
	} else if age >= 5 && age <= 22 {
		ticketPrice = 10
	} else {
		ticketPrice = 15
	}
	fmt.Printf("Ticket price is $%d", ticketPrice)
}
```

[Run in playground](https://go.dev/play/p/JHmL4h_MB2F)

在此程序中，`age`在第9行的`if`语句中被初始化。**`age`只能在`if`结构中访问。也就是说，它的作用域仅限于`if`、`else if`和`else`块。如果尝试在这些块外访问`age`，编译器将会报错。**

这种语法常用于声明仅在`if else`构造中使用的变量。它能确保变量的作用域只限于`if`语句。

### 注意事项

`else`语句必须紧跟在`if`语句的结束括号`}`所在的同一行。如果不是，编译器会报错。

让我们通过一个程序来理解：

```go
package main

import (
	"fmt"
)

func main() {
	num := 10
	if num % 2 == 0 { //checks if number is even
		fmt.Println("the number is even") 
	}  
    else {
		fmt.Println("the number is odd")
	}
}
```

[Run in playground](https://go.dev/play/p/RiZ3drcztcI)

在上面的程序中，`else`语句没有紧跟在第11行的`if`语句结束括号`}`之后，而是换行写在下一行。这在Go语言中是不允许的。如果运行该程序，编译器会打印如下错误：

```fallback
./prog.go:12:5: syntax error: unexpected else, expected }
```

这是因为Go语言的分号自动插入规则。可以阅读[此处](https://go.dev/ref/spec#Semicolons)的语法说明了解更多。

规则中说明，如果`}`是该行的最后一个标记，则在`}`后会自动插入一个分号。因此，Go编译器会在第11行的`if`语句结束括号`}`后插入分号。

因此，程序实际上变成：

```fallback
...
if num%2 == 0 { 
      fmt.Println("the number is even") 
};  //semicolon inserted by Go Compiler
else {
      fmt.Println("the number is odd")
}
```

由于`if {...} else {...}`是一条完整的语句，因此在中间不能有分号。故此程序无法通过编译。因此，`else`必须紧跟在`if`语句的结束括号`}`后。

让我们重新编写程序，将`else`放到同一行：

```go
package main

import (
	"fmt"
)

func main() {
	num := 10
	if num%2 == 0 { //checks if number is even
		fmt.Println("the number is even") 
	} else {
		fmt.Println("the number is odd")
	}
}
```

[Run in playground](https://go.dev/play/p/NKELadXVQse)

现在编译器会正常通过，我们也会感到满意 😃。

### Go的惯用写法

我们已经看到多种`if else`构造，也看到了多种方式实现相同功能的程序。例如，通过不同的`if else`构造编写检查数字是奇数还是偶数的程序。那么哪一种是Go语言的惯用写法？**在Go的惯用写法中，最好避免不必要的分支和代码缩进，同时尽可能早地返回。**

我提供了上一节的程序：

```go
package main

import (
	"fmt"
)

func main() {
	if num := 10; num % 2 == 0 { //checks if number is even
		fmt.Println(num,"is even") 
	}  else {
		fmt.Println(num,"is odd")
	}
}
```

[Run in playground](https://go.dev/play/p/ed3YK5cM3Hw)

在Go中，惯用写法是尽早返回并避免`else`分支：

```go
package main

import (
	"fmt"
)

func main() {
	num := 10;
	if num%2 == 0 { //checks if number is even
		fmt.Println(num, "is even")
		return
	}
	fmt.Println(num, "is odd")

}
```

[Run in playground](https://go.dev/play/p/JpWZRmFcdSG)

在此程序中，当确定数字是偶数后，立即返回。这避免了不必要的`else`分支。这是Go语言中的推荐做法 😃。编写Go程序时，请牢记这一点。

**下一篇教程 - [循环](../【GolangBot】9-循环/)**
