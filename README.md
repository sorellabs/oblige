Oblige
======

Oblige is an expressive type notation system for multi-paradigm
languages that fit the object-oriented/functional spectrum. It's mostly
designed towards annotating JavaScript source code, and in a way that
could be used to derive static analysis tools (and support type
inferencing).

> **Note**: This document is a draft, things here may change at any
> time.
    

## 0. Prelude

JavaScript (and most of its close-cousins, like CoffeeScript or
LiveScript) is an untyped language. Types are, however, a nice way of
communicating constraints and expectations to the next programmer
reading the code-base.

There are some attempts[¹][1] at providing a type notation for
JavaScript, despite its untyped nature, but they either do not provide a
concise or compelling syntax to reduce the burden of maintaining such
documentation, or plainly fall short on providing an expressive enough
framework to describe most kinds of programs.

**Oblige** is based on the Haskell type notation system, as specified by
Mark P. Jones[²][2], with additions and omissions as needed to account
for the nature of JavaScript applications and possible type checking
tools that could be implemented on top of this specification.


[2]: http://web.cecs.pdx.edu/~mpj/thih/TypingHaskellInHaskell.html
[1]: #appendix-a


## 1. Introduction

### 1.1. The concept of types

Types are sets of possible values that fit a particular domain. In the
most basic form, they can be encoded in Oblige as:

```hs
<name>: <rule>
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
byte  : 0 ... 2^8
int   : 0 ... 2^32
float : 0 ... 2^64
```

This is because all of those types include the value `1` as a possible
member of the set. This way, a type definition is nothing more than an
*alias* to a particular set of values. And these sets of values can be
combined through union, intersection and difference operations.


### 1.3. Type variables and scope

Types all share a single scope. Types that can't be resolved to a
previous declaration — type variables — are to be inferred by the
inferring system (if any). Since these share the same scope, explicit
type variables should be written as uppercase single letters.


## 2. Types

As mentioned before, types are just sets of possible values, and they're
matched structurally. At the end of the day, types are just names for
these sets, not the sets themselves. Which means that a particular set
may have several names — or even be spelled out fully.

Types are defined in the form:

```hs
<name> <parameters>: <rule>
```

Where `<rule>` is what defines the set of possible values. These can be
any combination of primitive types.


### 2.1. Primitive types

#### 2.1.1. Numbers

Number values use the same notation as JavaScript numbers, except they
refer to the exact value:

```hs
zero: 0
one: 1
```

Besides this, you get the constants `+infinity`, `-infinity` and
`nan`. And you can use exponentiation by way of the `n^m` notation:

```hs
byte: 2^8
```

Intervals can be specified by the `n ... m` syntax, which is similar to
the `[n, m)` notation in mathematics, where the endpoint is excluded.


#### 2.1.2. Unit

The unit type is something that holds no value, and is usually used to 
specify functions that are evaluated by their side-effects. It has the
name `void`.


#### 2.1.2. Tuples

Tuples are an ordered sequence of types, and are encoded in Oblige as
`#[type1, type2, ..., typeN]`. 


#### 2.1.3. Lists

Lists are collections of values, and are encoded in Oblige as
`[type1, type2, ..., typeN]`, differently from tuples, lists don't
specify the order in which the types should appear, but rather which
types can be stored in the collection.


#### 2.1.4. Records

Record types are structures where the symbol on the left has the type on
the right.

```
rect: { x: int, y: int, width: int, height: int }
```


### 2.1.5. Functions

Function types specify the domain and image of a particular unit of
computation. We use the `->` arrow function to specify that a function
takes the input on the left and returns the output on the right:

```hs
-- | Identity a type A and returns something of the same type.
identity: A -> A

-- | Add takes two numbers, and returns a number
add: number, number -> number
```

Of course, functions may return multiple values. This is not something
that happens in JavaScript, but it's something that happens in Lua and
concatenative languages:

```hs
foo A B: A, B -> A, B
```

A function that can take variadic arguments can specify it so by using
the splat suffix:

```hs
-- | Concat takes any number of lists, and returns a single list
concat: [A]... -> [A]
```

Optional parameters can be suffixed with a question mark:

```hs
slice: [A], number, number? -> [A]

-- | This is equivalent to the more laborious overloaded notation:
slice: [A], number -> [A]
slice: [A], number, number -> [A]
```


### 2.1.6. Delegation

Types can specify delegation fields by way of the `<|` operator.

```hs
-- | List delegates to all types on the right
list <| collection, sequence
```


## 2.2. Parametric types

Parametric types are those that can be specialised by way of a type
constraint. This fits nicely on container types, where you can, for
example, have a `list of integers` or `list of strings`. Instead of
having different types for each one, or an extremely loose type,
parametric types let you define the right specialisation.

Parameters are defined in a way similar to how function parameters are
written in JavaScript, sans the parenthesis and commas. For example, one
could define a list of "something" as:

```hs
list A: [A]
```

And then, he would be able to use this definition to say that a
particular code works with `lists of integers`:

```hs
listOfIntegers: list int
```

Specialisation of types share the same notation as definition, and has a
parallel with function application in JavaScript (sans the
parenthesis/commas).


