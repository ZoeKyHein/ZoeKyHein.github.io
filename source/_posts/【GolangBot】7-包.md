---
title: 【GolangBot】7-包
date: 2024-12-17 08:48:29
tags: 
    - GoLang
    - 教程
categories:
    - [学习心得, GoLangBot]
published: false
---

Welcome to tutorial number 7 in [Golang tutorial series](https://golangbot.com/learn-golang-series/).

### What are packages and why are they used?

So far we have seen Go programs that have only one file with a main [function](https://golangbot.com/functions/) and a couple of other functions. In real-world scenarios, this approach of writing all source code in a single file is not scalable. It becomes impossible to reuse and maintain code written this way. This is where packages are helpful.

**Packages are used to organize Go source code for better reusability and readability. Packages are a collection of Go sources files that reside in the same directory. Packages provide code compartmentalization and hence it becomes easy to maintain Go projects.**

For example, let’s say we are writing a fintech application in Go and some of the functionalities are simple interest calculation, compound interest calculation and loan repayment calculation. One simple way to organize this application is by functionality. We can create packages `simpleinterest`, `compoundinterest` and `loanrepayment`. If the `loanrepayment` package needs to calculate the simple interest, it can simply do so by importing the `simpleinterest` package. This way the code is reused.

We will learn packages by creating a simple application to determine the simple interest given principal, interest rate and the time duration in years.

### Go Modules

We will structure the code in such a way that all functionalities related to simple interest are in `simpleinterest` package. To do that we need to create a custom package `simpleinterest` which will contain the function to calculate the simple interest. Before creating custom packages, we need to understand Go Modules first, since **Go Modules** are needed to create custom packages.

**A Go Module is nothing but a collection of Go packages.** Now this question might come to your mind. Why do we need Go modules to create a custom package? The answer is **the import path for the custom package we create is derived from the name of the go module**. In addition to this, all the other third-party packages(such as source code from github) along with their versions which our application uses will be managed by the `go.mod` file. This `go.mod` file is created when we create a new module. You will understand this better in the next section.

### Creating a Go module

Run the command below to create a directory named learnpackage inside the current user’s Documents directory.

```fallback
1mkdir ~/Documents/learnpackage/ 
```

Make sure you are inside the directory `learnpackage` by typing `cd ~/Documents/learnpackage/`. Inside this directory run the following command to create a go module named *learnpackage*.

```fallback
go mod init learnpackage
```

The above command will create a file named `go.mod`. The following will be the contents of the file.

```v
module learnpackage

go 1.21.0
```

v

The line `module learnpackage` specifies that the module’s name is `learnpackage`. As we mentioned earlier, `learnpackage` will be the base path to import any package created inside this module. The line `go 1.21.0` specifies that the files in this module use go version `1.21.0`.

### Create the simple interest custom package

**All Go files that belong to a package should be placed in separate folders of their own. It is a convention in Go to name this folder with the same name as the package.**

Let’s create a folder named `simpleinterest` inside the `learnpackage` folder. `mkdir simpleinterest` will create this folder for us.

**`package packagename` specifies that a particular .go file belongs to package `packagename`. This should be the first line of every go source file.** Hence all files inside the *simpleinterest* folder should start with the line

```go
package simpleinterest
```

go

as they all belong to the `simpleinterest` package.

Create a file `simpleinterest.go` inside the *simpleinterest* folder.

The following will be the directory structure of our application.

```fallback
learnpackage/
├── go.mod
└── simpleinterest
    └── simpleinterest.go
```

Add the following code to the `simpleinterest.go` file.

```go
1package simpleinterest
2
3//Calculate calculates and returns the simple interest for a principal p, rate of interest r for time duration t years
4func Calculate(p float64, r float64, t float64) float64 {
5	interest := p * (r / 100) * t
6	return interest
7}
```

go

In the above code, we have created a function `Calculate` which calculates and returns the simple interest. This function is self-explanatory. It calculates and returns the simple interest.

*Note that the function name **Calculate** starts with caps. This is essential and we will explain shortly why this is needed.*

### main package and the main function

The next step of the tutorial is to import the `simpleinterest` package we just created and use it. We will import the simple interest package from the `main` package. Let’s first understand about the `main` function and the `main` package.

Every executable Go application must contain the `main` function. This function is the entry point for execution. The `main` function should reside in the `main` package.

Let’s get started by creating the `main` function and `main` package for our application.

Create a file named `main.go` inside our `learnpackage` directory with the following contents.

```go
1package main 
2
3import "fmt"
4
5func main() { 
6	fmt.Println("Simple interest calculation")
7}
```

go

The line of code `package main` specifies that this file belongs to the main package. The `import "packagename"` statement is used to import an existing package. `packagename.FunctionName()` is the syntax to call a function in a package.

In line no. 3, we import the `fmt` package to use the `Println` function. The `fmt` is a standard package and is available as a part of the Go standard library. Then there is the `main` function which prints `Simple interest calculation`

Compile the above program by moving to the `learnpackage` directory using

```fallback
cd ~/Documents/learnpackage/
```

and typing the following command

```fallback
go build
```

If all went well, our binary will be compiled and will be ready for execution. Type the command `./learnpackage` in the terminal and you will see the following output.

```fallback
Simple interest calculation
```

If you don’t understand how `go build` works, please visit [this hello world tutorial](https://golangbot.com/hello-world-gomod) to know more.

### Importing the simpleinterest package in main

To use a custom package we must import it first. The import path is the name of the go module concatenated by the directory where the package resides and the package name.

In our case the go module name is `learnpackage` and the package `simpleinterest` is in the `simpleinterest` folder directly under `learnpackage`

```fallback
├── learnpackage
│   └── simpleinterest
```

So the line `import "learnpackage/simpleinterest"` will import the *simpleinterest* package.

In case we have a directory structure like this

```fallback
learnpackage
│   └── finance
│       └── simpleinterest 
```

then the import statement would be `import "learnpackage/finance/simpleinterest"`

Add the following code to `main.go`

```go
 1package main
 2
 3import (
 4	"fmt"
 5	"learnpackage/simpleinterest"
 6)
 7
 8func main() {
 9	fmt.Println("Simple interest calculation")
10	p := 5000.0
11	r := 10.0
12	t := 1.0
13	si := simpleinterest.Calculate(p, r, t)
14	fmt.Println("Simple interest is", si)
15}
```

go

The above code imports the `simpleinterest` package and uses the `Calculate` function to find the simple interest. Packages in the standard library don’t need the module name prefix and hence `fmt` works without the module prefix. When the application is run, the output will be

```fallback
Simple interest calculation
Simple interest is 500
```

### A bit more on go build

Now that we understand how packages work, it’s time to talk a little bit more about `go build`. Go tools like `go build` work in the context of the current directory. Let’s understand what that means. Till now we have been running `go build` from the directory `~/Documents/learnpackage/`. If we try to run it from any other directory, it will fail.

Try cding into `cd ~/Documents/` and then running `go build learnpackage`. It will fail with the following error.

```go
package learnpackage is not in std (/usr/local/go/src/learnpackage)
```

go

Let’s understand the reason behind this error. `go build` takes an optional package name as a parameter(in our case the package name is `learnpackage`) and it tries to compile the main function if the package exists in the current directory from which it is run or in the parent directory or it’s parent directory and so on.

We are in `Documents` directory and there is no `go.mod` file there and hence `go build` complains that it cannot find the package `learnpackage`.

When we move to `~/Documents/learnpackage/`, there is a `go.mod` file there and our module name is `learnpackage` in that `go.mod` file.

so `go build learnpackage` will work from inside the `~/Documents/learnpackage/` directory.

But so far we have just been using `go build` and we did not specify the package name. If no package name is specified, `go build` will default to the module name in the current working directory. That’s why when `go build` is run without any package name from `~/Documents/learnpackage/` it worked. So the following 3 commands are equivalent when run from `~/Documents/learnpackage/`

```fallback
go build

go build .

go build learnpackage
```

I also mentioned that `go build` has the ability to recursively search the parent directory for a go.mod file. Let’s check whether that works.

```fallback
cd ~/Documents/learnpackage/simpleinterest/
```

The above command will take us to the `simpleinterest` directory. From that directory run

```fallback
go build learnpackage
```

go build will successfully find a `go.mod file` in the parent directory `learnpackage` that has the module `learnpackage` defined and hence it works :).

It’s also possible to change the name of the output binary file using `go build`. Move to `~/Documents/learnpackage` and type

```fallback
1go build -o fintechapp
```

The `-o` argument is used to specify the name of the output binary. In this case a binary file of name `fintechapp` will be created.

Run `./fintechapp` and the binary will be run successfully.

### Exported Names

We capitalized the function `Calculate` in the Simple interest package. This has a special meaning in Go. Any [variable](https://golangbot.com/variables/) or function which starts with a capital letter are exported names in go. Only exported functions and variables can be accessed from other packages. In our case, we want to access `Calculate` function from the main package. Hence this is capitalized.

If the function name is changed from `Calculate` to `calculate` in `simpleinterest.go`, and if we try to call the function using `simpleinterest.calculate(p, r, t)` in main.go, the compiler will error

```fallback
# learnpackage
./main.go:13:8: undefined: simpleinterest.calculate
```

Hence if you want to access a function outside of a package, it should be capitalized.

### init function

Each package in Go can contain an `init` function. The `init` function must not have any return type and it must not have any parameters. The init function cannot be called explicitly in our source code. It will be called automatically when the package is initialized. The init function has the following syntax

```fallback
1func init() {
2}
```

The `init` function can be used to perform initialization tasks and can also be used to verify the correctness of the program before the execution starts.

The order of initialization of a package is as follows

1. Package level variables are initialised first
2. init function is called next. A package can have multiple init functions (either in a single file or distributed across multiple files) and they are called in the order in which they are presented to the compiler.

If a package imports other packages, the imported packages are initialized first.

A package will be initialized only once even if it is imported from multiple packages.

Let’s make some modifications to our application to understand `init` functions.

To start with let’s add the `init` function to the `simpleinterest.go` file.

```go
 1package simpleinterest
 2
 3import "fmt"
 4
 5/*
 6 * init function added
 7 */
 8func init() {
 9	fmt.Println("Simple interest package initialized")
10}
11
12//Calculate calculates and returns the simple interest for principal p, rate of interest r for time duration t years
13func Calculate(p float64, r float64, t float64) float64 {
14	interest := p * (r / 100) * t
15	return interest
16}
```

go

We have added a simple init function which just prints `Simple interest package initialised`

Now let’s modify the main package. We know that the principal, rate of interest and time duration should be greater than zero when calculating simple interest. We will define this check using init function and package level variables in the `main.go` file.

Modify the `main.go` to the following,

```go
 1package main 
 2
 3import (
 4	"fmt"
 5	"learnpackage/simpleinterest" //importing custom package
 6	"log"
 7)
 8
 9var p, r, t = 5000.0, 10.0, 1.0
10
11/*
12* init function to check if p, r and t are greater than zero
13 */
14func init() {
15	fmt.Println("Main package initialized")
16	if p < 0 {
17		log.Fatal("Principal is less than zero")
18	}
19	if r < 0 {
20		log.Fatal("Rate of interest is less than zero")
21	}
22	if t < 0 {
23		log.Fatal("Duration is less than zero")
24	}
25}
26
27func main() {
28	fmt.Println("Simple interest calculation")
29	si := simpleinterest.Calculate(p, r, t)
30	fmt.Println("Simple interest is", si)
31}
```

go

The following are the changes made to `main.go`

1. **p**, **r** and **t** variables are moved to package level from the main function level.
2. An init function has been added. The *init* function prints a log and terminates the program execution if either the principal, rate of interest or time duration is less than zero using **log.Fatal** function.

The order of initialisation of the is as follows,

1. The imported packages are initialized first. Hence **simpleinterest** package is initialized first and it's init method is called.
2. Package level variables **p**, **r** and **t** are initialized next.
3. **init** function is called in main package
4. **main** function is called at last.

If you run the program, you will get the following output.

```go
Simple interest package initialized
Main package initialized
Simple interest calculation
Simple interest is 500
```

go

As expected the init function of the `simpleinterest` package is called first followed by the initialization of the package level variables `p`, `r` and `t`. The init function of the main package is called next. It checks whether `p`, `r` and `t` are lesser than zero and terminates if the condition is true. We will learn about `if` statement in detail in a [separate tutorial](https://golangbot.com/if-statement/). For now you can assume that `if p < 0` will check whether `p` is less than 0 and if it is, the program will be terminated. We have written a similar condition for `r` and `t`. In this case, all these conditions are false and the program execution continues. Finally, the main function is called.

Let’s modify this program a bit to learn the use of the init function.

Change the line

```fallback
var p, r, t = 5000.0, 10.0, 1.0
```

in `main.go` to

```fallback
var p, r, t = -5000.0, 10.0, 1.0
```

We have initialised `p` to negative.

Now if you run the application, you will see

```go
1Simple interest package initialized
2Main package initialized
32024/04/01 02:58:32 Principal is less than zero
```

go

*p* is negative. Hence when the init function runs, the program terminates after printing `Principal is less than zero`.

### Use of blank identifier

It is illegal in Go to import a package and not to use it anywhere in the code. The compiler will complain if you do so. The reason for this is to avoid bloating of unused packages which will significantly increase the compilation time. Replace the code in `main.go` with the following,

```go
1package main
2  
3import (
4        "learnpackage/simpleinterest"
5)
6
7func main() {
8
9}
```

go

The above program will error

```fallback
# learnpackage
./main.go:4:2: imported and not used: "learnpackage/simpleinterest"
```

But it is quite common to import packages when the application is under active development and use them somewhere in the code later if not now. The `_` blank identifier saves us in those situations.

The error in the above program can be silenced by the following code,

```go
 1package main
 2  
 3import (
 4        "learnpackage/simpleinterest"
 5)
 6
 7var _ = simpleinterest.Calculate
 8
 9func main() {
10
11}
```

go

The line `var _ = simpleinterest.Calculate` mutes the error. We should keep track of these kinds of error silencers and remove them including the imported package at the end of application development if the package is not used. Hence it is recommended to write error silencers in the package level just after the import statement.

Sometimes we need to import a package just to make sure the initialization takes place even though we do not need to use any function or variable from the package. For example, we might need to ensure that the `init` function of the `simpleinterest` package is called even though we plan not to use that package anywhere in our code. The _ blank identifier can be used in this case too as shown below.

```go
package main

import (
	_ "learnpackage/simpleinterest"
)

func main() {

}
```

go

Running the above program will output `Simple interest package initialized`. We have successfully initialized the `simpleinterest` package even though it is not used anywhere in the code.

This brings us to the end of this tutorial. Hope you enjoyed reading. Please leave your feedback and comments. Please consider sharing this tutorial on [twitter](https://twitter.com/intent/tweet?text=Go Packages&url=https%3a%2f%2fgolangbot.com%2fgo-packages%2f&tw_p=tweetbutton) and [LinkedIn](https://golangbot.com/go-packages/#linkedinshr). Have a good day.

**Next tutorial - [if else statement](https://golangbot.com/if-statement/)**
