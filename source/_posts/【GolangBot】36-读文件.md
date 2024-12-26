---
title: 【GolangBot】36-读文件
date: 2024-12-26 13:10:08
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---
欢迎来到 [Golang 教程系列](../golangbot/) 的第 36 个教程。

文件读取是任何编程语言中最常见的操作之一。在本教程中，我们将学习如何使用 Go 读取文件。

本教程包含以下部分：

- 将整个文件读入内存
  - 使用绝对文件路径
  - 将文件路径作为命令行标志传递
  - 将文件打包到二进制文件中
- 分块读取文件
- 逐行读取文件

### 将整个文件读入内存

最基本的文件操作之一是将整个文件读入内存。这可以通过 [os](https://pkg.go.dev/os) 包中的 [ReadFile](https://pkg.go.dev/os#ReadFile) 函数来实现。

让我们读取一个文件并打印其内容。

我通过运行 `mkdir ~/Documents/filehandling` 在 `Documents` 目录下创建了一个文件夹 `filehandling`。

在 `filehandling` 目录下运行以下命令，创建一个名为 `filehandling` 的 Go 模块。

```fallback
go mod init filehandling
```

我有一个文本文件 `test.txt`，它将从我们的 Go 程序 `filehandling.go` 中读取。`test.txt` 包含以下字符串：

```fallback
Hello World. Welcome to file handling in Go.
```

这是我的目录结构：

```fallback
├── Documents
│   └── filehandling
│       ├── filehandling.go
|       ├── go.mod
│       └── test.txt
```

让我们直接看代码。创建一个文件 `filehandling.go`，内容如下：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	contents, err := os.ReadFile("test.txt")
	if err != nil {
		fmt.Println("File reading error", err)
		return
	}
	fmt.Println("Contents of file:", string(contents))
}
```

请在本地环境中运行此程序，因为无法在 playground 中读取文件。

程序第 9 行读取文件并返回一个字节 [slice](../【GolangBot】11-数组和切片/#切片)，存储在 `contents` 中。在第 14 行，我们将 `contents` 转换为 `string` 并显示文件内容。

请在 **test.txt** 所在的目录下运行此程序。

如果 **test.txt** 位于 **~/Documents/filehandling**，则使用以下步骤运行此程序：

```fallback
cd ~/Documents/filehandling/
go install 
filehandling
```

如果你不知道如何运行 Go 程序，请访问 [/hello-world-gomod/](../GolangBot】2-Hello-World) 了解更多。如果你想了解更多关于包和 Go 模块的信息，请访问 [/go-packages/](../【GolangBot】7-包)

该程序将打印：

```fallback
Contents of file: Hello World. Welcome to file handling in Go.  
```

如果从其他位置运行此程序，例如尝试从 `~/Documents/` 运行程序：

```fallback
cd ~/Documents/
filehandling
```

它将打印以下错误：

```fallback
File reading error open test.txt: no such file or directory
```

原因是 Go 是一种编译型语言。`go install` 所做的是从源代码创建一个二进制文件。二进制文件独立于源代码，可以从任何位置运行。由于在运行二进制文件的位置找不到 `test.txt`，程序会抱怨找不到指定的文件。

有三种方法可以解决这个问题：

1. 使用绝对文件路径
2. 将文件路径作为命令行标志传递
3. 将文本文件与二进制文件打包在一起

让我们逐一讨论。


##### 1. 使用绝对文件路径

解决此问题的最简单方法是传递绝对文件路径。我修改了程序，并在第 9 行将路径更改为绝对路径。请将此路径更改为你的 `test.txt` 的绝对路径。

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	contents, err := os.ReadFile("/Users/naveen/Documents/filehandling/test.txt")
	if err != nil {
		fmt.Println("File reading error", err)
		return
	}
	fmt.Println("Contents of file:", string(contents))
}
```

现在，程序可以从任何位置运行，并打印 `test.txt` 的内容。

例如，即使我从主目录运行它，它也会工作：

```fallback
cd ~/Documents/filehandling
go install
cd ~
filehandling
```

该程序将打印 `test.txt` 的内容。

这似乎是一种简单的方法，但有一个缺点，即文件必须位于程序中指定的路径中，否则此方法将失败。

##### 2. 将文件路径作为命令行标志传递