### 2.3. Type predicates

Types can have their domains further reduced by specifying a type
predicate. A type predicate defines which set of values match the type
with basis on a universal quantification. For example, which this
information we might not necessarily know how the sort function works:

```hs
sort: [A] -> [A]
```

But it would be immediately obvious if the sorting domain was a little
less broad. For instance, an implementation of `sort` where all values
can be compared using the relational `greater than` or `less than`
operators would be straightforward:

```hs
relational A: { isGreater: (A, A -> bool)
              , isLesser:  (A, A -> bool)
              }
              
sort: relational A => [A] -> [A]
```

The last type could be read as "`sort` has a type `list of A`'s to `list
of A`'s, where all `A`'s match the `relational` type."

A useful case for type predicates is to specify which kind of objects a
function can be applied to in JavaScript, due to dynamic this binding:

```hs
pop: @[A] => [A] -> maybe A
```


### 2.4. Tagged unions

Tagged unions can be defined by separating constructors with a `|`
symbol, where each constructor defines a unique, possibly parametric,
tag for the type. Nullary constructors are supported, e.g.:

```hs
bool: false | true
```

As are n-ary constructors:

```hs
tree A: leaf A | node (tree A) (tree A)
```

### 2.5. Set operations

Types can be structurally combined through set operations, these
include:

  - `union`: By way of `a + b`, which creates a new type that matches
    either things on the type A or type B.
   
  - `complement`: By way of `a \ b`, which creates a new type that
    matches things on type A but not on type B.


## 3. Built-in types for JavaScript

The following is the list of built-in types for JavaScript:

### 3.1. Primitive types

```hs
-- | No-value types
type null
undefined: void

-- | Boolean types
bool: true | false

-- | Numeric types
uint8   :     0 ... 2^8
uint16  :     0 ... 2^16
uint32  :     0 ... 2^32
int8    :  -2^8 ... 2^8
int16   : -2^16 ... 2^16
int32   : -2^32 ... 2^32
float32 : -2^32 ... 2^32
float64 : -2^64 ... 2^64
number  : float64

-- | String types
char   : uint8
string : [char]

-- | Container types
array A  : [A]
object A : { string -> A }

-- | Umbrella types
nothing : null | void
falsy   : "" | 0 | false | nothing
truthy  : any \ falsy

-- | Built-in types
type date
type regexp
```

### 3.2. Common interfaces

```hs
array-like: { length: uint32 }
```


## 4. Formal syntax

```hs
typeDecl :: typeName typeName* ":" typeDef EOL

reserved :: "void"
typeName :: "a" .. "z" | "A" .. "Z" | "-" | "_" | "$"           ?(NOT reserved)

typeDef :: group
         | predicate
         | union
         | complement
         | function
         | delegation
         | unit
         | number
         | tuple
         | list
         | record

         
typeDefs :: typeDef ("," typeDef)*         
typeDecl :: typeDecl ("," typeDecl)*

group :: "(" typeDef ")"

-- | Numeric types         
digit       :: "0" .. "9"
digits      :: digit+
sign        :: "-" | "+"
number      :: "+infinity"
             | "-infinity"
             | "nan"
             | exponential
             | interval
             | sign? digits ("." digits)
exponential :: number "^" number
interval    :: number "..." number

-- | Collections
tuple  :: "#[" typeDefs "]"
        | "#[" "]"

list   :: "[" typeDefs "]"
        | "[" "]"

record :: "{" typeDecls "}"
        | "{" "}"
        
-- | Functions
function     :: functionArgs "->" functionArgs
functionArg  :: typeDef argSuffix
functionArgs :: functionArg ("," functionArg)*
argSuffix    :: argOptional
              | argVariadic
argOptional  :: "?"
argVariadic  :: "..."

-- | Delegation
delegation :: typeDef "<|" typeDefs

-- | Predicates
predicate :: typePred "=>" typeDef
typePred  :: "@" typeDef
           | typeDef
           
-- | Tagged Unions
taggedUnion :: tag ("|" tag)*
tag         :: typeName typeDef*

-- | Set operations
union      :: typeDef "+" typeDef
complement :: typeDef "\" typeDef

-- | Unit type
unit :: "void"
```


## Appendix A

( ... ) TODO: Previous attempts on type notations for JS.


## Licence

<p xmlns:dct="http://purl.org/dc/terms/" xmlns:vcard="http://www.w3.org/2001/vcard-rdf/3.0#">
  <a rel="license"
     href="http://creativecommons.org/publicdomain/zero/1.0/">
    <img src="http://i.creativecommons.org/p/zero/1.0/88x31.png" style="border-style: none;" alt="CC0" />
  </a>
  <br />
  To the extent possible under law,
  <a rel="dct:publisher"
     href="http://killdream.github.com/">
    <span property="dct:title">Quildreen Motta</span></a>
  has waived all copyright and related or neighboring rights to
  <span property="dct:title">Oblige</span>.
This work is published from:
<span property="vcard:Country" datatype="dct:ISO3166"
      content="BR" about="http://killdream.github.com/">
  Brazil</span>.
</p>
