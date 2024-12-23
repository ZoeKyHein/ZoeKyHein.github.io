---
title: 【GolangBot】9-循环
date: 2024-12-19 18:48:57
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---

# 【GolangBot】9-循环

欢迎来到[Golang教程系列](../golangbot/)的第9课

循环用于重复执行一段代码，直到满足某个条件为止。

在Go中，`for` 是唯一的循环。Go没有像C语言那样的`while`或`do while`循环。

### for 循环语法

```go
for 初始化; 条件; 后续操作 {
}
```

初始化语句只会执行一次。循环初始化后，会检查条件。如果条件为真，则会执行 `{}` 中的循环体代码，然后执行后续操作语句。后续操作语句会在每次成功迭代后执行。执行完后续操作语句后，会再次检查条件。如果条件为真，循环将继续执行，否则`for`循环终止。

在Go中，初始化、条件和后续操作这三个部分都是可选的。让我们通过一个例子来更好地理解`for`循环。

### 示例

我们来写一个程序，使用`for`循环打印从1到10的数字。

```go
package main

import (
	"fmt"
)

func main() {
	for i := 1; i <= 10; i++ {
		fmt.Printf("%d ", i)
	}
}
```

[在Playground中运行](https://go.dev/play/p/C7lOa1D_0-1)

在上述程序中，`i`被初始化为1。条件语句检查`i <= 10`是否为真。如果条件为真，则打印`i`的值，否则循环终止。后续操作语句在每次迭代结束时将`i`增加1。一旦`i`大于10，循环终止。

上述程序将打印：

```fallback
1 2 3 4 5 6 7 8 9 10
```

在`for`循环中声明的变量只能在循环的作用域内访问。因此，`i`无法在`for`循环体外部访问。

### break

`break`语句用于在`for`循环完成正常执行之前中断循环，并将控制转移到`for`循环之后的代码行。

让我们修改上述程序，使其在打印到5时中断。

```go
package main

import (
	"fmt"
)

func main() {
	for i := 1; i <= 10; i++ {
		if i > 5 {
			break //如果i > 5，循环终止
		}
		fmt.Printf("%d ", i)
	}
	fmt.Printf("
loop ended")
}
```

[在Playground中运行](https://go.dev/play/p/v5MIpHz6H6J)

在上述程序中，每次迭代都会检查`i`的值。如果`i`大于5，则执行`break`并终止循环。随后执行循环之后的打印语句。上述程序将输出：

```fallback
1 2 3 4 5 
loop ended
```

### continue

`continue`语句用于跳过当前迭代的剩余代码。`for`循环中`continue`之后的所有代码在当前迭代中将不会被执行。循环将直接跳到下一次迭代。

让我们编写一个程序，使用`continue`打印1到10之间的所有奇数。

```go
package main

import (
	"fmt"
)

func main() {
	for i := 1; i <= 10; i++ {
		if i%2 == 0 {
			continue
		}
		fmt.Printf("%d ", i)
	}
}
```

[在Playground中运行](https://go.dev/play/p/ZGsEFZzxYhD)

在上述程序中，第9行检查`i`除以2的余数是否为0。如果为0，则数字是偶数，执行`continue`语句，控制直接跳到循环的下一次迭代。因此，`continue`后的打印语句不会被调用，循环进入下一次迭代。上述程序的输出为：

```fallback
1 3 5 7 9
```

### 嵌套循环

一个`for`循环如果里面还有另一个`for`循环，那它就被称为嵌套循环。

我们写个程序打印如下序列来了解嵌套循环。

```fallback
*
**
***
****
*****
```

下面的程序使用嵌套打印序列。第8行的变量`n`存储了序列的行数。在我们的示例中是`5`。外层循环`i`从`0`迭代到`4`，内层循环`j`从`0`迭代到当前`i`的值。内层循环每次迭代都会打印`*`，外层循环每次迭代都会打印一个空行。运行这个程序你会看到序列打印出来作为输出。

```go
package main

import (
	"fmt"
)

func main() {
	n := 5
	for i := 0; i < n; i++ {
		for j := 0; j <= i; j++ {
			fmt.Print("*")
		}
		fmt.Println()
	}
}
```

[Run in playground](https://go.dev/play/p/0rq8fWjVDLb)

### 标签

标签可以用来从内层循环内部终止外层循环。让我们用一个简单的例子来理解这是什么意思。

```go
package main

import (
	"fmt"
)

func main() {
	for i := 0; i < 3; i++ {
		for j := 1; j < 4; j++ {
			fmt.Printf("i = %d , j = %d\n", i, j)
		}

	}
}
```

[Run in playground](https://go.dev/play/p/BnCKho2x5hM)

上面的程序的作用不言自明，它会打印

```fallback
i = 0 , j = 1
i = 0 , j = 2
i = 0 , j = 3
i = 1 , j = 1
i = 1 , j = 2
i = 1 , j = 3
i = 2 , j = 1
i = 2 , j = 2
i = 2 , j = 3
```

在这里没什么特别的:)。

如果我想在`i`和`j`相等的时候停止打印该怎么办呢？为了达成目的，我们需要从外层`for`循环`break`。当内层`for`循环`i`和`j`相等的时候添加一个`break`，将会只结束内层`for`循环。

```go
package main

import (
	"fmt"
)

func main() {
	for i := 0; i < 3; i++ {
		for j := 1; j < 4; j++ {
			fmt.Printf("i = %d , j = %d\n", i, j)
			if i == j {
				break
			}
		}

	}
}
```

[Run in playground](https://go.dev/play/p/uMjbF8Ii41d)

在上面的程序中，我已经在第10行内层`for`循环`i`和`j`相等的时候添加了一个`break`。这只是`break`掉了内层循环，而外层循环还在继续。这个程序将会打印

```fallback
i = 0 , j = 1
i = 0 , j = 2
i = 0 , j = 3
i = 1 , j = 1
i = 2 , j = 1
i = 2 , j = 2
```

这不是我们预期的输出。我们需要在`i`和`j`相等的时候停止打印，比如他们都等于`1`的时候。

这时就需要标签来救我们了。标签可以用来终止外层循环。我们用标签重写上面的程序。

```go
package main

import (
	"fmt"
)

func main() {
outer:
	for i := 0; i < 3; i++ {
		for j := 1; j < 4; j++ {
			fmt.Printf("i = %d , j = %d\n", i, j)
			if i == j {
				break outer
			}
		}

	}
}
```

[Run in playground](https://go.dev/play/p/BI10Rmp_Z3y)

在上面的程序中，我们已经为外层循环在第8行添加了一个`outer`标签，并且我们在第13行已通过指定标签break了外层循环。这个程序将会在`i`和`j`相等的时候停止打印。程序将会输出

```fallback
i = 0 , j = 1
i = 0 , j = 2
i = 0 , j = 3
i = 1 , j = 1
```

### 使用for循环实现while循环

我们在前面讨论过，`for`循环是go中唯一可用的循环语句。我们可以使用for循环的变体来实现`while`循环的功能。让我们来讨论一下如何实现。下面的程序打印从0到10的所有偶数。

```go
package main

import (
	"fmt"
)

func main() {
	i := 0
	for ;i <= 10; { // initialisation and post are omitted
		fmt.Printf("%d ", i)
		i += 2
	}
}
```

[Run in playground](https://go.dev/play/p/xstHXjQrWcc)

我们已经知道for循环的所有三个组成部分，即初始化、条件和后置条件，它们都是可选的。在山脉呢程序中忽略了初始化和后置条件。`i`在循环外面被初始化为`0`。只要`i<=10`循环就会执行。`i`在循环内每次加`2`。上面的程序将会打印`0 2 4 6 8 10 `。

上面程序中循环的封号也可以被省略。这种格式可以看作是`while`循环的一种替代格式。上面的程序可以重写为

```go
package main

import (
	"fmt"
)

func main() {
	i := 0
	for i <= 10 { //semicolons are ommitted and only condition is present. This is similar to while loop.
		fmt.Printf("%d ", i)
		i += 2
	}
}
```

[Run in playground](https://go.dev/play/p/4kU5944zDto)

### 多重变量声明

在一个`for`循环中声明和操作多个变量也是可以的。让我们写一个程序，用多重变量声明来打印如下的序列。

```fallback
10 * 1 = 10
11 * 2 = 22
12 * 3 = 36
13 * 4 = 52
14 * 5 = 70
15 * 6 = 90
16 * 7 = 112
17 * 8 = 136
18 * 9 = 162
19 * 10 = 190
```

``` go
package main

import (
	"fmt"
)

func main() {
	for no, i := 10, 1; i <= 10 && no <= 19; i, no = i+1, no+1 { //multiple initialisation and increment
		fmt.Printf("%d * %d = %d\n", no, i, no*i)
	}

}
```

[Run in playground](https://go.dev/play/p/oFcmLl3NHzK)

在上面的程序中`no`和`i`被声明，并且分别被初始化为10和1。每次循环结束后它们增加1。布尔操作符`&&`被用在条件里，它用来确保`i`小于等于10并且`no`小于等于19。

### 死循环

创建一个死循环的语法如下，

```go
for {
}
```

下面的程序将会持续不断地打印`Hello World`。

```go
package main

import "fmt"

func main() {
	for {
		fmt.Println("Hello World")
	}
}
```

如果你尝试着在[在线环境](https://go.dev/play/p/3Dmj7QVbQJz)运行上面的程序，你会得到错误`timeout running program`。请试着在你的本地环境运行，打印无限多的"Hello World"。

还有一种结构**range**可以在`for`循环中用于[数组操作](../【GolangBot】11-数组和切片/)。我们将会在数组章节来探讨它。

**下一篇教程 - [switch语句](../【GolangBot】10-switch语句/)**
