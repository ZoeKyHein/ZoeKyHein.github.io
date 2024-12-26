---
title: 【GolangBot】31-自定义错误
date: 2024-12-26 11:09:39
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---
欢迎来到 [Golang 教程系列](../golangbot/) 的第 31 个教程。

在[上一个教程](../【GolangBot】30-错误处理)中，我们学习了 Go 中错误的表示方式以及如何处理标准库中的错误。我们还学习了如何从错误中提取更多信息。

本教程将介绍如何创建我们自己的自定义错误，以便在我们的函数和包中使用。我们还将使用标准库采用的相同技术来提供有关自定义错误的更多详细信息。

### 使用 New 函数创建自定义错误

创建自定义错误的最简单方法是使用 [errors](https://pkg.go.dev/errors) 包的 [New](https://pkg.go.dev/errors#New) 函数。

在我们使用 New [函数](../【GolangBot】6-函数)创建自定义错误之前，让我们先了解它的实现方式。[errors 包](https://go.dev/src/errors/errors.go?s=293:320#L58)中 New 函数的实现如下：

```go
package errors

// New 返回一个将给定文本格式化为错误的错误。
// 即使文本相同，每次调用 New 都会返回一个不同的错误值。
func New(text string) error {
    return &errorString{text}
}

// errorString 是 error 的一个简单实现。
type errorString struct {
    s string
}

func (e *errorString) Error() string {
    return e.s
}
```

实现非常简单。`errorString` 是一个具有单个字符串字段 `s` 的[结构体](../【GolangBot】16-结构体)类型。`error` 接口的 `Error() string` [方法](../【GolangBot】17-方法)使用 `errorString` 的[指针接收者](../【GolangBot】17-方法/#指针接收者与值接收者)在第 14 行实现。

第 5 行的 `New` 函数接受一个 `string` 参数，使用该参数创建一个 `errorString` 类型的值，并返回其地址。因此，一个新的错误被创建并返回。

现在我们知道 `New` 函数的工作原理了，让我们在自己的程序中使用它来创建一个自定义错误。

我们将创建一个简单的程序来计算圆的面积，并在半径为负时返回错误。

```go
package main

import (
	"errors"
	"fmt"
	"math"
)

func circleArea(radius float64) (float64, error) {
	if radius < 0 {
		return 0, errors.New("Area calculation failed, radius is less than zero")
	}
	return math.Pi * radius * radius, nil
}

func main() {
	radius := -20.0
	area, err := circleArea(radius)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("Area of circle %0.2f", area)
}
```

[在 Playground 中运行](https://go.dev/play/p/_vuf6fgkqm)

在上面的程序中，我们在第 10 行检查半径是否小于零。如果是，我们返回面积为 0 以及相应的错误消息。如果半径大于 0，则计算面积并在第 13 行返回 `nil` 作为错误。

在主函数中，我们在第 19 行检查错误是否为 `nil`。如果不是 `nil`，我们打印错误并返回，否则打印圆的面积。

在这个程序中，半径小于零，因此它将打印：

```fallback
Area calculation failed, radius is less than zero
```

### 使用 Errorf 为错误添加更多信息

上面的程序运行良好，但如果能打印导致错误的实际半径岂不是更好？这就是 [fmt](https://pkg.go.dev/fmt) 包的 [Errorf](https://pkg.go.dev/fmt#Errorf) 函数的用武之地。该函数根据格式说明符格式化错误，并返回一个满足 `error` 接口的[字符串](../【GolangBot】14-Strings)值。

让我们使用 `Errorf` 函数并使程序更好。

```go
package main

import (
	"fmt"
	"math"
)

func circleArea(radius float64) (float64, error) {
	if radius < 0 {
		return 0, fmt.Errorf("Area calculation failed, radius %0.2f is less than zero", radius)
	}
	return math.Pi * radius * radius, nil
}

func main() {
	radius := -20.0
	area, err := circleArea(radius)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("Area of circle %0.2f", area)
}
```

[在 Playground 中运行](https://go.dev/play/p/HQ7bvjT4o2)


在上面的程序中，`Errorf` 在第 10 行用于打印导致错误的实际半径。运行该程序将输出：

```fallback
Area calculation failed, radius -20.00 is less than zero
```

### 使用结构体类型和字段为错误提供更多信息

也可以使用实现 error [接口](../【GolangBot】18-接口-I)的结构体类型作为错误。这为错误处理提供了更大的灵活性。在我们之前的示例中，如果我们想访问导致错误的半径，唯一的方法是解析错误描述 `Area calculation failed, radius -20.00 is less than zero`。这不是一个合适的方式，因为如果描述发生变化，我们的代码将会崩溃。

我们将使用上一教程中“[将错误转换为底层类型并从结构体字段中检索更多信息](../【GolangBot】30-错误处理/#1-将错误转换为底层类型并从结构体字段中检索更多信息)”部分中描述的策略，并使用结构体字段来提供对导致错误的半径的访问。我们将创建一个实现 error 接口的结构体类型，并使用其字段来提供有关错误的更多信息。

第一步是创建一个表示错误的结构体类型。错误类型的命名约定是名称应以文本 `Error` 结尾。因此，我们将我们的结构体类型命名为 `areaError`。

```go
type areaError struct {
	err    string
	radius float64
}
```

上述结构体类型有一个 `radius` 字段，用于存储导致错误的半径值，`err` 字段存储实际的错误消息。

下一步是实现 error 接口。

```go
func (e *areaError) Error() string {  
    return fmt.Sprintf("radius %0.2f: %s", e.radius, e.err)
}
```

在上面的代码片段中，我们使用指针接收者 `*areaError` 实现了 error 接口的 `Error() string` 方法。此方法打印半径和错误描述。

让我们通过编写 `main` 函数和 `circleArea` 函数来完成程序。

```go
package main

import (
	"errors"
	"fmt"
	"math"
)

type areaError struct {
	err    string
	radius float64
}

func (e *areaError) Error() string {
	return fmt.Sprintf("radius %0.2f: %s", e.radius, e.err)
}

func circleArea(radius float64) (float64, error) {
	if radius < 0 {
		return 0, &areaError{
			err:    "radius is negative",
			radius: radius,
		}
	}
	return math.Pi * radius * radius, nil
}

func main() {
	radius := -20.0
	area, err := circleArea(radius)
	if err != nil {
		var areaError *areaError
		if errors.As(err, &areaError) {
			fmt.Printf("Area calculation failed, radius %0.2f is less than zero", areaError.radius)
			return
		}
		fmt.Println(err)
		return
	}
	fmt.Printf("Area of rectangle %0.2f", area)
}
```

[在 Playground 中运行](https://go.dev/play/p/X4GvrehVTGA)

在上面的程序中，`circleArea` 函数在第 18 行用于计算圆的面积。该函数首先检查半径是否小于零，如果是，则使用导致错误的半径和相应的错误消息创建一个 `areaError` 类型的值，然后在第 20 行返回其地址以及面积为 0。**因此，我们使用自定义错误结构体的字段提供了有关错误的更多信息，在这种情况下是导致错误的半径。**

如果半径不为负，则此函数计算并返回面积，并在第 25 行返回 `nil` 错误。

在主函数的第 30 行，我们尝试计算半径为 -20 的圆的面积。由于半径小于零，将返回错误。

我们在第 31 行检查错误是否为 `nil`，并在第 33 行尝试将其转换为 `*areaError` 类型。**如果错误是 `*areaError` 类型，我们在第 34 行使用 `areaError.radius` 获取导致错误的半径，打印自定义错误消息并从程序返回。**

如果错误不是 `*areaError` 类型，我们只需在第 37 行打印错误并返回。如果没有错误，则会在第 40 行打印面积。

该程序将打印：

```fallback
Area calculation failed, radius -20.00 is less than zero
```

现在让我们使用上一教程中描述的[第二种策略](../【GolangBot】30-错误处理/#2-使用方法检索更多信息)，并在自定义错误类型上使用方法以提供有关错误的更多信息。


### 使用结构体类型上的方法为错误提供更多信息

在本节中，我们将编写一个计算矩形面积的程序。如果长度或宽度小于零，该程序将打印错误。

第一步是创建一个表示错误的结构体。

```go
type areaError struct {
	err    string //错误描述
	length float64 //导致错误的长度
	width  float64 //导致错误的宽度
}
```

上述错误结构体类型包含一个错误描述字段以及导致错误的长度和宽度。

现在我们有了错误类型，让我们实现 error 接口，并在错误类型上添加一些方法以提供有关错误的更多信息。

```go
func (e *areaError) Error() string {
	return e.err
}

func (e *areaError) lengthNegative() bool {
	return e.length < 0
}

func (e *areaError) widthNegative() bool {
	return e.width < 0
}
```

在上面的代码片段中，我们从 `Error() string` 方法返回错误的描述。`lengthNegative() bool` 方法在长度小于零时返回 true，`widthNegative() bool` 方法在宽度小于零时返回 true。**这两种方法提供了有关错误的更多信息，在这种情况下，它们表明面积计算失败是由于长度为负还是宽度为负。因此，我们使用了结构体错误类型上的方法来提供有关错误的更多信息。**

下一步是编写面积计算函数。

```go
func rectArea(length, width float64) (float64, error) {
	err := ""
	if length < 0 {
		err += "length is less than zero"
	}
	if width < 0 {
		if err == "" {
			err = "width is less than zero"
		} else {
			err += ", width is less than zero"
		}
	}
	if err != "" {
		return 0, &areaError{
			err:    err,
			length: length,
			width:  width,
		}
	}
	return length * width, nil
}
```

上面的 `rectArea` 函数检查长度或宽度是否小于零，如果是，则返回 `*areaError` 类型的错误，否则返回矩形的面积并将 `nil` 作为错误。

让我们通过创建主函数来完成这个程序。

```go
func main() {
	length, width := -5.0, -9.0
	area, err := rectArea(length, width)
	if err != nil {
		var areaError *areaError
		if errors.As(err, &areaError) {
			if areaError.lengthNegative() {
				fmt.Printf("error: length %0.2f is less than zero\n", areaError.length)
			}
			if areaError.widthNegative() {
				fmt.Printf("error: width %0.2f is less than zero\n", areaError.width)
			}
			return
		}
		fmt.Println(err)
		return
	}
	fmt.Println("area of rect", area)
}
```

在主函数中，我们在第 4 行检查错误是否为 `nil`。如果不是 nil，我们尝试将其转换为 `*areaError` 类型。然后使用 `lengthNegative()` 和 `widthNegative()` 方法，我们检查错误是否是由于长度为负或宽度为负。我们打印相应的错误消息并从程序返回。**因此，我们使用了错误结构体类型上的方法来提供有关错误的更多信息。**

如果没有错误，将打印矩形的面积。

以下是完整的程序供您参考。

```go
package main

import (
	"errors"
	"fmt"
)

type areaError struct {
	err    string  //错误描述
	length float64 //导致错误的长度
	width  float64 //导致错误的宽度
}

func (e *areaError) Error() string {
	return e.err
}

func (e *areaError) lengthNegative() bool {
	return e.length < 0
}

func (e *areaError) widthNegative() bool {
	return e.width < 0
}

func rectArea(length, width float64) (float64, error) {
	err := ""
	if length < 0 {
		err += "length is less than zero"
	}
	if width < 0 {
		if err == "" {
			err = "width is less than zero"
		} else {
			err += ", width is less than zero"
		}
	}
	if err != "" {
		return 0, &areaError{
			err:    err,
			length: length,
			width:  width,
		}
	}
	return length * width, nil
}

func main() {
	length, width := -5.0, -9.0
	area, err := rectArea(length, width)
	if err != nil {
		var areaError *areaError
		if errors.As(err, &areaError) {
			if areaError.lengthNegative() {
				fmt.Printf("error: length %0.2f is less than zero\n", areaError.length)
			}
			if areaError.widthNegative() {
				fmt.Printf("error: width %0.2f is less than zero\n", areaError.width)
			}
			return
		}
		fmt.Println(err)
		return
	}
	fmt.Println("area of rect", area)
}
```

[在 Playground 中运行](https://go.dev/play/p/9xxF9fVw1fe)

该程序将输出：

```fallback
error: length -5.00 is less than zero
error: width -9.00 is less than zero
```

我们已经看到了在[错误处理](../【GolangBot】30-错误处理/)教程中描述的三种方式中的两种示例，以提供有关错误的更多信息。

第三种方式使用[直接比较](../【GolangBot】30-错误处理/#3-直接比较)非常简单。我将把它留作练习，让您弄清楚如何使用此策略来提供有关我们自定义错误的更多信息。

本教程到此结束。

以下是我们在本教程中学到的内容的快速回顾：

- 使用 New 函数创建自定义错误
- 使用 Errorf 为错误添加更多信息
- 使用结构体类型和字段为错误提供更多信息
- 使用结构体类型上的方法为错误提供更多信息

**下一个教程 - [错误包装](../_posts/【GolangBot】32-错误包装)**
