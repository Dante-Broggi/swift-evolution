# Unordered Collections

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

Currently the Collection protocols express the assumption that all collections require an explicit linear order for their elements, despite the fact that the swift standard library has two types that violate that assumption, namely `Set` and `Dictionary`. This proposal fixes that, by adding an `UnorderedCollection` between `Sequence` and `Collection`, or if the `Iterable` proposal is accepted, then `UnorderedCollection` will inherit from `Iterable` and `Collection` will inherit from both `UnorderedCollection` and `Sequence`.

Swift-evolution thread: [Discussion thread topic for that proposal](https://lists.swift.org/pipermail/swift-evolution/)

## Motivation

Unordered collections such as `Set` and `Dictionary` currently are forced to have `first` and `last` properties, despite the fact that the implementations of these properties in those types reflect implementation details that should ideally be hidden, and fail to have the semantics that should be guaranteed by `Collection`, but has not been explicitly described to permit these types to fall through the cracks, namely that equal collections have equal first, last and gennerally nth elements<!-- (NOTE: collections that satisfy this condition need not be equal, it is a necessary, but certainly not sufficient condition) -->.

This means that programmers passing such unordered collections to collection generic functions or extensions must be vigilant to ensure the function or extension only uses properties that their unordered collection type has implemented with semantics close enough to the semantics that collection should require.

## Proposed solution

Add an `UnorderedCollection` between `Sequence` and `Collection`,
	or if the `Iterable` proposal is accepted, then `UnorderedCollection`
	will inherit from `Iterable` and `Collection` will inherit from both `UnorderedCollection` and `Sequence`.
This protocol will extract the requirements of `Collection` that do not imply a definitive order, and what automatic generic functionality is able to be implemented against the weaker constraints.

Add stricter semantic conditions to `Collection` to emphasize the new distinction between `UnorderedCollection` and `Collection`.

## Detailed design

```swift
/// An iterable whose elements can be traversed multiple times,
/// nondestructively, and accessed by an indexed subscript.
///
/// UnorderedCollections are used throughout the standard library. When you
/// use sets, dictionaries, and other unordered collections, you benefit from the
/// operations that the `UnorderedCollection` protocol declares and implements.
/// In addition to the operations that collections inherit from the `Iterable`
/// protocol, you gain access to methods that depend on accessing an element
/// at a specific position in a collection.
///
/// For example, if you want to print only the first word in a string, you can
/// search for the index of the first space, and then create a substring up to
/// that position.
///
///     let text = "Buffalo buffalo buffalo buffalo."
///     if let firstSpace = text.index(of: " ") {
///         print(text[..<firstSpace])
///     }
///     // Prints "Buffalo"
///
/// The `firstSpace` constant is an index into the `text` string---the position
/// of the first space in the string. You can store indices in variables, and
/// pass them to collection algorithms or use them later to access the
/// corresponding element. In the example above, `firstSpace` is used to
/// extract the prefix that contains elements up to that index.
///
/// Accessing Individual Elements
/// =============================
///
/// You can access an element of a collection through its subscript by using
/// any valid index, except the collection's `endIndex` property, if it has one.
/// This property is a "past the end" index that does not correspond with any element of the
/// collection, an is defined by `Collection`.
///
/// Here's an example of accessing the first character in a string through its
/// subscript:
///
///     let firstChar = text[text.startIndex]
///     print(firstChar)
///     // Prints "B"
///
/// The `UnorderedCollection` protocol declares and provides default implementations for
/// many operations that depend on elements being accessible by their
/// subscript.
///
/// You can pass only valid indices to collection operations. Any index returned
/// by a method that starts with `index` in this protocol are valid indices.
/// This protocol does not define whether any other values of the `Index` type are valid,
/// or invalid, indices for this collection.
///
/// Saved indices may become invalid as a result of mutating operations. For
/// more information about index invalidation in mutable collections, see the
/// reference for the `MutableCollection` and `RangeReplaceableCollection`
/// protocols, as well as for the specific type you're using.
///
/// Traversing an UnorderedCollection
/// =======================
///
/// Although an Iterable can be consumed as it is traversed, a collection is
/// guaranteed to be *multipass*: Any element can be repeatedly accessed by
/// saving its index. Moreover, a collection's indices form a finite range of
/// the positions of the collection's elements. The fact that all collections
/// are finite guarantees the safety of many of Iterable's operations, such as
/// using the `contains(_:)` method to test whether a collection includes an
/// element.
///
/// Iterating over the elements of a collection by their positions yields the
/// same elements in the same order as iterating over that collection using
/// its iterator. This example demonstrates that the `characters` view of a
/// string returns the same characters in the same order whether the view's
/// indices or the view itself is being iterated.
///
///     let word = "Swift"
///     for character in word {
///         print(character)
///     }
///     // Prints "S"
///     // Prints "w"
///     // Prints "i"
///     // Prints "f"
///     // Prints "t"
///
///     for i in word.indices {
///         print(word[i])
///     }
///     // Prints "S"
///     // Prints "w"
///     // Prints "i"
///     // Prints "f"
///     // Prints "t"
///
/// Conforming to the UnorderedCollection Protocol
/// =====================================
///
/// If you create a custom iterable that can provide repeated access to its
/// elements, make sure that its type conforms to the `UnorderedCollection`
/// protocol in order to give a more useful and more efficient interface for
/// iteration and collection operations. To add `UnorderedCollection conformance
/// to your type, you must declare at least the following requirements:
///
/// - A subscript and an iterator that provide at least read-only access to
///   your type's elements
///
/// - An Indices type and property that provide at least read-only access to
///   your type's indices
///
/// Expected Performance
/// ====================
///
/// Types that conform to `Collection` are expected to provide subscript access
/// to elements as O(1) operations. Types that are not able to guarantee this
/// performance must document the departure, because many collection operations
/// depend on O(1) subscripting performance for their own performance guarantees.
///
/// The performance of some collection operations depends on the type of index
/// that the collection provides. For example, because a forward
/// or bidirectional collection must traverse the entire collection to count
/// the number of contained elements, accessing its `count` property is an
/// O(*n*) operation.
public protocol UnorderedCollection: Sequence { //Iterable
    associatedtype Element

    /// A type that represents a position in the collection.
    ///
    /// Valid indices consist of the position of every element and a
    /// "past the end" position that's not valid for use as a subscript
    /// argument.
    associatedtype Index

    /// A type that provides the collection's iteration interface and
    /// encapsulates its iteration state.
    associatedtype Iterator: Sequence

    /// Returns an iterator over the elements of the collection.
    func makeIterator() -> Iterator

    /// Accesses the element at the specified position.
    ///
    /// The following example accesses an element of an array through its
    /// subscript to print its value:
    ///
    ///     var streets = ["Adams", "Bryant", "Channing", "Douglas", "Evarts"]
    ///     print(streets[1])
    ///     // Prints "Bryant"
    ///
    /// You can subscript a collection with any valid index
    ///
    /// - Parameter position: The position of the element to access. `position`
    ///   must be a valid index of the collection.
    ///
    /// - Complexity: O(1)
    subscript(position: Index) -> Element { get }

    /// A type that represents the indices that are valid for subscripting the
    /// collection.
    associatedtype Indices : UnorderedCollection
    where Indices.Element == Index,
    Indices.Index == Index

    /// The indices that are valid for subscripting the collection
    ///
    /// A collection's `indices` property can hold a strong reference to the
    /// collection itself, causing the collection to be nonuniquely referenced.
    /// If you mutate the collection while iterating over its indices, a strong
    /// reference can result in an unexpected copy of the collection.
    var indices: Indices { get }

    /// A Boolean value indicating whether the collection is empty.
    ///
    /// When you need to check whether your collection is empty, use the
    /// `isEmpty` property instead of checking that the `count` property is
    /// equal to zero. For collections that don't conform to
    /// `RandomAccessCollection`, accessing the `count` property iterates
    /// through the elements of the collection.
    ///
    ///     let horseName = "Silver"
    ///     if horseName.isEmpty {
    ///         print("I've been through the desert on a horse with no name.")
    ///     } else {
    ///         print("Hi ho, \(horseName)!")
    ///     }
    ///     // Prints "Hi ho, Silver!")
    ///
    /// - Complexity: O(1)
    var isEmpty: Bool { get }

    /// The number of elements in the collection.
    ///
    /// To check whether a collection is empty, use its `isEmpty` property
    /// instead of comparing `count` to zero. Unless the collection guarantees
    /// random-access performance, calculating `count` can be an O(*n*)
    /// operation.
    ///
    /// - Complexity: O(1) if the collection conforms to
    ///   `RandomAccessCollection`; otherwise, O(*n*), where *n* is the length
    ///   of the collection.
    var count: Int { get }
}

/// An unordered collection that supports subscript assignment.
///
/// collections that conform to `MutableUnorderedCollection` gain the ability to
/// change the value of their elements. This example shows how you can
/// modify one of the names in an array of students.
///
///     var students = ["Ben", "Ivy", "Jordell", "Maxime"]
///     if let i = students.index(of: "Maxime") {
///         students[i] = "Max"
///     }
///     print(students)
///     // Prints "["Ben", "Ivy", "Jordell", "Max"]"
///
/// Conforming to the MutableUnorderedCollection Protocol
/// ============================================
///
/// To add conformance to the `MutableUnorderedCollection` protocol to your own
/// custom collection, upgrade your type's subscript to support both read
/// and write access.
///
/// A value stored into a subscript of a `MutableUnorderedCollection`
/// instance must subsequently be accessible at that same position.
/// That is, for a mutable collection instance `a`, index `i`, and value `x`,
/// the two sets of assignments in the following code sample must be equivalent:
///
///     a[i] = x
///     let y = a[i]
///
///     // Must be equivalent to:
///     a[i] = x
///     let y = x
public protocol MutableUnorderedCollection: UnorderedCollection {

  /// Accesses the element at the specified position.
  ///
  /// For example, you can replace an element of an array by using its
  /// subscript.
  ///
  ///     var streets = ["Adams", "Bryant", "Channing", "Douglas", "Evarts"]
  ///     streets[1] = "Butler"
  ///     print(streets[1])
  ///     // Prints "Butler"
  ///
  /// You can subscript a collection with any valid index other than the
  /// collection's end index. The end index refers to the position one
  /// past the last element of a collection, so it doesn't correspond with an
  /// element.
  ///
  /// - Parameter position: The position of the element to access. `position`
  ///   must be a valid index of the collection that is not equal to the
  ///   `endIndex` property.
  subscript(position: Index) -> Element { get set }

  /// Exchanges the values at the specified indices of the collection.
  ///
  /// Both parameters must be valid indices of the collection and not
  /// equal to `endIndex`. Passing the same index as both `i` and `j` has no
  /// effect.
  ///
  /// - Parameters:
  ///   - i: The index of the first value to swap.
  ///   - j: The index of the second value to swap.
  mutating func swapAt(_ i: Index, _ j: Index)
}

///...
public protocol Collection:
	OrderedCollection, Sequence where SubSequence: Collection {

		///...

    /// The first element of the collection.
    ///
    /// Two equal colections must have equal first elements.
    /// ```swift
    /// let a, b : Collection
    /// ...
    /// if a == b {
    /// 	assert(a.first == b.first)
    /// }
    /// ```
    ///
    /// If the collection is empty, the value of this property is `nil`.
    ///
    ///     let numbers = [10, 20, 30, 40, 50]
    ///     if let firstNumber = numbers.first {
    ///         print(firstNumber)
    ///     }
    ///     // Prints "10"
    var first: Element? { get }

    ///...
}

///...
public protocol MutableCollection: Collection, MutableUnorderedCollection
where SubSequence: MutableCollection {
	///...
}
```

## Source compatibility

This proposal has no source concerns. However, there is some new semantic requirements on `Collection`, but only ones most people (TODO: FIXME: check if possible) probably were implementing anyways, or they were aware they were deviating from standard. The stdlib violations of the new semantic requirements, will have been fixed by this proposal.

## Effect on ABI stability

This proposal has no ABI concerns.

## Effect on API resilience

This proposal changes API in a compatible way: inserting a new protocol in a tree.

## Alternatives considered

Do nothing, and live with the ambiguity between ordered and unordered collections.

