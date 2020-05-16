---
layout: post
title:  "Go for C# Developers"
date:   2020-05-13 13:05:07 -0700
categories: jekyll update
---
# Go for C# Developers

## Overview

- Goals of this blog post
- Quick Go summary
- Basic similarities/differences
- Dependency management
- Brief tooling introduction
- Rapid-fire syntax rundown
- Concurrency and asynchronous programming in Go vs. C#
- Brief note on garbage collection
- Some helpful links for more information

## Goals of this Blog Post

Welcome! This post assumes working knowledge of C# and it's targetted toward those without Go experience. The aim is to provide just enough of an introduction to the language to make you dangerous :P. It's also somewhat influenced by [A half-hour to learn Rust](https://fasterthanli.me/blog/2020/a-half-hour-to-learn-rust/). When I start learning a new language, I tend to learn best by doing - that is, I like to experiment and make the dumb mistakes myself, before the deeper lessons sink in, and I'd like to make this the sort of blog post I wish I had when I was learning Go for the first time.

That being said, there's a couple basic things we have to discuss up front before we can jump straight into writing Go code. Let's start with a brief overview of the language and some of its major design decisions.

## Quick Go Summary

Go was developed at Google, first announced in 2009 and initially released in 2012. The language is:

- statically-typed
- compiled
- automatically garbage-collected

Go programs compile down to a single binary with all dependencies included. In contrast to C#, no runtime is needed, so applications can be run in very thin containers such as [alpine](https://hub.docker.com/_/alpine/).

