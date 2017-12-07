# Split Sequence

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

`Sequence` is currently the root protocol for allowing a type to be used in a `for in` loop.
However, dispite not guaranteeing anything more, it's name and helper functions imply such.
Thus, it is proposed we lift the core functionality to a new protocol with a new name: `Iterable`
and add guarantees to `Sequence` that the helper functions were assuming.
Additionaly, this proposal suggests the possibility of requiring Iterators (`IteratorProtocol`) to conform to `Sequence`, permitting anyone to recover a `Sequence` from an `Iterable` by explicitly requesting the type impose an order on it's elements by calling `makeIterator()`.

Swift-evolution thread: [Discussion thread topic for that proposal](https://lists.swift.org/pipermail/swift-evolution/)

## Motivation

This proposal seeks to separate the idea that an instance has elements one can iterate through, and the assumption that the provided order has any semantic meaning other than that imposed by the iteration being, an inherently linear proccess.

In addition, because there is no syntactic differences between the new `Iterable` and `Sequence`, if a type conforms to `Iterable` and not `Sequence`, it must be because there is no inherent order in the data being iterated over, thus it is unwise for the body of the loop to depend upon this order, and thus, the compiler can, possibly, verify this lack-of-a-guarantee with warnings or even errors.

## Proposed solution

This proposal adds a new protocol `Iterable` that takes on `Sequence`'s compiler visible role, and to which the latter conforms, and, if desired, two example `Iterable`s. The example `Iterable` but not `Sequence` types are wrappers, one arround `arc4random_uniform(:)` and the other arround an arbatrary Base `Iterable` that uses nondeterminism to verify that whatever the Base is, it itself does not heed any guarantees one would expect from a `Sequence`. 
And, if reqiring Iterators (`IteratorProtocol`) to conform to `Sequence` is requested, than that shall be done as well, and the default implementation for Iterator-Sequences shall have the `where Self: Sequence` removed.

## Detailed design

```swift
/// A type that provides iterated access to its elements.
///
/// An iterable abstractly contains values that you can step through. The
/// most common way to iterate over the elements of a sequence is to use a
/// `for`-`in` loop:
///
///     let oneTwoThree = 1...3
///     for number in oneTwoThree {
///         print(number)
///     }
///     // Prints "1"
///     // Prints "2"
///     // Prints "3"
///
/// While seemingly simple, this capability gives you access to a large number
/// of operations that you can perform on any iterable. As an example, to
/// check whether an iterable includes a particular value, you can test each
/// value sequentially until you've found a match or reached the end of the
/// Iterable's iterator. This example checks to see whether a particular 
/// insect is in an array.
///
///     let bugs = ["Aphid", "Bumblebee", "Cicada", "Damselfly", "Earwig"]
///     var hasMosquito = false
///     for bug in bugs {
///         if bug == "Mosquito" {
///             hasMosquito = true
///             break
///         }
///     }
///     print("'bugs' has a mosquito: \(hasMosquito)")
///     // Prints "'bugs' has a mosquito: false"
///
/// The `Iterable` protocol provides default implementations for many common
/// operations that depend on sequential access to a sequence's values. For
/// clearer, more concise code, the example above could use the array's
/// `contains(_:)` method, which performs the iteration for you 
/// as every iterable inherits it from `Iterable` instead of iterating manually:
///
///     if bugs.contains("Mosquito") {
///         print("Break out the bug spray.")
///     } else {
///         print("Whew, no mosquitos!")
///     }
///     // Prints "Whew, no mosquitos!"
///
/// Ordered Access
/// ===============
///
/// The `Iterable` protocol makes no requirement on conforming types regarding
/// whether they have a standardized, or even deterministic iteration order.
/// However, it does gaurantee that if it's iterator's `.next()` is caled until
/// it returns `nil` it will return each element exactly once for each time the
/// iterable contains that element. As a consequence, don't assume that whether
/// or not one element preceeds annother is a sensible question. for example, even
/// if a `.copy()` method exists with expected semantics (and Element is Equatable):
///
///     let a, b: type(of: iterable).Element
///
///     for element in iterable.copy() {
///         a = element
///         break
///     }
///
///     for element in iterable {
///         b = element
///         break
///     }
///
///     // No defined behavior
///     if a == b {
///         ...
///     } else {
///         ...
///     }
///
/// In this case, you cannot assume either an iterable is a sequence and the
/// elements will be equal, or that an iterable will have been reordered and
/// the elements will be unequal. A conforming iterable that is
/// not a sequence is allowed to produce elements in an arbitrary order.
///
/// To establish that a type you've created has an iteration order that is
/// preserved by equality, add conformance to the `Sequence` protocol.
///
/// Repeated Access
/// ===============
///
/// The `Iterable` protocol makes no requirement on conforming types regarding
/// whether they will be destructively consumed by iteration. As a
/// consequence, don't assume that multiple `for`-`in` loops on a iterable
/// will either resume iteration or restart from the beginning:
///
///     for element in sequence {
///         if ... some condition { break }
///     }
///
///     for element in sequence {
///         // No defined behavior
///     }
///
/// In this case, you cannot assume either that an iterable will be consumable
/// and will resume iteration, or that an iterable is a collection and will
/// restart iteration from the first element. A conforming iterable that is
/// not a collection is allowed to produce arbitrary elements
/// in the second `for`-`in` loop.
///
/// To establish that a type you've created supports nondestructive iteration,
/// add conformance to a collection protocol.
///
/// Conforming to the Iterable Protocol
/// ===================================
///
/// Making your own custom types conform to `Iterable` enables many useful
/// operations, like `for`-`in` looping and the `contains` method, without
/// much effort. To add `Sequence` conformance to your own custom type, add a
/// `makeIterator()` method that returns an iterator.
///
/// Alternatively, if your type is a sequence and can act as its own iterator,
/// implementing the requirements of the `Sequence` and `IteratorProtocol`
/// protocols are sufficient.
///
/// Here's a definition of a `Countdown` sequence that serves as its own
/// iterator. The `makeIterator()` method is provided as a default
/// implementation.
///
///     struct Countdown: Sequence, IteratorProtocol {
///         var count: Int
///
///         mutating func next() -> Int? {
///             if count == 0 {
///                 return nil
///             } else {
///                 defer { count -= 1 }
///                 return count
///             }
///         }
///     }
///
///     let threeToGo = Countdown(count: 3)
///     for i in threeToGo {
///         print(i)
///     }
///     // Prints "3"
///     // Prints "2"
///     // Prints "1"
///
/// Expected Performance
/// ====================
///
/// An iterable should provide its iterator in O(1). The `Iterable` protocol
/// makes no other requirements about element access.
protocol Iterable /*: Compiler_ForIn_Able*/ {
  /// A type representing the sequence's elements.
  associatedtype Element

  /// A type that provides the sequence's iteration interface and
  /// encapsulates its iteration state.
  associatedtype Iterator : IteratorProtocol where Iterator.Element == Element

  /// Returns an iterator over the elements of this sequence.
  func makeIterator() -> Iterator

  /// A value less than or equal to the number of elements in
  /// the sequence, calculated nondestructively.
  ///
  /// - Complexity: O(1)
  var underestimatedCount: Int { get }

  /// Calls the given closure on each element in the sequence in the same order
  /// as a `for`-`in` loop.
  ///
  /// The two loops in the following example produce the same output:
  ///
  ///     let numberWords = ["one", "two", "three"]
  ///     for word in numberWords {
  ///         print(word)
  ///     }
  ///     // Prints "one"
  ///     // Prints "two"
  ///     // Prints "three"
  ///
  ///     numberWords.forEach { word in
  ///         print(word)
  ///     }
  ///     // Same as above
  ///
  /// Using the `forEach` method is distinct from a `for`-`in` loop in two
  /// important ways:
  ///
  /// 1. You cannot use a `break` or `continue` statement to exit the current
  ///    call of the `body` closure or skip subsequent calls.
  /// 2. Using the `return` statement in the `body` closure will exit only from
  ///    the current call to `body`, not from any outer scope, and won't skip
  ///    subsequent calls.
  ///
  /// - Parameter body: A closure that takes an element of the sequence as a
  ///   parameter.
  func forEach(_ body: (Element) throws -> Void) rethrows  
}

///...
/// Ordered Access
/// ===============
///
/// The `Sequence` protocol requires conforming types have an order that is 
/// preserved by equality. As a consequence, one whether or not one element
/// preceeds annother is a sensible question. for example, even if a `.copy()`
/// method exists with expected semantics (and Element is Equatable):
///
///     let a, b: type(of: sequence).Element
///
///     for element in sequence.copy() {
///         a = element
///         break
///     }
///
///     for element in sequence {
///         b = element
///         break
///     }
///
///     assert(a == b, "\(sequence) is not a valid Sequence")
///...
protocol Sequence: Iterable {
  ...
}

struct UniformRandomNumbers: Iterable, Iterator {
  var _min, _difference: Int
  init(between: Int, and: Int) {
    var min, max: Int
    (min, max) = between < and ? (between, and) : (and, between)
    _min = min
    _difference = max - min
    precondition(_differance <= UInt32.max, "Implementation requires (max-min) to fit in a UInt32")
  }
  
  mutating func next() -> Int? {
    return Int(arc4random_uniform(numericCast(_difference))) + _min
  }
}

struct LazilyRandomise<Base: Iterable>: Iterable {
  typealias Element = Base.Element
  var _base: Base
  
  init(_ b: Base) {
    _base = b
  }
  
  func makeIterator() -> Iterator {
    return LazilyRandomiseIterator(b)
  }
}

struct LazilyRandomiseIterator<Base: Iterable>: Iterable {
  typealias Element = Base.Element
  var _base: Base.Iterator
  var _buf1, _buf2: Element?
  
  init(_ b: Base.) {
    _base = b.makeIterator()
    _buf1 = _base.next()
  }
  
  ///FIXME: Implementation wrong, use 2+ element buffer, currently the 
  ///last element of base is always the last element of self!
  func next() -> Element {
    guard let n = _base.next() else {
      return _buf1
    }
    if arc4random_uniform(1) == 0 {
      return n
    } else {
      return _buf1
    }
  }
}
```

## Source compatibility

This proposal has no source concerns. However, there is some new semantic requirements on `Sequence`, but only ones most people (TODO: FIXME: check if possible) probably were implementing anyways, or they were aware they were deviating from standard. The stdlib violations of the new semantic requirements, namely `Set` and `Dictionary` will be fixed when we extend the distinction between order being a public guarantee and being a private implementation detail further down the `Collection` protocol tree in later proposals.

## Effect on ABI stability

This proposal has no ABI concerns.

## Effect on API resilience

This proposal changes API in a compatible way: extracting a new root for a protocol tree, possibly adding 2 new public types, and, if desired, adding new requirements for a type that all have default implementations.

## Alternatives considered

Do nothing, and live with the ambiguity between iteration order being guaranteed or an implementation detail in `Sequence`

