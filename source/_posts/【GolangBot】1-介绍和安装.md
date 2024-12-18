---
title: 【GolangBot】1.介绍和安装
date: 2024-12-16 18:31:00
tags: 
    - GoLang
    - 教程
categories:
    - [学习心得, GoLangBot]
---

发现了一个不错的英文Golang初级教程，内容比较容易理解，开个新坑，翻译一下。一来能够巩固一下GoLang基础，二来可以提高一下英文水平，如果纰漏还请指出。那我们开始吧！

> [原文地址](https://golangbot.com/learn-golang-series/)

# 【GolangBot】1-介绍和安装

这是我们[GoLang系列教程](../golangbot/)的第一篇。这篇文章提供了对于Go的介绍，也讨论了在其他众多编程语言中，选择Go的优点。我们也会学习如何在MacOS、Windows和Linux上安装MacOS。

### 介绍

Go，也被称为Golang，是由Google开发的开源、编译型、静态类型编程语言。Go创立背后的关键人物是[Rob Pike](https://zh.wikipedia.org/wiki/Rob_Pike)、[Ken Thompson](https://zh.wikipedia.org/wiki/Ken_Thompson)和Robert Griesemer。Go于2009年11月公开发布。

Go是一种语法简单的通用语言，并且有强大的标准库支持。Go的主要优势之一是创建高度可用且扩展的Web应用。Go也可以用来创建命令行应用、桌面应用，甚至是移动应用。

### Go的优势

既然有Python、Ruby、NodeJS等大量其他语言可以完成相同的工作，为什么还要选择Go来作为你的服务端编程语言呢？

以下是我在选择Go时发现的一些优点。

#### 简单的语法

Go的语法简洁明了，没有多余的功能。这使得编写可读性和可维护性良好的代码变得容易。

#### 容易编写并发程序

[并发性](../【GolangBot】20-并发介绍/)是Go语言的固有部分。因此，对于Go来说，编写多线程程序是小菜一碟。这是通过[Goroutines](../【GolangBot】21-Goroutines/)和[Channels](../【GolangBot】22-Channels/)实现的，我们将在接下来的教程讨论他们。



#### 编译型语言

Go是一种编译型语言。源代码被编译成原生二进制文件。这种特性是诸如nodejs所使用的Javascript这类解释型语言所没有的。

#### 静态链接

Go编译器支持静态链接。整个Go工程都可以静态链接刀一个非常大的二进制文件中，并且可以很容易地部署在云服务器上，而且不需要为各种依赖所困扰。

#### Go工具链

Go的工具链特别值得提一下。Go附送了一套强大的工具，可以帮助开发者编写更好的代码。一些常用的工具有：

- gofmt - [gofmt](https://pkg.go.dev/cmd/gofmt)用于自动格式化Go的源代码。他使用制表符进行锁紧，使用空格进行对齐。
- vet - [vet](https://pkg.go.dev/cmd/vet)分析go的源代码，并且报告可疑的代码。vet报告的不是真正的问题，而是编译器不能捕获的错误。比如使用[Printf](https://pkg.go.dev/fmt#Printf)时使用了错误的格式指定符。
- staticcheck - [staticcheck](https://staticcheck.dev/)用来强制执行代码中的样式问题。

#### 垃圾收集

Go使用垃圾收集机制，因此内存管理基本是自动完成的，开发者不需要担心内存管理。这也有助于轻松编写并发程序。

#### 简单的语言规范

语言规范十分简单。[整个规范](go.dev/ref/spec)都有详细的记录，你甚至可以用它来编写自己的编译器:)。

#### 开源

最后，同样重要的是，Go是一个开雨啊项目。你可以参与[Go项目](https://go.dev/doc/contribute)并为其作出贡献。



### 使用Go构建的热门产品

下面是一些使用Go构建的热门产品。

- 谷歌使用Go开发了Kubernetes。
- Docker，世界闻名的容器化平台是用Go开发的。
- Dropbox一讲起性能关键组件从Python移植到了Go。
- Infoblox's 的下一代网络产品时使用Go开发的。



### 安装

Go可以安装在所有的三种平台上：Mac、Windows和Linux。你可以从 https://go.dev/dl/ 下载对应平台的二进制文件。

#### Mac OS

从 https://go.dev/dl/ 下载Mac OS安装器。双击开始安装。根据提示操作，Golang将会安装在`/usr/local/go`并且会把 `/usr/local/go/bin`这个文件夹添加到你的PATH环境变量中。

#### Windows

从 https://go.dev/dl/ 下载MSI安装器。双击开始安装，并根据提示进行操作。这将会把Go安装在`c:\Go`，并且会把目录 `c:\Go\bin`添加到path环境变量。

#### Linux

从 https://go.dev/dl/ 下载tar文件并且解压至`/usr/local`。添加`/usr/local/go/bin`到PATH环境变量。这将在Linux中安装Go。



### 验证安装

To verify that Go installed successfully, type the command `go version` in the terminal and it will output the installed Go version. Here is the output in my terminal.

要验证Go是否安装成功，在终端输入`go version`命令，它会输出安装好的Go版本。这是我的终端输出的内容。

```fallback
$ go version
go version go1.19.2 linux/amd64
```

`1.19.2`是在写这篇教程时最新的版本。这证实Go确实安装成功了。在下一篇教程中，我们将会用Go写我们的第一个[Hello World程序](../【GolangBot】2-Hello-World/)。



**下一篇教程 - [Hello World](../【GolangBot】2-Hello-World/)**

