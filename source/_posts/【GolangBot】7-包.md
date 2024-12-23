---
title: 【GolangBot】7-包
date: 2024-12-18 16:48:29
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---



欢迎来到[Golang系列教程](../golangbot/)的第七篇。

### 什么是包？我们为什么要用它？

到目前为止，我们已经看过了只有一个文件的Go程序，这个文件中有一个main[函数](../【GolangBot】6-函数/)和一些其他的函数。在实际的场景中，把所有的源码都写在一个文件中是不可扩展的。这样编写的代码不太可能复用和维护。这时候包就派上用场了。

**包被用于管理源代码，使其有更好的复用性和可读性。包是所有处于同一目录中文件的集合。包提供了代码分隔功能，因此可以轻松的维护Go项目。**

例如，加入我们正在用Go编写一个金融科技应用，其中一些功能包括单利计算、复利计算和贷款偿还计算。管理这个应用的一个简单的方式是通过功能划分。我们可以创建包`simpleinterest`, `compoundinterest` 和 `loanrepayment`。如果`loanrepayment`包需要计算单利，它可以直接导入`simpleinterest`包来完成。这样代码就实现了复用。

我们将通过创建一个简单的应用来学习包，在给定本金、利率和以年为单位的时间期限的情况下确定单利。

### Go Modules

我们将以这样的方式来组织代码，所有和单利相关的功能都放在`simpleinterest`包。为了实现目的，我们需要创建一个自定义包`simpleinterest`，在这里包含了计算单利的函数。在创建自定义包之前，我们需要先理解Go Modules，因为**Go Modules**被用于创建自定义包。

**Go Module只是Go软件包的集合。**现在你可能想到了一个问题，我们为什么需要Go Modules来创建自定义包？答案是**我们创建的自定义包的导入路径来自Go Modules的名字**。除此之外，所有的第三方包(例如来自github的源码)及其版本都由`go.mod`文件管理。`go.mod`文件是我们在创建新模块时创建的。你会在下一节更好地理解它。

### 创建一个Go module

运行下面的命令，在当前用户的文档目录下创建一个名为learnpackage的目录。

```fallback
mkdir ~/Documents/learnpackage/ 
```

通过输入`cd !/Documents/learnpackage`命令确保你在目录`learnpackage`当中。在这个目录中运行下面的命令，创建一个名为*learnpackage*的go module。

```fallback
go mod init learnpackage
```

上述命令会创建一个名为`go.mod`的文件。下面是文件中将会出现的内容。

```v
module learnpackage

go 1.21.0
```

`module learnpackage`表明模块的名称是`learnpackage`。正如我们前面提到的，`learnpackage`将是导入任何在这个模块中创建的包的基础路径。`go 1.21.0`指定这个模组中的文件使用的go版本是`1.21.0`。

### 创建一个简单的单利自定义包

**属于同一个包的所有Go文件应该被放在它们所属的单独的文件夹内。Go的惯例是将该文件夹命名为与包相同的名称。**

让我们在`learnpackage`文件夹中创建一个名为`simpleinterest`的文件夹，`mkdir simpleinterest`命令将会为我们创建这个文件夹。

**`package packagename`指定某个特定的.go文件属于`packagename`包。这应该在每个源文件的第一行。**因此所有*simpleinterest*文件夹中的文件都应该以这行代码开始，因为它们都属于`simpleinterest`包。

```go
package simpleinterest
```

在 *simpleinterest*包中创建一个`simpleinterest.go`文件。

下面是我们应用的目录结构。

```fallback
learnpackage/
├── go.mod
└── simpleinterest
    └── simpleinterest.go
```

在`simpleinterest.go`文件中添加如下代码。

```go
1package simpleinterest
2
3//Calculate calculates and returns the simple interest for a principal p, rate of interest r for time duration t years
4func Calculate(p float64, r float64, t float64) float64 {
5	interest := p * (r / 100) * t
6	return interest
7}
```

