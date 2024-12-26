---
title: 【GolangBot】36-读文件
date: 2024-12-17 09:20:08
tags: 
    - Golang
    - 教程
categories:
    - [学习心得, GolangBot]
published: false
---

Welcome to tutorial no. 36 in [Golang tutorial series](https://golangbot.com/learn-golang-series/).

File reading is one of the most common operations performed in any programming language. In this tutorial, we will learn about how files can be read using Go.

This tutorial has the following sections.

- Reading an entire file into memory
  - Using an absolute file path
  - Passing the file path as a command line flag
  - Bundling the file inside the binary
- Reading a file in small chunks
- Reading a file line by line

### Reading an entire file into memory

One of the most basic file operations is reading an entire file into memory. This is done with the help of the [ReadFile](https://pkg.go.dev/os#ReadFile) function of the [os](https://pkg.go.dev/os) package.

Let’s read a file and print its contents.

I have created a folder `filehandling` inside my `Documents` directory by running `mkdir ~/Documents/filehandling`.

Create a Go module named `filehandling` by running the following command from the `filehandling` directory.

```fallback
go mod init filehandling
```

I have a text file `test.txt` which will be read from our Go program `filehandling.go`. `test.txt` contains the following string

```fallback
Hello World. Welcome to file handling in Go.
```

Here is my directory structure.

```fallback
├── Documents
│   └── filehandling
│       ├── filehandling.go
|       ├── go.mod
│       └── test.txt
```

Let’s get to the code right away. Create a file `filehandling.go` with the following contents.

```go
 1package main
 2
 3import (
 4	"fmt"
 5	"os"
 6)
 7
 8func main() {
 9	contents, err := os.ReadFile("test.txt")
10	if err != nil {
11		fmt.Println("File reading error", err)
12		return
13	}
14	fmt.Println("Contents of file:", string(contents))
15}
```

go

Please run this program from your local environment as it’s not possible to read files in the playground.

Line no. 9 of the program above reads the file and returns a byte [slice](https://golangbot.com/arrays-and-slices/#slices) which is stored in `contents`. In line no. 14 we convert `contents` to a `string` and display the contents of the file.

Please run this program from the location where **test.txt** is present.

If **test.txt** is located at **~/Documents/filehandling**, then run this program using the following steps,

```fallback
cd ~/Documents/filehandling/
go install 
filehandling
```

If you are not aware of how to run a Go program, please visit [/hello-world-gomod/](https://golangbot.com/hello-world-gomod/) to know more. If you want to learn more about packages and Go modules, please visit [/go-packages/](https://golangbot.com/go-packages/)

This program will print,

```fallback
Contents of file: Hello World. Welcome to file handling in Go.  
```

If this program is run from any other location, for instance, try running the program from `~/Documents/`

```fallback
cd ~/Documents/
filehandling
```

It will print the following error.

```fallback
File reading error open test.txt: no such file or directory
```

The reason is Go is a compiled language. What `go install` does is, it creates a binary from the source code. The binary is independent of the source code and it can be run from any location. Since `test.txt` is not found in the location from which the binary is run, the program complains that it cannot find the file specified.

There are three ways to solve this problem,

1. Using absolute file path
2. Passing the file path as a command line flag
3. Bundling the text file along with the binary

Let’s discuss them one by one.

Get the free Golang tools cheat sheet

##### 1. Using absolute file path

The simplest way to solve this problem is to pass the absolute file path. I have modified the program and changed the path to an absolute one in line no. 9. Please change this path to the absolute location of your `test.txt`.

```go
 1package main
 2
 3import (
 4	"fmt"
 5	"os"
 6)
 7
 8func main() {
 9	contents, err := os.ReadFile("/Users/naveen/Documents/filehandling/test.txt")
10	if err != nil {
11		fmt.Println("File reading error", err)
12		return
13	}
14	fmt.Println("Contents of file:", string(contents))
15}
```

go

Now the program can be run from any location and it will print the contents of `test.txt`.

For example, it will work even when I run it from my home directory

```fallback
cd ~/Documents/filehandling
go install
cd ~
filehandling
```

The program will print the contents of `test.txt`

This seems to be an easy way but comes with the pitfall that the file should be located in the path specified in the program else this method will fail.

##### 2. Passing the file path as a command line flag

Another way to solve this problem is to pass the file path as a command line argument. Using the [flag](https://pkg.go.dev/flag) package, we can get the file path as input argument from the command line and then read its contents.

Let’s first understand how the `flag` package works. The `flag` package has a [String](https://golang.org/pkg/flag/#String) function. This function accepts 3 arguments. The first is the name of the flag, second is the default value and the third is a short description of the flag.

Let’s write a small program to read the file name from the command line. Replace the contents of `filehandling.go` with the following,

```go
 1package main
 2import (
 3	"flag"
 4	"fmt"
 5)
 6
 7func main() {
 8	fptr := flag.String("fpath", "test.txt", "file path to read from")
 9	flag.Parse()
10	fmt.Println("value of fpath is", *fptr)
11}
```

go

Line no. 8 of the program above, creates a string flag named `fpath` with default value `test.txt` and description `file path to read from` using the `String` function. This function returns the address of the string [variable](https://golangbot.com/variables/) that stores the value of the flag.

*flag.Parse()* should be called before accessing any flag.

We print the value of the flag in line no. 10

When this program is run using the command

```fallback
filehandling -fpath=/path-of-file/test.txt
```

we pass `/path-of-file/test.txt` as the value of the flag `fpath`.

This program outputs

```fallback
value of fpath is /path-of-file/test.txt
```

If the program is run using just `filehandling` without passing any `fpath`, it will print

```fallback
value of fpath is test.txt
```

since `test.txt` is the default value of `fpath`.

`flag` also provides a nicely formatted output of the different arguments that are available. This can be displayed by running

```fallback
filehandling --help
```

This command will print the following output.

```fallback
Usage of filehandling:
  -fpath string
    	file path to read from (default "test.txt")
```

Nice isn’t it :).

Now that we know how to read the file path from the command line, let’s go ahead and finish our file reading program.

```go
 1package main
 2
 3import (
 4	"flag"
 5	"fmt"
 6	"os"
 7)
 8
 9func main() {
10	fptr := flag.String("fpath", "test.txt", "file path to read from")
11	flag.Parse()
12	contents, err := os.ReadFile(*fptr)
13	if err != nil {
14		fmt.Println("File reading error", err)
15		return
16	}
17	fmt.Println("Contents of file:", string(contents))
18}
```

go

The program above reads the content of the file path passed from the command line. Run this program using the command

```fallback
filehandling -fpath=/path-of-file/test.txt
```

Please replace `/path-of-file/` with the absolute path of `test.txt`. For example, in my case, I ran the command

```fallback
filehandling --fpath=/Users/naveen/Documents/filehandling/test.txt
```

and the program printed.

```fallback
Contents of file: Hello World. Welcome to file handling in Go.
```

##### 3. Bundling the text file along with the binary

The above option of getting the file path from the command line is good but there is an even better way to solve this problem. Wouldn’t it be awesome if we are able to bundle the text file along with our binary? This is what we are going to do next.

The [embed](https://pkg.go.dev/embed) package from the standard library will help us achieve this.

After importing the `embed` package, the `//go:embed` directive can be used to read the contents of the file.

A program will make us understand things better.

Replace the contents of `filehandling.go` with the following,

```go
 1package main
 2
 3import (
 4	_ "embed"
 5	"fmt"
 6)
 7
 8//go:embed test.txt
 9var contents []byte
10
11func main() {
12	fmt.Println("Contents of file:", string(contents))
13}
```

go

In line no. 4 of the program above, we import the `embed` package with a underscore prefix. The reason is because `embed` is not explicitly used in the code but the `//go:embed` comment in line no. 8 needs some preprocessing by the compiler. Since we need to import the package without any explicit usage, we prefix it with underscore to make the compiler happy. If not, the compiler will complain stating that the package is not used anywhere.

The `//go:embed test.txt` in line no. 8 tells the compiler to read the contents of `test.txt` and assign it to the variable following that comment. In our case `contents` variable will hold the contents of the file.

Run the program using the following commands.

```fallback
cd ~/Documents/filehandling
go install
filehandling
```

and the program will print

```fallback
Contents of file: Hello World. Welcome to file handling in Go.
```

Now the file is bundled along with the binary and it is available to the go binary irrespective of where it’s executed from. For example, try running the program from a directory where `test.txt` doesn’t reside.

```fallback
cd ~/Documents
filehandling
```

The above command will also print the contents of the file.

Do note that the variable to which the contents of the file should be assigned to must be at the package level. Local variables won’t work. Try changing the program to the following.

```go
 1package main
 2
 3import (
 4	_ "embed"
 5	"fmt"
 6)
 7
 8func main() {
 9	//go:embed test.txt
10	var contents []byte
11	fmt.Println("Contents of file:", string(contents))
12}
```

go

The above program has `contents` as a local variable.

The program will now fail to compile with the following error.

```fallback
./filehandling.go:9:4: go:embed cannot apply to var inside func
```

If you are interested to know more about the design decision behind this, please read https://github.com/golang/go/issues/43216

Get the free Golang tools cheat sheet

### Reading a file in small chunks

In the last section, we learned how to load an entire file into memory. When the size of the file is extremely large it doesn’t make sense to read the entire file into memory especially if you are running low on RAM. A more optimal way is to read the file in small chunks. This can be done with the help of the [bufio](https://pkg.go.dev/bufio) package.

Let’s write a program that reads our `test.txt` file in chunks of 3 bytes. Replace the contents of `filehandling.go` with the following,

```go
 1package main
 2
 3import (
 4	"bufio"
 5	"flag"
 6	"fmt"
 7	"io"
 8	"log"
 9	"os"
10)
11
12func main() {
13	fptr := flag.String("fpath", "test.txt", "file path to read from")
14	flag.Parse()
15
16	f, err := os.Open(*fptr)
17	if err != nil {
18		log.Fatal(err)
19	}
20	defer func() {
21		if err = f.Close(); err != nil {
22			log.Fatal(err)
23		}
24	}()
25
26	r := bufio.NewReader(f)
27	b := make([]byte, 3)
28	for {
29		n, err := r.Read(b)
30		if err == io.EOF {
31			fmt.Println("finished reading file")
32			break
33		}
34		if err != nil {
35			fmt.Printf("Error %s reading file", err)
36			break
37		}
38		fmt.Println(string(b[0:n]))
39	}
40}
```

go

In line no. 16 of the program above, we open the file using the path passed from the command line flag.

In line no. 20, we defer the file closing.

Line no. 26 of the program above creates a new buffered reader. In the next line, we create a byte slice of length and capacity 3 into which the bytes of the file will be read.

The `Read` [method](https://golangbot.com/methods/) in line no. 29 reads up to `len(b)` bytes i.e up to 3 bytes and returns the number of bytes read. We store the bytes returned in a variable`n`. In line no. 38, the slice is read from index `0` to `n-1`, i.e up to the number of bytes returned by the `Read` method and printed.

Once the end of the file is reached, `read` will return an EOF error. We check for this error in line no. 30. The rest of the program is straight forward.

If we run the program above using the commands,

```fallback
cd ~/Documents/filehandling
go install
filehandling -fpath=/path-of-file/test.txt
```

the following will be output

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

### Reading a file line by line

In the section, we will discuss how to read a file line by line using Go. This can done using the [bufio](https://pkg.go.dev/bufio) package.

Please replace the contents in `test.txt` with the following

```fallback
Hello World. Welcome to file handling in Go.  
This is the second line of the file.  
We have reached the end of the file.  
```

The following are the steps involved in reading a file line by line.

1. Open the file
2. Create a new scanner from the file
3. Scan the file and read it line by line.

Replace the contents of `filehandling.go` with the following

```go
 1package main
 2
 3import (
 4	"bufio"
 5	"flag"
 6	"fmt"
 7	"log"
 8	"os"
 9)
10
11func main() {
12	fptr := flag.String("fpath", "test.txt", "file path to read from")
13	flag.Parse()
14
15	f, err := os.Open(*fptr)
16	if err != nil {
17		log.Fatal(err)
18	}
19    defer func() {
20	    if err = f.Close(); err != nil {
21		log.Fatal(err)
22	}
23	}()
24	s := bufio.NewScanner(f)
25	for s.Scan() {
26		fmt.Println(s.Text())
27	}
28	err = s.Err()
29	if err != nil {
30		log.Fatal(err)
31	}
32}
```

go

In line no. 15 of the program above, we open the file using the path passed from the command line flag. In line no. 24, we create a new scanner using the file. The `scan()` method in line no. 25 reads the next line of the file and the string that is read will be available through the `text()` method.

After Scan returns `false`, the `Err()` method will return any error that occurred during scanning. If the error is End of File, `Err()` will return `nil`.

If we run the program above using the commands,

```fallback
cd ~/Documents/filehandling
go install
filehandling -fpath=/path-of-file/test.txt
```

the contents of the file will be printed line by line as shown below.

```fallback
Hello World. Welcome to file handling in Go.
This is the second line of the file.
We have reached the end of the file.
```

This brings us to the end of this tutorial. Hope you enjoyed it. Please leave your feedback and comments. Please consider sharing this tutorial on [twitter](https://twitter.com/intent/tweet?text=Reading Files&url=https%3a%2f%2fgolangbot.com%2fread-files%2f&tw_p=tweetbutton) and [LinkedIn](https://golangbot.com/read-files/#linkedinshr). Have a good day.

**Next tutorial - [Writing Files](https://golangbot.com/write-files/)**
