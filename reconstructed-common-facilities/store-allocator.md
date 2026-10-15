# Store Allocator

> **Status:** Initial reconstruction file. Existing research and proposed interfaces have not yet been reconciled into this document.

## Documented facility

**DOCUMENTED OBSERVATION:** D. M. England, *Architectural Features of System 250*, §§35–36 and Figure 11, identifies the **Store Allocator** as one of the seven resource allocators accessible through Enter Capabilities in the Common Facilities Block. Allocates store blocks and returns capabilities to them.

**Primary source:** [England, *Architectural Features of System 250*](../transcriptions/Architectural%20Features%20of%20System%20250%20-%20D%20M%20England.rtf).

## Reconstruction recorded to date

**WORKING RECONSTRUCTION — record of proposals already developed, not a newly completed historical API.** The [COS-derived interface premise](../research/pp250-execution-and-process-model.md) treats COS commands as CLI expressions of underlying resource operations. The command interpreter's symbolic names, directory lookups and terminal presentation are omitted from the proposed machine-level interface. Capabilities are supplied and returned in capability registers; numerical inputs and outputs use data registers. No specific register assignments or error convention has been established.

### COS-derived operations discussed

| Proposed operation | COS evidence | Recorded proposal / boundary |
|---|---|---|
| `ALLOCATE` | `ALO <name> <size>` | Create a storage block and return its capability. England additionally describes an allocation request with length and access in the more developed virtual-store system; that must not silently be read into the COS command syntax. |
| `COPY` | `COP <old> <new> [larger size]` | Create a new block containing a copy and return its capability. |
| `RESIZE` | `COP` with identical old/new names | Change a block's size while retaining its SCT offset/identity, as the Pocket Reference specifies. |
| `FREE_SPACE` | `FST` | Report available storage by store module as numerical information. COS `FST` itself takes no arguments; no underlying calling signature was settled. |

**Related commands, not adopted here as additional ordinary allocator calls:** `GIV` allocates a capability-table entry, and `RSP` creates and stores a capability pointer with a specified access code. Existing protected `GIVE(base, limit)` and `TAKE(index)` proposals concern internal SCT management; retain their definitions and qualifications in [capability genesis and resource lifecycle](../research/capability-genesis-and-resource-lifecycle.md). `ADR` and reverse physical-address lookup were discussed as diagnostics/management facilities, without deciding their inclusion or authority. `BLK`, `REN` and name-prefix `REB` belong to COS symbolic-directory functions, not a capability-register allocator interface.

### Reference lifetime and deletion

The discussion distinguished **discarding a capability reference**, **automatic collection of unreachable objects**, and **explicit release/revocation of a block**. Discarding a reference should not itself invoke or direct garbage collection; dropping the last reachable reference leaves collection to the background mechanism. Under this **working interpretation**, COS `DEL` could remove the named directory reference rather than directly destroy an object; the Pocket Reference says only “Delete a block”, so this interpretation is **not documented COS semantics**.

England, [*Architectural Features of System 250*, §30(4)](../transcriptions/Architectural%20Features%20of%20System%20250%20-%20D%20M%20England.rtf), separately documents **explicit release** making backing-store space reusable by invalidating *all* capabilities to the released block, through a disk-sector sequence-number mechanism. The background collector removes invalid capabilities and unreachable blocks. Explicit release was considered a distinct potential operation, **not settled as an ordinary public Store Allocator entry**. The exact sequence-number representation and release authority were not established. See [SCT garbage collection and object lifetime](../research/sct-garbage-collection-and-object-lifetime.md).

### Other existing proposals retained in their research notes

Restricted construction of Internal Mode capabilities, Dump Stack and SCT capabilities, and initial virgin-object authority belong to the already recorded [capability genesis and resource lifecycle research](../research/capability-genesis-and-resource-lifecycle.md). Bootstrap responsibilities are discussed separately in [boot and processor startup](../research/pp250-boot-and-processor-startup.md). This section records their existence without merging them into the ordinary public API or choosing among implementation hypotheses.

## Existing research to reconcile

- [Capability genesis and resource lifecycle](../research/capability-genesis-and-resource-lifecycle.md) — existing Store Allocator, SCT-management and capability-construction proposals.
- [COS operator commands and process model](../research/pp250-execution-and-process-model.md) — COS-derived interface reconstruction premise.
- [Boot and processor startup](../research/pp250-boot-and-processor-startup.md) — bootstrap versus ordinary runtime responsibilities.
- [SCT garbage collection and object lifetime](../research/sct-garbage-collection-and-object-lifetime.md) — reference lifetime, explicit release and collection.
- [COS command definitions](../transcriptions/System%20250%20Pocket%20Reference%20pg11-pg12.txt) — primary command contracts.

