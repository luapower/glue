---
title:    design of glue
tagline:  how and why
---

## Scope

A place to accumulate good implementations of common Lua idioms, mainly so
that they can be copy-pasted in library code (a typical library only needs
a few functions from glue) or used directly in app code (where another
dependency is not a problem). In this context a good implementation is first
of all small and its corner cases well documented (since they are usually not
addressed in the implementation in order to keep the code small and fast).

## Naming

The idea is to find the most popular, familiar and _short_ names for _each
and every_ function (no underscores and no capitals). Python gets this right,
so does UNIX. A function with an unheard of name or alien semantics will be
avoided. People rather recall known names/semantics rather than learn
unfamiliar new names/semantics, even when those would be more clear.

## Semantics

They follow the general [api-design] rules.

### Objects vs glue

Don't provide data structures like list and set in a glue library, or a way
to do OOP. Instead just provide the mechanisms as functions working on bare
tables. Don't do both either: if your list type gets widely adopted, your
programs will now be a mixture of bare tables (this is inevitable) and lists
so now you have to decide which of your lists has a `sort()` method and which
need to be wrapped first.

### Write in Lua

String lambdas, callable strings, list comprehensions are all fun, but they
add syntax and a learning curve and should be generally avoided in contexts
where their use is spare.

### Sugar

Don't add shortcut functions except when calling the shortcut function makes
the intent clearer than when reading the equivalent Lua code.

If something is an [idiom][lua-tricks], don't add a function for it, use it
directly. Chances are its syntax will be more popular than its name. Eg. it's
harder to recall and trust semantic equivalence of `isnan(x)` to the odd
looking but mnemonic idiom `x ~= x` (eg. does `isnan` raise an error when `x`
is not a number?). That doesn't mean `a < b and a or b` is a good idiom for
`math.min(a, b)` though, `min()` itself is the idiom as we know it from math
(`sign()`, `clamp()`, etc. are idioms too).

Functional programming sugars like `compose` and `bind` makes code harder to
read because brains are slow to switch between abstraction levels unless it's
a self-contained DSL with radically different syntax and semantics than the
surrounding code. Eg. it's easier to read a Lua string pattern or an embedded
SQL string than it is to read expressions involving `bind` and `compose`
which force you to simulate the equivalent Lua syntax in your head.

Sugars like "message %s" % arg are the good ones: % is odd enough to put after
a string constant that it has an idiomatic quality, and its semantics is
self-evident by reading the format string literal, even for someone who never
heard of python's `%` operator. Also, a prefix notation is generally more
readable than a function call.

## Implementation

Keep the code readable and compact. Code changes that compromise these
qualities for optimization should come with a benchmark to justify them.

Document the limits of the algorithms involved with respect to input, like
when does it have non-linear performance and if and how it is stack bound.
Performance characteristics are not an implementation detail.
