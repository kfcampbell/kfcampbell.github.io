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

### Similarities

Both Go and C# are:

- statically-typed
- compiled
- automatically garbage-collected
- support multiple platforms
    - in Go, typically no changes are needed to support a binary e.g. compiled on macOS but running in Linux in production
- include robust standard libraries

### Differences

- Go has no runtime required (like the [.NET CLR](https://docs.microsoft.com/en-us/dotnet/standard/clr))
- Go has no generics
    - This is a consistent paint point for the Go community
    - See [here](https://go.googlesource.com/proposal/+/master/design/go2draft-generics-overview.md) for a proposal to add generics from language designer Russ Cox
- Go returns errors which are then checked by the caller, rather than try/catch blocks and exceptions for error handling
- Go has no concept of `await`ing an action (more on this later)
- Go places a strong emphasis on staying up to date with the language, in constrast to the fragmentation in the .NET and .NET Core ecosystems

## Dependency management

Dependency management used to be a somewhat contentious subject in the Go ecosystem, with a number of third party solutions attempting to address the gap in this language tooling. [v1.11](https://golang.org/doc/go1.11), however, introduced experimental support for Go modules, and [v1.14](https://golang.org/doc/go1.14) (released this February) standardized them as ready for production use. Modules allow reusable packages to be imported using their URL, as in `import github.com/go-sql-driver/mysql`. Similar to Nuget packages (and unlike Node modules), modules are cached at a user level and not at a repository level. Somewhat like a `Nuget.config` file, module dependencies are declared in a `go.mod` file  in the project's root. The syntax for this file is as follows:

```go
module github.com/github/go-sample-service

go 1.14

require (
	github.com/go-sql-driver/mysql v1.5.0
)
```

You'll notice three important sections: the top line is the declaration of the module name (also where it's hosted). The second line is the version of Go the module uses. After that, all the modules the application imports are declared and given a version.

Building and/or running your code will result in a generated file `go.sum` that contains the checksums for all the packages you use. Similar to a `package.json`, this should be checked into source control.

## Brief tooling introduction

- Development in Go is heavily reliant on a few tools I'll provide a brief overview of here. To see all the tools and their documentation, run `go help` and then `go <command> help` at your terminal. For now, let's stick with:

- `go get`
- `go run`


## Brief note on garbage collection

- Go is mark and sweep: https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html
    - Go hasn't done something more sophisticated because: https://groups.google.com/forum/m/#!topic/golang-nuts/KJiyv2mV2pU
- C# is mark-compact: https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals

## Miscellaneous/Questions

- discuss error handling in C# (exceptions) vs. Go (returned errors)
- have a section on "norms" like error handling and "everyone stays updated"?