在上面的代码中，我们已经创建了`Calculate`函数，它能够计算并返回单利。这个函数的作用不言自明。

*请注意，函数名称Calculate以大写字母开头。这一点很重要，我们很快就会解释为什么需要这么做。*

### main包和main函数

教程的下一步是导入我们刚刚创建的`simpleinterest`包并且使用它。我们将从`main`包中导入simpleinterest包。让我们首先来理解一下`main`函数和`main`包。

每个可执行的Go应用都应该包含`main`函数。这个函数是执行的入口点。`main`函数应该位于`main`包中。

让我们通过为我们的应用创建`main`函数和`main`包来开始。

在我们的`learnpackage`目录下创建一个名为`main.go`的文件，其中有以下内容。

```go
package main 

import "fmt"

func main() { 
	fmt.Println("Simple interest calculation")
}
```

`package main`指定这个文件属于main包。`import packagename`语句用来导入一个已经存在的包。`packagename.FunctionName()`是调用包中函数的语法。

在第3行，我们导入了`fmt`包来使用`Println`函数。`fmt`是一个标准包，作为Go标准库的一部分提供。然后是打印`Simple interest calculation`的`main`函数。

使用如下命令移动到`learnpackage`目录

```fallback
cd ~/Documents/learnpackage/
```

用下面的命令，编译上面的程序

```fallback
go build
```

如果一切正常，我们的二进制文件将会被编译出来，而且准备好运行。在终端中输入`./learnpackage`命令，你会看到下面的输出。

```fallback
Simple interest calculation
```

如果你不理解`go build`是如何工作的，请前往[Hello World教程](../【GolangBot】2-Hello-World/)了解详情。

### 在main包中印入simpleinterest包

为了使用自定义包，我们必须先导入它。导入路径是由包所在目录和包名称连接的go module的名称。

在我们的示例中，go module的名称是`learnpackage`，`simpleinterest`包在`learnpackage`正下方的`simpleinterest`文件夹中。

```fallback
├── learnpackage
│   └── simpleinterest
```

所以`import "learnpackage/simpleinterest"`这一行会引入*simpleinterest*包。

在这种情况下，我们的目录结构是这样的。

```fallback
learnpackage
│   └── finance
│       └── simpleinterest 
```

导入语句就是`import "learnpackage/finance/simpleinterest"`。

添加如下代码到`main.go`。

```go
package main

import (
	"fmt"
	"learnpackage/simpleinterest"
)

func main() {
	fmt.Println("Simple interest calculation")
	p := 5000.0
	r := 10.0
	t := 1.0
	si := simpleinterest.Calculate(p, r, t)
	fmt.Println("Simple interest is", si)
}
```

上面的代码引入了`simpleinterest`包并且使用了`Calculate`函数求单利。标准库中的包不需要使用模块名作为前缀，因此`fmt`不需要模组前缀就可以工作。当程序运行时，输出将会是

```fallback
Simple interest calculation
Simple interest is 500
```

### 关于go build的更多内容

现在我们理解了包如何工作，是时候聊一下`go build`了。像`go build`的Go工具在当前目录的上下文中工作。我们来理解一下这是什么意思。到目前为止，我们一直在`~/Documents/learnpackage`中运行`go build`。如果我们尝试在其他目录中运行，它就会报错。

试着用``cd ~/Documents/``进入`~/Documents/`，然后运行`go build learnpackage`。它会报下面的错误。

```go
package learnpackage is not in std (/usr/local/go/src/learnpackage)
```

我们来理解一下错误背后的原因。`go build`用一个可选的报名作为参数(在我们的例子中包名是`learnpackage`)，如果运行时的当前目录、它的父级目录或者父级目录的父级目录，以此类推……存在包，他就会尝试编译main函数。

我们在`Documents`目录中，这里没有`go.mod`文件，因此`go build`抱怨它找不到`learnpackage`包。

