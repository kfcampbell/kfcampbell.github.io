---
layout: post
title: "Go for C# People, Part One: The Overview"
date: 2020-06-01 00:00:00-0700
description: In which lies a brief introduction to the Go programming language, primarily geared towards C# people.
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

Anyways,


P.S. Throughout the entire length of this post, I've been trying to work in the language "Don't hate, allocate" in a clever manner and I've failed to do so.