---
layout: post
title: "Go for C# People, Part Two: Syntax, Concurrency, and Garbage Collection"
date: 2020-05-31 00:00:00-0700
description: We learn syntax rapid-fire, then delve into Go's concurrency and garbage collection models.
---

This is heavily influenced by [A half-hour to learn Rust](https://fasterthanli.me/blog/2020/a-half-hour-to-learn-rust/).

## In Which We Discuss The Following:

- Rapid-fire syntax rundown
- Concurrency and asynchronous programming in Go vs. C#
- Brief note on garbage collection
- Some helpful links for more information


## Rapid-fire syntax rundown

Seriously, you should skip this post and go see [A Tour of Go](https://tour.golang.org). If you're lazy and just want to read, or you want to hear my snarky comments, then fasten your seatbelt:

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

If statements are exactly what you'd expect. The Go attitude is "no parentheses, no problems."
```go
x := 0

if x > 0 {
    fmt.Printf("help i'm trapped inside a Go standard library")
} else if x < 0 {
    fmt.Printf("i can't get out of the fmt package")
} else {
    fmt.Printf("printing is my only hope")
}
// printing is my only hope
```

Switch statements are a bit different than C#. Go only allows a single case to be run at once, so you won't see `break` littered throughout the codeblock. You can switch on any variable type and even function results:
```go

x := "x"
switch tmp := fmt.Sprintf("%v", x); tmp {
    case "x":
        fmt.Printf("an x is an %v is an %v", x, x, x)
    default:
        fmt.Printf("this won't happen")
}
// an x is an x is an x

```

The `defer` statement is a construct pretty unique to Go (as far as my limited knowledge goes). Calling a function with the `defer` keyword will defer its execution until after wherever it's in returns. Explaining it is hard, but here's an example that'll make it all clear:
```go
func ex() int {
    x := 42
    defer fmt.Printf("%v", x)
    x = -1
    return x
}

ex()
// 42

```

You'll commonly see this used for closing database connections or other similar resource cleanup. It's a nice way of making sure you don't have to track down every single codepath to make sure certain tasks are performed. I like to think of it as Go looking out for future @kfcampbell.

Go's only sort-of object-oriented. There's no `class` keyword, so upfront one wonders what's going on, but `struct`s will get you pretty much all the way there. You can declare one like so:

```go
type Movie struct {
    ID    int
    Title string
}

func (*m Movie) PrintTitleAndId() {
    fmt.Printf("movie id: %v, title: %v", m.ID, m.Title)
}

mov := &Movie {
    ID:     0,
    Title: "Fantastic Mr. Fox",
}

fmt.Printf("%v", mov.Title)
mov.Print
// Fantastic Mr. Fox
```

I snuck a couple of extra things into this because ~I'm getting impatient of writing this~ it was a teachable moment. First, note that `gofmt` spaces the declarations of structs out so that they form two nice little columns. [That's pretty neat!](https://www.youtube.com/watch?v=Hm3JodBR-vs). 

Second, you'll note that I initialized the movie struct with an `&` character. That's a convention in Go, although not strictly required in this case. It means that you're creating a _pointer_ to a Movie, and initializing that movie with the following content.

Don't be scared! 

Let me say it again. 

DO. NOT. BE. ALARMED.

Points in Go are very normal and it's pretty difficult to hurt yourself with it (although there is one way. I need to put it in an appendix or something. TODO(kfcampbell))



Oh! And I almost forgot: we still need to talk about slices.

TODO(kfcampbell) still:

## Concurrency and asynchronous programming in Go and C#


## Brief note on garbage collection

- Go is mark and sweep: https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html
    - Go hasn't done something more sophisticated because: https://groups.google.com/forum/m/#!topic/golang-nuts/KJiyv2mV2pU
- C# is mark-compact: https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals

## Miscellaneous/Questions

- discuss error handling in C# (exceptions) vs. Go (returned errors)
- have a section on "norms" like error handling and "everyone stays updated"?