当我们移动到`~/Documents/learnpackage`,这里有`go.mod`文件，并且在那个`go.mode`文件中，我们的模组名就是`learnpackage`。

所以`go build learnpackage`在`~/Documents/learnpackage/`目录中就会奏效。

但是到目前为止，我们只是使用了`go build`但并没有指定一个包名。如果没有包名被指定，`go build`会默认使用当前工作目录下的模块名称。这就是为什么在`~/Documents/learnpackage/`中不带包名运行`go build`正常的原因。所以当我们在`~/Documents/learnpackage/`中运行时，以下3种命令都是同样的。

```fallback
go build

go build .

go build learnpackage
```

我也提到了`go build`能够递归搜索父目录的go.mod文件。让我们看下这是否有效。

```fallback
cd ~/Documents/learnpackage/simpleinterest/
```

上面的命令让我们进入`simpleinterest`目录。在这个目录运行

```fallback
go build learnpackage
```

`go build`将会从定义了`learnpackage`模组的`learnpackage`目录成功找到`go.mod`文件，因此它正常运行:)。

还可以使用`go build`更改输出二进制文件的位置。移至`~/Documents/learnpackage`并键入

```fallback
1go build -o fintechapp
```

`-o`参数用于指定输出的二进制文件的名称。在示例中，一个名为`fintechapp`的二进制文件将会被创建。运行`./fintechapp`，二进制文件将会成功运行。

### 导出的名称

我们在计算单利包中大写了`Calculate`函数。这在Go中有特殊意义。在go中任意大写字母开头的[变量](../【GolangBot】3-变量/)或者函数都是可导出的名称。只有可导出函数和变量才在其他包中可用。在我们的示例中，我们想要在main包中使用`Calculate`函数。因此这是大写。

如果在`simpleinterest.go`中，函数名称从`Calculate`改为`calculete`，如果我们尝试在main.go中使用``simpleinterest.calculate(p, r, t)``，编译会出错。

```fallback
# learnpackage
./main.go:13:8: undefined: simpleinterest.calculate
```

因此如果你想在包外使用函数，它就应该被大写。

### init函数

Go的每一个包都可以有一个`init`函数。`init`函数必须不含邮任何返回类型，也不能有任何参数。init函数不能在我们的源码中被显式调用。当包被初始化的时候它会被自动调用。init函数有如下格式	

```fallback
func init() {
}
```

`init`函数可被用于执行初始化任务，也可以用来在执行开始前验证程序的正确性。

一个包的初始化顺序如下：

1. 包等级变量首先被初始化
2. init函数接着被调用。一个包可以有多个init函数(可以在单个文件中，也可以分布在多个文件当中)，并且它们依照出现在编译器的顺序被依次调用。

如果一个包引入了其他包，被引入的包首先初始化。

如果一个包被多个包引用，它只会初始化一次。

让我们在我们的应用上做些修改，来理解`init`函数。

首先，我们在`simpleinterest.go`文件添加一个`init`函数。

```go
package simpleinterest

import "fmt"

/*
 * init function added
 */
func init() {
	fmt.Println("Simple interest package initialized")
}

//Calculate calculates and returns the simple interest for principal p, rate of interest r for time duration t years
func Calculate(p float64, r float64, t float64) float64 {
	interest := p * (r / 100) * t
	return interest
}
```

我们已经添加了一个简单的init函数，它只打印`Simple interest package initialised`。

现在让我们修改main包。我们知道，但计算单利的时候，本金、利率和时间期限应该比0大。我们会在`main.go`文件中使用init函数和包级别变量定义一个检查。

按如下修改`main.go`。

```go
package main 

import (
	"fmt"
	"learnpackage/simpleinterest" //importing custom package
	"log"
)

var p, r, t = 5000.0, 10.0, 1.0

/*
* init function to check if p, r and t are greater than zero
 */
func init() {
	fmt.Println("Main package initialized")
	if p < 0 {
		log.Fatal("Principal is less than zero")
	}
	if r < 0 {
		log.Fatal("Rate of interest is less than zero")
	}
	if t < 0 {
		log.Fatal("Duration is less than zero")
	}
}

func main() {
	fmt.Println("Simple interest calculation")
	si := simpleinterest.Calculate(p, r, t)
	fmt.Println("Simple interest is", si)
}
```