另一种解决此问题的方法是将文件路径作为命令行参数传递。使用 [flag](https://pkg.go.dev/flag) 包，我们可以从命令行获取文件路径作为输入参数，然后读取其内容。

让我们首先了解 `flag` 包的工作原理。`flag` 包有一个 [String](https://golang.org/pkg/flag/#String) 函数。该函数接受 3 个参数。第一个是标志的名称，第二个是默认值，第三个是标志的简短描述。

让我们编写一个小程序来从命令行读取文件名。将 `filehandling.go` 的内容替换为以下内容：

```go
package main
import (
	"flag"
	"fmt"
)

func main() {
	fptr := flag.String("fpath", "test.txt", "file path to read from")
	flag.Parse()
	fmt.Println("value of fpath is", *fptr)
}
```

程序第 8 行使用 `String` 函数创建了一个名为 `fpath` 的字符串标志，默认值为 `test.txt`，描述为 `file path to read from`。此函数返回存储标志值的字符串 [变量](https://golangbot.com/variables/) 的地址。

在访问任何标志之前，应调用 *flag.Parse()*。

我们在第 10 行打印标志的值。

当使用以下命令运行此程序时：

```fallback
filehandling -fpath=/path-of-file/test.txt
```

我们将 `/path-of-file/test.txt` 作为标志 `fpath` 的值传递。

该程序输出：

```fallback
value of fpath is /path-of-file/test.txt
```

如果仅使用 `filehandling` 运行程序而不传递任何 `fpath`，它将打印：

```fallback
value of fpath is test.txt
```

因为 `test.txt` 是 `fpath` 的默认值。

`flag` 还提供了一个格式良好的输出，显示可用的不同参数。可以通过运行以下命令显示：

```fallback
filehandling --help
```

此命令将打印以下输出：

```fallback
Usage of filehandling:
  -fpath string
    	file path to read from (default "test.txt")
```

不错吧 :)。

现在我们知道如何从命令行读取文件路径，让我们继续完成我们的文件读取程序。

```go
package main

import (
	"flag"
	"fmt"
	"os"
)

func main() {
	fptr := flag.String("fpath", "test.txt", "file path to read from")
	flag.Parse()
	contents, err := os.ReadFile(*fptr)
	if err != nil {
		fmt.Println("File reading error", err)
		return
	}
	fmt.Println("Contents of file:", string(contents))
}
```

上面的程序读取从命令行传递的文件路径的内容。使用以下命令运行此程序：

```fallback
filehandling -fpath=/path-of-file/test.txt
```

请将 `/path-of-file/` 替换为 `test.txt` 的绝对路径。例如，在我的情况下，我运行了以下命令：

```fallback
filehandling --fpath=/Users/naveen/Documents/filehandling/test.txt
```

程序打印：

```fallback
Contents of file: Hello World. Welcome to file handling in Go.
```

##### 3. 将文本文件与二进制文件打包在一起

上述从命令行获取文件路径的选项很好，但有一种更好的方法可以解决此问题。如果能够将文本文件与我们的二进制文件打包在一起，那不是很棒吗？这就是我们接下来要做的。

标准库中的 [embed](https://pkg.go.dev/embed) 包将帮助我们实现这一点。

导入 `embed` 包后，可以使用 `//go:embed` 指令读取文件内容。

一个程序将帮助我们更好地理解这一点。

将 `filehandling.go` 的内容替换为以下内容：

```go
package main

import (
	_ "embed"
	"fmt"
)

//go:embed test.txt
var contents []byte

func main() {
	fmt.Println("Contents of file:", string(contents))
}
```

在程序的第 4 行，我们使用下划线前缀导入 `embed` 包。原因是 `embed` 没有在代码中显式使用，但第 8 行的 `//go:embed` 注释需要编译器进行一些预处理。由于我们需要在不显式使用的情况下导入包，因此我们使用下划线前缀以使编译器满意。如果不这样做，编译器会抱怨该包未在任何地方使用。

第 8 行的 `//go:embed test.txt` 告诉编译器读取 `test.txt` 的内容并将其分配给该注释后面的变量。在我们的例子中，`contents` 变量将保存文件的内容。

使用以下命令运行程序：

```fallback
cd ~/Documents/filehandling
go install
filehandling
```

程序将打印：

```fallback
Contents of file: Hello World. Welcome to file handling in Go.
```

现在，文件与二进制文件打包在一起，无论从何处执行，Go 二进制文件都可以访问它。例如，尝试从 `test.txt` 不存在的目录运行程序。

```fallback
cd ~/Documents
filehandling
```

上述命令也将打印文件的内容。

请注意，文件内容应分配给的变量必须在包级别。局部变量不起作用。尝试将程序更改为以下内容：

```go
package main

import (
	_ "embed"
	"fmt"
)

func main() {
	//go:embed test.txt
	var contents []byte
	fmt.Println("Contents of file:", string(contents))
}
```

上面的程序将 `contents` 作为局部变量。

程序现在将无法编译，并出现以下错误：

```fallback
./filehandling.go:9:4: go:embed cannot apply to var inside func
```

如果你有兴趣了解更多关于此设计决策的信息，请阅读 https://github.com/golang/go/issues/43216


### 分块读取文件

在上一节中，我们学习了如何将整个文件加载到内存中。当文件的大小非常大时，尤其是在 RAM 不足的情况下，将整个文件读入内存是没有意义的。更优化的方法是分块读取文件。这可以通过 [bufio](https://pkg.go.dev/bufio) 包来实现。

让我们编写一个程序，以 3 字节的块读取我们的 `test.txt` 文件。将 `filehandling.go` 的内容替换为以下内容：

```go
package main

import (
	"bufio"
	"flag"
	"fmt"
	"io"
	"log"
	"os"
)

func main() {
	fptr := flag.String("fpath", "test.txt", "file path to read from")
	flag.Parse()

	f, err := os.Open(*fptr)
	if err != nil {
		log.Fatal(err)
	}
	defer func() {
		if err = f.Close(); err != nil {
			log.Fatal(err)
		}
	}()

	r := bufio.NewReader(f)
	b := make([]byte, 3)
	for {
		n, err := r.Read(b)
		if err == io.EOF {
			fmt.Println("finished reading file")
			break
		}
		if err != nil {
			fmt.Printf("Error %s reading file", err)
			break
		}
		fmt.Println(string(b[0:n]))
	}
}
```

在程序的第 16 行，我们使用从命令行标志传递的路径打开文件。

在第 20 行，我们延迟关闭文件。

程序的第 26 行创建了一个新的缓冲读取器。在下一行，我们创建了一个长度为 3 的字节切片，文件字节将被读取到其中。

第 29 行的 `Read` [方法](../【GolangBot】17-方法) 读取最多 `len(b)` 字节，即最多 3 个字节，并返回读取的字节数。我们将返回的字节存储在变量 `n` 中。在第 38 行，切片从索引 `0` 读取到 `n-1`，即读取到 `Read` 方法返回的字节数，并打印。

一旦到达文件末尾，`read` 将返回一个 EOF 错误。我们在第 30 行检查此错误。程序的其余部分很简单。

如果我们使用以下命令运行上述程序：

```fallback
cd ~/Documents/filehandling
go install
filehandling -fpath=/path-of-file/test.txt
```

将输出以下内容：

```fallback
Hel
lo
Wor
ld.
 We
lco
me
to
fil
e h
and
lin
g i
n G
o.
finished reading file
```

### 逐行读取文件

在本节中，我们将讨论如何使用 Go 逐行读取文件。这可以使用 [bufio](https://pkg.go.dev/bufio) 包来完成。

请将 `test.txt` 的内容替换为以下内容：

```fallback
Hello World. Welcome to file handling in Go.  
This is the second line of the file.  
We have reached the end of the file.  
```

以下是逐行读取文件的步骤：

1. 打开文件
2. 从文件创建一个新的扫描器
3. 扫描文件并逐行读取。

将 `filehandling.go` 的内容替换为以下内容：

```go
package main

import (
	"bufio"
	"flag"
	"fmt"
	"log"
	"os"
)

func main() {
	fptr := flag.String("fpath", "test.txt", "file path to read from")
	flag.Parse()

	f, err := os.Open(*fptr)
	if err != nil {
		log.Fatal(err)
	}
    defer func() {
	    if err = f.Close(); err != nil {
		log.Fatal(err)
	}
	}()
	s := bufio.NewScanner(f)
	for s.Scan() {
		fmt.Println(s.Text())
	}
	err = s.Err()
	if err != nil {
		log.Fatal(err)
	}
}
```

在程序的第 15 行，我们使用从命令行标志传递的路径打开文件。在第 24 行，我们使用文件创建一个新的扫描器。第 25 行的 `scan()` 方法读取文件的下一行，读取的字符串将通过 `text()` 方法可用。

当 Scan 返回 `false` 后，`Err()` 方法将返回扫描期间发生的任何错误。如果错误是文件结束，`Err()` 将返回 `nil`。

如果我们使用以下命令运行上述程序：

```fallback
cd ~/Documents/filehandling
go install
filehandling -fpath=/path-of-file/test.txt
```

文件的内容将逐行打印，如下所示：

```fallback
Hello World. Welcome to file handling in Go.
This is the second line of the file.
We have reached the end of the file.
```


**下一个教程 - [写入文件](../【GolangBot】37-写文件)**