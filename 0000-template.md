# Protocol Conformance

* Proposal: [SE-NNNN](NNNN-filename.md)
* Authors: [Dante Broggi](https://github.com/Dante-Broggi)
* Review Manager: TBD
* Status: **Awaiting implementation**

*During the review process, add the following fields as needed:*

* Implementation: [apple/swift#NNNNN](https://github.com/apple/swift/pull/NNNNN)
* Decision Notes: [Rationale](https://lists.swift.org/pipermail/swift-evolution/), [Additional Commentary](https://lists.swift.org/pipermail/swift-evolution/)
* Bugs: [SR-NNNN](https://bugs.swift.org/browse/SR-NNNN), [SR-MMMM](https://bugs.swift.org/browse/SR-MMMM)
* Previous Revision: [1](https://github.com/apple/swift-evolution/blob/...commit-ID.../proposals/NNNN-filename.md)
* Previous Proposal: [SE-XXXX](XXXX-filename.md)

## Introduction

A protocol can currently `inherit` from other protocols, but not `conform` to them. This means that while one protocol can automatically add the requirements of another, and create a subtype relationship between them, it cannot provide a specific, concrete, implementation of another's requirements that is definitively used by the concrete types that conform to it. This proposal suggests we fix that.

Note that the feature here described is slightly, but significantly different to that described in [Generics Manifesto](https://github.com/apple/swift/blob/master/docs/GenericsManifesto.md#conditional-conformances-via-protocol-extensions), because the feature described there implies that the functions implemented with that feature use a variant on dynamic dispatch, while this feature's goal is to provide static dispatch for protocols.

Swift-evolution thread: [Discussion thread topic for that proposal](https://lists.swift.org/pipermail/swift-evolution/)

## Motivation

This proposal seeks to adress the case where one protocol abstracts some functionality, and another abstracts some similar functionality, where the second protocol can provide the first's requirements, but has additional invariants of it's own it must satisfy if it is doing so. For example:
```swift
protocol Fooable {
	func foo() -> Int
}

/// Could inherit `Fooable`, but the semantics of `Fooable` and `Barable`
/// combined imply that: if x is Fooable & Barable
/// then assert(x.foo() == x.bar() + 1) must hold.
/// (I don't know how or why, but they do, and swift's typesystem has no way
/// of ensuring that the condition hold, in the face of realitic programmers)
protocol Barable /*: Fooable*/ {
	func bar() -> Int
}

```

## Proposed solution

The proposed solution to this problem, in short, is to allow (unconstrained) extensions of protocols to have conformance clauses.
The reason to still forbid constrained, conformance providing extensions, is because they are an addition to this feature that can be handled later, in a separate proposal. However, even without that there are numerous challenges, both in design and implementation to overcome.

## Detailed design

With 2 protocols P and Q, and an extension P: Q:
- If a variable's concrete type is known
	- functions are dispatched as usual
- If a variable is only known to directly conform to P
	- calls to P's functions are dispatched as usual
	- calls to Q's functions are dispatched statically to the implementations in the P: Q extension.
- If a variable is only known to directly conform to Q
	- calls to P's functions are unavailable
	- calls to Q's functions are dispatched as usual
- If a variable is known to directly conform to P & Q
	- calls to functions defined in both P and Q are compile-tome ambiguity errors.
	- calls to P's functions are dispatched as if it only conformed to P
	- calls to Q's functions are dispatched as if it only conformed to Q

<!--
An example:

`\` `swift
/// TODO: FIXME:
protocol P {
	func foo() -> Int
	func baz()
}

protocol Q {
	func bar() -> Double
	func baz()
	func zaz()
}

extension P: Q {
	/// this function *must* be defined in this extension because `P` does not
	/// have a function with the same signature, and Q does not have a default
	/// implementation for it.
	func bar() -> Double {
		print("P: Q - bar")
		return Double(foo())
	}

	/// this function *may* be defined in this extension because `P` has a
	/// function with the same signature. If it is, it is considered to be a
	/// normal default implenentation for P.baz()
	func baz() -> Double {
		print("P: Q - baz")
	}

	/// this function *may* be defined in this extension because `P` does not
	/// have a function with the same signature, and Q has a default
	/// implementation for it.
	//func zaz() -> Double {
	//	print("P: Q - zaz")
	//	return Double(foo())
	//}
}

extension Q {
	/// a default implementation for Q.zaz()
	func zaz() -> Double {
		print("Q - zaz")
		return 0
	}
}

struct S1: P {
	/// this function *must* be defined, as usual.
	func foo() -> Int {
		print("S1 - foo")
		return 42
	}

	/// this function *must* be defined, as usual.
	func baz() {
		print("S1 - baz")
	}
}

struct S2: P, Q {

	/// this function *must* be defined, as usual.
	func foo() -> Int {
		print("S2 - foo")
		return 0
	}

	/// this function *must* be defined, as usual.
	func baz() {
		print("S2 - baz")
	}

	/// this function *may* be defined because `P` conforms to `Q`
	//func bar() -> Double {
	//	print("S2 - bar")
	//	return Double(foo())
	//}

	/// this function *may* be defined because `Q` has a default implementation
	//func zaz() -> Double {
	//	print("S2 - zaz")
	//	return 42
	//}
}

let s1 = S1()
let s2 = S2()

let p1 : P = s1
let p2 : P = s2

let q1 : Q = s1
let q2 : Q = s2

let pq1 : P & Q = s1
let pq2 : P & Q = s2

///
s1.foo() // "S1 - foo"
s2.foo() // "S2 - foo"
s1.baz() // "S1 - baz"
s2.baz() // "S2 - baz"

s1.bar() // "P: Q - bar"
s2.bar() // "P: Q - bar"
s1.zaz() // "Q - zaz"
s2.zaz() // "Q - zaz"

///
p1.foo() // P -> "S1 - foo"
p2.foo() // P -> "S2 - foo"
p1.baz() // P -> "S1 - baz"
p2.baz() // P -> "S2 - baz"

p1.bar() // P: Q -> "S1 - foo"
p2.foo() // P -> "S2 - foo"
p1.baz() // P -> "S1 - baz"
p2.baz() // P -> "S2 - baz"

q1.baz() // P -> "S1 - baz"
q2.baz() // P -> "S2 - baz"

`\` `
 -->

## Source compatibility

No source breakage, because this proposal gives a definition to syntax that has previously been an error.

## Effect on ABI stability

This proposal does not change ABI because it's direct effects are purely static and use the existing ABI.

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

Doing nothing, because this proposal is backwards compatible, and we have managed fine without it so far.