下面是对`main.go`文件所做的修改

1. **p**, **r** 和 **t** 从main函数级别移到了package级别。
2. 添加了一个init函数。*init*函数打印一条日志，并通过**log.Fatal**在本金、利率或时间期限小于0时终止程序的运行。

初始化的顺序如下，

1. 首先初始化导入的包。因此**simpleinterest**包首先被初始化，init方法被调用。
2. package级别变量**p**, **r** 和 **t**被初始化。
3. main包中的**init**函数被调用。
4. **main**函数最后被调用。

如果你运行程序，你会得到如下的输出。

```go
Simple interest package initialized
Main package initialized
Simple interest calculation
Simple interest is 500
```

正如我们所预期的，`simpleinterest`包首先被调用，接下来是package级别的变量`p`, `r` 和 `t`被初始化。main包中的init函数接着被调用，它检查`p`, `r` 和 `t`是否小于0，在条件为true的时候终止程序。我们将会在[单独的教程](../【GolangBot】8-if-else语句/)中详细学习`if`语句。到目前为止，可以假设。`if p < 0`会检查`p`是否小于0，如果它确实小于0，程序将被终止。我们已经为`r`和`t`写了类似的条件。在这个例子中，所有这些条件都是false，程序执行会继续。最后，main函数被调用。

让我们微调一下程序，学习init函数的使用。

在`main.go`中找到这一行，

```fallback
var p, r, t = 5000.0, 10.0, 1.0
```

修改为

```fallback
var p, r, t = -5000.0, 10.0, 1.0
```

我们已经把`p`初始化成了负数。

现在如果你运行程序，你会看到

```go
Simple interest package initialized
Main package initialized
2024/04/01 02:58:32 Principal is less than zero
```

*p*是负数。因此当init函数运行时，程序会在打印`Principal is less than zero`后终止。

### 空白标识符的使用

在Go中，引入一个包但是不使用它是不合法的。如果你这么做，编译器会抱怨。原因是避免为使用的包膨胀，浙江显著增加编译时间。把`main.go`中的代码替换为如下所示

```go
package main
  
import (
        "learnpackage/simpleinterest"
)

func main() {

}
```

上面的程序将会报错

```fallback
# learnpackage
./main.go:4:2: imported and not used: "learnpackage/simpleinterest"
```

但是，当程序正在处于开发活跃状态时，导入包并稍后(如果不是现在)在代码中的某个位置使用它们是很常见的。在这些情况下，`_`空白标识符可以拯救我们。

上面的程序的错误可以被下面的代码改动所消除。

```go
package main
  
import (
        "learnpackage/simpleinterest"
)

var _ = simpleinterest.Calculate

func main() {

}
```

这一行，`var _ = simpleinterest.Calculate`,屏蔽了错误。我们应该跟踪这些类型的错误消音器，并在应用程序开发结束时移除它们，包括导入的包(如果不使用的话)。因此，建议在导入语句后在package级别编写错误消音器。

有时候，我们需要导入一个包只是为了确保该包的初始化过程得以执行，即使我们并不需要在代码中使用该包的任何函数或变量。例如，我们可能需要确保 `simpleinterest` 包中的 `init` 函数被调用，即使我们并不打算在代码的任何地方使用这个包。在这种情况下，也可以使用**空白标识符** _ 来实现，如下所示。

```go
package main

import (
	_ "learnpackage/simpleinterest"
)

func main() {

}
```

运行上述代码，程序会输出`Simple interest package initialized`。我们已经成功初始化了`simpleinterest`包，即使它没有在任何地方使用。

**下一篇教程 - [if else 语句](../【GolangBot】8-if-else语句/)**
