---
title: "Go 101: String or Byte Slice"
layout: post
published: false
categories: golang
description: "Go has two types for representing text. Don't be confused. Let's walk through the differences and when to use what."
---

{{site.go_101_blurb}}

One of the first things you'll notice in Go is that two different types are commonly used for representing text. `string` and `[]byte`. A quick example is the [regexp package](http://golang.org/pkg/regexp/#Regexp.FindAll) which has functions for both `string` and `[]byte`.

**What is the difference?**

`string` is immutable and `[]byte` it mutable. Both can contain arbitrary bytes.

The name "string" implies unicode text but this is not enforced. Operating on `string` is like operating on `[]byte`. You are working with bytes not characters.

They are nearly identical and differ only in mutability. The [`strings`](http://golang.org/pkg/strings/) and [`bytes`](http://golang.org/pkg/bytes/) packages are nearly identical apart from the type that they use.


> **Q:** If strings are just arbitrary bytes, then how do you work with characters?
> 
> **A:** What you are thinking of as a character, Go calls a [rune](http://golang.org/ref/spec#Rune_literals). One way to iterate the characters in a string is to use the `for...range` loop. Range will parse the string as UTF-8 and iterate the runes. Read [the `for` loop](http://golang.org/doc/effective_go.html#for) section of Effective Go for more information.

**When to use `string`?**

Ask not when to use `string` but rather, when to use `[]byte`. Always start with `string` and switch to `[]byte` when justified.

**When to use `[]byte`?**

Use `[]byte` when you need to make many changes to a string. Since `string` is immutable, any change will allocate a new `string`. You can get better performance by using `[]byte` and avoiding the allocations.

> C# perspective: `[]byte` is to [`System.StringBuilder`][stringbuilder] as `string` is to [`System.String`][systemstring] when it comes to performance.

Even if your code isn't directly manipulating the string, you may want to use `[]byte` if you are using packages which require it so you can avoid the conversion.

Converting to and from `[]byte` is easy. Just remember that each conversion is a copy.

	s := "some string"
	b := []byte(s) // convert string -> []byte
	s2 := string(b) // convert []byte -> string

**More about strings**

The Go blog has posted in detail about [strings, bytes, runes, and characters in Go](http://blog.golang.org/strings). You should definitely read that post to fully understand the topic.

[stringbuilder]: http://msdn.microsoft.com/en-us/library/system.text.stringbuilder(v=vs.110).aspx
[systemstring]: http://msdn.microsoft.com/en-us/library/system.string(v=vs.110).aspx