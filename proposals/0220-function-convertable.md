# Function Convertable Types

* Proposal: [SE-NNNN](NNNN-function-convertable.md)
* Authors: [Dante Broggi](https://github.com/dante-broggi)<!--, [Author 2](https://github.com/swiftdev)-->
* Review Manager: TBD
* Status: **Awaiting implementation**

*During the review process, add the following fields as needed:*

* Implementation: [apple/swift#NNNNN](https://github.com/apple/swift/pull/NNNNN)
<!--* Decision Notes: [Rationale](https://forums.swift.org/), [Additional Commentary](https://forums.swift.org/)-->
<!--* Bugs: [SR-NNNN](https://bugs.swift.org/browse/SR-NNNN), [SR-MMMM](https://bugs.swift.org/browse/SR-MMMM)-->
<!--* Previous Revision: [1](https://github.com/apple/swift-evolution/blob/...commit-ID.../proposals/NNNN-filename.md)-->
<!--* Previous Proposal: [SE-XXXX](XXXX-filename.md)-->

## Introduction

This proposal adds the ability for arbatrary types to be used as functions, either directly or being passed as one, and adds this new capability to keypaths.

This proposal is the merger of two separately concieved ideas, the use of keypaths as functions, which <!--, I believe, -->was not formally proposed as it did not have good obvious syntax, although it had motivating examples, and non-dynamic custom callable types, suggested during the dynamically callable proposal, because that proposal was adding custom callable types for interop with other languages, without that capability being in Swift for itself first.

Swift-evolution thread: [Function Convertable Types](https://forums.swift.org/)

## Motivation

One feature oft mentioned as desireable is the ability to use a keypath as a function, most recently in the [Sort `Collection` using `KeyPath`](https://forums.swift.org/t/sort-collection-using-keypath/14554) thread [here](https://forums.swift.org/t/sort-collection-using-keypath/14554/2) but also in [KeyPath based map, flatMap, filter](https://forums.swift.org/t/pitch-keypath-based-map-flatmap-filter/6266) and elsewhere.

This concept was suggested in the [Dynamically Callable](https://forums.swift.org/t/se-0216-user-defined-dynamically-callable-types/13615) thread [here](https://forums.swift.org/t/se-0216-user-defined-dynamically-callable-types/13615/5)
The name `_` was suggested [here](https://forums.swift.org/t/se-0216-user-defined-dynamically-callable-types/13615/51), in the same thread.


This use can be approximated either by having overloads of the numerous functions one would want to pass a keypath instead of a normal function or by having either an operator or method to convert a keypath into a function, which is boilerplate without good syntax.

```swift
#warning("Insert examples here")
```

Other examples where this feature would be usefull include the use of methods of the form `func _()` to create concice, and probably easily identifiable <!-- FIXME: word choice -->syntax for resolving `Result` , `Future` and similar types into their values.

```swift
#warning("Insert examples here")
```

This feature would also permit changing the `@autoclosure () -> T` syntax into `Autoclosure<T>` syntax, which, if it were proposed, would remove the amiguity of "can I put a parameter for the autoclosure".

Additionally, It would seem to be benificial to have  "augmented" functions, which would be able to e.g. have properties for captured values or express that they have certain semantics, while still being usable as normal functions.
While this proposal doesn't express these automatically, it does reduce the boilerplate required to approximate it, in a way that is purely benificial for a proposal that seeks to express such things automatically. 

This use can be approximated by having a wraper type arround the desired function, conforming to the protocols and exposing the properties one disires, but one needs either an operator or method to convert the wrapper into a function for use, which is again boilerplate without good syntax.

<!--Finally, this feature would potentially permit features like pure functions and compiler evaluatable functions to be expressed as compiler generated protocols instead of `@attribute`s if desired. -->

```swift
#warning("Insert examples here")
```

## Proposed solution

### Function Convertability

Any method (instance function), either declared in a type or extention, shall be permitted to, instead of an operator or identifier name, use the symbol `_`. 
For the time being, `static` and `class` functions will not be permitted to be named `_`, as this would create potential ambiguity with the prexisting call syntax on metatypes that this proposal does not desire to resolve. However, a future proposal may lift this restriction. Such future work should keep in mind that call syntax on metatypes is already meaningful, both for implicit `init` calls and, after [SE-0213](https://github.com/apple/swift-evolution/blob/master/proposals/0213-literal-init-via-coercion.md), coersion from literals, and that ambiguity would have to be resolved somehow (e.g. through the most specific rule) or be subsumed.

Any type: struct, enum, class, protocol, with at least one method with the name `_` is permitted to:
* be called as if it were one of its methods so named
* be considered a subtype of the type of each of its methods so named

Additionally, to preserve type recovery, any function may be dynamically downcasted with `as?` or `as!` into any type which implements a method of name `_` of the same type as the function being casted, which will return the instance the function was converted from, if it was converted from an instance of the type being casted to.

### Similarity to [SE-0216 Dynamically Callable](https://github.com/apple/swift-evolution/blob/master/proposals/0216-dynamic-callable.md)
Although this feature is similar to the dynamically callable feature, it has significant differences, both mentioned here and in SE-0216. 
In particular, the dynamically callable proposal had to be able to express methods where the number of arguments and their labels were not defined by the implementor of the method, but could require all of the method's arguments be the same type. Whereas this proposal must be able to express the entire spectrum of swift methods as instance calls, but does not need arbatrary argument labels.

This proposal does require that, if a method named `_` is a visible member on an `@dynamicCallable` type, all applicable methods named `_`  will be prefered over the `dynamicallyCall` methods.

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

* A function name mangling for functions named `_` must be added.
* The casting from a function instance into a type which may be the function's original type, *may* not be ABI changing, as the conversion of a type into a function *should* be ABI compatible with ordnary closures.
* In addition, it may be desireable to implement a `@convention`  to describe "a normal type" used as a function, separate from `@convention(swift)` the convention for normal swift functions, but in this draft that question is considered an implementation detail.

## Effect on API resilience

* Methods will be able to be named `_`, these methods will obey all of the normal method resiliance requirements and capabilities.
* The new methods on the keypath types will be adding to the Standard Library API and ABI.

## Future directions

* Permit adding `static func _`, aka permitting call syntax on metatypes to be overloaded.

* Use methods of the form `func _()`, if or when `Result` or `Future` types are added to the Standard Library.

* Replacing  `@autoclosure () -> T` syntax with `Autoclosure<T>` syntax, potentialy wth an `ExpressibleWithExpressionOf<T>` parameterized protocol, which would lift `Autoclosure<T>` out of bing a special case in the compiler and remove the amiguity of "can I put a parameter for the autoclosure".

## Alternatives considered

* Not adding this.
* Using operators or methods for the conversions

