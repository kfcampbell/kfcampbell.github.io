---
layout: post
title: "Go for C# People, Part One: The Overview"
date: 2020-05-21 00:00:00-0700
description: In which lies a brief introduction to the Go programming language, primarily geared towards C# people.
---

## In which we discuss the following:

- Goals of this blog post
- Quick Go summary
- Basic similarities/differences
- Dependency management
- Brief tooling introduction

## Goals of this blog post

Welcome! This post assumes working knowledge of C# and it's targetted toward those without Go experience. The aim is to provide just enough of an introduction to the language to make you dangerous :P. 

When I start learning a new language, I tend to learn best by doing - that is, I like to experiment and make the dumb mistakes myself, before the deeper lessons sink in, and I'd like to make this the sort of blog post I wish I had when I was learning Go for the first time.

That being said, there's a couple basic things we have to discuss up front before we can jump straight into writing Go code (I know, I'm sorry!). But trust me, it won't be so bad, and we'll let you stare at compiler errors yourself in Part Two (coming soon!). 

For now, let's start with a brief overview of the language and some of its major design decisions.

## Quick Go summary

Go was developed at Google, first announced in 2009 and initially released in 2012. The language is:

- statically-typed
- compiled
- automatically garbage-collected

Go programs compile down to a single binary with all dependencies included. In contrast to C#, no runtime is needed, so applications can be run in very thin containers such as [scratch](https://hub.docker.com/_/scratch) or [alpine](https://hub.docker.com/_/alpine/).

In addition, Go has a strong focus on concurrency. 

