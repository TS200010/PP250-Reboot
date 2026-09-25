# PP250 / System 250 Patent Audit

**Status:** active research inventory  
**Created:** 25 September 2026  
**Purpose:** identify the patent corpus relevant to reconstruction of the Plessey PP250 / System 250, including patents that explicitly describe PP250/System 250, closely related Plessey capability/fault/multiprocessor patents, later developments of the architecture, and patents by principal designers that may expose otherwise undocumented implementation details.

## Evidence and inclusion policy

This inventory is deliberately broader than a search for patents containing the words "System 250" or "PP250". Contemporary patents often describe the architecture generically as a data-processing system. Searches therefore include patent-family relationships, citations, Plessey assignee records, explicit PP250/System 250 descriptions, and named designers.

Entries are classified as:

- **A — Direct PP250/System 250:** explicitly identifies PP250/System 250 or is strongly established as describing the contemporary machine.
- **B — PP250 architectural lineage:** Plessey capability/process/fault/store architecture closely connected with System 250, even where the patent does not explicitly use the product name.
- **C — Later System 250 development:** later Plessey work explicitly building on the System 250 architecture.
- **D — Designer-related lead:** patent by a System 250 designer which may provide relevant context but is not currently established as part of PP250.

A patent being cited by a PP250 patent is **not** sufficient for inclusion as a PP250 invention. Patent families are recorded so that GB/US/other publications of the same invention are not accidentally counted as separate inventions.

## Repository holdings at start of audit

The repository initially contained four patent PDFs:

- `US3787813A-Data-Processing-Devices-Using-Capability-Registers.pdf`
- `US4383297-internal-register-addressing.pdf`
- `US4408274-memory-protection-capability-registers.pdf`
- `US4486831-process-suspension.pdf`

The audit establishes that this is substantially incomplete.

## Core original PP250 / System 250 patent families

| Class | Priority | GB publication / family | US publication | Inventors | Subject | Repo PDF | Reconstruction relevance |
|---|---|---|---|---|---|---|---|
| **A** | 1969-01-02 | family of GB 193/69 | **US3657736A** | Roger J. Boom; John M. Cotton; Martin J. Goodier; David C. Cosserat | Method of assembling subroutines; distributed-algorithm multiprocessor communication using input/output wells and queues | No | Early architectural ancestry; useful for evolution into System 250 |
| **A/B** | 1970-05-26 | **GB1329721A** | **US3787813A** | Alun J. Cole; David C. Cosserat; John M. Cotton | Data processing devices; capability registers and memory protection | **Yes** | Fundamental capability architecture |
| **A** | 1970-09-02 | family of GB 41951/70 | **US3757307A** | David C. Cosserat; John M. Cotton; Michael O'Halloran; Frederick M. Trapnell | Program interrupt facilities in modular data-processing systems | No | Interrupt demand/masking, timers and handler entry; highly relevant to M and indicators |
| **A** | 1971-03-04 | **GB1344474A** | **US3814919A** | Charles S. Repton; Peter C. Venton; Kenneth J. Hamer-Hodges | Fault detection and isolation | No | **Explicit PP250**; fault checkout, special capability pointer/table, fault containment; essential to M reconstruction |
| **A** | 1971-06-24 | **GB1394431A** | **US3787818A** | John S. Arnold; George M. Beck; Roger J. Boom; David C. Cosserat; Kenneth J. Hamer-Hodges; Michael O'Halloran; Theodor D. Sandeman; Roger M. Williams | Multi-processor data processing system | No | Core processor/store/peripheral/bus topology and fault-tolerant modular system architecture |
| **A** | 1972-01-26 | **GB1410631A** | **US3771146A** | John M. Cotton; James J. L. Williams; David C. Cosserat | Data processing system interrupt arrangements | No | Capability-register restoration, characteristic codes and process suspension/interrupt entry; essential to M/H/T boundary |
| **A** | 1972-06-03 | **GB1422952A** | **US3879712A** | Gordon Edge; George Worthington | Data processing system fault diagnostic arrangements | No | **Explicit PP250**; diagnostic interface, microbits, microprogram stop/monitor facilities; unusually valuable evidence about processor implementation |

### Notes on the two interrupt patents

`US3757307A` and `US3771146A` are distinct inventions and should not be collapsed into one family.

- `US3757307A` describes system interrupt-demand bits, masks, processor interrogation, internal timer demands and transition to an interrupt handler.
- `US3771146A / GB1410631A` describes capability-register characteristic codes and capability restoration arrangements which provoke suspension/interrupt handling when the characteristic code is encountered.

Both are potentially relevant to reconstructing the relationship between M, machine indicators and H/T software.

## Later patents explicitly tied to System 250

| Class | Priority | GB/family | US publication | Inventors | Subject | Repo PDF | Relevance |
|---|---|---|---|---|---|---|---|
| **A/C** | 1975-05-01 | family of GB18129/75 | **US4050059A** | John Lloyd Williams; Roger J. Leaman; Robert V. Moberly; Geoffrey B. K. Stagg; Graham J. Wisdom | Data processing read-and-hold facility | No | Processor bus atomic/read-hold operation; MCT modification; parity failure automatically enters fault interrupt |
| **C** | 1975-10-08 | **GB1548401A** | **US4121286A** | Peter C. Venton; Michael Blench; Anthony K. Sutherland; Kenneth J. Hamer-Hodges | Memory-space allocation/deallocation | No | MCT/SCT capability lifecycle, garbage/visited semantics, stale capability-pointer handling |
| **A/C** | c. 1975 | family to US filing 1976-05-17 | **US4041460A** | Graham J. Wisdom; Peter V. Creteau; John Lloyd Williams | Multi-processor data-processing-system peripheral-equipment access units | No | Explicitly states that each System 250 store/peripheral connects to processor buses through a common-form access unit |
| **C** | 1979-09-29 | GB7933857 family / EP0026590 | **US4408274A** | Nigel J. Wheatley; Martyn P. Andrews | Improved memory protection using capability registers | **Yes** | Later capability classes, load-on-use, propagation control, access reduction; must not be silently back-projected onto original PP250 |
| **C** | 1979-09-29 | GB7933856 family / EP0026589 | **US4486831A** | Nigel J. Wheatley; Martyn P. Andrews | Multi-programming process suspension | **Yes** | Explicit System 250 Dump Stack/process-suspension development; highly relevant comparative evidence for CHP |
| **C** | later Plessey family | — | **US4383297** | see patent PDF | Internal-register addressing | **Yes** | Relevant to special/internal processor registers; version must be kept explicit |

