---
layout: post
title: "Go Allocators"
date: 2020-06-23 00:00:00-0700
description: In which we sit around the campfire and tell scary stories about Go memory allocation.
---

I was driving and listening to [episode 100 of the Go Time podcast](https://changelog.com/gotime/100) the other day, and heard something from language creator Rob Pike that gave me pause: there's actually two allocators in Go, `new` and `make`. 

In writing day-to-day Go, most people don't use the `new` keyword. In fact, I didn't know that it even existed (though it's very common in other languages)! The figuratively first thing I did when I got home was sit down at my computer to test it out. First, let's re-familiarize ourselves with the normal `&` syntax:

```go
package main

import (
	"fmt"
)

type ContrivedExample struct {
	Text string
}

func main() {
	ex := &ContrivedExample{
		Text: "This feels familiar.",
	}
	fmt.Printf(ex.Text) // This feels familiar.
}
```

That's not too spooky. We can switch it up a bit so that we initialize the object without providing a value for `Text`, and then set that later:


```go
package main

import (
	"fmt"
)

type ContrivedExample struct {
	Text string
}

func main() {
	ex := &ContrivedExample{}
	ex.Text = "This feels familiar."
	fmt.Printf(ex.Text) // This feels familiar.
}
```

Now let's take the `new` keyword for a spin:

```go
package main

import (
	"fmt"
)

type ContrivedExample struct {
	Text string
}

func main() {
	ex := new(ContrivedExample)
	ex.Text = "What the hell is really going on?"

	fmt.Printf(ex.Text) // What the hell is really going on?
}
```

That's crazy! It looks weird to me to see that in a working Go program. Also it might be a good way to annoy coworkers in your next code review.

Go uses the `new` keyword (or its suspiciously sugary syntax shortcut, `&`) for _direct_ memory allocation: primitives, structs, any type you define.

So let's take a peek next at `make` next.

Go uses the `make` keyword for three certain types that require Go magic to work with easily: 

If we [read the docs](https://github.com/golang/go/blob/7b872b6d955d3e749ea62dbfced68ab5c61eae91/src/builtin/builtin.go#L172) in the source code of the allocator, we can see this described:

```go
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. 
```

What happens if we try to use the `new` keyword to instantiate a slice, for example? Well, the compiler gets angry:

```go
package main

import (
	"fmt"
)

func main() {
	ex := new([]int)
	fmt.Printf("%T\n", ex) // would print *[]int if this compiled
	ex = append(ex, 42) // invalid argument: ex (variable of type *[]int) is not a slice

	fmt.Printf("%v\n", ex[0]) // invalid operation: cannot index ex (variable of type *[]int)
}
```

Aha! We're getting a little closer: it turns out that `new` returns a _pointer_ to whatever type you want to create.

But that same code works just fine when we try it with `make`:

```go
package main

import (
	"fmt"
)

func main() {
	ex := make([]int, 0)
	fmt.Printf("%T\n", ex) // int[]
	ex = append(ex, 42)

	fmt.Printf("%v\n", ex[0]) // 42
}
```

And like the docs state, `make` returns a slice, not a pointer to it. Why?

A pointer is only a location to a specific location in memory. It's as close as you can get to the direct memory location of your variable.

Slices, maps, and channels need a little bit more magic to work correctly. 

Incidentally, _slicing_ an array returns a slice type as well:

```go
package main

import "fmt"

func main() {
	ex := make([]int, 50, 100)
	fmt.Printf("%T\n", ex) // []int

	why := new([100]int)
	fmt.Printf("%T\n", why) // *[100]int

	zee := new([100]int)[0:50]
	fmt.Printf("%T\n", zee) // []int
}
```

Note that `why`'s type `*[100]int` reflects both a pointer to a specific location and not the object itself, and a given size for the length of the (fixed) array.

Anyways, we were looking at the difference between `make` and `new`, so let's read some source code. [new](https://github.com/golang/go/blob/master/src/runtime/malloc.go#L1192) is implemented like so:

```go
// implementation of new builtin
// compiler (both frontend and SSA backend) knows the signature
// of this function
func newobject(typ *_type) unsafe.Pointer {
	return mallocgc(typ.size, typ, true)
}
```

For our purposes, we're not very concerned with the particulars of `mallocgc()`; it's enough to know that `new` returns an `unsafe.Pointer`.

The `make` source, however, is split up into three pieces. The Go compiler creates calls to either [makeslice()](https://github.com/golang/go/blob/master/src/runtime/slice.go#L83), [makemap()](https://github.com/golang/go/blob/master/src/runtime/map.go#L303), or [makechan()](https://github.com/golang/go/blob/master/src/runtime/chan.go#L71) depending on what developer requires.

Note that due to optimizations, these functions may not be called _exactly_. For example, there's a [makemap_small()](https://github.com/golang/go/blob/master/src/runtime/map.go#L292) function called under certain conditions instead of `makemap`.

However, eagle-eyed readers will notice that `makeslice()`, for example, _also_ returns an `unsafe.Pointer`.

What's going on here? Earlier we said that calls to `make()` returned a type, not a pointer to a type.

Well, `makeslice()` is called by other helper initialization code. This is the actual definition of what we think of as a [slice](https://github.com/golang/go/blob/master/src/runtime/slice.go#L13):

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

This is known as the _slice header_. I wanted to make a little drawing, so here's a brief explanation of each part:

<img class="col three" src="{{ site.baseurl }}assets/img/slice_header.png">
<div class="col three caption">
    I'm sorry, I really tried to draw a decent row of blocks for that array. I'm not spacially gifted.
</div>
<br/>

However, slice/map/channel internals are out of scope for this blog post. If I [feel cute later](https://knowyourmeme.com/memes/feeling-cute-might-delete-later), I might write another blog post on them, idk.

Is there a moral to this whole story? Not really, no.

Or maybe, there's a couple of small ones, like "you should know Go has two allocators", or "slices are made up of a pointer to an array, and integer values for length and capacity", or "Keegan can't draw to save his life". 

Regardless, I thought it was interesting and wanted to write about it. If you've made it this far, congratulations! I hope you learned something.

P.S. Throughout the entire length of this post, I've been trying to work in the language "Don't hate, allocate" in a clever manner and I've failed to do so. If you can think of a better way to do it, please get at me [@keeganfcampbell](https://twitter.com/keeganfcampbell) on Twitter.