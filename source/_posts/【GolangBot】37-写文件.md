---
title: 【GolangBot】37-写文件
date: 2024-12-17 09:20:14
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: false
---

Welcome to tutorial no. 37 in [Golang tutorial series](https://golangbot.com/learn-golang-series/).

In this tutorial, we will learn how to write data to files using Go. We will also learn how to write to a file concurrently.

This tutorial has the following sections

- Writing string to a file
- Writing bytes to a file
- Writing data to a file line by line
- Appending to a file
- Writing to a file concurrently

Please run all the programs of this tutorial in your local system as playground doesn’t support file operations.

### Writing string to a file

One of the most common file writing operations is writing a string to a file. This is quite simple to do. It consists of the following steps.

1. Create the file
2. Write the string to the file

Let’s get to the code right away.

```go
 1package main
 2
 3import (
 4	"fmt"
 5	"os"
 6)
 7
 8func main() {
 9	f, err := os.Create("test.txt")
10	if err != nil {
11		fmt.Println(err)
12		return
13	}
14	l, err := f.WriteString("Hello World")
15	if err != nil {
16		fmt.Println(err)
17        f.Close()
18		return
19	}
20	fmt.Println(l, "bytes written successfully")
21	err = f.Close()
22	if err != nil {
23		fmt.Println(err)
24		return
25	}
26}
```

go

The `create` function in line no. 9 of the program above creates a file named *test.txt*. If a file with that name already exists, then the create function truncates the file. This function returns a [File descriptor](https://pkg.go.dev/os#File).

In line no 14, we write the string **Hello World** to the file using the `WriteString` method. This method returns the number of bytes written and error if any.

Finally, we close the file in line no. 21.

The above program will print

```fallback
11 bytes written successfully
```

You can find a file named **test.txt** created in the directory from which this program was executed. If you open the file using any text editor, you can find that it contains the text **Hello World**.

### Writing bytes to a file

Writing bytes to a file is quite similar to writing a string to a file. We will use the [Write](https://pkg.go.dev/os#File.Write) method to write bytes to a file. The following program writes a slice of bytes to a file.

```go
 1package main
 2
 3import (
 4	"fmt"
 5	"os"
 6)
 7
 8func main() {
 9	f, err := os.Create("/home/naveen/bytes")
10	if err != nil {
11		fmt.Println(err)
12		return
13	}
14	d2 := []byte{104, 101, 108, 108, 111, 32, 98, 121, 116, 101, 115}
15	n2, err := f.Write(d2)
16	if err != nil {
17		fmt.Println(err)
18        f.Close()
19		return
20	}
21	fmt.Println(n2, "bytes written successfully")
22	err = f.Close()
23	if err != nil {
24		fmt.Println(err)
25		return
26	}
27}
```

go

In the program above, in line no. 15 we use the **Write** method to write a slice of bytes to a file named `bytes` in the directory `/home/naveen`. You can change this directory to a different one. The remaining program is self-explanatory. This program will print `11 bytes written successfully` and it will create a file named `bytes`. Open the file and you can see that it contains the text `hello bytes`

### Writing strings line by line to a file

Another common file operation is the need to write strings to a file line by line. In this section, we will write a program to create a file with the following content.

```fallback
Welcome to the world of Go.
Go is a compiled language.
It is easy to learn Go.
```

Let’s get to the code right away.

```go
 1package main
 2
 3import (
 4	"fmt"
 5	"os"
 6)
 7
 8func main() {
 9	f, err := os.Create("lines")
10	if err != nil {
11		fmt.Println(err)
12                f.Close()
13		return
14	}
15	d := []string{"Welcome to the world of Go1.", "Go is a compiled language.", "It is easy to learn Go."}
16
17	for _, v := range d {
18		fmt.Fprintln(f, v)
19        if err != nil {
20			fmt.Println(err)
21			return
22		}
23	}
24	err = f.Close()
25	if err != nil {
26		fmt.Println(err)
27		return
28	}
29	fmt.Println("file written successfully")
30}
```

go

In line no. 9 of the program above, we create a new file named **lines**. In line no. 17 we iterate through the array using a for range loop and use the [Fprintln](https://golang.org/pkg/fmt/#Fprintln) function to write the lines to a file. The **Fprintln** function takes a `io.writer` as parameter and appends a new line, just what we wanted. Running this program will print `file written successfully` and a file `lines` will be created in the current directory. The content of the file `lines` is provided below.

```fallback
Welcome to the world of Go1.
Go is a compiled language.
It is easy to learn Go.
```

### Appending to a file

In this section, we will append one more line to the `lines` file which we created in the previous section. We will append the line **File handling is easy** to the `lines` file.

The file has to be opened in append and write only mode. These flags are passed as parameters to the [Open](https://pkg.go.dev/os#OpenFile) function. After the file is opened in append mode, we add the new line to the file.

```go
 1package main
 2
 3import (
 4	"fmt"
 5	"os"
 6)
 7
 8func main() {
 9	f, err := os.OpenFile("lines", os.O_APPEND|os.O_WRONLY, 0644)
10	if err != nil {
11		fmt.Println(err)
12		return
13	}
14	newLine := "File handling is easy."
15	_, err = fmt.Fprintln(f, newLine)
16	if err != nil {
17		fmt.Println(err)
18                f.Close()
19		return
20	}
21	err = f.Close()
22	if err != nil {
23		fmt.Println(err)
24		return
25	}
26	fmt.Println("file appended successfully")
27}
```

go

In line no. 9 of the program above, we open the file in append and write only mode. After the file is opened successfully, we add a new line to the file in line no. 15. This program will print `file appended successfully`. After running this program, the contents of the `lines` file will be,

```fallback
Welcome to the world of Go1.
Go is a compiled language.
It is easy to learn Go.
File handling is easy.
```

### Writing to file concurrently

When multiple [goroutines](https://golangbot.com/goroutines/) write to a file concurrently, we will end up with a [race condition](https://golangbot.com/mutex/#criticalsection). Hence concurrent writes to a file must be coordinated using a channel.

We will write a program that creates 100 goroutines. Each of this goroutine will generate a random number concurrently, thus generating hundred random numbers in total. These random numbers will be written to a file. We will solve the [race condition](https://golangbot.com/mutex/#criticalsection) problem by using the following approach.

1. Create a channel that will be used to read and write the generated random numbers.
2. Create 100 producer goroutines. Each goroutine will generate a random number and will also write the random number to a channel.
3. Create a consumer goroutine that will read from the channel and write the generated random number to the file. Thus we have only one goroutine writing to a file concurrently thereby avoiding race condition :)
4. Close the file once done.

Let’s write the `produce` function first which generates the random numbers.

```fallback
1func produce(data chan int, wg *sync.WaitGroup) {
2	n := rand.Intn(999)
3	data <- n
4	wg.Done()
5}
```

The function above generates a random number and writes it to the channel `data` and then calls `Done` on the [waitgroup](https://golangbot.com/buffered-channels-worker-pools/#waitgroup) to notify that it is done with its task.

Let’s move to the function which writes to the file now.

```fallback
 1func consume(data chan int, done chan bool) {
 2	f, err := os.Create("concurrent")
 3	if err != nil {
 4		fmt.Println(err)
 5		return
 6	}
 7	for d := range data {
 8		_, err = fmt.Fprintln(f, d)
 9		if err != nil {
10			fmt.Println(err)
11			f.Close()
12			done <- false
13			return
14		}
15	}
16	err = f.Close()
17	if err != nil {
18		fmt.Println(err)
19		done <- false
20		return
21	}
22	done <- true
23}
```

The `consume` function creates a file named `concurrent`. It then reads the random numbers from the `data` channel and writes to the file. Once it has read and written all the random numbers, it writes `true` to the `done` channel to notify that it’s done with its task.

Let’s write the `main` function and complete this program. I have provided the entire program below.

```go
 1package main
 2
 3import (
 4	"fmt"
 5	"math/rand"
 6	"os"
 7	"sync"
 8)
 9
10func produce(data chan int, wg *sync.WaitGroup) {
11	n := rand.Intn(999)
12	data <- n
13	wg.Done()
14}
15
16func consume(data chan int, done chan bool) {
17	f, err := os.Create("concurrent")
18	if err != nil {
19		fmt.Println(err)
20		return
21	}
22	for d := range data {
23		_, err = fmt.Fprintln(f, d)
24		if err != nil {
25			fmt.Println(err)
26			f.Close()
27			done <- false
28			return
29		}
30	}
31	err = f.Close()
32	if err != nil {
33		fmt.Println(err)
34		done <- false
35		return
36	}
37	done <- true
38}
39
40func main() {
41	data := make(chan int)
42	done := make(chan bool)
43	wg := sync.WaitGroup{}
44	for i := 0; i < 100; i++ {
45		wg.Add(1)
46		go produce(data, &wg)
47	}
48	go consume(data, done)
49	go func() {
50		wg.Wait()
51		close(data)
52	}()
53	d := <-done
54	if d {
55		fmt.Println("File written successfully")
56	} else {
57		fmt.Println("File writing failed")
58	}
59}
```

go

The main function creates the `data` channel in line no. 41 from which random numbers are read from and written. The `done` channel in line no. 42 is used by the `consume` goroutine to notify `main` that it is done with its task. The `wg` waitgroup in line no. 43 is used to wait for all the 100 goroutines to finish generating random numbers.

The `for` loop in line no. 44 creates 100 goroutines. The goroutine call in line no. 49 calls `wait()` on the waitgroup to wait for all 100 goroutines to finish creating random numbers. After that, it closes the channel. Once the channel is closed and the `consume` goroutine has finished writing all generated random numbers to the file, it writes `true` to the `done` channel in line no. 37 and the main goroutine is unblocked and prints `File written successfully`.

Now you can open the file **concurrent** in any text editor and see the 100 generated random numbers :)

This brings us to the end of this tutorial. Hope you enjoyed reading. Have a great day.

Please leave your feedback and comments. Please consider sharing this tutorial on [twitter](https://twitter.com/intent/tweet?text=Writing Files using Go&url=https%3a%2f%2fgolangbot.com%2fwrite-files%2f&tw_p=tweetbutton) or [LinkedIn](https://golangbot.com/write-files/#linkedinshr).

Previous tutorial - [Reading Files](https://golangbot.com/read-files/)
