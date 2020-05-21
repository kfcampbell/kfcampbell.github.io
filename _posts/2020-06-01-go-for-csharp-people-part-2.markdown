---
layout: post
title:  "Go for C# Developers Part Two: Syntax Attack"
date:   2020-05-13 13:05:07 -0700
categories: go golang csharp
---
# Go for C# Developers Part Two: Syntax, Concurrency, and Garbage Collection

It's also somewhat influenced by [A half-hour to learn Rust](https://fasterthanli.me/blog/2020/a-half-hour-to-learn-rust/).

## In Which We Discuss The Following:

- Rapid-fire syntax rundown
- Concurrency and asynchronous programming in Go vs. C#
- Brief note on garbage collection
- Some helpful links for more information


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




TODO(kfcampbell) still:

## Concurrency and asynchronous programming in Go and C#


## Brief note on garbage collection

- Go is mark and sweep: https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html
    - Go hasn't done something more sophisticated because: https://groups.google.com/forum/m/#!topic/golang-nuts/KJiyv2mV2pU
- C# is mark-compact: https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals

## Miscellaneous/Questions

- discuss error handling in C# (exceptions) vs. Go (returned errors)
- have a section on "norms" like error handling and "everyone stays updated"?