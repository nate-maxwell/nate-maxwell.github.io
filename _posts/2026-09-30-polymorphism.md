---
layout: post
title: "[LANG] The Poly Morphs of Polymorphism"
date: 2026-09-30
permalink: /the-poly-morphs-of-polymorphism/
author: Nate Maxwell
categories:
    - "Language"
tags: 
    - "language"
    - "polymorphism"
---

# The Poly Morphs of Polymorphism

_"A monad is a monoid in the category of endofunctors, what's the problem?"_ - James Iry

I accidentally stumbled across the polymorphism model in my language, Silver.
I couldn't think of another language that did the exact behavior I was.
Whenever you think you've invented something new in a field with decades of
research, especially in academia, one of two things happened:

* The rare event that you've invented something new.

### _Or_

* Your idea is widely regarded as a bad move.

Luckily for me, I do not think, therefore I do not am.

While searching around I was surprised at the number of types of polymorphism
across languages. It seems like this is maybe the most whatever-floats-your-boat
topic in programming language design. Although this post is about the many kinds
of polymorphism, there are so many that I've elected to not mention a bunch of
them here for the sake of brevity.

## Parametric Polymorphism

Starting off is Parametric Polymorphism. Perhaps the most "academic" of
polymorphisms. The short answer is that you can think of parametric polymorphism
as generics. You write a function once and it accepts anything.

I'm told Haskell is the canonical example of parametric polymorphism, but I'm
also told that releasing to production in Haskell and writing a white paper are
the same.

In Go, functions and methods that take generics actually box the value at the
call site and ship the boxed value to the function. The value is then unboxed
inside the function body. The boxes are generated at compile time and are how
the compiler maintains strict typing. Go's compiler refers to this as GC Shape
Stenciling.
```go
// SumIntsOrFloats sums the values of map m. It supports both int64 and float64
// as types for map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

Depending on your strictness of the definition of parametric, C++ templates may
or may not count. They achieve the same practical goal as parametric polymorphism
— write one implementation that works for many types:
```cpp
template<typename T>
T identity(T x) {
    return x;
}

identity(5)      // works for int
identity(5.0)    // works for float
identity("hi")   // works for string
```

On the other hand, they aren't truly parametric in the theoretical sense — they
use duck typing at compile time rather than true parametric polymorphism.
The compiler generates a separate concrete implementation for each type the
template is instantiated with. It is essentially code generation.

## Subtyping Polymorphism

Subtyping is probably the most widely recognized form of polymorphism, as it is
almost universally used in the OOP world. Interestingly it doesn't always take
the same form. As long as the passed value belongs to a greater category, it
counts as a subtype. This includes both inheritance and something like interfaces
in Java:
```java
interface Drawable {
    void draw();
}

class Circle implements Drawable { ... }
class Rectangle implements Drawable { ... }
```

I'm almost surprised it isn't called `Subcategory` instead of `Subtyping`.

## Structural and Duck Typing

Java interfaces and Rust traits are often compared to Go's interfaces, but they're
still distinctly different.

```go
type Drawable interface {
    Draw()
}

func Render(d Drawable) {
    d.Draw()
}
```

Subtyping is a behavioral guarantee. A subtype can be substituted for a supertype
anywhere the supertype is expected. Structural typing checks the structural
compatibility at a specific point. Does this object, whatever it is, contain
the expected behaviors?

Structural typing is statically typed and checked at compile time. The dynamic
runtime equivalent is Duck Typing, "how _does_ this thing behave", rather than
"_how_"?

Think Python Protocols
```python
from typing import Protocol

class Flyer(Protocol):
    altitude: int  # Required attribute
    
    def fly(self, destination: str) -> None:
        ...  # Required method signature

class Airplane:
    def __init__(self) -> None:
        self.altitude = 0

    def fly(self, destination: str) -> None:
        print(f"Flying to {destination} at {self.altitude}ft.")

def launch(vehicle: Flyer) -> None:
    vehicle.fly("Paris")
```

## Ad Hoc Polymorphism

The same item has different implementations selected per type. "Ad hoc" meaning
the polymorphism is not systematic - each type gets its own specific implementation
rather than a uniform one.

If parametric is one implementation for all types, then ad hoc is all types get
their own implementation. Overloading...

### C++
```cpp
class Point {
public:
    int x, y;
    Point(int x = 0, int y = 0) : x(x), y(y) {}

    // Overloading the '+' operator
    Point operator+(const Point& other) {
        return Point(this->x + other.x, this->y + other.y);
    }
};
```

### Python
```python
class Point:
    def __init__(self, x: int, y: int) -> None:
        self.x = x
        self.y = y

    # Overloading the '+' operator
    def __add__(self, other: Point) -> "Point":
        if isinstance(other, Point):
            return Point(self.x + other.x, self.y + other.y)
        return NotImplemented
```

## Row Polymorphism

The most interesting of my finds is Row Polymorphism, a type system feature
where a function can accept any record that has at least the required fields,
with any additional fields allowed. The "row" refers to the fields of a record,
and the function is polymorphic over the unknown remaining rows. The remaining
fields are tracked by the type system as a row variable — an explicit placeholder
for whatever else the record might have.

```ocaml
(* A function that accepts any object with a name field *)
let greet (obj : < name : string; .. >) =
  "Hello, " ^ obj#name

(* Two different object types with different fields *)
let person = object
  method name = "Ada"
  method age = 36
end

let animal = object
  method name = "Rex"
  method species = "Dog"
end

(* Both work because both have a name field *)
let () =
  print_endline (greet person);  (* Hello, Ada *)
  print_endline (greet animal)   (* Hello, Rex *)
```
The `..` in `< name : string; .. >` is the row variable - it represents the
unknown remaining fields.

Row polymorphism in its original formulation is about record fields and their
types, not behavior. But in a language where functions are first class values
and can be stored in record fields, the distinction collapses.

## Silver's Destructuring Polymorphism

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
