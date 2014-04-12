---
title: "Go 101: Methods on Pointers vs. Values"
layout: post
published: false
categories: golang
description: "A quick breakdown of the subtle difference between declaring a method on a pointer vs. value in Go."
---

{{site.go_101_blurb}}

Methods can be declared on both pointers and values. The difference is subtle but important.

	type Person struct {
	     age int
	}

	// Method's receiver is the value, `Person`.
	func (p Person) Age() int {
	     return p.age
	}

	// Method's receiver is a pointer, `*Person`.
	func (p *Person) SetAge(age int) {
	     p.age = age
	}

> In reality, you'd only define getter and setter functions like this if you needed to implement additional logic. In an example like this you'd just make the `Age` field public.

This is how you define getter and setter functions in Go. Notice that we defined the `Age()` function on the value but `SetAge()` on the pointer (i.e. `*Person`). This is important.

**Go always passes by value.** Function parameters are  always passed by copying them as opposed to passing a reference. ([Read more.](http://golang.org/doc/faq#pass_by_value))

> Even pointers are technically passed by value. The memory address is copied, the value it points to is *not* copied.

Here is the *wrong* way to define `SetAge`. Let's see what happens.

	func (p Person) SetAge(age int) {
	     p.age = age
	}

	p := Person{}
	p.SetAge(10)
	fmt.Printf("Age: %v", p.Age()) // Age: 0

[&#9654; Run it.](http://play.golang.org/p/CJZfqBrAIC)

Notice that the output is `0` instead of `10`? This is 'pass by value' in action.

Calling `p.SetAge(10)` passes a copy of `p` to the `SetAge` function. `SetAge` sets the `age` property on the copy of `p` that it received which is discarded after the function returns.

Now let's do it the right way.

	func (p *Person) SetAge(age int) {
	     p.age = age
	}

	p := Person{}
	p.SetAge(10)
	fmt.Printf(“Age: %v”, p.Age()) // Age: 10

[&#9654; Run it.](http://play.golang.org/p/BbIlSUQBCr)

The rule of thumb is this: declare the method on the pointer if it needs to modify the receiver. Otherwise declare the method on the value.

An important caveat is if the method is on a large struct. In this case, it will be more efficient to use the pointer and avoid the copy.

[Read the FAQ](http://golang.org/doc/faq#methods_on_values_or_pointers) "Should I define methods on values or pointers?" for more insight.