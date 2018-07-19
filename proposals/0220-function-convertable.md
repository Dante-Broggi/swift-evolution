# Function Convertable Types

* Proposal: [SE-0220](0220-function-convertable.md)
* Authors: [Author 1](https://github.com/swiftdev)<!--, [Author 2](https://github.com/swiftdev)-->
* Review Manager: TBD
* Status: **Awaiting implementation**

*During the review process, add the following fields as needed:*

* Implementation: [apple/swift#NNNNN](https://github.com/apple/swift/pull/NNNNN)
<!--* Decision Notes: [Rationale](https://forums.swift.org/), [Additional Commentary](https://forums.swift.org/)-->
<!--* Bugs: [SR-NNNN](https://bugs.swift.org/browse/SR-NNNN), [SR-MMMM](https://bugs.swift.org/browse/SR-MMMM)-->
<!--* Previous Revision: [1](https://github.com/apple/swift-evolution/blob/...commit-ID.../proposals/NNNN-filename.md)-->
<!--* Previous Proposal: [SE-XXXX](XXXX-filename.md)-->

## Introduction

This proposal adds the ability for arbatrary types to be used as functions, either directly or being passed as one.

Swift-evolution thread: [Discussion thread topic for that proposal](https://forums.swift.org/)

## Motivation

One feature oft mentioned as desireable is the ability to use a keypath as a function, most recently in the [Sort `Collection` using `KeyPath`](https://forums.swift.org/t/sort-collection-using-keypath/14554) thread [here](https://forums.swift.org/t/sort-collection-using-keypath/14554/2) but also in [KeyPath based map, flatMap, filter](https://forums.swift.org/t/pitch-keypath-based-map-flatmap-filter/6266) and elsewhere.

This use can be approximated either by having overloads of the numerous functions one would want to pass a keypath instead of a normal function or by having either an operator or method to convert a keypath into a function, which is boilerplate without good syntax.

```swift
#warning("Insert examples here")
```

Additionally, It would seem to be benificial to have  "augmented" functions, which would be able to e.g. have properties for captured values or express that they have certain semantics, while still being usable as normal functions.
While this proposal doesn't express these automatically, it does reduce the boilerplate required to approximate it, in a way that is purely benificial for a proposal that seeks to express such things automatically. 

This use can be approximated by having a wraper type arround the desired function, conforming to the protocols and exposing the properties one disires, but one needs either an operator or method to convert the wrapper into a function for use, which is again boilerplate without good syntax.

```swift
#warning("Insert examples here")
```

## Proposed solution

### Function Convertability

Any method (instance function), either declared in a type or extention, shall be permitted to, instead of an operator or identifier name, use the symbol `_`. 

Any type: struct, enum, class, protocol, with at least one method with the name `_` is permitted to:
* be called as if it were one of its methods so named
* be considered a subtype of the type of each of its methods so named

Additionally, to preserve type recovery, any function may be dynamically downcasted with `as?` or `as!` into any type which implements a method of name `_` and the same type as the function being casted, which will return the instance the function was converted from, if it was converted from an instance of the type being casted to.

### Standard Library modification

The keypath types will get `_` named methods from `Base` to `Target` etc. 
```swift
#warning("Insert new API here")
```

## Detailed design

```swift
#warning("Insert new grammar here")

#warning("Insert new API here")
```

Describe the design of the solution in detail. If it involves new
syntax in the language, show the additions and changes to the Swift
grammar. If it's a new API, show the full API and its documentation
comments detailing what it does. The detail in this section should be
sufficient for someone who is *not* one of the authors to be able to
reasonably implement the feature.

## Source compatibility

This should be purely source-additive

## Effect on ABI stability

The only part of this proposal that *may* impact ABI stability is the casting from a function instance into a type which may be the function's origional type, as the conversion of a type into a function *should* be ABI compatible with ordnary closures.

In addition, it may be desireable to implement a `@convention`  to describe "a normal type" used as a function, separate from `@convention(swift)` the convention for normal swift functions, but in this draft that question is considered an implementation detail

## Effect on API resilience

API resilience describes the changes one can make to a public API
without breaking its ABI. Does this proposal introduce features that
would become part of a public API? If so, what kinds of changes can be
made without breaking ABI? Can this feature be added/removed without
breaking ABI? For more information about the resilience model, see the
[library evolution
document](https://github.com/apple/swift/blob/master/docs/LibraryEvolution.rst)
in the Swift repository.

## Alternatives considered

* Not adding this.
* Using operators or methods for the conversions

