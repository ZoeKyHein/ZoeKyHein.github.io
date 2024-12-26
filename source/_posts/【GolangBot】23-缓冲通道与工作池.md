---
title: 【GolangBot】23-缓冲通道与工作池
date: 2024-12-25 11:54:19
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: true
---



欢迎来到[Golang系列教程](../golangbot/)的第23讲。

## 什么是缓冲通道？

我们在[上一篇教程](../【GolangBot】22-Channels)中讨论的所有通道基本上都是无缓冲的。正如我们在[channels教程](../【GolangBot】22-Channels)中详细讨论的那样，对无缓冲通道的发送和接收都是阻塞的。

可以创建一个带缓冲的通道。只有当缓冲区已满时，向缓冲通道发送数据才会被阻塞。同样，只有当缓冲区为空时，从缓冲通道接收数据才会被阻塞。

通过向`make`函数传递一个额外的容量参数来创建缓冲通道，该参数指定了缓冲区的大小。

```fallback
ch := make(chan type, capacity)
```

上述语法中的*capacity*必须大于0才能使通道具有缓冲区。无缓冲通道的容量默认为0，因此我们在[上一教程](../【GolangBot】22-Channels)中创建通道时省略了容量参数。

让我们编写一些代码并创建一个缓冲通道。

### 示例

```go
package main

import (
    "fmt"
)


func main() {
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println(<- ch)
    fmt.Println(<- ch)
}
```

