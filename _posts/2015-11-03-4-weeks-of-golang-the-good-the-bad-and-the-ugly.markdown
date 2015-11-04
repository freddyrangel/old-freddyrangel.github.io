---
layout: post
title: "4 Weeks of Golang: The Good, the Bad, and the Ugly"
author: freddyrangel
excerpt: "Nothing is a clear win -- there are always tradeoffs"
modified:
categories: 
tags: []
comments: true
image:
  feature:
date: 2015-11-03T15:02:29-08:00
---
HelloSign's API has SDKs in a number of programming languages, but there are a few more we would like to officially support -- one of those languages is Go. I've worked in a number of programming languages over the years, but most of those have been high level languages. I wanted to work on a slightly lower level programming language that I felt I could pick up pretty quickly. I have a number of friends who work in Go at their day jobs and have wonderful things to say about it. Therefore I decided to give Go a try by writing a wrapper for the HelloSign API in Go.

So far I've spent 4 weeks in Go, quickly becoming my new favorite programming language. That said, nothing is ever a clear win -- there are always tradeoffs. The job of the developer is know when those tradeoffs make sense and when they don't. With that in mind, I want to run through what I think is really great about Go and why you should take a serious look at it, what isn't as great about Go, and what I just don't like personally for aesthetic/style reasons.

## What is Go?

Go, also known as golang, is an open source systems language developed at Google and used in production by companies such as Dropbox, SendGrid, and Google itself. Go was originally intended to be a descendent of C++ for the 21st Century. Go can be used for anything C++ would be used for but that could benefit from fast compilation, easier syntax, and an efficient garbage collector. Go is now heavily used as a general-purpose programming language that's a pleasure to use and maintain.

It's compiled and statically typed with some dynamic typing capabilities. Go is very C-like with a lot of features found in higher-level languages, such as Unicode strings, flexible data structures, high-level concurrency support through goroutines and channels, and a sophisticated garbage collector, but also keeps some of the best lower level features such as pointers and references. It also has a very extensive standard library, allowing you to build large projects with very few outside dependencies. 

## What makes Go different?

Although Go is quite a small language, it is a very rich and expressive language (as measured in syntactic constructs, concepts, and idioms), so there is a surprising amount to learn. Go is an object-oriented language but it has design elements that distinguish itself from the crowd.

## The Good:

### No Classes. No Type Inheritance

Go does not have any classes to speak of, but you do have structs, functions, and interfaces. You can declare a struct with certain attributes, and later attach functions to that struct, creating a method. 

```
type Pet struct {
  Name string
}

function (pet *Pet) Walk() {
  // Do things
}
```

Go is similar to JavaScript in that it also lacks classes, but unlike JavaScript, Go has no inheritance to speak of. What you do have is a way to emulate through "anonymous fields". It essentially replaces inheritance with delegation. 

