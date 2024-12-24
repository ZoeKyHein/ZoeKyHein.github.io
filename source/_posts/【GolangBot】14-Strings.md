---
title: 【GolangBot】14-Strings
date: 2024-12-17 08:51:01
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: false
---

# 【GolangBot】14-Strings

欢迎来到[Golang系列教程](../golangbot/)第14篇。

字符串在 Go 中值得特别提及，因为与其他语言相比，它们的实现有所不同。

### 什么是字符串？

**在 Go 中，字符串是字节的[切片](../【GolangBot】11-数组和切片/)。可以通过将一组字符用双引号 `""` 包括起来来创建字符串。**

让我们通过一个简单的示例来创建一个 `string` 并打印它。

```go
package main

import (
	"fmt"
)

func main() {
	name := "Hello World"
	fmt.Println(name)
}
```

[Run in playground](https://play.golang.org/p/o9OVDgEMU0)

上述程序将打印 `Hello World`。

Go 中的字符串是[符合 Unicode 标准](https://naveenr.net/unicode-character-set-and-utf-8-utf-16-utf-32-encoding/)的，并且采用[UTF-8 编码](https://naveenr.net/unicode-character-set-and-utf-8-utf-16-utf-32-encoding/)。

### 访问字符串的各个字节

由于字符串是字节的切片，因此可以访问字符串的每个字节。

```go
package main

import (
	"fmt"
)

func printBytes(s string) {
	fmt.Printf("Bytes: ")
	for i := 0; i < len(s); i++ {
		fmt.Printf("%x ", s[i])
	}
}

func main() {
	name := "Hello World"
	fmt.Printf("String: %s\n", name)
	printBytes(name)
}
```

[Run in playground](https://play.golang.org/p/B3KgBBQhiN9)

**%s 是用于打印字符串的格式说明符。** 在上述程序的第 16 行，打印了输入的字符串。在第 9 行，**`len(s)` 返回字符串中的字节数**，并使用[for 循环](../【GolangBot】9-循环/)以十六进制格式打印这些字节。**%x 是表示十六进制的格式说明符。**上述程序的输出为：

```fallback
String: Hello World
Bytes: 48 65 6c 6c 6f 20 57 6f 72 6c 64  
```

这些是 `Hello World` 的 [Unicode UTF-8 编码](https://mothereff.in/utf-8#Hello World)值。要更好地理解字符串，需要对 Unicode 和 UTF-8 有基本的了解。推荐阅读 https://naveenr.net/unicode-character-set-and-utf-8-utf-16-utf-32-encoding/ 以了解更多关于 Unicode 和 UTF-8 的内容。

### 访问字符串中的单个字符

让我们稍微修改一下上面的程序，以打印字符串的字符。

```go
package main

import (
	"fmt"
)

func printBytes(s string) {
	fmt.Printf("Bytes: ")
	for i := 0; i < len(s); i++ {
		fmt.Printf("%x ", s[i])
	}
}

func printChars(s string) {
	fmt.Printf("Characters: ")
	for i := 0; i < len(s); i++ {
		fmt.Printf("%c ", s[i])
	}
}

func main() {
	name := "Hello World"
	fmt.Printf("String: %s\n", name)
	printChars(name)
	fmt.Printf("\n")
	printBytes(name)
}
```

[Run in playground](https://play.golang.org/p/ZkXmyVNsqv7)

在上面程序的第 17 行，`%c` 格式说明符用于在 `printChars` 方法中打印字符串的字符。程序输出如下：

```fallback
String: Hello World
Characters: H e l l o   W o r l d 
Bytes: 48 65 6c 6c 6f 20 57 6f 72 6c 64 
```

尽管上面的程序看起来是访问字符串中各个字符的合法方式，但实际上存在一个严重的漏洞。我们来看看这个漏洞是什么。

```go
package main

import (
	"fmt"
)

func printBytes(s string) {
	fmt.Printf("Bytes: ")
	for i := 0; i < len(s); i++ {
		fmt.Printf("%x ", s[i])
	}
}

func printChars(s string) {
	fmt.Printf("Characters: ")
	for i := 0; i < len(s); i++ {
		fmt.Printf("%c ", s[i])
	}
}

func main() {
	name := "Hello World"
	fmt.Printf("String: %s\n", name)
	printChars(name)
	fmt.Printf("\n")
	printBytes(name)
	fmt.Printf("\n\n")
	name = "Señor"
	fmt.Printf("String: %s\n", name)
	printChars(name)
	fmt.Printf("\n")
	printBytes(name)
}
```

[Run in playground](https://play.golang.org/p/2hyVf8l9fiO)

上面的程序的输出是

```fallback
String: Hello World
Characters: H e l l o   W o r l d 
Bytes: 48 65 6c 6c 6f 20 57 6f 72 6c 64 

String: Señor
Characters: S e Ã ± o r 
Bytes: 53 65 c3 b1 6f 72 
```

在上面的程序第30行，我们尝试打印**Señor**的字符，结果输出为**S e Ã ± o r**，这是错误的。为什么`Señor`会出错，而`Hello World`却能正常工作呢？原因是`ñ`的Unicode代码点是`U+00F1`，其[UTF-8编码](https://mothereff.in/utf-8#ñ)占用2个字节`c3`和`b1`。我们在尝试打印字符时，假设每个代码点仅占用一个字节，这是错误的。**在UTF-8编码中，一个代码点可能占用多个字节**。那么我们该如何解决这个问题呢？这就是**rune**的作用所在。

### Rune

在Go中，rune是一个内建类型，它是int32的别名。Rune表示Go中的Unicode代码点。无论代码点占用多少字节，都可以用rune表示。让我们修改上面的程序，使用rune来打印字符。

```go
package main

import (
	"fmt"
)

func printBytes(s string) {
	fmt.Printf("Bytes: ")
	for i := 0; i < len(s); i++ {
		fmt.Printf("%x ", s[i])
	}
}

func printChars(s string) {
	fmt.Printf("Characters: ")
	runes := []rune(s)
	for i := 0; i < len(runes); i++ {
		fmt.Printf("%c ", runes[i])
	}
}

func main() {
	name := "Hello World"
	fmt.Printf("String: %s\n", name)
	printChars(name)
	fmt.Printf("\n")
	printBytes(name)
	fmt.Printf("\n\n")
	name = "Señor"
	fmt.Printf("String: %s\n", name)
	printChars(name)
	fmt.Printf("\n")
	printBytes(name)
}
```

[Run in playground](https://play.golang.org/p/n8rsfagm2SJ)

在上面的程序第16行，字符串被转换为[rune切片](../【GolangBot】11-数组和切片/)。然后，我们遍历该切片并显示字符。该程序输出：

```fallback
String: Hello World
Characters: H e l l o   W o r l d 
Bytes: 48 65 6c 6c 6f 20 57 6f 72 6c 64 

String: Señor
Characters: S e ñ o r 
Bytes: 53 65 c3 b1 6f 72 
```

上面的输出是完美的。正是我们想要的 😀。

### 使用 for range 循环访问单个 rune

上面的程序是迭代字符串中单个 rune 的完美方式。但是 Go 提供了一个更简单的方法，使用 **for range** 循环。

```go
package main

import (  
    "fmt"
)

func charsAndBytePosition(s string) {
	for index, rune := range s {
		fmt.Printf("%c starts at byte %d\n", rune, index)
	}
}

func main() {  
    name := "Señor"
    charsAndBytePosition(name)
}
```

[Run in playground](https://play.golang.org/p/0ldNBeffjYI)

在上面的程序的第8行，使用 `for range` 循环迭代字符串。循环返回了每个 rune 开始的位置以及该 rune 本身。该程序的输出为：

```fallback
S starts at byte 0
e starts at byte 1
ñ starts at byte 2
o starts at byte 4
r starts at byte 5
```

从上面的输出可以看出，`ñ` 占用了 2 个字节，因为下一个字符 `o` 从字节 4 开始，而不是从字节 3 开始 😀。

### 从字节切片创建字符串

```go
package main

import (
	"fmt"
)

func main() {
	byteSlice := []byte{0x43, 0x61, 0x66, 0xC3, 0xA9}
	str := string(byteSlice)
	fmt.Println(str)
}
```

[Run in playground](https://play.golang.org/p/Vr9pf8X8xO)

*byteSlice* 在上面程序的第8行包含了字符串 `Café` 的 [UTF-8 编码](https://mothereff.in/utf-8#Café) 十六进制字节。程序输出

```fallback
Café
```

如果我们有十六进制值的十进制等价物，上面的程序还会有效吗？让我们来检查一下。

```go
package main

import (
	"fmt"
)

func main() {
	byteSlice := []byte{67, 97, 102, 195, 169}//decimal equivalent of {'\x43', '\x61', '\x66', '\xC3', '\xA9'}
	str := string(byteSlice)
	fmt.Println(str)
}
```

[Run in playground](https://play.golang.org/p/jgsRowW6XN)

十进制值也可以正常工作，上面的程序将打印 `Café`。

### 从 rune 切片创建字符串

```go
package main

import (
	"fmt"
)

func main() {
	runeSlice := []rune{0x0053, 0x0065, 0x00f1, 0x006f, 0x0072}
	str := string(runeSlice)
	fmt.Println(str)
}
```

[Run in playground](https://play.golang.org/p/m8wTMOpYJP)

在上面的程序中，`runeSlice` 包含字符串 `Señor` 的 Unicode 代码点的十六进制表示。程序的输出是：

```fallback
Señor
```

### 字符串长度

可以使用 [utf8 包](https://golang.org/pkg/unicode/utf8/#RuneCountInString) 的 `RuneCountInString(s string) (n int)` 函数来获取字符串的长度。该方法接受一个字符串作为参数，并返回其中的 rune 数量。

如前所述，`len(s)` 用于获取字符串的字节数，而不是字符串的长度。正如我们之前讨论的那样，一些 Unicode 字符的代码点占用了超过 1 个字节。使用 `len` 来计算这些字符串的长度会返回不正确的字符串长度。

```go
package main

import (
	"fmt"
	"unicode/utf8"
)

func main() {
	word1 := "Señor"
	fmt.Printf("String: %s\n", word1)
	fmt.Printf("Length: %d\n", utf8.RuneCountInString(word1))
	fmt.Printf("Number of bytes: %d\n", len(word1))

	fmt.Printf("\n")
	word2 := "Pets"
	fmt.Printf("String: %s\n", word2)
	fmt.Printf("Length: %d\n", utf8.RuneCountInString(word2))
	fmt.Printf("Number of bytes: %d\n", len(word2))
}
```

[Run in playground](https://play.golang.org/p/KBQg1qagnfC)

上面的程序将会输出

```fallback
String: Señor
Length: 5
Number of bytes: 6

String: Pets
Length: 4
Number of bytes: 4
```

上述输出确认了 `len(s)` 和 `RuneCountInString(s)` 返回的值不同 😀。

### 字符串比较

使用 `==` 运算符可以比较两个字符串是否相等。如果两个字符串相等，则结果为 `true`，否则为 `false`。

```go
package main

import (
	"fmt"
)

func compareStrings(str1 string, str2 string) {
	if str1 == str2 {
		fmt.Printf("%s and %s are equal\n", str1, str2)
		return
	}
	fmt.Printf("%s and %s are not equal\n", str1, str2)
}

func main() {
	string1 := "Go"
	string2 := "Go"
	compareStrings(string1, string2)
	
	string3 := "hello"
	string4 := "world"
	compareStrings(string3, string4)

}
```

[Run in playground](https://play.golang.org/p/JEAMexbvJ1s)

在上面的 `compareStrings` 函数中，第 8 行使用 `==` 运算符比较两个字符串 `str1` 和 `str2` 是否相等。如果它们相等，则打印相应的消息，并且 [函数](../【GolangBot】6-函数/) 返回。

上述程序打印：

```fallback
Go and Go are equal
hello and world are not equal
```

### 字符串拼接

在 Go 中，有多种方法可以进行字符串拼接。我们来看其中的几种方法。

最简单的字符串拼接方式是使用 `+` 运算符。

```go
package main

import (
	"fmt"
)

func main() {
	string1 := "Go"
	string2 := "is awesome"
	result := string1 + " " + string2
	fmt.Println(result)
}
```

[Run in playground](https://play.golang.org/p/RCL8SGkrBe9)

在上面的程序中，程序的第10行将 `string1` 和 `string2` 用一个空格连接起来。该程序输出：

```fallback
Go is awesome
```

第二种连接字符串的方法是使用 `fmt` 包中的 [Sprintf](https://golang.org/pkg/fmt/#Sprintf) 函数。

`Sprintf` 函数根据输入的格式说明符格式化字符串，并返回结果字符串。让我们使用 `Sprintf` 函数重写上面的程序。

```go
package main

import (
	"fmt"
)

func main() {
	string1 := "Go"
	string2 := "is awesome"
	result := fmt.Sprintf("%s %s", string1, string2)
	fmt.Println(result)
}
```

[Run in playground](https://play.golang.org/p/AgqI29aQQDu)

在上面的程序中，第10行的 `%s %s` 是 `Sprintf` 的格式说明符输入。这个格式说明符接收两个字符串作为输入，并且它们之间有一个空格。这样就会将两个字符串连接起来，中间有一个空格。结果字符串会被存储在 `result` 变量中。该程序也会打印：

```fallback
Go is awesome
```

### 字符串是不可变的

在 Go 中，字符串是不可变的。一旦字符串被创建，就无法修改它。

```go
package main

import (  
    "fmt"
)

func mutate(s string)string {
	s[0] = 'a'//any valid unicode character within single quote is a rune 
	return s
}
func main() {  
    h := "hello"
    fmt.Println(mutate(h))
}
```

[Run in playground](https://play.golang.org/p/bv4SlSd_hp)

在上面的程序的第 8 行，我们试图将字符串的第一个字符更改为 `'a'`。在单引号内的任何有效 Unicode 字符都是一个 rune。我们尝试将 rune `a` 赋值给切片的第零个位置。这是不允许的，因为字符串是不可变的，因此程序无法编译并报错 **./prog.go:8:7: cannot assign to s[0]**。

为了解决字符串不可变的问题，可以将字符串转换为 [rune 切片](../【GolangBot】11-数组和切片/)，然后在该切片上进行所需的修改，最后将其转换回一个新的字符串。

```go
package main

import (  
    "fmt"
)

func mutate(s []rune) string {
	s[0] = 'a' 
	return string(s)
}
func main() {  
    h := "hello"
    fmt.Println(mutate([]rune(h)))
}
```

[Run in playground](https://play.golang.org/p/GL1cm17IP1)

在上面的程序的第 7 行，`mutate` 函数接受一个 rune 切片作为参数。然后，它将切片的第一个元素更改为 `'a'`，并将 rune 切片转换回字符串并返回。在程序的第 13 行调用了这个方法。`h` 被转换为 rune 切片并传递给 `mutate`。这个程序的输出是 `aello`。

我已经将我们讨论过的内容整合到一个 GitHub 程序中，你可以 [点击这里下载](https://github.com/golangbot/stringsexplained)。

这就是关于字符串的所有内容。祝你有个愉快的一天！

**下一篇教程 - [指针](../【GolangBot】15-指针/)**
