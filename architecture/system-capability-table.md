# System 250 Architecture — System Capability Table

## System Capability Table (SCT)

### Role

The SCT is the system indirection structure used when an active stored capability is expanded into a capability register. A stored active capability carries an SCT reference/index rather than a raw physical base address. The processor uses the SCT reference relative to the special SCT capability register `C(C)` / `C12` to obtain the physical segment bounds.

This gives a useful separation:

```text
stored active capability
    = capability form/type + access rights + SCT identity/reference

SCT entry
    = physical realisation and current state of that segment identity

loaded capability register
    = physical base/bounds + access authority,
      or a distinguished unusable/trap representation
```

Consequently, relocating a segment need not require rewriting every stored capability that designates it: its SCT identity can remain stable while the SCT entry is changed.

### Entry structure

A normal SCT entry occupies **three 24-bit words**:

| Entry word | Contents |
|---:|---|
| 0 | Sum-check / validity word |
| 1 | Base |
| 2 | Limit/extent plus additional access/spare-bit capacity used by later system mechanisms |

Halton states that capability loading uses the System Capability Table and that the table contains a sum-check formed from the base and limit values. The patent material describes the corresponding three-word descriptor access and validation sequence.

Later Plessey store-allocation/deallocation patent material establishes that the third SCT word has spare capacity in its access-code portion. Two such bits are used by the described garbage-collection mechanism as **GARBAGE** and **VISITED** bits. This is concrete primary-source evidence for additional SCT flag/state bits and is likely the basis of later descriptions referring to special SCT flags.

The ordinary capability access rights are **not supplied by the SCT entry**. They originate in the stored capability and are combined with the base/limit information obtained through the SCT to form the loaded capability-register representation.

### Sum-check, relocation and unavailable segments

The sum-check protects the integrity of the base/limit descriptor during capability loading. Patent material describes zeroing the check word as a mechanism for making an SCT entry temporarily unavailable while its descriptor is being changed, including relocation.

The interrupt patent further establishes an important distinction: encountering such an unavailable SCT state during capability loading need not immediately execute the whole software recovery action. The capability-loading machinery can establish a distinguished **unusable/trap representation** in the destination capability register. Hardware detects that state when the capability register is subsequently used and enters the normal interrupt/change-process machinery.

Thus the mechanism is approximately:

```text
stored active capability (11)
        |
        v
      LC / SCT lookup
        |
        +-- valid descriptor --> normal expanded C register
        |
        +-- unavailable state --> distinguished unusable C-register state
                                      |
                                      v
                              attempted use of C register
                                      |
                                      v
                              hardware trap detection
                                      |
                                      v
                                   C(N)
                                      |
                                      v
                           Normal Interrupt Block
                                      |
                                      v
                         target Dump Stack capability
                                      |
                                      v
                              automatic CHP
                                      |
                                      v
                         Normal Interrupt process
```

This is more precise than saying simply that "LC page-faults": capability loading establishes the protected state that causes the later attempted use to trap.

### Active versus passive representation

Do not equate "segment is not currently resident" with "every reference to it has type `10`." An active (`11`) reference identifies an SCT entry and can remain meaningful while the SCT entry is unavailable. A passive (`10`) representation instead carries backing-store identity/address information and is used when authority itself has been converted to its backing-store/outform representation.

When capability-containing blocks are moved between main store and backing store, embedded capability representations may therefore require inform/outform conversion. The SCT identity mechanism allows active references elsewhere in main store to remain stable across relocation/page movement.

## Segment sharing and virtual store: source context

**Documented, [EP-E1], paragraphs 19–20:** the **System Capability Table (SCT)** holds the physical base/limit information for store blocks. A special processor capability identifies that table. Programs traverse capability structures without supplying physical addresses for the referenced segments.

```text
process's capability environment
          |
          v
stored capability --SCT reference--> SCT entry
          |                              |
       rights                         base / limit
          +---------------+--------------+
                          v
                 expanded capability
                          |
                checked segment access
                          v
                 shared Store Module
```

Multiple processes can possess different rights to the same segment [EP-P4, “Description of Prior Art”]. A shared table does not mean universal authority: the reachable capability network and each capability's rights constrain a process's accesses.

**Documented, [EP-E1], paragraph 30:** virtual store extends the capability structure to disk. The store-management package moves blocks between backing store and main store, and an attempted access can trigger bringing a block into main store. A main-store capability still needs an SCT entry when its target block exists only on disk. On-disk capabilities replace the SCT offset with disk identity/address information; moving a capability block entails converting its contained capabilities.

**SECONDARY EVIDENCE, [EP-L1], sections 4.3–4.5:** the SCT is shared by processors, with synchronization required during updates. Primary-memory capabilities are called inform/active, and disk capabilities outform/passive; the latter identify the disk object rather than its current SCT slot. LC retains the SCT index in the Process Dump Stack, and SC combines that identity with register rights to reconstruct the stored capability.

Thus PP250 virtual memory is **segment/capability oriented**, not a conventional separate flat paged address space for each process. Sharing follows segment identity and authority. The disk representation is documented at this conceptual level; exact outform fields, conversion procedures and the applicability of each OS's implementation remain **UNKNOWN**. This does not establish an INFORM or OUTFORM machine instruction, a page-table mechanism, or a disk-based cold loader.

**Strong inference:** a common SCT supports processor-independent segment identity. Updating an SCT entry must also account for any descriptors already expanded in running processors; changing the table alone must not be assumed sufficient for coherent relocation. The exact synchronization protocol for the target machine remains to be established.

### Sources for the relocated material

Source identifiers prefixed `EP-` retain the provenance and verification limits of the execution/process note; this reorganisation does not constitute a new source verification.

- **[EP-E1] PRIMARY EVIDENCE:** D. M. England, *Architectural Features of System 250* (1972), [repository paper](../sources/1972/1972-England-Architectural-Features-of-System-250.pdf). Paragraphs 4–9: topology; 16–20: rights, integrity and SCT; 21–26: domains/CALL/Dump Stack; 30: virtual store and disk capabilities; 31: process management and CPU independence. PDF text inspected; diagram bit boundaries not treated as verified from extraction.
- **[EP-P4] PRIMARY EVIDENCE:** US 4,408,274, *Memory protection system using capability registers*, [repository PDF](../patents/US4408274-memory-protection-capability-registers.pdf), [patent text](https://patents.google.com/patent/US4408274A/en). Locators: Description of Prior Art, Capability Formats, LC/load-on-use and SC descriptions, Figures 5–9 references. Text read; enhanced pointer format and propagation controls are version-qualified.
- **[EP-L1] SECONDARY EVIDENCE:** Henry M. Levy, *Capability-Based Computer Systems*, Digital Press, 1984, [chapter 4, “The Plessey System 250”](https://homes.cs.washington.edu/~levy/capabook/Chapter4.pdf), especially sections 4.2–4.5 and Figure 4-1. Read as corroboration and for terminology; it does not override the pocket reference or resolve version conflicts.
