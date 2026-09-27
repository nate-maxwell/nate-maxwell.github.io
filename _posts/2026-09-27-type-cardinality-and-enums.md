---
layout: post
title: "[LANG] Type Cardinality and Enums"
date: 2026-09-27
permalink: /type-cardinality-and-enums/
author: Nate Maxwell
categories:
    - "Language"
tags: 
    - "language"
    - "types"
---

# Type Cardinality and Enums

_"Simple is better than complex. Complex is better than complicated."_ - The
Zen of Python

In my last post I wrote about how I started working on my own programming
language [Silver](https://github.com/nate-maxwell/silver-lang).
I briefly explained how I started understanding where to take the language,
beyond the most generic of features, once I stumbled across argument destructuring
as a form of polymorphism. From there I began researching type systems. My
favorite part of learning anything is getting to know the lingo. Even if you
aren't super knowledgeable, its sort of feels like you're in the club. Like
you've mastered the shibboleth and get to hang with the gang.

I also explained in my last post how I use this blog to formalize my understanding
of things I've learned. So today I am talking about type cardinality and enums
(as the title says).

## What is "Cardinality"?

Cardinality is the number of values a type can have.
Things like `null`, `nil`, `void`, or `None` have a cardinality of 0.
A bool has a cardinality of 2: `True` and `False`.
An unsigned 8-bit integer has a cardinality of 256.
Pretty straightforward so far.

## Product Types

Now let's imagine a struct of two bools:
```
type Example = struct {
    first: bool
    second: bool
}
```
Here, the `Example` struct could be represented as
* `Example{ True, False }`
* `Example{ False, True }`
* `Example{ True, True }`
* `Example{ False, False }`

Which gives us a cardinality of 4.

We call these "Product Types". Types whose cardinality is calculated by
multiplying the cardinality of all attributes that make up the type.
Structs are the most common product type.

## Sum Types

Sum types, on the other hand, are a data type that holds a value that can belong
to exactly one of several distinct categories. Unions are a classic example:
```python
numerical = Union[float, int]
json_types = Union[str, numerical, bool, list, map, None]
```
The cardinality of a sum type is the number of items that make up the type.
Remember, sum types hold a value that belong to a distinct category, which is
how they differ from something like a `uint8`.

## Algebraic Data Types

Algebraic Data Types (ADTs) are a combination of sum and product types.
Think enums.
Types that are reasoned about by their "variants". Each variant has a name.

```
type Shape struct {
    color: Red | Green | Blue
    geometry: Circle | Rectangle | Triangle
}
```

ADTs can be:
* Product types of sum types | Like typed enums
* Sum types of product types | A union of types or objects
* Product types of product types | A struct of structs
* Sum types of sum types

## Enums In The Wild

I was looking to add enums to Silver. Having a background primarily in Python
and Go I started looking around at how other languages implement enums and much
to my surprise, nobody can agree on how enums should work.

In C/C++ they're basically just mappings of symbols to integers
```cpp
enum TrafficLight {
    GREEN,  // Defaults to 0
    YELLOW, // Defaults to 1
    RED     // Defaults to 2
};
```

This is somewhat helpful as you can bit shift them for use with bitwise OR and
bitwise AND operatations:
```c
enum Permission {
    READ    = 1 << 0,  // 0001 = 1
    WRITE   = 1 << 1,  // 0010 = 2
    EXECUTE = 1 << 2,  // 0100 = 4
    DELETE  = 1 << 3,  // 1000 = 8
}

int user_perms = READ | WRITE;  // 0011 = 3

if (user_perms & READ) { ... }    // true
if (user_perms & EXECUTE) { ... } // false
```

---

In TypeScript they are mappings of symbols to values
```typescript
// numeric enum
enum Color {
    Red,    // 0
    Green,  // 1
    Blue    // 2
}

// string enum
enum Color {
    Red = "RED",
    Green = "GREEN",
    Blue = "BLUE"
}

// heterogeneous (mixed, generally avoided)
enum Mixed {
    No = 0,
    Yes = "YES"
}
```
Interestingly, in TypeScript you can index into an array via an integer enum
```typescript
const colors = ["Red", "Green", "Blue"]
colors[Color.Red]  // "Red"
```

---

Go doesn't have true enums, but has two popular substitutes. The first of which
is the classic iota usage in a list of constants:
```go
const (
    Red = iota   // 0
    Green        // 1
    Blue         // 2
)
```

The second takes it a bit further and makes a sibling type to use with a map,
but still using the iota constant:
```go
type Color int

const (
    Red Color = iota
    Green
    Blue
)

var colorName = map[Color]string{
    Red:   "Red",
    Green: "Green",
    Blue:  "Blue",
}

func (c Color) String() string {
    return colorName[c]
}
```

Although Go does not have an express enum type, the `iota` still lets us bit
shift the values

---

In Java, enums can have methods and are actually full classes under the hood,
but declared with an `enum` keyword:
```java
// basic enum
enum Color {
    RED, GREEN, BLUE
}

// enum with fields and methods
enum Color {
    RED("#FF0000"),
    GREEN("#00FF00"),
    BLUE("#0000FF");

    private final String hex;

    Color(String hex) {
        this.hex = hex;
    }

    public String getHex() {
        return hex;
    }
}

Color.RED.getHex()  // "#FF0000"
```

---

Rust has what is generally agreed to be the most capable enum types, full
algebraic data types where each variant can carry different data:
```rust
// simple enum
enum Color {
    Red,
    Green,
    Blue,
}

// enum with associated data per variant
enum Shape {
    Circle(f64),                    // carries a radius
    Rectangle(f64, f64),            // carries width and height
    Triangle(f64, f64, f64),        // carries three sides
}

// enum with named fields per variant
enum Message {
    Move { x: i32, y: i32 },        // struct-like variant
    Write(String),                  // tuple-like variant
    ChangeColor(u8, u8, u8),        // tuple-like variant
    Quit,                           // unit variant, no data
}
```
Which are somewhat similar to the enum class in Python, which are just Python
classes, and therefore completely dynamic in shape.

## Generalized Algebraic Types

Topping the complexity curve we have Generalized Algebraic Data Types (GADTs),
an extension of ADTs to make them even more expressive. In many cases, these are
ADTs whose variant are subjected to some constraint on their validity.

Here is an ADT in OCaml, which is essentially an enum.

```ocaml
type _ expr =
  | Lit  : int -> int expr
  | Bool : bool -> bool expr
  | Add  : int expr * int expr -> int expr
  | If   : bool expr * 'a expr * 'a expr -> 'a expr
```

Each expression to the right of the `:` can differ per variant, which makes it
a _generalized_ ADT.

This is partly why most languages begin their development in OCaml. OCaml has
many features that map really, really well to lambda calculus (the math primarily
used to describe language design), in addition to having a plethora of ways to
express data.

## Enums in Silver

Perhaps anticlimactically, Silver's enums are very simple:
```
type Color = enum {
    Red,
    Green,
    Blue
}
```

They do not map to value literals, like integers or strings, they do not hold
constraints or per variant expressions, and they do not hold methods like
classes.

They _can_ be used in a switch and if statements:
```
let print_color = fn(color: Color) {
    switch color {
    case Red:
        io.println("Red")
    case Green:
        io.println("Green")
    case Blue:
        io.println("Blue")
    default:
        io.println("Beyond the rainbow")
    }
}


if foo == Color.Red { ... }
```

Personally, I've never needed enums to be anything more than a modal sentinel
value.
```
type PrivilegeLevel = enum {
    Vendor,
    Production,
    Developer,
    Lead,
    Admin
}
```
Its always seemed strange to me that anyone would want to reference the value
stored in the field. Isn't that just a class or a map? Sure it eliminates bugs
with map lookups, and it means there's only one kind of value stored in a variable
at a time, instead of an entire class with all its fields.

I've never needed any more complexity when defining a function or object than a
simple group of values in which a variable can only be one of at a time.

I'm not making a language to make other languages. I'm not making a language for
data science or mapping behavior models to programming concepts. Currently, I'm
just trying to make a language that lets me program the way I want. . .
```
type User = struct {
    name: str
    authenticated: bool
    privilege: PrivilegeLevel
}
```

So. . . that's it. 
