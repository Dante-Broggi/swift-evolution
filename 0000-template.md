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

Describe the design of the solution in detail. If it involves new
syntax in the language, show the additions and changes to the Swift
grammar. If it's a new API, show the full API and its documentation
comments detailing what it does. The detail in this section should be
sufficient for someone who is *not* one of the authors to be able to
reasonably implement the feature.

## Source compatibility

This proposal has no source concerns. However, there is some new semantic requirements on `Collection`, but only ones most people (TODO: FIXME: check if possible) probably were implementing anyways, or they were aware they were deviating from standard. The stdlib violations of the new semantic requirements, will have been fixed by this proposal.

## Effect on ABI stability

This proposal has no ABI concerns.

## Effect on API resilience

This proposal changes API in a compatible way: inserting a new protocol in a tree.

## Alternatives considered

Describe alternative approaches to addressing the same problem, and
why you chose this approach instead.

