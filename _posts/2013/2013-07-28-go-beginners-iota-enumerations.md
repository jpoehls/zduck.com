---
title: "Go Beginners: Iota Enumerations"
layout: post
categories: golang
description: "Enum constants in Go have a super power. As opposed to C# where you only have two options for assigning enum values, Go has a third option. Say hello to `iota` and repeating expressions."
---

Enum constants in Go have a super power. As opposed to C# where you only have two options for enums, either you assign each value yourself or let the compiler increment each value by one, Go has a third option.

**Auto-incremented enum in C#**

<pre data-language="csharp">
enum Example
{
     One = 1,
     Two,
     Three
}
</pre>

Go's third option is that you can make use of repeating expressions. This works because of Go's `iota` enumerator. The best example can by found in [the Effective Go docs](http://golang.org/doc/effective_go.html#constants).

<pre data-language="go">
type ByteSize float64

const (
     _           = iota // ignore first value by assigning to blank identifier
     KB ByteSize = 1 &lt;&lt; (10 * iota)
     MB
     GB
     TB
     PB
     EB
     ZB
     YB
) 
</pre>

`iota` is only valid when assigning constants and with each `const` block it starts out life with a value of zero. Each time it is used it is incremented by one.

In the example above, the first constant is `_ = iota`. `_` is a throwaway variable in Go so we are ignoring the first value of `iota` (zero) and starting with a value of 1 on the next line.

`KB` is assigned an expression that uses `iota`: `1 << (10 * iota`. The subsequent constants don't have any explicit value so Go repeats the last assignment. Thus, repeating expressions. Because `iota` increments each time, we end up with a very useful pattern generator.

Flag enumerations are a great use of this. In C# it is a manual process to assign each flag a value that increments in powers of 2.

<pre data-language="csharp">
[Flags]
enum FlagExample : int
{
     Zero = 0,
     Two = 2,
     Four = 4,
     Eight = 8
}
</pre>

`iota` makes this much simpler in Go.

<pre data-language="go">
const FlagExample (
     Zero = 1 &lt;&lt; iota
     Two
     Four
     Eight
)
</pre>

Of course you don't have to use `iota` at all if you don't want to.

<pre data-language="go">
const Example (
     One = 1
     Two = 2
     Four = 4
)
</pre>

Be sure and [read the docs](http://golang.org/doc/effective_go.html#constants) for even more info. Also, [here's a playground example](http://play.golang.org/p/ASLfPwNc2S) of what we've covered.

**Go** forth and code.