The approach it takes is a version of [Communicating Sequential Processes](https://en.wikipedia.org/wiki/Communicating_sequential_processes), which is a formal way of specifying rules for interaction in concurrent systems whose details are beyond the scope of this post. 

For our purposes, it's enough to know that Go "shares memory by communicating", in which various concurrent processes called "goroutines" pass objects between them. 

This is a large departure from more traditional object-oriented concurrency schemes, in which applications instead "communicate by sharing memory": traditionally multiple threads share a reference to an object, and writes are controlled through a delicate balance of locks and mutexes.

Go is frequently compared to C due to a similar syntax, pointer usage, and relatively un-object-oriented nature. 

However, Go is much safer than C (it's much harder to cut yourself on a sharp pointer) and its concurrency tooling and robust standard library means it offers significant capability on top of vanilla C.

Go tends to align well with a couple of different use cases. Its small binary size, concurrency tooling, and fast execution speed means it's frequently used for high-volume distributed webservices, most frequently ones that speak RPC but RESTful services as well. 

Some of these factors also make it useful for writing CLIs or interactive tools for testing services. You'll commonly see a simple CLI included with a Twirp service for testing purposes here at GitHub.

Go plays pretty nicely with C, and includes functionality that both lets applications call C code from Go, and export Go code to C standard libraries (which can then be called by numerous other languages, including C, Node, Python, Java, and Ruby). 

For this reason, it's a common choice when writing libraries and frameworks, even ones that need to integrate with other languages.

Many of the attributes discussed above also make Go a reliable choice for automation, monitoring, and data processing tools. For more information on what the Go community is using the language for, you can view the results of the [yearly developer survey](https://blog.golang.org/survey2019-results).

Go is a highly opinionated language. 

All Go source code is formatted the same, thanks to the `gofmt` command. Uncapitalized members are always kept private, whereas capitalized ones are public (there are no `public`/`private` modifier keywords!). There is no concept of inheritance; instead the language enforces composition. 

All of these decisions require every developer to either get with the program or leave, but once you re-align your thinking to the Go way of doing things, it allows for a surprising amount of speed and freedom. 


## Basic similarities and differences

### Same-same

Both Go and C# are:

- statically-typed
- compiled
- automatically garbage-collected
- support multiple platforms
    - in Go, typically no changes are needed to support a binary e.g. compiled on macOS but running in Linux in production
- include robust standard libraries

### But different

- Go has no runtime required (like the [.NET CLR](https://docs.microsoft.com/en-us/dotnet/standard/clr))
- Go has no generics
    - This is a consistent paint point for the Go community
    - See [here](https://go.googlesource.com/proposal/+/master/design/go2draft-generics-overview.md) for a proposal to add generics from language designer Russ Cox
- Go returns errors which are then checked by the caller, rather than try/catch blocks and exceptions for error handling
- Go has no concept of `await`ing an action (more on this later)
- Go places a strong emphasis on staying up to date with the language, in constrast to the fragmentation in the .NET and .NET Core ecosystems

## Dependency management

Dependency management used to be a somewhat contentious subject in the Go ecosystem, with a number of third party solutions attempting to address the gap in this language tooling. 

[v1.11](https://golang.org/doc/go1.11), however, introduced experimental support for Go modules, and [v1.14](https://golang.org/doc/go1.14) (released this February) standardized them as ready for production use.

Before we can get into modules, however, we need to briefly discuss packages. 

All Go source files must declare a package in their first line, and every Go file must have at least one `package main` file that declares a `func main()`. Other than the file that contains `main`, code must be placed in a directory whose name matches the `package` of files within it. 

As previously mentioned, all uncapitalized members and functions are private, and all capitalized ones are publically visible. 

Modules allow reusable packages to be imported using their URL, as in `import github.com/go-sql-driver/mysql`. Interestingly, packages from within a project as well as third party dependencies are imported using _the exact same syntax_. 

Put another way: if a function is public in your code, and your code is accessible, anybody can import and use it! 

This can lead to some weird issues [like this one](https://github.com/github/hub/issues/2517#issuecomment-615531677), in which parts of an application are consumed as a library by other applications.

Similar to Nuget packages (and unlike Node modules), modules are cached at a user level and not at a repository level. Somewhat like a `Nuget.config` file, module dependencies are declared in a `go.mod` file  in the project's root. The syntax for this file is as follows:

```go
module github.com/github/go-sample-service

go 1.14

require (
	github.com/go-sql-driver/mysql v1.5.0
)
```

You'll notice three important sections: the top line is the declaration of the module name (also where it's hosted). 

The second line is the version of Go the module uses. 

After that, all the modules the application imports are declared and given a version.

Building and/or running your code will result in a generated file `go.sum` that contains the checksums for all the packages you use. 

Similar to a `package.json` (although NOT a lockfile, it contains enough information for reproducible builds), this should be checked into source control.

## Brief tooling introduction

Development in Go is heavily reliant on a few tools I'll provide a brief overview of here. 

To see all the tools and their documentation, run `go help` and then `go <command> help` at your terminal. For now, what you need to know are the following:

- `go get`
    - Adds and installs dependencies to your current project
    - Example use: `go get github.com/go-sql-driver/mysql`
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

Go has a [sweet playground](https://play.golang.org/) for you to experiment in! 

But if you're the type who's reading this in Mutt through an RSS feed, [install Go](https://golang.org/doc/install) (or on MacOS, just run `brew install go`) and create a file called `main.go` and fill it with the following content:

```go
package main

import (
    "fmt"
)

func main() {
	fmt.Printf("identity theft is not a joke, jim")
}
```

Remember how opinionated Go is? The "Go-y" place to store your source is `~/go/src/url/to/your/code`. 

In practice, that looks something like `~/go/src/github.com/github/hub`. For those of you following along at home, you can dump your newly created file in a directory like `~/go/src/github.com/yourGitHubName/go-experiments`.

If you've done everything right, you can `cd` to that directory and run `go run main.go`, and you'll be able to see the string output to stdout.

## That's it for now!

Part Two on this topic is coming soon! If you're interested, I can be reached [@kfcampbell on GitHub](https://github.com/kfcampbell) or [@keeganfcampbell on Twitter](https://twitter.com/keeganfcampbell), because apparently that's a thing I have now.
