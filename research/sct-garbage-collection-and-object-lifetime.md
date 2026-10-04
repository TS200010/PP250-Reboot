# SCT garbage collection, object lifetime and lazy traversal

> Status: historical mechanism now substantially documented from England and the 1975-priority Venton/Blench/Sutherland/Hamer-Hodges allocation/deallocation patent family. Inform/Outform belongs to the VM/storage-management path and exact SCT/representation details are generation-specific; neither is treated as a current architectural blocker. Modern extensions remain hypotheses.

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

This is now supported by direct documentary evidence rather than reconstruction alone.

England, *Architectural Features of System 250*, §30(4), states that a background **"garbage collection program"** seeks out and destroys invalid capabilities and blocks severed from the main capability network. Explicit release is a distinct operation: backing-store space is made reusable by invalidating existing capabilities referring to it, using a disk-sector sequence-number mechanism.

The 1975-priority Venton/Blench/Sutherland/Hamer-Hodges allocation/deallocation patent family (GB1548401A / US4121286A) documents the SCT/MCT-side reachability mechanism. SCT entries contain **GARBAGE** and **VISITED** state. During collection, capability-pointer blocks are traversed and the capabilities they contain are loaded. Loading a capability causes the referenced SCT entry's GARBAGE state to be set, thereby recording that the object is live/referenced. VISITED supports traversal of capability-containing blocks so that the capability graph can be walked without treating ordinary data as possible capabilities.

The broad documented model is therefore:

    roots
      |
      +--> capability-pointer block
                 |
                 +-- LC(pointer) --> referenced SCT entry [GARBAGE set]
                 |
                 +-- if referenced object is itself a capability block
                         -> visit/traverse it
                         -> LC its contained capability pointers
                         -> mark their SCT entries

This also supplies the historical answer to an important concurrency question. Normal execution that loads a capability participates in setting the referenced SCT entry's GARBAGE state. The marking operation is therefore integrated with ordinary capability use rather than depending solely on a stop-the-world scan.

An SCT entry that remains unmarked after the collection rules have been satisfied can become eligible for reclamation/reuse. This is the security-critical operation: an SCT identity must not be reassigned while a live capability pointer can still confer authority through that identity.

This separates two kinds of reclamation that should not be conflated:

    backing/physical storage reclamation
        - storage-management and explicit-release concern

    SCT identity reclamation
        - capability reachability concern

### Relationship to LDP

Capability pointers are central to this garbage-collection mechanism, but the evidence found so far does **not** show LDP being used by the collector.

The documented traversal uses **LC** on capability pointers. LC interprets the compact pointer through the SCT and, as part of that machinery, causes the referenced entry to be marked live. The collector therefore does not need to use LDP merely to extract the SCT index into a data register and mark the entry in software.

This is an important negative result:

- capability pointers: **documented as fundamental to GC traversal**;
- LC: **documented as participating in GC marking**;
- LDP: **no documented GC role found so far**.

The purpose of LDP must therefore remain a separate architectural question unless further primary evidence connects it to lifecycle management.

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

## Concurrency: historical mechanism now identified

The patent evidence shows that collection was designed to coexist with ordinary capability use: loading a capability marks its referenced SCT entry. This is the key historical mechanism that prevents a capability actively used during collection from remaining invisible merely because the collector has already passed another part of the graph.

## Modern extensions must remain distinct from the historical mechanism

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

## Research aside: forgery of outform capabilities on backing store

A separate question arises if an attacker can write arbitrary raw data to backing store. The attacker might manufacture a bit pattern resembling an outform capability, including one intended to behave as an Enter Capability associated with malicious code and a fabricated protected environment.

An isolated forged representation is not necessarily useful. To become callable, it must somehow become part of the existing protected capability graph: a legitimate protected structure must acquire a capability edge through which the fabricated object can be reached. Creating or substituting that edge may itself require existing valid authority.

The historical PP250 mechanism needs to be established rather than inferred. In particular:

1. What exactly validates an outform capability when it is informed or otherwise returned to active use?
2. Can arbitrary raw backing-store writes manufacture a representation that Inform would accept as a valid capability?
3. What prevents substitution of one otherwise-valid outform reference for another within an existing capability-bearing structure?
4. Is authenticity derived from persistent object metadata, sequence numbers, the capability graph, storage-manager state, or some combination of these?
5. At what point does a representation read from backing store acquire protected capability status, and what authority is required to make that transition?

This is deliberately left as a research question. The SCT is an active-system structure and should not be assumed to be the ultimate persistent record of capability validity.
