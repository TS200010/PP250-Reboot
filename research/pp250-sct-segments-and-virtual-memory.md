# PP250 SCT, segments and virtual memory

Status: research reconstruction, 27 September 2026. Material relocated from [PP250 execution and process model](pp250-execution-and-process-model.md). Evidence labels, source qualifications and unresolved questions are retained; this note does not amend the architecture specification.

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
