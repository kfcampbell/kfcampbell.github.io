---
layout: post
title: "Go Allocators"
date: 2020-06-09 00:00:00-0700
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

So let's take a peek next at `make`:

Go uses the `make` keyword for three certain types that require Go magic to work with easily: 

If we [read the source code of the allocator](https://github.com/golang/go/blob/7b872b6d955d3e749ea62dbfced68ab5c61eae91/src/builtin/builtin.go#L172), we can see this described:

```go
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. 
```

For the sake of simplicity, we'll focus on slices for the rest of this post.

What happens if we try to use the `new` keyword to instantiate a slice? Well, the compiler gets angry:

```go
package main

import (
	"fmt"
)

func main() {
	ex := new([]int)
	fmt.Printf("%T\n", ex) // would print: *[]int
	ex = append(ex, 42) // invalid argument: ex (variable of type *[]int) is not a slice

	fmt.Printf("%v\n", ex[0]) // invalid operation: cannot index ex (variable of type *[]int)
}
```

Aha! We're getting a little closer: it turns out that `new` returns a _pointer_ to whatever type you want to create.

But that same code works just fine when we create it with `make`:

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

A pointer is only a location to a specific location in memory. It's as close as you can get to the direct bits of your variable.

Slices need a little bit more magic to work correctly. Just how are they suspiciously always ready to accept new items, even though arrays cannot be resized?

If you pop the hood, the structure of a slice looks like this:

<div class="img_row">
    <img class="col three" src="{{ site.baseurl }}/assets/img/slice_header.jpg">
</div>
<div class="col three caption">
    I'm sorry, I really tried to draw a decent row of blocks for that array. I'm not spacially gifted.
</div>

When you insert an item past the end of the slice's underlying array, Go will create a new, larger underlying array for you and copy the contents over. You can see this reflected in the slices `capacity` value:

```go
package main

import "fmt"

func main() {
	ex := make([]int, 0)
	fmt.Printf("length: %v, capacity: %v\n", len(ex), cap(ex)) // length: 0, capacity: 0
	ex = append(ex, 42)
	fmt.Printf("length: %v, capacity: %v\n", len(ex), cap(ex)) // length: 1, capacity: 1
	ex = append(ex, 43)
	fmt.Printf("length: %v, capacity: %v\n", len(ex), cap(ex)) // length: 2, capacity: 2
	ex = append(ex, 44)
	fmt.Printf("length: %v, capacity: %v\n", len(ex), cap(ex)) // length: 3, capacity: 4
}
```

Interestingly, the slice header is made up of two _values_, the length and capacity, and one _pointer_, to the underlying array.

This can lead to weird effects, like this example from [Rob Pike's post on slices](https://blog.golang.org/slices), reproduced here for convenience because I can't figure out how to directly link to the example itself: 

```go
package main

import (
	"fmt"
)

func Extend(slice []int, element int) []int {
    n := len(slice)
    slice = slice[0 : n+1]
    slice[n] = element
    return slice
}

func main() {
    var iBuffer [10]int
    slice := iBuffer[0:0]
    for i := 0; i < 20; i++ {
        slice = Extend(slice, i)
        fmt.Println(slice)
    }
}
/*
[0]
[0 1]
[0 1 2]
[0 1 2 3]
[0 1 2 3 4]
[0 1 2 3 4 5]
[0 1 2 3 4 5 6]
[0 1 2 3 4 5 6 7]
[0 1 2 3 4 5 6 7 8]
[0 1 2 3 4 5 6 7 8 9]
panic: runtime error: slice bounds out of range [:11] with capacity 10

goroutine 1 [running]:
main.Extend(...)
	/Users/kfcampbell/go/src/github.com/kfcampbell/go-experiments/main.go:9
main.main()
	/Users/kfcampbell/go/src/github.com/kfcampbell/go-experiments/main.go:18 +0x100
exit status 2
*/
```

Why is this significant? 

It's important to remember when copying or creating slices that while the array portion is passed by reference, the values for the length and capacity aren't. It may help you avoid an ugly panic in the future [unlike this poor child](https://preview.redd.it/oepz8q6lopy41.png?width=538&auto=webp&s=1e0901f3b884b2b636691f50ecb5fff068b8d2b3).

Is there a moral to this whole story? Not really, no.

Or maybe, there's a couple of small ones, like "you should know Go has two allocators", or "slices are made up of a pointer to an array, and integer values for length and capacity", or "Keegan can't draw to save his life". 

Regardless, I thought it was interesting and wanted to write about it. If you've made it this far, congratulations! I hope you learned something.

P.S. Throughout the entire length of this post, I've been trying to work in the language "Don't hate, allocate" in a clever manner and I've failed to do so. If you can think of a better way to do it, please get at me [@keeganfcampbell](https://twitter.com/keeganfcampbell) on Twitter.