Oblige
======

Oblige is an expressive type notation system for multi-paradigm
languages that fit the object-oriented/functional spectrum. It's mostly
designed towards annotating JavaScript source code, and in a way that
could be used to derive static analysis tools (and support type
inferencing).


## 0. Prelude

JavaScript (and most of its close-cousins, like CoffeeScript or
LiveScript) is an untyped language. Types are, however, a nice way of
communicating constraints and expectations to the next programmer
reading the code-base.

There are some attempts[¹][appendix-a] at providing a type notation for
JavaScript, despite its untyped nature, but they either do not provide a
concise or compelling syntax to reduce the burden of maintaining such
documentation, or plainly fall short on providing an expressive enough
framework to describe most kinds of programs.

**Oblige** is based on the Haskell type notation system, as specified by
Mark P. Jones[²][2], with additions and omissions as needed to account
for the nature of JavaScript applications and possible type checking
tools that could be implemented on top of this specification.


[2]: http://web.cecs.pdx.edu/~mpj/thih/TypingHaskellInHaskell.html


## 1. Introduction

### 1.1. The concept of types

Types are sets of possible values that fit a particular domain. In the
most basic form, they can be encoded in Oblige as:

```hs
<name> :: <rule>
```

Where `name` is a human readable description of the domain, and `rule`
defines the possible values for the domain, similar to what you have in
set-theory.

*Type checking*, then, refers to the process of determining whether a
particular value belongs to a domain or not. Because this is all based
on sets, a value is said to belong to a particular domain if it matches
the range of values that the type describes **not the type name**.

For example, the value `1` would match all the following types at once:

```hs
byte  :: 0 ... 2^8
int   :: 0 ... 2^32
float :: 0 ... 2^64
```

This is because all of those types include the value `1` as a possible
member of the set. This way, a type definition is nothing more than an
*alias* to a particular set of values. And these sets of values can be
combined through union, intersection and difference operations.


### 1.2. The notation

As mentioned before, types are encoded as simple `<name> :: <rule>`
definitions, which can be read as `<name> is the type of all
<rule>`. E.g.: `int :: 0 ... 2^32` would be read as `int is the type of
all numbers between 0 and 2³²`.

Rules can contain simple values: `0`, `1`, ...; ranges of values: `0
... 2` (not inclusive); unique symbols: `'false`. 

They can be aggregated using tuples: `(0, 1)` which are ordered
sequences of values; lists: `[0]` which are repetitions of values; and
maps: `{ 0 -> 1 }`.

Types can be composed using the usual set operations/type theory: `|` is
a disjoint union; `&` is intersection; and `-` is complement.

Of course, values can always be substituted by types, so if you have a
`zero :: 0` declaration, it makes no difference if you use `binary :: 0
| 1` or `binary :: zero | 1`.


## 2. Basic types

### 2.1. The Universal Type

The most basic type of all is the **universal type**, which encodes all
the possible values that can ever come into existence in a given
program. This is also called the `Top` type (⊤) in type theory.

```hs
any :: ⊤
```

### 2.2. Primitive types

JavaScript has only a handful of conceptual primitive types. Oblige
provides a few more conceptual types that form the basis of JavaScript
complex primitive types like `String` or `Number`:


```hs
-- | No values
null
undefined

-- | Boolean types
bool :: true | false

-- | Numeric types
uint8   ::     0 ... 2^8
uint16  ::     0 ... 2^16
uint32  ::     0 ... 2^32
int8    ::  -2^8 ... 2^8
int16   :: -2^16 ... 2^16
int32   :: -2^32 ... 2^32
float32 :: -2^32 ... 2^32
float64 :: -2^64 ... 2^64

-- | Character types
char :: uint8

-- | Higher-level aliases
number :: float64
string :: [char]
```

### 2.3. Parametric types

Parametric types are those that can be specialised by way of a type
constraint. This fits nicely on container types, where you can, for
example, have a `list of integers` or `list of strings`. Instead of
having different types for each one, or an extremely loose type,
parametric types let you define the right specialisation.

Parameters are defined in a way similar to how function parameters are
written in JavaScript, sans the parenthesis and commas. For example, one
could define a list of "something" as:

```hs
list a :: [a]
```

And then, he would be able to use this definition to say that a
particular code works with `lists of integers`:

```hs
listOfIntegers :: list int
```

Specialisation of types share the same notation as definition, and has a
parallel with function application in JavaScript (sans the
parenthesis/commas).



### 2.4. Container types

Container types are parametric types that can contain other primitive
values. In JavaScript's world, these are `Array` and `Object` most of
the time.

```hs
array a  :: [a]
object a :: { string -> a }
```


### 2.5. Umbrella types

Umbrella types are disjoint unions of high-level conceptual interfaces
that we occasionally refer to. They're basically "bags" of types that
are handled similarly in certain situations (e.g.: `Falsy` in boolean
comparisons).


```hs
nothing :: null | undefined
falsy   :: "" | 0 | false | nothing
truthy  :: any - falsy
```


### 2.6. Function types




## Appendix A

( ... ) Previous attempts on type notations for JS.
