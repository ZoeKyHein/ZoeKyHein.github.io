---
title: 【GolangBot】28-多态
date: 2024-12-26 10:35:08
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: false
---

欢迎来到 [Golang 教程系列](../golangbot/) 的第 28 个教程。

Go 中的多态性是通过[接口](../【GolangBot】18-接口-I)实现的。正如我们已经讨论过的，接口在 Go 中是隐式实现的。如果一个类型为接口中声明的所有[方法](../【GolangBot】17-方法)提供了定义，那么它就实现了该接口。让我们通过一个例子来看看如何在 Go 中利用接口实现多态性。

### 使用接口实现多态性

任何为接口的所有方法提供定义的类型都被称为隐式实现了该接口。当我们讨论多态性的例子时，这一点会更加清楚。

**接口类型的变量可以保存任何实现了该接口的值。接口的这一特性用于在 Go 中实现多态性。**

让我们通过一个计算组织净收入的程序来理解 Go 中的多态性。为了简单起见，假设这个虚构的组织有两种项目的收入，即**固定账单**和**时间和材料**。组织的净收入是通过这些项目的收入总和来计算的。为了保持本教程的简单性，我们假设货币是美元，并且我们不处理美分。它将用 `int` 表示。（我建议阅读 https://forum.golangbridge.org/t/what-is-the-proper-golang-equivalent-to-decimal-when-dealing-with-money/413 以了解如何表示美分。感谢评论部分的 *Andreas Matuschek* 指出这一点。）

首先，我们定义一个接口 `Income`。

```go
type Income interface {
	calculate() int
	source() string
}
```

上面定义的 `Income` 接口包含两个方法：`calculate()` 用于计算并返回收入来源的收入，`source()` 用于返回收入来源的名称。

接下来，我们为 `FixedBilling` 项目类型定义一个结构体。

```go
type FixedBilling struct {
	projectName  string
	biddedAmount int
}
```

`FixedBilling` 项目有两个字段：`projectName` 表示项目的名称，`biddedAmount` 是组织为该项目投标的金额。

`TimeAndMaterial` 结构体将表示时间和材料类型的项目。

```go
type TimeAndMaterial struct {
	projectName string
	noOfHours   int
	hourlyRate  int
}
```

`TimeAndMaterial` 结构体有三个字段：`projectName`、`noOfHours` 和 `hourlyRate`。

下一步是为这些结构体类型定义方法，计算并返回实际收入和收入来源。

```go
func (fb FixedBilling) calculate() int {
	return fb.biddedAmount
}

func (fb FixedBilling) source() string {
	return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {
	return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {
	return tm.projectName
}
```

对于 `FixedBilling` 项目，收入就是为项目投标的金额。因此，我们从 `FixedBilling` 类型的 `calculate()` 方法中返回该值。

对于 `TimeAndMaterial` 项目，收入是 `noOfHours` 和 `hourlyRate` 的乘积。该值从 `TimeAndMaterial` 类型的 `calculate()` 方法中返回。

我们从 `source()` 方法中返回项目的名称作为收入来源。

由于 `FixedBilling` 和 `TimeAndMaterial` 结构体都为 `Income` 接口的 `calculate()` 和 `source()` 方法提供了定义，因此这两个结构体都实现了 `Income` 接口。

让我们声明 `calculateNetIncome` 函数，它将计算并打印总收入。

```go
func calculateNetIncome(ic []Income) {
	var netincome int = 0
	for _, income := range ic {
		fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
		netincome += income.calculate()
	}
	fmt.Printf("Net income of organization = $%d", netincome)
}
```

上面的 `calculateNetIncome` [函数](../【GolangBot】6-函数)接受一个 `Income` 接口的[切片](../【GolangBot】11-数组和切片/#切片)作为参数。它通过遍历切片并对每个项目调用 `calculate()` 方法来计算总收入。它还通过调用 `source()` 方法来显示收入来源。根据 `Income` 接口的具体类型，将调用不同的 `calculate()` 和 `source()` 方法。因此，我们在 `calculateNetIncome` 函数中实现了多态性。

将来，如果组织添加了新的收入来源，该函数仍然可以正确计算总收入，而无需更改任何代码 :)。

程序中唯一剩下的部分是 `main` 函数。

```go
func main() {
	project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
	project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
	project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
	incomeStreams := []Income{project1, project2, project3}
	calculateNetIncome(incomeStreams)
}
```

在上面的 `main` 函数中，我们创建了三个项目，其中两个是 `FixedBilling` 类型，一个是 `TimeAndMaterial` 类型。接下来，我们使用这三个项目创建一个 `Income` 类型的切片。由于每个项目都实现了 `Income` 接口，因此可以将所有三个项目添加到 `Income` 类型的切片中。最后，我们调用 `calculateNetIncome` 函数并将此切片作为参数传递。它将显示各种收入来源及其收入。

