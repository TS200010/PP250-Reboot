# Store Allocator — API Reference

**Status:** Working reconstruction of the interfaces developed to date. Names and register allocation are proposed, not verified historical entry-point definitions.

The Store Allocator is one of the seven Common Facilities Block resources documented by England. This reference separates **ordinary callable operations**, **private implementation functions**, and **restricted services**. Capability values use capability registers (`C`); numerical values use data registers (`D`). Specific registers and error-return conventions have not been assigned.

## 1. Public interface

Accessible to ordinary authorised callers through the Store Allocator's protected interface.

| Operation | Inputs | Result | Behaviour |
|---|---|---|---|
| `ALLOCATE` | `D: size` (requested access is also part of England's allocator description) | `C: segment capability` | Create a new storage segment and return authority to it. |
| `COPY` | `C: source`; `D: size` if enlargement requested | `C: new segment capability` | Allocate a separate segment containing a copy of the source. |
| `RESIZE` | `C: segment`; `D: new size` | Existing segment identity retained | Adjust segment size without changing its SCT identity. |
| `FREE_SPACE` | No settled argument convention | `D: free-space information` | Report free space in the store modules. |

A caller relinquishes a reference by discarding or replacing its capability. **Garbage collection is automatic**, not an operation exposed by this interface.

## 2. Private Store Allocator functions

**Not externally callable.** These functions belong to the Store Allocator's implementation.

| Function | Inputs | Result | Behaviour |
|---|---|---|---|
| `GIVE` | `base, limit` | SCT entry index | Obtain and populate an available SCT entry for a new segment. |
| `TAKE` | SCT entry index | — | Return an SCT entry to the free list once safe reuse has been established. |

## 3. Restricted services

Protected entry points available only to specifically authorised system components; **not part of the public interface**.

| Service | Authorised consumer | Result / function |
|---|---|---|
| `Create Dump Stack` | Process Manager | Construct the protected Dump Stack resource needed for process creation. |
| `Get SCT Capability` | Authorised SCT-management / garbage-collection code | Return a rights-limited capability to the current SCT. |
| Restricted Internal Mode capability construction | Authorised processor-management services | Create appropriately bounded and rights-limited Internal Mode capabilities. |

Specific call parameters for these restricted services have not been fixed in the existing proposals.

## References

- [COS command definitions](../transcriptions/System%20250%20Pocket%20Reference%20pg11-pg12.txt) — `ALO`, `COP`, `FST` and capability-table operations.
- [England, *Architectural Features of System 250*](../transcriptions/Architectural%20Features%20of%20System%20250%20-%20D%20M%20England.rtf) — Common Facilities Block, resource allocation and storage lifetime.
- [Capability genesis and resource lifecycle](../research/capability-genesis-and-resource-lifecycle.md) — private `GIVE`/`TAKE` and restricted-service proposals.
- [COS-derived API premise and process model](../research/pp250-execution-and-process-model.md).
- [SCT garbage collection and object lifetime](../research/sct-garbage-collection-and-object-lifetime.md).
