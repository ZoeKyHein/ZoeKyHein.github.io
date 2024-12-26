---
title: 【GolangBot】27-组合而不是继承
date: 2024-12-26 10:17:24
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---

欢迎来到 [Golang 教程系列](../golangbot/) 的第 27 个教程。

Go 不支持继承，但它支持组合。组合的通用定义是“组合在一起”。一个组合的例子是**汽车**。汽车由轮子、发动机和其他各种部件组成。

### 通过嵌入结构体实现组合

在 Go 中，可以通过将一个[结构体](../【GolangBot】16-结构体/)类型嵌入到另一个结构体来实现组合。

博客文章是组合的一个完美例子。每篇博客文章都有标题、内容和作者信息。这可以通过组合完美地表示。在本教程的后续步骤中，我们将学习如何实现这一点。

让我们首先创建 `author` 结构体。

```go
package main

import (
	"fmt"
)

type author struct {
	firstName string
	lastName  string
	bio       string
}

func (a author) fullName() string {
	return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}
```

在上面的代码片段中，我们创建了一个 `author` 结构体，包含字段 `firstName`、`lastName` 和 `bio`。我们还添加了一个 `fullName()` 方法，该方法以 `author` 作为接收者类型，并返回作者的全名。

下一步是创建 `blogPost` 结构体。

```go
type blogPost struct {
	title   string
	content string
	author
}

func (b blogPost) details() {
	fmt.Println("Title: ", b.title)
	fmt.Println("Content: ", b.content)
	fmt.Println("Author: ", b.author.fullName())
	fmt.Println("Bio: ", b.author.bio)
}
```

`blogPost` 结构体包含字段 `title` 和 `content`。它还有一个嵌入的[匿名](../【GolangBot】16-结构体/#匿名字段)字段 `author`。该字段表示 `blogPost` 结构体由 `author` 组成。现在，`blogPost` 结构体可以访问 `author` 结构体的所有字段和方法。我们还向 `blogPost` 结构体添加了 `details()` 方法，用于打印标题、内容、作者的全名和简介。

当一个结构体字段嵌入到另一个结构体时，Go 允许我们像访问外部结构体的字段一样访问嵌入的字段。这意味着上面代码中第 11 行的 `p.author.fullName()` 可以替换为 `p.fullName()`。因此，`details()` 方法可以重写如下：

```go
func (p blogPost) details() {
	fmt.Println("Title: ", p.title)
	fmt.Println("Content: ", p.content)
	fmt.Println("Author: ", p.fullName())
	fmt.Println("Bio: ", p.bio)
}
```

现在我们已经准备好了 `author` 和 `blogPost` 结构体，让我们通过创建一篇博客文章来完成这个程序。

```go
package main

import (
	"fmt"
)

type author struct {
	firstName string
	lastName  string
	bio       string
}

func (a author) fullName() string {
	return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type blogPost struct {
	title   string
	content string
	author
}

func (b blogPost) details() {
	fmt.Println("Title: ", b.title)
	fmt.Println("Content: ", b.content)
	fmt.Println("Author: ", b.fullName())
	fmt.Println("Bio: ", b.bio)
}

func main() {
	author1 := author{
		"Naveen",
		"Ramanathan",
		"Golang Enthusiast",
	}
	blogPost1 := blogPost{
		"Inheritance in Go",
		"Go supports composition instead of inheritance",
		author1,
	}
	blogPost1.details()
}
```

[在 Playground 中运行](https://play.golang.org/p/fmF-2oJwbCP)

上述程序的 `main` 函数在第 31 行创建了一个新作者。在第 36 行，通过嵌入 `author1` 创建了一篇新文章。该程序输出：

```fallback
Title:  Inheritance in Go
Content:  Go supports composition instead of inheritance
Author:  Naveen Ramanathan
Bio:  Golang Enthusiast
```

### 嵌入结构体切片

我们可以进一步扩展这个例子，使用博客文章的[切片](../【GolangBot】11-数组和切片)创建一个网站 :)。

让我们首先定义 `website` 结构体。请将以下代码添加到现有程序的 `main` 函数之前并运行它。

```go
type website struct {
	[]blogPost
}

func (w website) contents() {
	fmt.Println("Contents of Website\n")
	for _, v := range w.blogPosts {
		v.details()
		fmt.Println()
	}
}
```

当你在添加上述代码后运行程序时，编译器会报以下错误：

```fallback
main.go:31:9: syntax error: unexpected [, expecting field name or embedded type
```

此错误指向嵌入的结构体切片 `[]blogPost`。原因是无法匿名嵌入切片。需要一个字段名称。因此，让我们修复此错误并使编译器满意。

```go
type website struct {
	blogPosts []blogPost
}
```

我添加了字段 `blogPosts`，它是一个 `[]blogPost` 切片。

现在让我们修改 `main` 函数，为我们的新网站创建几篇文章。

修改后的完整程序如下：

```go
package main

import (
	"fmt"
)

type author struct {
	firstName string
	lastName  string
	bio       string
}

func (a author) fullName() string {
	return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type blogPost struct {
	title   string
	content string
	author
}

func (p blogPost) details() {
	fmt.Println("Title: ", p.title)
	fmt.Println("Content: ", p.content)
	fmt.Println("Author: ", p.fullName())
	fmt.Println("Bio: ", p.bio)
}

type website struct {
	blogPosts []blogPost
}

func (w website) contents() {
	fmt.Println("Contents of Website\n")
	for _, v := range w.blogPosts {
		v.details()
		fmt.Println()
	}
}

func main() {
	author1 := author{
		"Naveen",
		"Ramanathan",
		"Golang Enthusiast",
	}
	blogPost1 := blogPost{
		"Inheritance in Go",
		"Go supports composition instead of inheritance",
		author1,
	}
	blogPost2 := blogPost{
		"Struct instead of Classes in Go",
		"Go does not support classes but methods can be added to structs",
		author1,
	}
	blogPost3 := blogPost{
		"Concurrency",
		"Go is a concurrent language and not a parallel one",
		author1,
	}
	w := website{
		blogPosts: []blogPost{blogPost1, blogPost2, blogPost3},
	}
	w.contents()
}
```

[在 Playground 中运行](https://play.golang.org/p/dv2gdcptwgS)

在上面的 `main` 函数中，我们创建了一个作者 `author1` 和三篇文章 `post1`、`post2` 和 `post3`。最后，我们在第 62 行通过嵌入这 3 篇文章创建了网站 `w`，并在下一行显示了内容。

该程序将输出：

```fallback
Contents of Website

Title:  Inheritance in Go
Content:  Go supports composition instead of inheritance
Author:  Naveen Ramanathan
Bio:  Golang Enthusiast

Title:  Struct instead of Classes in Go
Content:  Go does not support classes but methods can be added to structs
Author:  Naveen Ramanathan
Bio:  Golang Enthusiast

Title:  Concurrency
Content:  Go is a concurrent language and not a parallel one
Author:  Naveen Ramanathan
Bio:  Golang Enthusiast
```

**下一个教程 - [多态](../GolangBot】28-多态)**