以下是完整的程序供您参考。

```go
package main

import (
	"fmt"
)

type Income interface {
	calculate() int
	source() string
}

type FixedBilling struct {
	projectName  string
	biddedAmount int
}

type TimeAndMaterial struct {
	projectName string
	noOfHours   int
	hourlyRate  int
}

func (fb FixedBilling) calculate() int {
	return fb.biddedAmount
}

func (fb FixedBilling) source() string {
	return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {
	return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {
	return tm.projectName
}

func calculateNetIncome(ic []Income) {
	var netincome int = 0
	for _, income := range ic {
		fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
		netincome += income.calculate()
	}
	fmt.Printf("Net income of organization = $%d", netincome)
}

func main() {
	project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
	project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
	project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
	incomeStreams := []Income{project1, project2, project3}
	calculateNetIncome(incomeStreams)
}
```

[在 Playground 中运行](https://play.golang.org/p/uexie1DCcvh)

该程序将输出：

```fallback
Income From Project 1 = $5000
Income From Project 2 = $10000
Income From Project 3 = $4000
Net income of organization = $19000
```

### 向上述程序添加新的收入流

假设组织通过广告找到了一种新的收入流。让我们看看在不更改 `calculateNetIncome` 函数的情况下，添加这个新的收入流并计算总收入是多么简单。这得益于多态性。

首先，我们定义 `Advertisement` 类型以及 `Advertisement` 类型的 `calculate()` 和 `source()` 方法。

```go
type Advertisement struct {
	adName     string
	CPC        int
	noOfClicks int
}

func (a Advertisement) calculate() int {
	return a.CPC * a.noOfClicks
}

func (a Advertisement) source() string {
	return a.adName
}
```

`Advertisement` 类型有三个字段：`adName`、`CPC`（每次点击成本）和 `noOfClicks`（点击次数）。广告的总收入是 `CPC` 和 `noOfClicks` 的乘积。

让我们稍微修改一下 `main` 函数以包含这个新的收入流。

```go
func main() {
	project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
	project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
	project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
	bannerAd := Advertisement{adName: "Banner Ad", CPC: 2, noOfClicks: 500}
	popupAd := Advertisement{adName: "Popup Ad", CPC: 5, noOfClicks: 750}
	incomeStreams := []Income{project1, project2, project3, bannerAd, popupAd}
	calculateNetIncome(incomeStreams)
}
```

我们创建了两个广告，分别是 `bannerAd` 和 `popupAd`。`incomeStreams` 切片包括我们刚刚创建的两个广告。

以下是添加 `Advertisement` 后的完整程序。

```go
package main

import (
	"fmt"
)

type Income interface {
	calculate() int
	source() string
}

type FixedBilling struct {
	projectName  string
	biddedAmount int
}

type TimeAndMaterial struct {
	projectName string
	noOfHours   int
	hourlyRate  int
}

type Advertisement struct {
	adName     string
	CPC        int
	noOfClicks int
}

func (fb FixedBilling) calculate() int {
	return fb.biddedAmount
}

func (fb FixedBilling) source() string {
	return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {
	return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {
	return tm.projectName
}

func (a Advertisement) calculate() int {
	return a.CPC * a.noOfClicks
}

func (a Advertisement) source() string {
	return a.adName
}

func calculateNetIncome(ic []Income) {
	var netincome int = 0
	for _, income := range ic {
		fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
		netincome += income.calculate()
	}
	fmt.Printf("Net income of organization = $%d", netincome)
}

func main() {
	project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
	project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
	project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
	bannerAd := Advertisement{adName: "Banner Ad", CPC: 2, noOfClicks: 500}
	popupAd := Advertisement{adName: "Popup Ad", CPC: 5, noOfClicks: 750}
	incomeStreams := []Income{project1, project2, project3, bannerAd, popupAd}
	calculateNetIncome(incomeStreams)
}
```

[在 Playground 中运行](https://play.golang.org/p/HF1P9tNt2H_6)

上面的程序将输出：

```fallback
Income From Project 1 = $5000
Income From Project 2 = $10000
Income From Project 3 = $4000
Income From Banner Ad = $1000
Income From Popup Ad = $3750
Net income of organization = $23750
```

你会注意到，尽管我们添加了新的收入流，但我们没有对 `calculateNetIncome` 函数进行任何更改。由于多态性，它仍然可以正常工作。由于新的 `Advertisement` 类型也实现了 `Income` 接口，我们能够将其添加到 `incomeStreams` 切片中。`calculateNetIncome` 函数也无需更改即可正常工作，因为它能够调用 `Advertisement` 类型的 `calculate()` 和 `source()` 方法。

**下一个教程 - [Defer](../【GolangBot】29-Defer)**