In addition, Go has a strong focus on concurrency. The approach it takes is a version of [Communicating Sequential Processes](https://en.wikipedia.org/wiki/Communicating_sequential_processes), which is a formal way of specifying rules for interaction in concurrent systems whose details are beyond the scope of this post. For our purposes, it's enough to know that Go "shares memory by communicating", in which various concurrent processes called "goroutines" pass objects between them. This is a large departure from more traditional object-oriented concurrency schemes, in which applications instead "communicate by sharing memory": traditionally multiple threads share a reference to an object, and writes are controlled through a delicate balance of locks and mutexes.

Go is frequently compared to C due to a similar syntax, pointer usage, and relatively un-object-oriented nature. However, Go is much safer than C (it's much harder to cut yourself on a sharp pointer) and its concurrency tooling and robust standard library means it offers significant capability on top of vanilla C.

Go tends to align well with a couple of different use cases. Its small binary size, concurrency tooling, and fast execution speed means it's frequently used for high-volume distributed webservices, most frequently ones that speak RPC but RESTful services as well. Some of these factors also make it useful for writing CLIs or interactive tools for testing services. You'll commonly see a simple CLI included with a Twirp service for testing purposes here at GitHub.

Go plays pretty nicely with C, and includes functionality that both let applications call C code from Go, and export Go code to C standard libraries (which can then be called by numerous other languages, including C, Node, Python, Java, and Ruby). For this reason, it's a common choice when writing libraries and frameworks, even ones that need to integrate with other languages.

Many of the attributes discussed above also make Go a reliable choice for automation, monitoring, and data processing tools. For more information on what the Go community is using the language for, you can view the results of the [yearly developer survey](https://blog.golang.org/survey2019-results).

Go is a highly-opinionated language. All Go source code is formatted the same, thanks to the `gofmt` command. Uncapitalized members are always kept private, whereas capitalized ones are public (there are no `public`/`private` modifier keywords!). There is no concept of inheritance; instead the language enforces composition. All of these decisions require every developer to either get with the program or leave, but once you re-align your thinking to the Go way of doing things, it allows for a surprising amount of speed and freedom. 


## Basic similarities and differences

### Same-Same

Both Go and C# are:

- statically-typed
- compiled
- automatically garbage-collected
- support multiple platforms
    - in Go, typically no changes are needed to support a binary e.g. compiled on macOS but running in Linux in production
- include robust standard libraries

### But Different

- Go has no runtime required (like the [.NET CLR](https://docs.microsoft.com/en-us/dotnet/standard/clr))
- Go has no generics
    - This is a consistent paint point for the Go community
    - See [here](https://go.googlesource.com/proposal/+/master/design/go2draft-generics-overview.md) for a proposal to add generics from language designer Russ Cox
- Go returns errors which are then checked by the caller, rather than try/catch blocks and exceptions for error handling
- Go has no concept of `await`ing an action (more on this later)
- Go places a strong emphasis on staying up to date with the language, in constrast to the fragmentation in the .NET and .NET Core ecosystems

## Dependency management

Dependency management used to be a somewhat contentious subject in the Go ecosystem, with a number of third party solutions attempting to address the gap in this language tooling. [v1.11](https://golang.org/doc/go1.11), however, introduced experimental support for Go modules, and [v1.14](https://golang.org/doc/go1.14) (released this February) standardized them as ready for production use.

Before we can get into modules, however, we need to briefly discuss packages. All Go source files must declare a package in their first line, and every Go file must have at least one `package main` file that declares a `func main()`. Other than the file that contains `main`, code must be placed in a directory whose name matches the `package` of files within it. As previously mentioned, all uncapitalized members and functions are private, and all capitalized ones are publically visible. 

Modules allow reusable packages to be imported using their URL, as in `import github.com/go-sql-driver/mysql`. Interestingly, packages from within a project as well as third party dependencies are imported using _the exact same syntax_. Put another way: if a function is public in your code, and your code is accessible, anybody can import and use it! This can lead to some weird issues [like this one](https://github.com/github/hub/issues/2517#issuecomment-615531677), in which parts of an application are consumed as a library by other applications.

Similar to Nuget packages (and unlike Node modules), modules are cached at a user level and not at a repository level. Somewhat like a `Nuget.config` file, module dependencies are declared in a `go.mod` file  in the project's root. The syntax for this file is as follows:

```go
module github.com/github/go-sample-service

go 1.14

require (
	github.com/go-sql-driver/mysql v1.5.0
)
```

You'll notice three important sections: the top line is the declaration of the module name (also where it's hosted). The second line is the version of Go the module uses. After that, all the modules the application imports are declared and given a version.

Building and/or running your code will result in a generated file `go.sum` that contains the checksums for all the packages you use. Similar to a `package.json` (although NOT a lockfile, it contains enough information for reproducible builds), this should be checked into source control.

## Brief tooling introduction

- Development in Go is heavily reliant on a few tools I'll provide a brief overview of here. To see all the tools and their documentation, run `go help` and then `go <command> help` at your terminal. For now, what you need to know are the following:

- `go get`
    - Adds and installs dependencies to your current project
- `go build`
    - Compiles packages and dependencies
    - Example use: `go build ./...` to recursively build every package in the current directory
- `go run`
    - Compiles _and_ runs a Go program
    - Example use: `go run cmd/twirp/main.go` in the [go-sample-service](https://github.com/github/go-sample-service)
- `gofmt`
    - Will reformat source code in your package
    - Many editors have plugins to perform this operation automatically on saving
- `go mod`
    - Command to interact with modules
    - Example use: `go mod tidy` will add any missing modules necessary to build and remove any unused modules
- `go doc`
    - Invaluable tool for quickly looking up Go documentation
    - Example use: `go doc fmt.Sprintf`


### Creating an example application for a sandbox

Go has a [sweet playground](https://play.golang.org/) for you to experiment in! But if you're the type who's reading this in Mutt through an RSS feed, [install Go](https://golang.org/doc/install) (or on MacOS, just run `brew install go`) and create a file called `whateverYouDamnWellPlease.go` and fill it with the following content:

```go
package main

import (
    "fmt"
)

func main() {
	fmt.Printf("identity theft is not a joke, jim")
}
```

Remember how opinionated Go is? The "Go-y" place to store your source is `~/go/src/url/to/your/code`. In practice, that looks something like `~/go/src/github.com/github/hub`. For those of you following along at home, you can dump your newly created file in a directory like `~/go/src/github.com/yourGitHubName/go-experiments`.

If you've done everything right, you can `cd` to that directory and run `go run whateverYouDamnWellPlease.go`, and you'll be able to see the string output to stdout.

## Rapid-fire syntax rundown

Buckle up, because we'll be quickly running through all the syntax you'll need in order to lose thousands of dollars with a bot that calls Robin Hood's undocumented API to trade stocks:

Create a new variable by using the `:=` operator

```go
    ans := 42
```

After the variable is created, use `=` to reassign it:

```go
    ans = 1
```

If you need to create a variable without initialization, try
```go
    var ans int
    ans = 42
```

Strings use `"` (or single backticks for multi-line strings)
```go
    res := "error fetching profile"
```

Functions are declared like so:
```go
func add(x int, y int) int {
    return x + y
}
```
Note that the returned value appears after the function arguments. Omit this if your function doesn't return a value.
Also note the lack of any `public` or `private` modifiers. This function is private to its package because it begins with a lowercase `a` instead of `A`.

Zero values are 0 (for numeric types), the empty string "" (for strings), and `false` (for booleans). Unlike C#, there is no `string.Empty` function. Use the literal value `""` if you need to check a value against an empty string.
```go
var x int
var y string
var z bool
	
fmt.Printf("%v, %v, %v", x, y, z)
// 0, , false
```
Note the usage of the standard library `fmt` to print content. To see the docs on this function, run `go doc fmt.Printf` from your terminal. This is a super quick way to access documentation without invoking [your favorite search engine](https://duckduckgo.com/), and it'll make you look 1337 in your next pair programming session.

Go only has one type of loop, but with a little massaging, it can easily replicate the normal loop types (for, while, infinite). Observe:
```go
for i := 0; i < 10; i++ {
    fmt.Printf("%v, ", i)
}
// 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 

i := 0
for i < 10 {
    fmt.Printf("%v, ", i)
    i++
}
// 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 

i := 0 
for {
    fmt.Printf("%v, ", i)
    i++
}
// 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23,
// timeout running program
```

If statements are exactly what you'd expect. The Go attitude is no parentheses, no problems.
```go
x := 0

if x > 0 {
    fmt.Printf("help i'm trapped inside a Go standard library")
} else if x < 0 {
    fmt.Printf("i can't get out of the fmt package")
} else {
    fmt.Printf("please i have kids")
}
// please i have kids
```

Switch statements are a bit different than C#. Go only allows a single case to be run at once, so you won't see `break` littered throughout the codeblock. 


## Concurrency and asynchronous programming in Go and C#


## Brief note on garbage collection

- Go is mark and sweep: https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html
    - Go hasn't done something more sophisticated because: https://groups.google.com/forum/m/#!topic/golang-nuts/KJiyv2mV2pU
- C# is mark-compact: https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals

## Miscellaneous/Questions

- discuss error handling in C# (exceptions) vs. Go (returned errors)
- have a section on "norms" like error handling and "everyone stays updated"?
- add section of things