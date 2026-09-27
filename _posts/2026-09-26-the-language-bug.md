---
layout: post
title: "[LANG] The Language Bug"
date: 2026-09-26
permalink: /language-bug/
author: Nate Maxwell
categories:
    - "Silver"
tags: 
    - "language"
    - "silver"
---

# The Language Bug

_"A language that doesn't affect the way you think about programming is not worth
knowing"_ - Alan Perlis.

It's been a few months. Typically, I start these blog posts to help me put my
thoughts into explainable words, rather than tell a narrative-like explanation
of some finding. The result is that many of my blog posts read more like project
README files instead of actual blogs.

As of the last few months I have been bitten by the language bug.
I am in the middle of researching language design and will likely dedicate the
next few posts to my findings. This should double as a narrative detailing my
journey into learning language development, as well as help me formalize my
understandings.

I have dabbled in language design before, but never enough to get anything
 of my own design working.
I was inspired to start language development again after watching a
[Logan Smith](https://youtu.be/ebqKYLKjL6U?si=feEhpK_Y4MgmLmtT) video talking
about the Verse scripting langauge coming in Unreal Engine 6. Previous to this
I had never seen nor heard of an effect system in a programming langauge. I was
instantly captivated by this language concept. A completely new way of thinking
about programming. I started to write down some ideas. I _love_ making diagrams
and making notes with arrows on [Excalidraw](https://app.excalidraw.com/). I
began thinking about the kinds of things I do in programming: personal practices,
the entertainment industry, tasks and problems I've had to solve. Nothing I came
up with really stood out to me as anything special. Still riding this inspiration
I started to write the basics of an interpreter over a weekend. _Very very_
small in scope. I figured if I just started writing, maybe something would come
to me as I started to see the pieces fit together. Unfortunately, this didn't
actually happen. By the end of the weekend I had:

* dynamic variables
* functions
* if/else control-flow
* ints, floats, bools and strings
* maps and arrays
* basic repl
* arithmetic operators
* few comparison operators
* built-in print function

But at this point I still had no idea what I wanted to make. Languages are tools,
and tools are designed to solve specific problems. C was made as an abstraction
for assembly, Java was made to reduce memory and pointer bugs, python was made
to simplify reading and writing code, and on and on.

What was I trying to solve?

## The Problem of Problems

I started to think about the kinds of problems I was facing in my job, my personal
projects. Things I noticed in my coworker's work, things I noticed in young
code-bases, mature code-bases. Anything I had ever worked on.

There are many criticisms of C++. One of which is any time someone makes a
criticism of C++, C++ developers always come back with how you should just be a
better developer and the language shouldn't hold your hand the entire way.
Almost every problem I thought about I could fairly easily tuck behind the ol'
"Skill Issue" label.

One of the things I love doing most in software development when I'm faced with
a particularly tricky problem is to look at what I'm currently doing, and see if
I can do the opposite instead. Sounds strange, right? A lot of the time it's
something like push instead of pull. In this case it was to stop looking for
things to add to solve some problem and instead to simply remove problems I had
with programming languages.

What problems did I have with languages?

After much reflection I realized I really dislike chained constructors, which was
interesting to realize because I _love_ encapsulation. I don't necessarily dislike
constructors.

I hate when I have a class that I inherit from, but I need to change
some value. Unfortunately that value is initialized on the parent constructor,
and the parent constructor uses that value to initialize the class's state.
Changing the value before calling the parent's constructor means it gets overridden
by the parent constructor, and setting it after means the parent has already
initialized its state using the wrong value. Invariably I always have to rewrite
the parent's constructor to make the value parameterized, but depending on the
language I can't always set a default value, and therefore affect all the places
that instantiate this object, so I end up rewriting the parent's constructor on
my derived class instead.

I love having a little container of data that I can send around with a tiny little
library attached to it that affects the contained data, or produces some
behavior using that data. Structs and struct methods are one of the design
decisions I really like about Go.

Switching from classes to structs means I need some way to do polymorphism. Go
solves this through interfaces, or structural typing, but I didn't want to do that.
I wanted to make an interpreted language and structural typing works best in
compiled languages, in my opinion.

That's when I had an idea for my first original feature. Original to me, anyway.
I hadn't seen it in other languages and that usually means one of two things:
* Its truly original
* Everyone has decided not to do this for a good reason

Either way, I thought it was interesting and made for a different kind of thinking
when programming, so I went with it. The very beginnings of a language identity
began to take shape.

Enter [Silver](https://github.com/nate-maxwell/silver-lang).

## Silver

Silver is the programming language I have been working on for several weeks now.
It is early days, and thus changes quite drastically as I continue.

I won't cover many language features here, as I will likely be exploring many
language concept in following posts, and describing my take on them and how they
are implemented in Silver.

Starting with today's feature - Silver has some interesting type features. Many
features of the language steer the user towards using structs. One of which is
**Argument Destructuring**.

First, the usual defining of terms. _Arguments_ are what get passed into a
function, while _Parameters_ are the accepted variables in the function signature.

```
let x: int = 32    # variable

# function with a parameter called "param"
let my_function = fn(param: int) {
    ...
}

my_function(x)    # x is an argument passed to my_function
```

In silver, when you pass a struct to a function, if the struct does not match
the type of the corresponding positional parameter, the language searches for
fields on the struct with matching names and types as the remaining parameters
of the function.

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
>> Position{x: 10, Y: 20}
```

Here, `print_postion` takes an integer named "x", and an integer named "y". We
instead pass a `Position` struct in, and since it isn't an integer named "x",
Silver offers the struct's fields as the remaining arguments.

I find this incredibly interesting because it's not a way I've thought about
polymorphism before. Typically, when you think of polymorphism, you think of a
single parameter of a function that can accept multiple types. Destructuring is
more like function overloading but without having to rewrite the function.

In C++ this would be represented as
```cpp
#include <iostream>

// overload for explicit arguments
void print_position(int x, int y) {
    std::cout << x << " " << y << std::endl;
}

// overload for Position struct
void print_position(Position p) {
    std::cout << p.x << " " << p.y << std::endl;
}
```
With a single function signature, we have preemptively written all the possible
overloads we could ever want, so long as the objects contain fields of the same
name and type to offer up. This is super fascinating for library authoring. You
can write the behavior you want, and consumers can define their preferred objects
while still using library functions with minimal conversion.

There are many more features that make use of, or extend the destructuring
system, but I won't go over them here. I'll slowly explain them over more posts
as I write about my findings on other topics.

Finally, because I recently learned about mermaid (how have I gone so long with
knowing?), here is a mermaid diagram of Silver's current architecture.

## Language Architecture

<img src="https://i.imgur.com/HIqqDvO.png">
