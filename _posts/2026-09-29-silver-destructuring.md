---
layout: post
title: "[SLVR] Silver Object Destructuring"
date: 2026-09-29
permalink: /silver-object-destructuring/
author: Nate Maxwell
categories:
    - "Language"
tags: 
    - "language"
    - "types"
    - "silver"
---

# Silver's Object Destructuring

_"The bigger the interface, the weaker the abstraction"_ - Rob Pike.

I like objects. I like composing them, shipping them around, packing and
unpacking them.

I absolutely cannot stand getting feedback in a PR about using
```python
if isinstance(obj, SomeClass):
    ...
```
or
```go
switch object := object.(type){
    ...
}
```
and how I must have completely mismanaged my inheritance tree. I die a little
inside every time I get this feedback. It works. I do not have to redo my
hierarchy to make the same code do the same thing. Would it be better if the
hierarchy accommodated this? Yes. Would it be better if I didn't have to check
the object type occasionally? Yes. Is it the end of the world when this happens?
No.

I often prefer composition to inheritance, anyway (again, shamelessly plugging Go).
Often times I write functions that take an object, then later stare at them
wondering if they could be used in systems that don't operate on my objects. Do
I want to refactor these functions to take primitives instead? What if the
functions are never used? Do I really want to unpack the objects in all the
callsites just to make this function work on more universal data? Would the
versatility make for a better library?

This eventually led to the destructuring system in my programming language,
Silver.

In a previous post I explain the language's polymorphism using this example:
```
type Position = struct {
    x: int
    y: int
}

let print_position = fn(x: int, y: int) {
    io.print(x, y)
}

let p = Position{10, 20}
print_position(p)
```
```
>> 10, 20
```
Here, silver first offers the struct into the function. After seeing that the
struct does not match the type of the corresponding positional argument, Silver
instead offers up the fields of matching names and types from the struct to as
many remaining function arguments as it can match. i.e. it says "hey, this isn't
supposed to be a Position struct, lets see if this struct has any 'x' or 'y'
integer fields that can be passed instead". 

This solves the two issues described earlier:
* I no longer have to worry about checking object types, if multiple types
contain the same fields then both are usable.
* Functions are reusable across types with no inheritance hierarchy involved. So
callers can use whichever types they want, literals or custom objects.

Neato, but it doesn't stop there. Being the fan of Go that I am, I have borrowed
the struct embedding feature.
```
type Person = struct {
    name: str
}

type Details = struct {
    person:: Person
    age: int
}

let details = Details{ Person{"Ada"}, 44 }
details.name # Ada
```
Here, `Person` is embedded in `Details`, so any of `Person`'s fields can be
accessed through the `Details` namespace.

Under the hood, in Silver struct objects have a `.get()` method. When something
requests a field on a struct it calls the `.get()` method. The `.get()` method
checks immediate fields, and then checks embedded structs for their fields
recursively. When a struct is passed to a function, and it doesn't match the
parameter type, the function calls the`.get()` method on the struct and attempts
to get a field with a matching name to use instead.

Silver refers to this as "destructuring". The opposite of structuring, pulling
the pieces apart to use separately. This also sounds pretty similar to row
polymorphism. So how is it different? Let's take a look at the following:

```
type Point = struct {
    x: int
    y: int
}

let plot = fn(x: int, y: int, offset: int) {
    ...
}

let current_position = Point{ 1, 4 }
plot(current_position, 7)
```

Here, `plot()` takes 3 arguments. When `plot()` is called, a `Point` is passed
in, as well as the integer `7`. Traditionally row polymorphism takes a singular
record and extracts all expected rows by name and type from the passed record.
In silver the passed values can be mixed. Here is a slightly more complex
example:

```
type Record1 = struct {
    a: int
    b: int
}

type Record2 = struct {
    c: int
    d: int
}

let signature = fn(a: int, b: int, c: int, d: int, e: str) { ... }

signature( Record1{1, 2}, Record2{3, 4}, "hello" )
```
In silver, this is completely valid. In a row polymorphic language, each
destructurable record could have to be declared separately. Using some pseudocode
it would look something like this:
```
signature :: { a: int, b: int | r1 } -> { c: int, d: int | r2 } -> str -> ()
```
with an argument, `| r1`, representing unknown rows that would also be received.

Another way to put it is that row polymorphism operates at a _type_ level while
Silver operates at a _value_ level. And working at the value level means that
function signatures are shaped exactly as their operations demand while still
accepting multiple types.

This isn't to be confused with duck typing or structural typing either.
Typing like Python protocols, Rust traits, or Go interfaces. All of them pass
more data than the functions require and are more behaviorally focussed.

---

What's fascinating is that modules behave much the same way inside the
interpreter. Modules in Silver are objects, like in OCaml or Python, and can be
passed to functions. From this, module destructuring can take place:
```
let io = import("core:io")

let announce = fn(print: call) {
    print("ready")
}

announce(io)
```
```
>> ready
```
The `io` standard library module contains the various print functions. In this
example `announce()` takes a field called `print` of type `call`. In silver,
`call` is a type used to represent a function, similar to `typing.Callable` in
Python. `io` is passed to `announce()`, silver sees that `io` isn't a `call`
type, and begins to look through its contents for a symbol called `print`. Once
found the `print` function is passed into `announce()` and the parameter is
fulfilled.

---

If the corresponding positional parameter type to a passed value matches, then
the whole item is passed instead.
```
type Foo = struct {
    value: int
}

let example = fn(f: Foo) { io.println(f.value) }

example( Foo{32} )
```
```
>> 32
```

---

As I've stated before: I love encapsulation, just not inheritance. I love having
little containers of data with behavior attached to them. Naturally, Silver has
struct methods, like Rust or Go. Unlike other languages, Silver takes advantage
of its polymorphism in its method implementation.

```
type Counter = struct {
    value: int
    increment: call(self: Counter, amount: int) int
}

let inc_function = fn(self: Counter, amount: int) int {
    self.value = self.value + amount
    return self.value
}

let counter = Counter{0, inc_function}
counter.increment(3) # equivalent to inc_function(counter, 3)
```

Here, `Counter` is a struct with an `increment` field whose type must be a
function with the signature `(self: Counter, amount: int) int` - one that
takes a `Counter` and integer, and returns an integer.

Later, a function named `inc_function()` is declared, matching that signature.
Then a counter is created with `inc_function()` given to it. From here
the function can be called by the `Counter` namespace as `counter.increment(3)`.
Silver first looks to see if the host struct can be passed as the first argument.
If so, the host struct is passed through and destructuring begins. Then the
integer `3` is given and the signature is fulfilled.

Host struct destructuring can be ignored and a method can be pure, if desired.

```
type Dog = struct {
    name: str
    shout: call(str)
}

let shout = fn(word: str) { io.print(word) }

let fido = Dog("Fido", shout)
fido.shout("bark!")
```
```
>> bark!
```

---

There are even more features in Silver that take advantage of this destructuring
system, but I think they deserve their own posts. Hopefully this conveys the
novelty of Silver's type system, if not its flexibility.
