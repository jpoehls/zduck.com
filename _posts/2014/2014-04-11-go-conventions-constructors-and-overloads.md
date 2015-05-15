---
title: "Go 101: Constructors and Overloads"
redirect_to: http://joshua.poehls.me/2014/04/go-conventions-constructors-and-overloads
layout: redirect_post
categories: golang
description: "Mini-post showing how to implement constructors
and function overloads in Go."
---

{{site.go_101_blurb}}

Go doesn't have constructors in the traditional sense. The convention is to make the zero value useful whenever possible.

	type Person struct {
	     Age int
	}

	// These are equivalent.
	// `p1` and `p2` are initialized to the zero value of Person.
	// Neither of these are nil.
	var p1 Person // type Person
	p2 := Person{} // type Person

	// You could also use `new` to allocate which returns a pointer
	p3 := new(Person) // type *Person

> It is most common to use the struct initializer. e.g. `p := Person{}` or `p := &Person{}` if you need the pointer.

Sometimes you want special initialization logic. If your type is named `Person` then the convention would be create a function named `NewPerson` that returns a pointer to an initialized `Person` type.

	func NewPerson(int age) *Person {
	     p := Person{age}
	     return &p
	}

	myPerson := NewPerson(10) // type *Person

Multiple constructors can be implemented by having multiple initializer functions. Go doesn't support function overloads so you will need to name your functions intelligently.

	import "time"

	func NewPersonAge(int age) *Person {
	     p := Person{age}
	     return &p
	}

	func NewPersonBirthYear(int birthYear) *Person {
	     p := Person{time.Now().Year() - birthYear}
	     return &p
	}

[Read more](http://golang.org/doc/effective_go.html#composite_literals) in Effective Go.

**Update:** Thanks to Joe Shaw for the comments! I've updated the article with his suggestions.