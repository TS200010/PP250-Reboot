# Store Allocator

> **Status:** Initial reconstruction file. Existing research and proposed interfaces have not yet been reconciled into this document.

## Documented facility

**DOCUMENTED OBSERVATION:** D. M. England, *Architectural Features of System 250*, §§35–36 and Figure 11, identifies the **Store Allocator** as one of the seven resource allocators accessible through Enter Capabilities in the Common Facilities Block. Allocates store blocks and returns capabilities to them.

**Primary source:** [England, *Architectural Features of System 250*](../transcriptions/Architectural%20Features%20of%20System%20250%20-%20D%20M%20England.rtf).

## Reconstruction

No facility-specific calling convention, complete operation set or internal implementation is asserted here yet. Consolidate the relevant existing repository findings before specifying them; distinguish documented behaviour from working reconstruction.

## Existing research to reconcile

- [Capability genesis and resource lifecycle](../research/capability-genesis-and-resource-lifecycle.md) — existing Store Allocator, SCT-management and capability-construction proposals.
- [COS operator commands and process model](../research/pp250-execution-and-process-model.md) — COS-derived interface reconstruction premise.
- [Boot and processor startup](../research/pp250-boot-and-processor-startup.md) — bootstrap versus ordinary runtime responsibilities.
- [SCT garbage collection and object lifetime](../research/sct-garbage-collection-and-object-lifetime.md) — reference lifetime, explicit release and collection.
- [COS command definitions](../transcriptions/System%20250%20Pocket%20Reference%20pg11-pg12.txt) — primary command contracts.

