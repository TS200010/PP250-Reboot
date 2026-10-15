# Store Allocator

> **Status:** Initial reconstruction file. Existing research and proposed interfaces have not yet been reconciled into this document.

## Documented facility

**DOCUMENTED OBSERVATION:** D. M. England, *Architectural Features of System 250*, §§35–36 and Figure 11, identifies the **Store Allocator** as one of the seven resource allocators accessible through Enter Capabilities in the Common Facilities Block. Allocates store blocks and returns capabilities to them.

**Primary source:** [England, *Architectural Features of System 250*](../transcriptions/Architectural%20Features%20of%20System%20250%20-%20D%20M%20England.rtf).

## Reconstruction recorded to date

**WORKING RECONSTRUCTION — record of proposals already developed, not a newly completed historical API.** The [COS-derived interface premise](../research/pp250-execution-and-process-model.md) treats COS commands as CLI expressions of underlying resource operations. The command interpreter's symbolic names, directory lookups and terminal presentation are omitted from the proposed machine-level interface. Capabilities are supplied and returned in capability registers; numerical inputs and outputs use data registers. No specific register assignments or error convention has been established.

### Three distinct interfaces in the existing reconstruction

**WORKING RECONSTRUCTION — three categories of functions, not one flat public API.** The Store Allocator work already distinguished:

1. **External/public user API.** Normal protected operations available to ordinary callers, with capabilities and numerical values passed/returned in capability and data registers respectively. The COS-derived `ALLOCATE`, `COPY`, `RESIZE` and `FREE_SPACE` proposals belong here. `DELETE` was an initial candidate but was revised following the distinction between relinquishing a reference, garbage collection and explicit object release. `ADDRESS` and `FIND_BLOCK` were explored from COS diagnostic commands, **not** established as unrestricted public functions.
2. **Private functions internal to the Store Allocator.** The previously proposed `GIVE(base, limit) -> SCT index` and `TAKE(index)` manage SCT slots and safe reuse as part of the Store Allocator's implementation. **They are not exposed as an API or independently callable service.** The earlier research note also explored a protected SCT Manager entry interface; that exploratory proposal is not adopted in this reconstruction. Its historical reasoning and free-list proposal remain preserved in [capability genesis and resource lifecycle](../research/capability-genesis-and-resource-lifecycle.md#12-october-2026--sct-manager-interface-and-free-list-reconstruction).
3. **Restricted special-purpose services.** The already proposed **Create Dump Stack** service is for authorised process construction, in particular the Process Manager—not ordinary allocation callers. The separate **Get SCT Capability** service was proposed for authorised SCT-management/garbage-collection code, returning a suitably restricted capability to the current SCT. These services are not general permission to manufacture or recover arbitrary capabilities, and their exact historical mechanisms are not established. See [restricted SCT capability service](../research/capability-genesis-and-resource-lifecycle.md#10-october-2026--restricted-sct-capability-service-and-allocator-initialisation) and the earlier Dump Stack construction discussion in that research.

The distinction is between **public operations**, **private implementation functions**, and **restricted services for specifically authorised callers**. **Do not expose internal SCT functions or Process Manager-only construction services through the ordinary caller API.** The following table preserves the earlier *exploratory list of calls*, including ones subsequently assigned elsewhere or left undecided; it is not a declaration that all entries are public.

### Original callable API proposal (preserved in full)

The following is the **initial proposed interface**, recorded before the subsequent discussion of garbage collection and explicit release. It is a **WORKING RECONSTRUCTION**, not a verified historical calling convention. Capability operands/results use capability registers (`C`); numerical operands/results use data registers (`D`). The names below designate proposed calls, not demonstrated software entry-point names.

| Proposed call | Inputs | Outputs / proposed effect | Later status |
|---|---|---|---|
| `ALLOCATE` | `D: size`; access requirements were also considered | `C: new segment capability` | Retained candidate. COS `ALO` documents size, not an access argument; England separately describes requested access. |
| `COPY` | `C: source segment`; `D: new size` when requested | `C: capability to separately allocated copy` | Retained candidate; COS optional size applies when enlarging. |
| `RESIZE` | `C: existing segment`; `D: new size` | Same segment identity / SCT offset after size adjustment | Retained candidate; no detailed movement/coherence mechanism selected. |
| `DELETE` | `C: segment` | Initially proposed to delete the block and return status (`D`) | **Revised**: ordinary reference discard need not call an allocator; do not interpret this as automatic destruction or immediate SCT reuse. |
| `FREE_SPACE` | Initially considered `D: store module number` | `D: available space` | Signature **undecided**: COS `FST` actually takes no operands and reports all store modules. |
| `ADDRESS` | `C: segment`, `D: offset` | `D: absolute physical address` | Considered from COS `ADR`; not accepted as an ordinary public operation. |
| `FIND_BLOCK` | `D: absolute physical address` | `C: corresponding segment capability`, `D: offset` | Considered from reverse COS `ADR`/`NAM`; authority to return a capability was **not established**. |
| `GIV` / explicit SCT entry creation | `D: base`, `D: size` | Initially contemplated creation/return of an SCT-backed capability or entry | Treated as potentially restricted/internal, not adopted as an ordinary allocator operation. Distinct from the existing internal `GIVE(base, limit)` proposal. |

These are the operations *actually considered*, including the ones subsequently questioned. `ADDRESS` and `FIND_BLOCK` expose a physical-address diagnostic/management concern; reverse lookup must not be presumed to authorise arbitrary capability creation. The initial `DELETE` proposal was reconsidered in favour of reference relinquishment plus independent garbage collection; England's **explicit object release**, which invalidates all extant capabilities, remains distinct and was not assigned a settled public API signature.

### COS-derived operations discussed

| Proposed operation | COS evidence | Recorded proposal / boundary |
|---|---|---|
| `ALLOCATE` | `ALO <name> <size>` | Create a storage block and return its capability. England additionally describes an allocation request with length and access in the more developed virtual-store system; that must not silently be read into the COS command syntax. |
| `COPY` | `COP <old> <new> [larger size]` | Create a new block containing a copy and return its capability. |
| `RESIZE` | `COP` with identical old/new names | Change a block's size while retaining its SCT offset/identity, as the Pocket Reference specifies. |
| `FREE_SPACE` | `FST` | Report available storage by store module as numerical information. COS `FST` itself takes no arguments; no underlying calling signature was settled. |

**Related commands, not adopted here as additional ordinary allocator calls:** `GIV` allocates a capability-table entry, and `RSP` creates and stores a capability pointer with a specified access code. Existing `GIVE(base, limit)` and `TAKE(index)` proposals are private Store Allocator functions for SCT management, not externally exposed calls; retain their definitions and qualifications in [capability genesis and resource lifecycle](../research/capability-genesis-and-resource-lifecycle.md). `ADR` and reverse physical-address lookup were discussed as diagnostics/management facilities, without deciding their inclusion or authority. `BLK`, `REN` and name-prefix `REB` belong to COS symbolic-directory functions, not a capability-register allocator interface.

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

