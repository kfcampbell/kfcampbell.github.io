---
layout: post
title: "Go Allocators"
date: 2020-06-01 00:00:00-0700
description: Our fearless heroes talk about memory allocation in Go.
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

Now let's get a little wild with Mr. Pike's `new` keyword:

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

So...what's the story here? What's going on under the hood?

Well, let's listen to Mr. Pike a bit more:

```markdown
Both maps and slices have the property, which is not true of anything in C, at least at the base level, which is that the memory representation is somewhat hidden from the user. They come with a more complex structure to hold the length of the array, or the hash buckets for the map, or whatever. And in C you never have anything like that at the basic level language… So that was a challenge. It turned out to be a challenge later, because in order to make slices and maps work properly, they have to be passed as the address of that in a descriptor block, and we struggled with how to best hide those pointers from the user.

For a while they were explicit, but that got kind of uncomfortable, so eventually we just broke down and made them completely hidden. But to do that, we kind of had to change the way memory allocation worked a bit, which is why there’s two allocators - new and make. And I was never happy with that; I don’t think anybody was really happy with how it all worked out…
```

People ("they", in a really broad hand-wavy sense), frequently compare Go to C...I just checked, and it turns out [I'm one of those people now](https://kfcampbell.com/blog/2020/go-for-csharp-people-part-one/). Well, if you can't beat them, then join them.

Anyways, we've stumbled upon a pretty large difference in between the two languages here: 

Go uses the `new` keyword (or its suspiciously sugary syntax shortcut, `&`) for _direct_ memory allocation: primitives, structs, any type you define.

Go uses the `make` keyword for certain types that require Go magic to work with easily. 

Maps and slices are the two easiest representations of this, although the same holds true of channels. 

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

And `make` returns a slice, not a pointer to it. Why?

A pointer is only a location to a specific location in memory. It's as close as you can get to the direct bits of your variable.

What's going on with the `make` command, then? What's Rob Pike crafting back there, hiding behind the scenes and working his magic with your variable allocations?

Welllllllll, slices need a little bit of magic to work correctly. They're suspiciously always ready to accept new items, even though arrays cannot be resized.

As so many programmers before them, Go designers rely on an extra layer of abstraction to run slices smoothly.

I took a slice out back and popped the hood, and it turns out the structure of a slice is like so:

TODO(kfcampbell): make a pretty diagram and put it here

If we [read the source code of the allocator](https://github.com/golang/go/blob/7b872b6d955d3e749ea62dbfced68ab5c61eae91/src/builtin/builtin.go#L172), this is described a bit further:

```go
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. 
```

Is there a moral to this story? Not really, no. I just thought it was interesting and decided to write about it. If you've made it this far, congratulations! 


P.S. Throughout the entire length of this post, I've been trying to work in the language "Don't hate, allocate" in a clever manner and I've failed to do so. If you can think of a better way to do it, please get [@keeganfcampbell](https://twitter.com/keeganfcampbell) on Twitter.