## Additional diagnostic lineage

A further family, **US4066883A / GB1563288A**, concerns a test vehicle for selectively inserting diagnostic signals into a bus-connected data-processing system. It cites the PP250 diagnostic patent `US3879712A / GB1422952A`. It is a useful follow-up lead but has not yet been classified as direct PP250 architecture.

## Principal designer search set

The audit uses the contemporary System 250 literature and the inventor network exposed by the patents to search at least the following names and spelling variants:

- David Cockburn Cosserat / Coserat
- John Michael Cotton
- Kenneth James Hamer-Hodges / Hamer Hodges / Hodges
- Charles Samuel Repton
- Peter Charles Venton
- Roger John Boom
- John Spencer Arnold
- George Morland Beck
- Michael O'Halloran
- Theodor Duncan Sandeman
- Roger Morley Williams
- James Jeffrey Llewelyn Williams
- Alun John Cole
- Frederick Mackay Trapnell
- Gordon Edge
- George Worthington
- Roger John Leaman
- Geoffrey Brian Kenneth Stagg
- Graham John Wisdom
- John Lloyd Williams
- Peter Verne Creteau
- Robert Valentine Moberly
- Michael Blench
- Anthony Keith Sutherland
- Nigel J. Wheatley
- Martyn P. Andrews

Published System 250 authors including D. M. England, D. Halton and W. A. C. Hemmings are also search targets even where they have not yet been found as inventors on a relevant patent.

## Designer-related patents not presently treated as PP250

Inventor searches deliberately produce false positives and later career work. These should be retained as leads rather than folded into the architecture.

Examples:

- **US3680053A**, Cotton and Peter Anthony Lloyd, *Data transmission systems* — ring/highway arbitration. Plessey, contemporary, but not yet established as PP250.
- Later Cotton/Hamer-Hodges telecommunications switching patents from their subsequent work — historically interesting but not evidence for original PP250 unless a concrete architectural connection is established.

## Acquisition priority

### Priority 1 — essential processor/M reconstruction

1. **US3814919A / GB1344474A** — fault detection and isolation
2. **US3771146A / GB1410631A** — capability-based interrupt arrangements
3. **US3757307A** — program interrupt facilities
4. **US3787818A / GB1394431A** — multiprocessor System 250 architecture
5. **US3879712A / GB1422952A** — PP250 diagnostic/microprogram interface

### Priority 2 — capability/store/bus completeness

6. **US4121286A / GB1548401A** — allocation/deallocation and capability lifecycle
7. **US4050059A** — read-and-hold/MCT atomicity/fault response
8. **US4041460A** — System 250 access units and processor buses

### Priority 3 — ancestry and related diagnostic material

9. **US3657736A** — early distributed-algorithm architecture
10. **US4066883A / GB1563288A** — later diagnostic test vehicle; classify after review

## Immediate implications for the M/H/T reconstruction

The missing corpus is not peripheral to the current emulator work. Several absent patents address exactly the mechanisms currently unresolved:

- fault entry and restricted authority;
- the special capability pointer/table;
- interrupt demand and masking indicators;
- timers and internal interrupt indications;
- process suspension and restoration;
- automatic entry into fault-interrupt machinery;
- microbits and the physical microprogram diagnostic interface;
- MCT/SCT manipulation and atomicity.

Accordingly, claims about M state, SPECIAL, MIP/MIS indicators, fault indicators, automatic CHP and the M→H/T handoff should be checked against this patent set before the emulator state vector is frozen.

## Known corrections to earlier informal patent notes

- **US4001813** is not a PP250 patent; it is unrelated and must not be used as System 250 evidence.
- `GB1410631A` is the British publication; its US family member is **US3771146A**. There is no corresponding "US 1,410,631".
- `US3757307A` and `US3771146A` are separate interrupt inventions.
- Patents must be tracked by **family**, not by isolated publication number, to avoid duplicate counting and mistaken lineage.

## Audit method and completeness

Search routes used include:

1. explicit `PP250` and `System 250` text;
2. Plessey assignee + architecture keywords;
3. known patent-family members and foreign priority records;
4. forward/backward citation chains from established System 250 patents;
5. inventor-name searches for principal System 250 designers;
6. later patents that explicitly cite System 250 papers or the foundational GB patents.

This file is intended to be the repository's living patent inventory. "All patents by a designer" is not synonymous with "all PP250 patents": unrelated work should remain in the designer-lead section unless evidence connects it to System 250.

## Source anchors

Principal public records used in this audit include Google Patents family records, USPTO-derived patent text, Justia patent records, the 1973 MIT bibliography of System 250 papers, and Henry Levy's historical treatment of System 250. The patent PDFs themselves should become the preferred primary evidence once acquired into this repository.
