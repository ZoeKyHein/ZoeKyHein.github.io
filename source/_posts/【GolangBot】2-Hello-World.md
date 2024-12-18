---
title: 【GolangBot】2-Hello World
date: 2024-12-17 08:47:41
tags: 
    - GoLang
    - 教程
categories:
    - [学习心得, GoLangBot]
---

# 【GolangBot】2-Hello World
这是我们[Golang系列教程](../golangbot/)的第二篇。请阅读我们前一篇教程[Golang安装和介绍](../【GolangBot】1-介绍和安装/)来了解什么是Golang，以及如何安装Golang。

## 安装开发环境

让我们在想要写Hello World程序的位置创建一个目录。打开终端运行如下命令。

```fallback
mkdir ~/Documents/learngo/
```

上面的命令将会在当前用户的文档目录下创建一个名为`learngo`的目录。你可以随意在希望存放代码的位置创建目录。

## 创建一个Go Module

下一步是在`~/Documents/learngo/`文件夹中创建一个名为`learngo`的go module。Go modules 用来追踪应用的依赖和版本。我们将会在我们学习[包](../【GolangBot】7-包/)的时候详细讨论Go modules。

在`~/Documents/learngo/`目录中运行`go mod init learngo`。这将会创建一个名为`go.mod`的文件。运行`go mod init learngo`命令后，下面的内容将会被打印出来

Run `go mod init learngo` inside the `~/Documents/learngo/` directory. This will create a file named `go.mod`. The following will be printed after running the `go mod init learngo` command

```v
go: creating new go.mod: module learngo
```

`go.mod`文件的内容如下。

```v
module learngo

go 1.21.0
```

The first line `module learngo` specifies the module name. The next line `1.21.0` indicates that the files in this module use go version 1.21.0

第一行`module learngo`制定了module的名称。下一行`1.21.0`表明在这个module当中，使用的go版本是1.21.0。

## Hello World

