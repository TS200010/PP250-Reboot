# Reconstructed System 250 Common Facilities

## Purpose

This directory is the navigation point for reconstruction of the System 250 **Common Facilities Block** and the individual resource-allocation facilities accessible through it. It is organised by the historically documented facilities, not by a newly invented API classification.

**Primary evidence:** D. M. England, *Architectural Features of System 250*, §§35–36, Figure 11: [repository transcription](../transcriptions/Architectural%20Features%20of%20System%20250%20-%20D%20M%20England.rtf). Figure 11 shows a read capability to the Common Facilities Block, which contains Enter Capabilities to seven resource allocators. England describes each allocation subroutine as returning a capability to the created resource.

## Facilities — Figure 11

The following facility documents follow England's Figure 11. They are initial reconstruction files; their detailed interfaces have not yet been consolidated.

| Document | Documented facility | Status |
|---|---|---|
| [store-allocator.md](store-allocator.md) | Store Allocator | Created; reconstruction to be consolidated |
| [process-allocator.md](process-allocator.md) | Process Allocator | Created; reconstruction not yet consolidated |
| [flag-allocator.md](flag-allocator.md) | Flag Allocator | Created; reconstruction not yet consolidated |
| [stream-allocator.md](stream-allocator.md) | Stream Allocator | Created; reconstruction not yet consolidated |
| [textfile-allocator.md](textfile-allocator.md) | Textfile Allocator | Created; reconstruction not yet consolidated |
| [directory-allocator.md](directory-allocator.md) | Directory Allocator | Created; reconstruction not yet consolidated |
| [job-allocator.md](job-allocator.md) | Job Allocator | Created; reconstruction not yet consolidated |

## Existing research to reconcile

- [Capability genesis and resource lifecycle](../research/capability-genesis-and-resource-lifecycle.md) — existing Store Allocator, SCT-management and capability-construction proposals.
- [COS operator commands and process model](../research/pp250-execution-and-process-model.md) — COS-derived interface reconstruction premise.
- [Boot and processor startup](../research/pp250-boot-and-processor-startup.md) — bootstrap versus ordinary runtime responsibilities.
- [SCT garbage collection and object lifetime](../research/sct-garbage-collection-and-object-lifetime.md) — reference lifetime, explicit release and collection.
- [COS command definitions](../transcriptions/System%20250%20Pocket%20Reference%20pg11-pg12.txt) — primary command contracts.

