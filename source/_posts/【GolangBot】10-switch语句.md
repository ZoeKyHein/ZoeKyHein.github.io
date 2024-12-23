---
title: 【GolangBot】10-switch语句
date: 2024-12-23 08:49:05
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---

# 【GolangBot】10-switch语句

这是[Golang系列教程](../golangBot/)的第10篇。

### 什么是switch语句？



**Switch是一种条件语句，它会评估一个表达式，将其与一系列可能的匹配项进行比较，并执行相应的代码块。它可以被视为代替复杂的`if else`语句的一种简洁方式。**

### 示例

一例胜千言。让我们以一个简单的例子开始。这个例子接收一个手指数作为输入，并且输出这个手指的名字:)。比如，1是拇指，2是食指，以此类推。

```go
package main

import (
	"fmt"
)

func main() {
	finger := 4
	fmt.Printf("Finger %d is ", finger)
	switch finger {
	case 1:
		fmt.Println("Thumb")
	case 2:
		fmt.Println("Index")
	case 3:
		fmt.Println("Middle")
	case 4:
		fmt.Println("Ring")
	case 5:
		fmt.Println("Pinky")

	}
}
```

[Run in playground](https://go.dev/play/p/94ktmJWlUom)

在上面的程序中的第十行`switch finger`，将`finger`的值与每一个`case`语句进行比较。Case们从上往下执行，第一个匹配表达式的case会执行。在这个例子中，`finger`的值是`4`因此打印如下。

```fallback
Finger 4 is Ring
```

### 重复的case是不允许的

拥有相同常量值的重复case是不允许的。

```go
package main

import (
	"fmt"
)

func main() {
	finger := 4
	fmt.Printf("Finger %d is ", finger)
	switch finger {
	case 1:
		fmt.Println("Thumb")
	case 2:
		fmt.Println("Index")
	case 3:
		fmt.Println("Middle")
	case 4:
		fmt.Println("Ring")
	case 4: //duplicate case
		fmt.Println("Another Ring")
	case 5:
		fmt.Println("Pinky")

	}
}
```

[Run in playground](https://go.dev/play/p/7qrmR0hdvHH)

运行上面的程序将会得到如下的编译错误

```fallback
./prog.go:19:7: duplicate case 4 (constant of type int) in expression switch
./prog.go:17:7: previous case
```

### 默认case

我们受伤只有5根手指。如果输入了错误的手指数会发生什么呢？这就是默认case该登场的时候了。默认case将会在其它case都不匹配的情况下运行。

```go
package main

import (
	"fmt"
)

func main() {
	switch finger := 8; finger {
	case 1:
		fmt.Println("Thumb")
	case 2:
		fmt.Println("Index")
	case 3:
		fmt.Println("Middle")
	case 4:
		fmt.Println("Ring")
	case 5:
		fmt.Println("Pinky")
	default: //default case
		fmt.Println("incorrect finger number")
	}
}
```

[Run in playground](https://go.dev/play/p/P3zBv7zHITF)

在上面的程序中，`finfer`是`8`，并且他不匹配任何case。因此在默认case中的`incorrct finger number`被打印。`default`并不一定要放在switch语句的最后一个case。它可以出现在switch语句中的任何地方。

你也许注意到了`finger`定义的小改动。它在switch自身中被定义。Switch可以包含一个可选语句，这个语句在表达式执行之前被执行。在第8行，`finger`是首先被定义，然后在表达式中被使用。在这个例子中`finger`的作用域仅限于switch代码块。

### case中的多表达式

在一个case中，包含多个用逗号分隔的表达式也是可以的。

```go
package main

import (
	"fmt"
)

func main() {
	letter := "i"
	switch letter {
	case "a", "e", "i", "o", "u": //multiple expressions in case
		fmt.Printf("%s is a vowel", letter)
	default:
		fmt.Printf("%s is not a vowel", letter)
	}
}
```

[Run in playground](https://go.dev/play/p/fYI-vbKNGfc)

上面的程序检测`letter`是不是元音字母。第10行的代码`case "a", "e", "i", "o", "u":`匹配所有的元音字母。因为`i`是原因，这个程序将会打印

```fallback
i is a vowel
```

### 无表达式switch

"switch 语句中的表达式是可选的，可以省略。如果省略了表达式，switch 会被视为 `switch true`，每个 `case` 表达式都会被求值判断其真假，并执行相应的代码块。"

```go
package main

import (
	"fmt"
)

func main() {
	hour := 15 // hour in 24 hour format
	// Using switch to determine the work shift
	switch {
	case hour >= 6 && hour < 12:
		fmt.Println("It's the morning shift.")
	case hour >= 12 && hour < 17:
		fmt.Println("It's the afternoon shift.")
	case hour >= 17 && hour < 21:
		fmt.Println("It's the evening shift.")
	case (hour >= 21 && hour <= 24) || (hour >= 0 && hour < 6):
		fmt.Println("It's the night shift.")
	default:
		fmt.Println("Invalid hour.")
	}
}
```

[Run in playground](https://go.dev/play/p/Bso-wGY040c)

在上面的程序中，switch中没有表达式，因此他被认为是true并且每个case都执行。13行的`hour >= 12 && hour < 17`是`true`并且程序打印

```fallback
It's the afternoon shift.
```

这种类型的switch可以被看作多个`if else`语句的替代方案。

### Fallthrough

"在 Go 语言中，执行完一个 case 后，控制流会立即跳出 switch 语句。`fallthrough` 语句用于将控制转移到刚执行完的 case 之后的下一个 case 的第一条语句。

让我们编写一个程序来理解 fallthrough。我们的程序将检查输入的数字是否小于 50、100 或 200。例如，如果我们输入 75，程序将打印出 75 小于 100 和 200。我们将使用 `fallthrough` 来实现这个功能。"

```go
package main

import (
	"fmt"
)

func number() int {
	num := 15 * 5
	return num
}

func main() {
	switch num := number(); { //num is not a constant
	case num < 50:
		fmt.Printf("%d is lesser than 50\n", num)
		fallthrough
	case num < 100:
		fmt.Printf("%d is lesser than 100\n", num)
		fallthrough
	case num < 200:
		fmt.Printf("%d is lesser than 200", num)
	}
}
```

[Run in playground](https://go.dev/play/p/yt3kMvYlzLR)

switch 和 case 表达式不一定要是常量，它们也可以在运行时求值。在上面的程序中，`num` 在第 13 行被初始化为函数 `number()` 的返回值。控制流进入 switch 后开始求值各个 case。第 17 行的 `case num < 100:` 判断为真，程序打印出 `75 小于 100`。下一条语句是 `fallthrough`。当遇到 `fallthrough` 时，控制流会移动到下一个 case 的第一条语句，因此也会打印出 `75 小于 200`。程序的输出结果是

```fallback
75 is lesser than 100
75 is lesser than 200
```

*`fallthrough` 必须是 `case` 中的最后一条语句。如果它出现在中间位置，编译器会报错：`fallthrough statement out of place`（fallthrough 语句位置不正确）*

### 即使case为false，fallthrough也会运行

`fallthrough` 使用时有一个需要特别注意的细节：即使 case 的条件判断为 false，fallthrough 也会执行。 让我们看看下面这个程序。

```go
package main

import (
	"fmt"
)

func main() {
	switch num := 25; { 
	case num < 50:
		fmt.Printf("%d is lesser than 50\n", num)
		fallthrough
	case num > 100:
		fmt.Printf("%d is greater than 100\n", num)		
	}

}
```

go

[Run in playground](https://go.dev/play/p/u2AN8LhYYMM)

在上面的程序中，`num`是25，它小于50，因此第9行的case是`true`。`fallthrough`出现在第11行。下一个`casecase num > 100:`在第12行，因为num < 100所以它是false。但是fallthrough不考虑这个。即便case是false，fallthrough也会运行。

上面的程序将会打印

```fallback
25 is lesser than 50
25 is greater than 100
```

所以使用fallthrough的时候，确保你知道你在干什么。

还有一点要注意，`fallthrough` 不能用在 switch 的最后一个 case 中，因为后面没有更多的 case 可以执行。如果在最后一个 case 中使用 `fallthrough`，会导致编译错误：`cannot fallthrough final case in switch`（不能在 switch 的最后一个 case 中使用 fallthrough）。

One more thing is `fallthrough` cannot be used in the last case of a switch since there are no more cases to fallthrough. If `fallthrough` is present in the last case, it will result in the following compilation error `cannot fallthrough final case in switch`

### 中断switch

`break` 语句可以用来提前终止 switch 的执行。让我们修改上面的例子，创建一个简单的示例来理解 break 是如何工作的。 让我们添加一个条件：如果 `num` 小于 0，则终止 switch 的执行。

```go
package main

import (
	"fmt"
)

func main() {
	switch num := -5; {
	case num < 50:
		if num < 0 {
			break
		}
		fmt.Printf("%d is lesser than 50\n", num)
		fallthrough
	case num < 100:
		fmt.Printf("%d is lesser than 100\n", num)
		fallthrough
	case num < 200:
		fmt.Printf("%d is lesser than 200", num)
	}

}
```

[Run in playground](https://go.dev/play/p/UHwBXPYLv1B)

在上面的程序中，`num` 的值是 `-5`。当控制流到达第 10 行的 [if 语句](../【GolangBot】8-if-else语句) 时，由于 `num < 0` 这个条件成立。break 语句会在 switch 完成之前终止它的执行，因此程序不会打印任何内容 :)。

### Breaking the outer for loop中断外层for循环

当switch在一个[for循环](../【GolangBot】9-循环)中时，可能需要提前中断for循环。这可以通过给for循环打标签，使用switch语句中的标签来中断for循环。我们来看一个例子。

让我们写一个程序生成一个随机奇数。

我们会创建一个死循环，使用switch case来断定生成的随机数是不是奇数。如果是奇数，生成数被打印，for循环使用它的标签中止。`rand`[包]()`中的Intn`函数用来生成非负伪随机数。

```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
randloop:
	for {
		switch i := rand.Intn(100); {
		case i%2 == 0:
			fmt.Printf("Generated even number %d", i)
			break randloop
		}
	}

}
```

[Run in playground](https://go.dev/play/p/0bLYOgs2TUk)

在上面的程序中，第9行的for循环被打了标签`randloop`。第11行使用`Intn`函数生成了一个0到99的随机数(100不包含在内)。如果生成数是奇数，循环将在14行使用标签中断。

这个程序打印，

```fallback
Generated even number 18
```

**请注意，如果break语句没有和标签一起使用，仅仅只会有switch语句被中断，而循环还会继续。所以要中断外层循环，为循环打标签，并用它在switch语句里它中断循环是必要的。**



**下一篇教程 - [数组和切片](../【GolangBot】11-数组和切片)**
