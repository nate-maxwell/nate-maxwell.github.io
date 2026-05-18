---
layout: post
title: "[PY] Dataclass Path Templating"
date: 2026-02-15
permalink: /dc-path-templating/
author: Nate Maxwell
categories:
    - "Python"
tags: 
    - "python"
    - "path"
    - "templating"
    - "dataclass"
    - "dataclasses"
    - "dictionaries"
    - "generics"
    - "intellisence"
---

# Dataclass Path Templating and Generics

I abhor dictionaries. I _haaate_ them. Sometimes they are the correct type or data
structure to use. However, most of the time I interact with one I find myself
saying "Couldn't this have been a class?" Typically, I'm asking this question because
the dictionary does not vary in shape. Dynamic dictionaries are where the type
shines the most. Most of the time dictionaries are used because they're a
primitive that all libraries/systems share.

I love intellisence. I _looove_ it. I want syntax highlighting and auto-suggestion
whenever I'm coding. I find it productive. I disdain AI auto-completion, but I
adore intellisense.

Every time I interact with a dictionary I have to ask if its `"filepath"`,
`"file_path"`, `"file"`, `"fp"`, etc.

I could use a typed-dictionary, but that's a lot of extra work.

I could use a named tuple, but named tuple methods are annoying to path when
unit testing.

I _looooooooooooooooove_ dataclasses. They're so easy, so convenient. You can
freeze them like tuples, you can instantiate copies, you can write comparators
for them. They're amazing. Most importantly: they have named fields just like
any other class but are bite-sized like a dict. That being said I don't actually
like pydantic models very much. If you don't use validation methods, why aren't
you just using dataclasses? If you are using validation methods, how have you not
screamed upwards at god for your lost performance?

Why not just use dataclasses? The problem is that normally you can't determine
which dataclass type you are currently working with. I'm not a fan of team
`Fat Struct`, where you use a struct with every possible field your system uses.
I am team `Struct++`: I want a small object that I can do a _little_ bit more with.
Enter generics...

I'm doing this in python 3.10, and I know generics have had some improvements
in the later version. I've also been writing code in PyCharm, so the effectiveness
of generics may vary from editor to editor.

First, we start with a `TypeVar`

```python
from typing import TypeVar

ContextT = TypeVar("ContextT")
```

Now we can parameterize a resolver with whatever dataclass we want it to speak in.
Here is the shape of `PathResolver`, stripped down to the parts that matter for
typing:

```python
from typing import Generic
from pathlib import Path

class PathResolver(Generic[ContextT]):
    def __init__(self, context_type: type[ContextT], ...) -> None:
        self.context_type = context_type
        ...

    def resolve(self, name: str, context: ContextT) -> Path: ...
    def parse_path(self, path: Path) -> Optional[ContextT]: ...
    def find_matches(self, context: ContextT) -> list[str]: ...
```

`Generic[ContextT]` is the part doing the heavy lifting. When you construct a resolver,
the `TypeVar` gets bound to whatever dataclass you hand it, and that binding
propagates through every method signature. The IDE doesn't need to guess — it
reads `Generic[ContextT]` and treats the class as a template that gets specialized
at the call site.

Here is what that looks like in practice. Say we have a `ShotContext` for
shot-level work:

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class ShotContext(object):
    show: str
    seq: str
    shot: str
    task: Optional[str] = None
    version: Optional[str] = None
```

We construct a resolver against it:

```python
resolver = PathResolver(ShotContext)
resolver.register("shot_root", "/shows/<show>/<seq>/<shot>")
resolver.register("task_dir", "/shows/<show>/<seq>/<shot>/<task>")
```

At construction time, `ContextT` is bound to `ShotContext`. Every method on
`resolver` is now specialized: `resolve` takes a `ShotContext`, `parse_path` returns
an `Optional[ShotContext]`, `find_matches` takes a `ShotContext`. The IDE knows
this without us writing a single explicit annotation at the call site.

```python
ctx = resolver.parse_path(Path("/shows/foo/sq010/sh020"))
# IDE infers: ctx: Optional[ShotContext]

if ctx is not None:
    ctx.shot     # ✓ auto-completes, recognized field
    ctx.task     # ✓ auto-completes, recognized field
    ctx.artist   # ✗ red squiggle - not a field on ShotContext
```

That last line is the payoff. Without generics, `parse_path` would have to return
`Any` or some base class, and `ctx.artist` would just sail through with no warning
until it blew up at runtime. With `Generic[ContextT]`, the resolver carries the
dataclass type with it everywhere it goes, and the IDE can tell us we're wrong
before we even hit save.

The same thing happens in reverse on `resolve`:

```python
resolver.resolve("shot_root", ShotContext(show="foo", seq="sq010", shot="sh020"))  # ✓
resolver.resolve("shot_root", {"show": "foo"})                                     # ✗ type error
```

The signature `resolve(self, name: str, context: ContextT) -> Path` becomes
`resolve(self, name: str, context: ShotContext) -> Path` from the IDE's
perspective. Pass a dict and the type checker complains immediately.

The win compounds when you have multiple resolvers in the same module.
A` PathResolver[ShotContext]` and a `PathResolver[AssetContext]` are not the
same type — the IDE will not let you cross the streams, even though both
resolvers are running the same code underneath:

```python
shot_resolver: PathResolver[ShotContext] = PathResolver(ShotContext)
asset_resolver: PathResolver[AssetContext] = PathResolver(AssetContext)

shot = shot_resolver.parse_path(some_path)   # Optional[ShotContext]
asset = asset_resolver.parse_path(some_path) # Optional[AssetContext]

shot_resolver.resolve("shot_root", asset)    # ✗ type error - wrong context type
```

This is partly what I meant earlier by `Struct++`. Each context is still bite-sized —
`ShotContext` has the fields it needs and nothing else — but the resolver knows
which one it's working with at all times. No `Fat Struct` carrying every possible
field across the whole pipeline, no `dict[str, Any]` black hole, no guessing whether
the key is `"filepath"` or `"file_path"`. Just a small dataclass, a generic resolver,
and an IDE that's keeping up with both.

You can check out the full library [here](https://github.com/nate-maxwell/templar).
