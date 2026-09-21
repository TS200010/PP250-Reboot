# SCT garbage collection, object lifetime and lazy traversal

> Status: research note preserving an exploratory discussion. The historical PP250 garbage-collection mechanism is not yet fully reconstructed. Statements about a modern implementation are hypotheses until checked against original System 250 documentation.

## The problem was initially framed incorrectly

When considering C/C++ object safety, an apparent problem was stale authority after memory reuse:

    object A uses physical block X
    object A is deleted
    object B later uses physical block X

For PP250 this is not, by itself, the important problem.

The program's authority is not supposed to be a raw physical-memory address. A virtual segment has an identity mediated by the SCT and virtual-memory system. A newly created segment need not receive physical memory until first use, and when backing storage is assigned it may receive any suitable block.

Therefore reuse of physical block X does not imply reuse of the old segment's authority.

## The real reuse problem is the SCT identity

The genuine temporal-safety problem occurs if an SCT entry is recycled while an old capability referring to that entry still exists.

For example:

    SCT[n] -> segment A

Capabilities referring to SCT[n] may have propagated into other capability segments, saved process state, or other architecturally valid storage.

If segment A is destroyed and SCT[n] is immediately reassigned:

    SCT[n] -> segment B

then a surviving old capability to A could become authority over B.

Therefore an SCT entry cannot safely be recycled merely because the physical backing of its former segment has been released.

The system has to establish that the SCT identity is no longer reachable through live capabilities.

## This is the SCT garbage-collection problem

The PP250 architecture contains garbage-collection support associated with SCT entries. The discussion recalled bits in SCT entries used while walking and marking the capability structure.

The broad model is a reachability traversal:

    roots
      |
      +--> SCT A [mark]
      |      |
      |      +--> capability -> SCT C [mark]
      |
      +--> SCT B [mark]
             |
             +--> capability -> SCT D [mark]

An allocated SCT entry not demonstrated reachable after the required traversal can eventually become eligible for reclamation.

This separates two kinds of reclamation that should not be conflated:

    physical storage reclamation
        - a VM/backing-store concern

    SCT identity reclamation
        - a capability reachability concern

The second is the security-critical issue for stale capabilities.

## Capability blocks, not arbitrary data, are traversed

A useful property of the PP250 model is the architectural distinction between ordinary data and capability-containing storage.

Garbage collection therefore need not be described as "scan every word of RAM looking for things that might be pointers." The capability graph is structured.

That may make SCT reachability collection substantially simpler than conservative garbage collection on a conventional machine.

The exact historical rules governing capability segments/blocks must be taken from the source documentation.

## Outform is a critical complication

A capability block in outform cannot simply be treated as though its capabilities were resident and directly traversable.

The recollection/discussion is that an outformed capability block has to be explicitly loaded/informed before the contained capability relationships can be walked.

Conceptually:

    marked SCT entry
          |
          v
    capability block
          |
          +-- already in usable capability form
          |       -> walk contained capabilities
          |
          +-- outform
                  -> explicitly load/inform
                  -> then walk contained capabilities

This means the historical GC algorithm cannot have been a naive in-memory mark pass. It had to cooperate with the mechanisms for capability representation and secondary/backing storage.

This is an important target for documentary reconstruction.

## The concurrency question

A subtler question arose: can the SCT collection be lazy or concurrent with ordinary execution?

Imagine that the collector has already traversed capability block A. While collection continues elsewhere, a running process copies a capability to previously unmarked segment X into A.

    A [already traversed]
      |
      +---- newly copied capability ----> X [not yet marked]

If the collector never revisits A and has no other way to learn about this new edge, it could incorrectly conclude that X is unreachable and recycle its SCT identity.

This is the classic problem of mutating a graph while a reachability collector is traversing it.

## A modern answer is easy to imagine, but should not be substituted for the historical answer

Because PP250 capability operations are architecturally distinguished from ordinary data writes, a modern implementation could maintain a collector invariant whenever a capability is stored or copied.

For example, a capability store into an already-traversed part of the graph could mark or enqueue the referenced SCT entry.

This resembles a write barrier in incremental garbage collectors.

However, this should currently be treated only as a design hypothesis.

The important realization is that **the original PP250 had to have an answer to this problem** if SCT collection occurred while a multiprocessor system continued to create/copy capabilities.

Possible historical answers include, but are not limited to:

* stopping or constraining capability mutation during a collection phase;
* using SCT state bits as part of an interlock/barrier protocol;
* making capability-copy operations participate in marking;
* revisiting changed parts of the graph;
* a different algorithm not yet reconstructed.

We should find the original mechanism before designing a replacement.

## Why lazy collection is attractive

Unreachable SCT entries are not necessarily urgent to reclaim. The physical storage belonging to a dead virtual segment is a separate issue. The SCT identity only becomes scarce when the pool of available SCT entries is under pressure.

This suggests that a modern implementation might be able to collect very lazily:

    normal execution
        |
        +--> capability graph evolves
        |
        +--> collector consumes spare cycles
        |
        +--> accelerate only when SCT free space is low

If the historical mechanism already did something similar, preserving it may be preferable to inventing a new design.

## Relationship to C/C++ temporal safety

If a C allocation is represented as its own data segment:

    allocation -> segment -> SCT identity

then:

* segment bounds provide spatial isolation;
* physical backing may be reclaimed/reassigned independently;
* stale authority remains tied to the old SCT identity;
* that identity must not be recycled until capability reachability permits it.

Thus the SCT garbage collector may provide the key mechanism required for temporal safety in a PP250-derived object model.

This differs structurally from CHERIoT's revocation approach. The comparison should be developed only after the historical SCT collector is understood accurately.

## Research questions

The immediate documentary questions are:

1. What are the exact GC-related fields/bits in an SCT entry?
2. What are their defined transitions during a collection?
3. What constitutes the root set?
4. How are resident capability blocks walked?
5. Exactly how are outformed capability blocks brought back into the traversal?
6. Can collection proceed while processors continue to copy/store capabilities?
7. If so, what prevents a new edge into an already-traversed part of the graph from being missed?
8. What synchronisation exists between processors and the SCT collector?
9. At what point may an SCT entry actually be reused?
10. Which parts of this were hardware, microcode, COS/POS/ROS software, or cooperation between them?

Until those questions are answered, modern incremental-GC ideas should remain clearly labelled as hypotheses rather than descriptions of PP250.