用你最喜欢的文本编辑器在`learngo`目录下创建一个名为`main.go`的文件，其中包含以下内容。

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello World")
} 
```

按照Go的管理，包含`main`函数的文件命名为`main.go`，但其他名称也同样适用。

## 运行go程序

有几种运行Go程序的不同方式。让我们一个个地来看。

### 1. go install

第一种运行Go程序的方法是使用`go install`命令。让我们`cd`进我们刚刚创建的`learngo`目录。

```fallback
cd ~/Documents/learngo/
```

运行如下命令。

```fallback
go install
```

上面的命令将会变异程序并且安装(拷贝)二进制文件到`~/go/bin`路径。二进制文件的名称是go module的名称。在我们的示例中，它会被命名为`learngo`。

当你尝试安装程序的时候，你可能会碰到下面的错误。

```fallback
go install: no install location for directory /home/naveen/Documents/learngo outside GOPATH
For more details see: 'go help gopath'
```

上面的错误通常的意思是指，`go install`找不到位置安装编译后的二进制文件。所以让我们继续，并且给他一个位置。这个位置是由`GOBIN`环境变量控制的。

```fallback
export GOBIN=~/go/bin/
```

上面的环境变量制定了`go install`应该拷贝并且编译二进制文件到`~go/bin`路径。这是对Go的二进制文件来说一个约定俗成的路径，但你也可以随心所欲的修改它到任意你想要的位置。现在试着重新运行`go install`，程序应该会正常编译并且运行了。

你可以在终端里面输入`ls -al ~/go/bin/learngo`，你会发现实际上`go install`已经把二进制文件放到了`~/go/bin`路径下。

现在，让我们来运行编译后的二进制文件。

```fallback
~/go/bin/learngo
```

上面的命令将会运行`learngo`二进制文件并且打印如下输出。

```fallback
Hello World
```

恭喜！你已经成功运行了你的第一个Go程序。

如果你不想每次运行程序的时候都输入完整的路径`~/go/bin/learngo`，你可以把`~/go/bin/`添加到你的PATH环境变量中。

```fallback
export PATH=$PATH:~/go/bin
```

现在，你只需要在终端输入`learngo`就可以运行你的程序了。

你也许想知道当`learngo`目录包含多个go文件而不只有一个`main.go`时会发生什么。在这种情况下`go install`会发生什么？请别着急，我们将会在[包和Go Modules](../【GolangBot】7-包/)一节中讨论这些。

### 2. go build

运行程序的第二个选择是使用`go build`。`go build`和`go install`非常相似，除了它不会在`~/go/bin/`路径下安装(拷贝)二进制文件。相反，它会在安装`go build`的位置创建二进制文件。

在终端输入以下命令将当前目录更改为`learngo`。

```fallback
cd ~/Documents/learngo/
```

在那之后，输入下面的命令。

```fallback
go build
```

上方的命令会在当前目录创建一个名为`learngo`的二进制文件。运行`ls -al`会发现一个名为`learngo`的文件被创建了。

输入`./learngo`运行程序。这也会打印

```fallback
Hello World
```

我们用`go build`同样成功运行了我们的第一个Go程序:)

### 3. go run

第三种运行程序的方式是使用`go run`命令。

在终端中输入`cd ~/Documents/learngo/`命令来改变当前目录到`learngo`。

然后，输入如下的命令。

```fallback
go run main.go
```

执行上面的命令后，我们会看到输出

```fallback
Hello World
```

`go run`与`go build/go install`命令的一个不同就在于，`go run`需要`.go`文件的名字作为一个参数。

在底层，`go run`和`go build`非常相似。但`go run`不在当前目录编译和安装程序，而是向一个临时目录编译文件，并在该目录运行文件。如果你有兴趣想知道`go run`把文件编译到了哪里，请在运行`go run`的时候加上`--work`参数。

```fallback
go run --work main.go
```

运行上面的命令，我这里输出

```fallback
WORK=/tmp/go-build199689936
Hello World
```

关键字`WORK`的值制定了程序将会被编译到的临时位置。在我的示例中，程序已经被编译到了`/tmp/go-build199689936`。你的运行情况可能有所不同:)

### 4. Go在线运行环境。

最后一种方式是使用Go在线运行环境。尽管有一些限制，但当我们想要运行简单的程序时，此方法将会派上用场，它使用浏览器运行，不需要在本地安装GO:)。我已经为Hello World 程序创建了一个在线环境。[点击这里](https://go.dev/play/p/oXGayDtoLPh)在线运行程序。

你也可以使用Go在线运行环境来与其他人分享你的源码。

现在我们知道了运行程序的四种不同的方式，你可能纠结于使用哪一种。答案是，视情况而定。当我想要做一个快速逻辑检查或一个标准库函数是否生效时，我一般会选择在线环境。在大多数情况下，我更喜欢`go install`，因为它给我一种选择，让我可以在终端里可以从任意目录运行程序，因为它将所有的程序编译到靠准`~/go/bin`路径。

### 对Hello World程序的简短解释

这里是我们刚刚写的Hello World程序。

```go
package main 

import "fmt" 

func main() { 
	fmt.Println("Hello World") 
}
```

我们会简要讨论程序的每一行的作用。我们将在接下来的每一篇教程中深入探讨程序的各个部分。

**package main - 每一个go文件都必须从`package name`语句开始**。包用于提供代码的分隔和可重用性。这里用到了包名`main`。`main`函数应该永远处于main包中。

**import “fmt”** - import语句用来引用其他的包。在我们的示例中，`fmt`包被引用并且它用于在main函数中向标准输出打印文本。

**fmt.Println(“Hello World”)** - `fmt`包的 `Println` 函数用于向标准输出打印文本。`package.Function()`是调用包内函数的格式。

代码可以在[GitHub](https://github.com/golangbot/hello)上下载。

你现在可以去[【GolangBot】3-变量]()来学习Go中的变量。

**下一篇教程 - [变量](../【GolangBot】3-变量/)**