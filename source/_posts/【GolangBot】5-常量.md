---
title: 【GolangBot】5-常量
date: 2024-12-18 09:48:12
tags: 
    - GoLang
    - 教程
categories:
    - [学习心得, GoLangBot]
published: false
---

Welcome to tutorial no. 5 in our [Golang tutorial series](https://golangbot.com/learn-golang-series/).

欢迎来到我们

### What is a constant?

Constants in Go is used to denote fixed static values such as

```fallback
95 
"I love Go" 
67.89 
```

and so on. Constants are generally used to represent values that do not change throughout the life time of an application.

### Declaring a constant

The keyword `const` is used to declare a constant in Go. Let’s see how to declare a constant by means an example.

```go
package main

import (
	"fmt"
)

func main() {
	const a = 50
	fmt.Println(a)
}
```

go

[Run in playground](https://go.dev/play/p/mv3B-q3h0zh)

In the above code `a` is a constant and it is assigned the value `50`.

### Declaring a group of constants

There is also another syntax to define a group of constants using a single statement. An example to define a group of constants using this syntax is provided below.

```go
package main

import (
	"fmt"
)

func main() {
	const (
		retryLimit = 4
		httpMethod = "GET"
	)

	fmt.Println(retryLimit)
	fmt.Println(httpMethod)
}
```

go

[Run in playground](https://go.dev/play/p/gSPJC0y49Sm)

In the above program, we have declared 2 constants `retryLimit` and `httpMethod`. The above program prints,

```fallback
4
GET
```

Constants, as the name indicate, cannot be reassigned again to any other value. In the program below, we are trying to assign another value `89` to `a`. This is not allowed since `a` is a constant. This program will fail to run with compilation error `cannot assign to a (neither addressable nor a map index expression)`.

```go
package main

func main() {  
    const a = 55 //allowed
    a = 89 //reassignment not allowed
}
```

go

[Run in playground](https://go.dev/play/p/5ggPss1iSsl)

**The value of a constant should be known at compile time.** Hence it cannot be assigned to a value returned by a [function call](https://golangbot.com/functions/) since the function call takes place at run time.

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	var a = math.Sqrt(4) //allowed
	fmt.Println(a)
	const b = math.Sqrt(4) //not allowed
	fmt.Println(b)
}
```

go

[Run in playground](https://go.dev/play/p/fFLfoN0L3Nf)

In the above program, `a` is a [variable](https://golangbot.com/variables/) and hence it can be assigned to the result of the function `math.Sqrt(4)` (We will discuss functions in more detail in a [separate tutorial](https://golangbot.com/functions/)).

`b` is a constant and the value of `b` needs to be known at compile time. The function `math.Sqrt(4)` will be evaluated only during run time and hence `const b = math.Sqrt(4)` fails to compile with error

```fallback
./prog.go:11:12: math.Sqrt(4) (value of type float64) is not constant
```

### String Constants, Typed and Untyped Constants

Any value enclosed between double quotes is a string constant in Go. For example, strings like `"Hello World"`, `"Sam"` are all constants in Go.

What type does a string constant belong to? The answer is they are **untyped**.

**A string constant like “Hello World” does not have any type**.

```go
const hello = "Hello World"
```

go

In the above line of code, the constant `hello` doesn’t have a type.

Go is a strongly typed language. All variables require an explicit type. How does the following program which assigns a variable `name` to an untyped constant `n` work?

```go
package main

import (
	"fmt"
)

func main() {
	const n = "Sam"
	var name = n
	fmt.Printf("type %T value %v", name, name)

}
```

go

[Run in playground](https://go.dev/play/p/CFqbC8yfe6C)

**The answer is untyped constants have a default type associated with them and they supply it if and only if a line of code demands it. In the statement `var name = n` in line no. 9, `name` needs a type and it gets it from the default type of the string constant `n` which is a `string`.**

Is there a way to create a **typed constant**? The answer is yes. The following code creates a typed constant.

```go
const name string = "Hello World"
```

go

*name* in the above code is a constant of type `string`.

Go is a strongly typed language. Mixing types during the assignment is not allowed. Let’s see what this means with the help of a program.

```go
package main

import "fmt"

func main() {
	var defaultName = "Sam" //allowed
	type myString string
	var customName myString = "Sam" //allowed
	customName = defaultName        //not allowed
	fmt.Println(customName)
}
```

go

[Run in playground](https://go.dev/play/p/sJ1rCtzebkP)

In the above code, we first create a variable `defaultName` and assign it to the constant `Sam`. **The default type of the constant `Sam` is a `string`, so after the assignment `defaultName` is of type `string`.**

In the next line, we create a new type `myString` which is an alias of `string`.

Then we create a variable `customName` of type `myString` and assign the constant `Sam` to it. Since the constant `Sam` is untyped, it can be assigned to any `string` variable. Hence this assignment is allowed and `customName` gets the type `myString`.

Now we have a variable `defaultName` of type `string` and another variable `customName` of type `myString`. Even though we know that `myString` is an alias of `string`, Go’s strong typing policy disallows variables of one type to be assigned to another. **Hence the assignment `customName = defaultName` is not allowed and the compiler throws the error `./prog.go:9:15: cannot use defaultName (variable of type string) as myString value in assignment`**

To make the above program work, `defaultName` must be converted to type `myString`. This is done in the following program in line no. 9

```go
package main

import "fmt"

func main() {
	var defaultName = "Sam" //allowed
	type myString string
	var customName myString = "Sam"    //allowed
	customName = myString(defaultName) //allowed
	fmt.Println(customName)
}
```

go

[Run in playground](https://go.dev/play/p/EObC2BTq4KW)

The above program will print `Sam`

### Boolean Constants

Boolean constants are no different from string constants. They are two untyped constants `true` and `false`. The same rules for string constants apply to booleans so we will not repeat them here. The following is a simple program to explain boolean constants.

```go
package main

func main() {
	const trueConst = true
	type myBool bool
	var defaultBool = trueConst //allowed
	var customBool myBool = trueConst //allowed
	defaultBool = customBool //not allowed
}
```

go

[Run in playground](https://go.dev/play/p/YWZ80x94Q1D)

The above program is self-explanatory.

### Numeric Constants

Numeric constants include integers, floats and complex constants. There are some subtleties in numeric constants.

Let’s look at some examples to make things clear.

```go
package main

import (
	"fmt"
)

func main() {
	const c = 5
	var intVar int = c
	var int32Var int32 = c
	var float64Var float64 = c
	var complex64Var complex64 = c
	fmt.Println("intVar", intVar, "\nint32Var", int32Var, "\nfloat64Var", float64Var, "\ncomplex64Var", complex64Var)
}
```

go

[Run in playground](https://go.dev/play/p/9MzdS-nmA1Z)

In the program above, the const `c` is `untyped` and has a value `5`. **You may be wondering what is the default type of c and if it does have one, how do we then assign it to variables of different types**. The answer lies in the syntax of `c`. The following program will make things more clear.

```go
package main

import (
	"fmt"
)

func main() {
	var i = 5
	var f = 5.6
	var c = 5 + 6i
	fmt.Printf("i's type is %T, f's type is %T, c's type is %T", i, f, c)
}
```

go

[Run in playground](https://go.dev/play/p/XVEGb9aUsxs)

In the program above, the type of each variable is determined by the syntax of the numeric constant. **5** is an integer by syntax, **5.6** is a float and **5 + 6i** is a complex number by syntax. When the above program is run, it prints

```fallback
i's type is int, f's type is float64, c's type is complex128
```

With this knowledge, let’s try to understand how the below program worked.

```go
package main

import (
	"fmt"
)

func main() {
	const c = 5
	var intVar int = c
	var int32Var int32 = c
	var float64Var float64 = c
	var complex64Var complex64 = c
	fmt.Println("intVar", intVar, "\nint32Var", int32Var, "\nfloat64Var", float64Var, "\ncomplex64Var", complex64Var)
}
```

go

[Run in playground](https://go.dev/play/p/9MzdS-nmA1Z)

In the program above, the value of `c` is `5` and the syntax of `c` is generic. It can represent a float, integer or even a complex number with no imaginary part. Hence it is possible to be assigned to any compatible type. The default type of these kinds of constants can be thought of as being generated based on the context where they are used. `var intVar int = c` requires `c` to be `int` so it becomes an `int` constant. `var complex64Var complex64 = c` requires `c` to be a complex number and hence it becomes a complex constant. Pretty neat :).

### Numeric Expressions

Numeric constants are free to be mixed and matched in expressions and a type is needed only when they are assigned to variables or used in any place in code which demands a type.

```go
package main

import (
	"fmt"
)

func main() {
	var a = 5.9 / 8
	fmt.Printf("a's type is %T and value is %v", a, a)
}
```

go

[Run in playground](https://go.dev/play/p/Nsak9scUAWg)

In the program above, `5.9` is a float by syntax and `8` is an integer by syntax. Still, `5.9/8` is allowed as both are numeric constants. The result of the division is `0.7375` is a `float` and hence variable `a` is of type `float`. The output of the program is

```fallback
a's type is float64 and value is 0.7375
```

This brings us to the end of this tutorial.

I hope you liked this tutorial. Please leave your feedback and comments. Please consider sharing this tutorial on [twitter](https://twitter.com/intent/tweet?text=Constants&url=https%3a%2f%2fgolangbot.com%2fconstants%2f&tw_p=tweetbutton) or [LinkedIn](https://golangbot.com/constants/#linkedinshr). Have a good day.