Play with the code here: [http://play.golang.org/p/m-7Mkp8UfZ](http://play.golang.org/p/m-7Mkp8UfZ)

```
package main
import "fmt"

type Car struct {
  wheelCount int
}

type Honda struct {
  Car
}

func main() {
  car := Car{wheelCount: 4}
  honda := Honda{Car: car}
  fmt.Println(honda.wheelCount)
}
```

### Primitives for Concurrency

High level concurrency is built into Go via _goroutines_ and _channels_. It provides the best support for concurrency outside of Erlang, but without the drawback of learning a confusing programming paradigm and awkward syntax. There are very few languages out there that come close to competing with Go in this area.

Play with code here: [http://play.golang.org/p/nCqaVzXPQF](http://play.golang.org/p/nCqaVzXPQF)

```
package main

import "fmt"

func ComputeAValue(c chan<- float64) {
   // Do the computation.
   c <- 2.3423 / 534.23423
}

func UseAGoroutine() float64 {
  channel := make(chan float64)
  go ComputeAValue(channel)
  // do something else for a while
  return <- channel
}

func main() {
  value := UseAGoroutine()
  fmt.Printf("Result was: %v", value)
}
```

### Duck Typing

If you’re not familiar with duck typing (used extensively in languages like Ruby), it basically means a type is defined not by what it is, but what it can do. Duck typing is less concerned about the exact type of an object and more about the methods it should respond to. If it looks like a duck and quacks like a duck, treat it as a duck. The difference between Ruby's duck typing and Go's is that you have to explicitly define a type that defines the duck. In Ruby, duck types are implied and can blow up at runtime if you're not careful. Not a problem in Go since you define a duck with an “interface” which is then checked at compile time. A struct passed to a function expecting a given interfaced signature must implement the same method signatures as the interface declaration itself.

Play with the code here: [http://play.golang.org/p/8vSbEM-MLH](http://play.golang.org/p/8vSbEM-MLH)

```
package main

import "fmt"

type Cat struct {
  Name string
}

type Dog struct {
  Name string
}

func (cat Cat) SaySomething() string {
  return "meow"
}

func (dog Dog) SaySomething() string {
  return "woof"
}

type Pet interface {
  SaySomething() string
}

func AllPetsSaySomething(pets ...Pet) {
  for _, pet := range pets {
    fmt.Printf(pet.SaySomething())
  }
}

func main() {
  cat := Cat{Name: "Marco"}
  dog := Dog{Name: "Polo"}
  AllPetsSaySomething(cat, dog)
}
```

### Variable Length Arrays

In addition to regular arrays with a set length, Go has variable length arrays called slices. They provide a convenient way of working with variable lengths of data. 

```
// Regular array
a := [2]string{"no", "cats"}

// Another regular array
b := [...]string{"no", "cats"}

// A slice
c :=  []string{"no", "cats"}
```

Arrays are not seen too much in Go code -- slices are much more popular.

### The "go" Tool

Go ships with a command line tool simply called "go" which automates the downloading, building, installation, and testing of Go packages and commands. There are very few other languages that provide this level of tooling out of the box. The explicit goal of the "go" tool was provide a way to compile Go programs with just the information in the source code itself, and not through a complicated configuration in a makefile. All you need is to include the necessary import statements in your code and use the "go" command.


### Aversion to Dependencies

Go has a totally different philosophy when it comes to package management. The Go community is much more careful about managing third-party dependencies than is many other popular languages. Frameworks and large libraries tend to be frowned upon. Unlike the Ruby or JavaScript communities that rely heavily on frameworks, Go programs are mostly written with just the standard library. Part of this is because the Go standard library is really massive and useful. The other is Go as a language has thought very long and hard about managing and packaging dependencies, which leads to an aversion to creating dependencies on third-party libraries.

## The Bad:

### List Comprehension in Go Kinda Sucks

Go has improved on C's "for" loops, making them intuitive and easy to use. But for being a modern programming language, it's disappointing there aren't basic functional methods like map/filter/reduce etc. You can implement these methods, but its verbose and requires more boiler plate. It was probably an intentional choice to leave these methods out of the language but they're very common in other languages and incredibly powerful. 

### Slices Can Be Dangerous If You Don't Pay Attention To What They Actually Doing

Slices is technically a "slice" of an array which can cause trouble if you're not careful. Here’s an example.

Play with the code here: [https://play.golang.org/p/8IE5zbJkcK](https://play.golang.org/p/8IE5zbJkcK)

```
package main

import "fmt"

func main() {
  array := [10]string{"a", "b", "c", "d", "e", "f"}
  slice1 := array[:3]
  slice2 := array[3:]
  fmt.Printf("so far so good: slice1 %v slice2 %v\n", slice1, slice2)
  slice1 = append(slice1, "BANG")
  fmt.Printf("append to slice1: %v\n", slice1)
  fmt.Printf("slice2 is now corrupt: %v\n", slice2)
}
```

What's leading to the unexpected behavior is that slice1 and slice2 are actually indexing an array on the stack, not making a copy of that array. Both slices are indexing the same array, so if you make changes to that array you change the slice as well. 

## The Ugly

### Error Handling

Golang functions can return multiple values which is useful in some ways but painful in other ways. A typical Go function will return two or more values, the last one is usually an error value. While this does force you to handle errors, it's also incredibly tedious and makes it very hard to create concise code. This isn't a deal breaker for using Go and in fact is a great feature of the language. It's Go's way of forcing you to eat your vegetables. That said, it would be nice to have explicit error handling that isn't so verbose, repetitive, and noisy.

```
f, err := os.Open("somefile.txt")
if err != nil {
  log.Fatal(err)
}
```

Now imagine having this 10 times in one file and you get the idea of how cluttered it can start to look.

### Testing In Go Is Kinda Painful In Some Ways But Not Others

In a language like Ruby where you can change any class at runtime, mocking is trivial. But in a statically compiled language like Go, you have to rely on dependency injection to test things in isolation or prevent real HTTP calls. Sometimes though dependency injection is just not practical in certain cases, making testing incredibly difficult in those cases. On the other hand, the Go compiler does a lot of type checking that would otherwise have to be done by hand with unit tests. You still have to write unit tests but not as extensively as a dynamically typed language.

## How To Learn Golang

I recommend reading "An Introduction to Programming in Go". It's an easy read that touches on the most important aspects of the language. It's short enough to go though in a few days and be up and running in golang. After that I recommend "Programming in Go" which is more in depth and requires a better understanding of the language.

## Conclusion

All in all, I’m having a great time writing programs in Go. As far as programming languages go, it’s a huge step in the right direction. I certainly consider it one of my favorite programming languages thus far.