[Run program in playground](https://play.golang.org/p/It-em11etK)

在上面的程序中，第9行我们创建了一个容量为2的缓冲通道。由于通道的容量为2，所以可以向通道写入2个字符串而不会被阻塞。我们在第10行和第11行向通道写入2个字符串，通道不会阻塞。我们分别在第12行和第13行读取写入的2个字符串。这个程序打印：

```fallback
naveen
paul
```

### 另一个示例

让我们看一个缓冲通道的另一个示例，其中值是在并发Goroutine中写入通道，并从main Goroutine中读取。这个示例将帮助我们更好地理解向缓冲通道写入时何时会阻塞。

```go
package main

import (  
    "fmt"
    "time"
)

func write(ch chan int) {  
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Println("successfully wrote", i, "to ch")
    }
    close(ch)
}
func main() {  
    ch := make(chan int, 2)
    go write(ch)
    time.Sleep(2 * time.Second)
    for v := range ch {
        fmt.Println("read value", v,"from ch")
        time.Sleep(2 * time.Second)

    }
}
```

[Run program in playground](https://play.golang.org/p/bKe5GdgMK9)

在上面的程序中，在main Goroutine的第16行创建了一个容量为`2`的缓冲通道`ch`，并在第17行传递给write Goroutine。然后main Goroutine休眠2秒。在此期间，write Goroutine并发运行。write Goroutine有一个`for`循环，将数字0到4写入`ch`通道。这个缓冲通道的容量是`2`，因此write `Goroutine`将能够立即将值`0`和`1`写入`ch`通道，然后它会阻塞，直到至少从`ch`通道读取一个值。所以这个程序会立即打印以下2行。

```fallback
successfully wrote 0 to ch
successfully wrote 1 to ch
```

打印完上面两行后，write Goroutine中向`ch`通道的写入会被阻塞，直到有人从`ch`通道读取。由于main Goroutine在开始从通道读取之前休眠2秒，所以程序在接下来的2秒内不会打印任何内容。main Goroutine在2秒后醒来，并开始使用第19行的`for range`循环从`ch`通道读取，打印读取的值，然后再次休眠2秒，这个循环继续，直到`ch`被关闭。所以程序会在2秒后打印以下行：

```fallback
read value 0 from ch
successfully wrote 2 to ch
```

这将持续到所有值都写入通道并在write Goroutine中关闭。最终输出将是：

```fallback
successfully wrote 0 to ch
successfully wrote 1 to ch
read value 0 from ch
successfully wrote 2 to ch
read value 1 from ch
successfully wrote 3 to ch
read value 2 from ch
successfully wrote 4 to ch
read value 3 from ch
read value 4 from ch
```

### 死锁

```go
package main

import (
    "fmt"
)

func main() {
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    ch <- "steve"
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```

[Run program in playground](https://play.golang.org/p/FW-LHeH7oD)

在上面的程序中，我们向一个容量为2的缓冲通道写入3个字符串。当控制到达第11行的第三次写入时，写入被阻塞，因为通道已超出其容量。现在必须有某个Goroutine从通道读取，写入才能继续进行，但在这种情况下，没有并发的例程从这个通道读取。因此会出现**死锁**，程序会在运行时出现panic，并显示以下消息：

```fallback
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
	/tmp/sandbox091448810/prog.go:11 +0x8d
```

### 关闭缓冲通道

我们已经在[上一教程](../【GolangBot】23-缓冲通道与工作池/#关闭缓冲通道)中讨论了关闭通道。除了我们在上一教程中学到的内容外，关闭缓冲通道时还有一个需要考虑的细节。

可以从已关闭的缓冲通道读取数据。通道将返回已写入通道的数据，一旦所有数据都被读取，它将返回通道的零值。

让我们编写一个程序来理解这一点。

```go
package main

import (
    "fmt"
)

func main() {
    ch := make(chan int, 5)
    ch <- 5
    ch <- 6
    close(ch)
    n, open := <-ch 
    fmt.Printf("Received: %d, open: %t\n", n, open)
    n, open = <-ch 
    fmt.Printf("Received: %d, open: %t\n", n, open)
    n, open = <-ch 
    fmt.Printf("Received: %d, open: %t\n", n, open)
}
```

[Run program in playground](https://play.golang.org/p/19cZrsy2X4v)

在上面的程序中，我们在第8行创建了一个容量为`5`的缓冲通道。然后我们向通道写入`5`和`6`。通道在第11行之后关闭。即使通道已关闭，我们仍然可以读取已写入通道的值。这是在第12行和第14行完成的。在第12行，`n`的值将是`5`，open将是`true`。在第14行，`n`的值将是`6`，open再次为`true`。我们现在已经读取了通道中的`5`和`6`，没有更多的数据可读。现在当在第16行再次读取通道时，`n`的值将是`0`，这是`int`的零值，`open`将是`false`，表示通道已关闭。

这个程序将打印：

```fallback
Received: 5, open: true
Received: 6, open: true
Received: 0, open: false
```

同样的程序也可以使用for range循环来编写。

```go
package main

import (
	"fmt"
)

func main() {
	ch := make(chan int, 5)
	ch <- 5
	ch <- 6
	close(ch)
	for n := range ch {
		fmt.Println("Received:", n)
	}
}
```

[Run program in playground](https://play.golang.org/p/xVgkn64Jado)

上面程序第12行的`for range`循环将读取写入通道的所有值，并在没有更多值可读时退出，因为通道已经关闭。

这个程序将打印：

```fallback
Received: 5
Received: 6
```

### 长度与容量

缓冲通道的容量是通道可以容纳的值的数量。这是我们使用`make`函数创建缓冲通道时指定的值。

缓冲通道的长度是当前队列中的元素数量。

一个程序可以让事情变得清晰 😀

```go
package main

import (
    "fmt"
)

func main() {
    ch := make(chan string, 3)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println("capacity is", cap(ch))
    fmt.Println("length is", len(ch))
    fmt.Println("read value", <-ch)
    fmt.Println("new length is", len(ch))
}
```

[Run program in playground](https://play.golang.org/p/2ggC64yyvr)

在上面的程序中，通道创建时的容量为`3`，也就是说它可以容纳3个字符串。然后我们在第9行和第10行分别向通道写入2个字符串。现在通道中有2个字符串排队，因此其长度为`2`。在第13行，我们从通道读取一个字符串。现在通道中只有一个字符串排队，因此其长度变为`1`。这个程序将打印：

```fallback
capacity is 3
length is 2
read value naveen
new length is 1
```

### WaitGroup

本教程的下一节是关于*Worker Pools*。为了理解worker pools，我们首先需要了解`WaitGroup`，因为它将在Worker pool的实现中使用。

WaitGroup用于等待一组Goroutines完成执行。控制会被阻塞，直到所有Goroutines完成执行。假设我们有3个从`main` Goroutine生成的并发执行的Goroutines。`main` Goroutines需要等待其他3个Goroutines完成才能终止。这可以使用WaitGroup来实现。

让我们停止理论，直接开始编写代码 😀

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func process(i int, wg *sync.WaitGroup) {
	fmt.Println("started Goroutine ", i)
	time.Sleep(2 * time.Second)
	fmt.Printf("Goroutine %d ended\n", i)
	wg.Done()
}

func main() {
	no := 3
	var wg sync.WaitGroup
	for i := 0; i < no; i++ {
		wg.Add(1)
		go process(i, &wg)
	}
	wg.Wait()
	fmt.Println("All go routines finished executing")
}
```

[Run in playground](https://play.golang.org/p/CZNtu8ktQh)

[WaitGroup](https://golang.org/pkg/sync/#WaitGroup)是一个结构类型，我们在第18行创建了一个WaitGroup类型的零值变量。WaitGroup的工作方式是通过使用计数器。当我们对WaitGroup调用`Add`并传递一个`int`时，WaitGroup的计数器会增加传递给`Add`的值。减少计数器的方法是在WaitGroup上调用`Done()`方法。`Wait()`方法会阻塞调用它的`Goroutine`，直到计数器变为零。

在上面的程序中，我们在第20行的`for`循环内调用`wg.Add(1)`，循环迭代3次。所以计数器现在变成3。`for`循环还生成了3个`process` Goroutines，然后在第23行调用的`wg.Wait()`使`main` Goroutine等待，直到计数器变为零。计数器通过在`process` Goroutine中第13行调用`wg.Done`来递减。一旦所有3个生成的Goroutines完成执行，也就是`wg.Done()`被调用三次，计数器将变为零，main Goroutine将被解除阻塞。

**在第21行传递`wg`的指针很重要。如果不传递指针，那么每个Goroutine将有自己的WaitGroup副本，并且main不会在它们完成执行时得到通知。**

这个程序输出：

```fallback
started Goroutine  2
started Goroutine  0
started Goroutine  1
Goroutine 0 ended
Goroutine 2 ended
Goroutine 1 ended
All go routines finished executing
```

你的输出可能与我的不同，因为Goroutines的执行顺序可能会有所不同 :)。

### Worker Pool 实现

缓冲通道的一个重要用途是实现[worker pool](https://en.wikipedia.org/wiki/Thread_pool)。

一般来说，worker pool是等待分配任务的线程集合。一旦他们完成分配的任务，就会再次变得可用于下一个任务。

我们将使用缓冲通道来实现worker pool。我们的worker pool将执行找出输入数字的各位数字之和的任务。例如，如果传入234，输出将是9（2 + 3 + 4）。worker pool的输入将是一个伪随机整数列表。

以下是我们worker pool的核心功能：

- 创建一个监听输入缓冲通道的Goroutine池，等待分配任务
- 向输入缓冲通道添加任务
- 任务完成后将结果写入输出缓冲通道
- 从输出缓冲通道读取并打印结果

我们将逐步编写这个程序，以使其更容易理解。

第一步将是创建表示任务和结果的结构体。

```fallback
type Job struct {
    id       int
    randomno int
}
type Result struct {
    job         Job
    sumofdigits int
}
```

每个`Job`结构体都有一个`id`和一个需要计算各位数字之和的`randomno`。

`Result`结构体有一个`job`字段，它是持有结果（各位数字之和）的任务，结果存储在`sumofdigits`字段中。

下一步是创建用于接收任务和写入输出的缓冲通道。

```fallback
var jobs = make(chan Job, 10)
var results = make(chan Result, 10)
```

Worker Goroutines监听`jobs`缓冲通道等待新任务。一旦任务完成，结果就会写入`results`缓冲通道。

下面的`digits`函数实际执行找出整数各位数字之和并返回的任务。我们将在这个函数中添加2秒的休眠，只是为了模拟这个函数需要一些时间来计算结果。

```go
func digits(number int) int {
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
```

接下来，我们将编写一个创建worker Goroutine的函数。

```go
func worker(wg *sync.WaitGroup) {
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
```

上面的函数创建一个worker，它从`jobs`通道读取，使用当前的`job`和`digits`函数的返回值创建一个`Result`结构体，然后将结果写入`results`缓冲通道。这个函数接受一个WaitGroup `wg`作为参数，当所有`jobs`完成时，它将在该参数上调用`Done()`方法。

`createWorkerPool`函数将创建一个worker Goroutine池。

```go
func createWorkerPool(noOfWorkers int) {
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
```

上面的函数接受要创建的workers数量作为参数。在创建Goroutine之前调用`wg.Add(1)`来增加WaitGroup计数器。然后通过将WaitGroup `wg`的指针传递给`worker`函数来创建worker Goroutines。创建完所需的worker Goroutines后，它通过调用`wg.Wait()`等待所有Goroutines完成执行。在所有Goroutines完成执行后，它关闭`results`通道，因为所有Goroutines都已完成执行，没有人会进一步写入`results`通道。

现在我们已经准备好worker pool，让我们继续编写将任务分配给workers的函数。

```go
func allocate(noOfJobs int) {
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
```

上面的`allocate`函数接受要创建的任务数量作为输入参数，生成最大值为`998`的伪随机数，使用随机数和for循环计数器`i`作为id创建`Job`结构体，然后将它们写入`jobs`通道。写入所有任务后关闭`jobs`通道。

下一步将是创建读取`results`通道并打印输出的函数。

```go
func result(done chan bool) {
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
```

`result`函数读取`results`通道并打印任务id、输入随机数和随机数的各位数字之和。result函数还接受一个`done`通道作为参数，一旦打印完所有结果就向该通道写入。

现在我们已经准备就绪。让我们继续最后一步，从`main()`函数调用所有这些函数。

```go
func main() {
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

我们首先在main函数的第2行存储程序的执行开始时间，在最后一行（第12行）我们计算endTime和startTime之间的时间差，并显示程序运行所需的总时间。这是必需的，因为我们将通过改变Goroutines的数量来做一些基准测试。

`noOfJobs`设置为100，然后调用`allocate`向`jobs`通道添加任务。

然后创建`done`通道并传递给`result` Goroutine，这样它就可以开始打印输出并在一切都打印完成时通知。

最后通过调用`createWorkerPool`函数创建一个包含`10`个worker Goroutines的池，然后main在`done`通道上等待所有结果打印完成。

这是完整的程序供你参考。我也导入了必要的包。

```go
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type Job struct {
    id       int
    randomno int
}
type Result struct {
    job         Job
    sumofdigits int
}

var jobs = make(chan Job, 10)
var results = make(chan Result, 10)

func digits(number int) int {
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
func worker(wg *sync.WaitGroup) {
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
func createWorkerPool(noOfWorkers int) {
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
func allocate(noOfJobs int) {
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
func result(done chan bool) {
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
func main() {
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

[Run in playground](https://play.golang.org/p/au5islUIbx)

请在本地机器上运行此程序以获得更准确的总时间计算。

这个程序将打印：

```fallback
Job id 1, input random no 636, sum of digits 15
Job id 0, input random no 878, sum of digits 23
Job id 9, input random no 150, sum of digits 6
...
total time taken  20.01081009 seconds
```

将打印总共100行，对应100个任务，然后最后一行打印程序运行的总时间。你的输出会与我的不同，因为Goroutines可以以任何顺序运行，总时间也会根据硬件而变化。在我的情况下，程序完成大约需要20秒。

现在让我们将`main`函数中的`noOfWorkers`增加到`20`。我们已经将workers的数量翻倍了。由于worker Goroutines增加了（准确地说是翻倍），程序完成所需的总时间应该减少（准确地说是减半）。在我的情况下，它变成了10.004364685秒，程序打印：

```fallback
...
total time taken  10.004364685 seconds
```

现在我们可以理解，随着worker Goroutines数量的增加，完成任务所需的总时间会减少。我把它作为一个练习留给你，在`main`函数中将`noOfJobs`和`noOfWorkers`设置为不同的值并分析结果。

**下一教程 - [Select](../【GolangBot】24-Select)**
