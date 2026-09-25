# PP250 / System 250 Patent Audit

**Status:** completed systematic audit; maintain as a living inventory when new primary evidence appears  
**Audit date:** 25 September 2026  
**Project:** PP250-Reboot

## Purpose

This document identifies the patent corpus relevant to reconstruction of Plessey PP250 / System 250. The audit deliberately goes beyond patents whose titles contain `PP250` or `System 250`: the contemporary patents usually describe the machine generically as a data-processing system.

The search covered:

1. explicit `PP250` and `System 250` patent text;
2. the known capability, fault, interrupt, multiprocessor, store and peripheral patents and their patent families;
3. Plessey assignee records around the System 250 period;
4. backward and forward patent-family citations;
5. the principal designers named in the patents and the contemporary System 250 bibliography;
6. later Plessey capability-machine patents which explicitly build on the earlier architecture;
7. patents by the principal designers outside the PP250 corpus, to distinguish relevant lineage from unrelated later work.

The audit is organised by **invention family**, not by publication number. GB, US, German, Canadian and other publications of the same invention are not separate inventions.

## Classification

- **A — Direct PP250/System 250:** explicitly identifies PP250/System 250, or is established by subject, date, inventors and family context as a contemporary System 250 mechanism.
- **B — Immediate architectural ancestry/adjacency:** contemporary Plessey work by the design group which informs the route into System 250 but is not asserted to be part of the delivered PP250 processor.
- **C — Later Plessey capability/System-250 development:** subsequent architecture that cites or explicitly develops the System 250 mechanisms. Useful evidence, but must not be projected backwards without corroboration.
- **D — Designer patent outside the PP250 lineage:** retained only to prevent inventor-name searches from being mistaken for architectural evidence.

## Repository holdings before this audit

The `patents/` directory contained four patent PDFs:

- `US3787813A-Data-Processing-Devices-Using-Capability-Registers.pdf`
- `US4383297-internal-register-addressing.pdf`
- `US4408274-memory-protection-capability-registers.pdf`
- `US4486831-process-suspension.pdf`

Only one of those four (`US3787813A`) belongs to the original 1970–72 architectural nucleus. The other three are later developments.

---

# 1. Original PP250 / System 250 architectural corpus

These are the patent families that should form the primary patent collection for reconstruction of the original machine.

| Class | Priority | Principal GB publication/family | US publication | Inventors | Mechanism | Repo PDF? |
|---|---:|---|---|---|---|---|
| **A** | 1970-05-26 | **GB1329721A** | **US3787813A** | Alun J. Cole; David C. Cosserat; John M. Cotton | Data processing devices; capability registers, segment descriptors, base/limit/access protection | **YES** |
| **A** | 1970-09-02 | GB application 41951/70 family | **US3757307A** | David C. Cosserat; John M. Cotton; Michael O'Halloran; Frederick M. Trapnell | Program interrupt facilities; demand bits, masks, periodic interrogation and interrupt-handler entry | **NO** |
| **A** | 1971-03-04 | **GB1344474A** | **US3814919A** | Charles S. Repton; Peter C. Venton; Kenneth J. Hamer-Hodges | Fault detection/handling and isolation; PP250 checkout and restricted fault authority | **NO** |
| **A** | 1971-06-24 | **GB1394431A** | **US3787818A** | John S. Arnold; George M. Beck; Roger J. Boom; David C. Cosserat; Kenneth J. Hamer-Hodges; Michael O'Halloran; Theodor D. Sandeman; Roger M. Williams | Multiprocessor System 250 structure: processors, stores, channels/buses and modular access | **NO** |
| **A** | 1972-01-26 | **GB1410631A** | **US3771146A** | John M. Cotton; James J. L. Williams; David C. Cosserat | Capability-based interrupt arrangements; characteristic codes, capability restoration and process suspension | **NO** |
| **A** | 1972-06-03 | **GB1422952A** | **US3879712A** | Gordon Edge; George Worthington | PP250 fault diagnostic arrangements; microprogram/microbit diagnostic access | **NO** |

## Why the two interrupt patents are separate

`US3757307A` and `US3771146A` are not duplicate publications of one invention.

- **US3757307A** provides the broader modular-system interrupt-demand mechanism: shared demand bits, masks/interrogation and entry into an interrupt handler.
- **US3771146A / GB1410631A** provides a capability-register mechanism in which a discrete characteristic code restored into a capability register is detected when that register is used and causes interrupt/process-suspension action.

Both therefore matter to reconstruction of M, but at different levels.

## Family confirmation

The capability-register family is represented in Germany by `DE2126206...`; the interrupt-capability family by `DE2303596...`; and the multiprocessor family by `DE2230830...`. The Canadian multiprocessor family includes `CA958489A`. These are family members, not additional inventions.

---

# 2. Immediate architectural ancestry

| Class | Priority | US publication | Inventors | Subject | Assessment |
|---|---:|---|---|---|---|
| **B** | 1969-01-02 | **US3657736A** | Roger J. Boom; John M. Cotton; Martin J. Goodier; David C. Cosserat | Method of assembling subroutines; asynchronous distributed-algorithm routines using input/output wells and queues | Important pre-System-250 Plessey design-group work. Preserve as ancestry, but do not claim it is a PP250 processor patent. |
| **B/D** | contemporary | **US3680053A** | John M. Cotton; Peter A. Lloyd | Data-transmission ring/highway arbitration | Contemporary Plessey communications work. No sufficient evidence found that it is a PP250 mechanism; retain as a designer lead only. |

The first of these is especially useful historically because it demonstrates that Boom, Cotton and Cosserat were already working on modular/distributed real-time processing before the capability patent family.

---

# 3. Later System 250 / capability-machine developments

These patents are strongly relevant to the reconstructed lineage, but their mechanisms must be dated explicitly. They cannot automatically be treated as properties of the 1970–72 PP250.

| Class | Priority | GB/family | US publication | Inventors | Mechanism | Repo PDF? |
|---|---:|---|---|---|---|---|
| **C/A** | 1975-05-01 | **GB1536853A** | **US4050059A** | John Lloyd Williams; Roger J. Leaman; Robert V. Moberly; Geoffrey B. K. Stagg; Graham J. Wisdom | Read-and-hold; atomic modification of shared data including MCT entries; parity-verified hold and automatic fault entry | **NO** |
| **C/A** | c. 1975 | family of US filing 1976-05-17 | **US4041460A** | Graham J. Wisdom; Peter V. Creteau; John Lloyd Williams | Peripheral/store access units; patent text explicitly says each store/peripheral in **System 250** connects to processor buses through a common-form access unit | **NO** |
| **C** | 1975-10-08 | **GB1548401A** | **US4121286A** | Peter C. Venton; Michael Blench; Anthony K. Sutherland; Kenneth J. Hamer-Hodges | MCT/capability lifecycle, garbage/visited indicators, cancellation of extant capability pointers and safe MCT-entry reuse | **NO** |
| **C** | 1979-09-29 | GB7933857 / EP0026590 family | **US4408274A** | Nigel J. Wheatley; Martyn P. Andrews | Later memory protection/capability-register architecture, including later capability semantics | **YES** |
| **C** | 1979-09-29 | GB7933856 / EP0026589 family | **US4486831A** | Nigel J. Wheatley; Martyn P. Andrews | Process suspension, Dump Stack and later processor state; extremely useful comparative CHP evidence | **YES** |
| **C** | 1979-era family | — | **US4383297A** | Nigel J. Wheatley; Martyn P. Andrews | Internal-register addressing in the later multiprocessor architecture | **YES** |

### Important evidence in the later process-suspension patent

`US4486831A` explicitly describes special capability registers, process Dump Stack state and four indicator registers (Primary, Fault, Test and Historical), and says that the Primary Indicator Register is changed by CHANGE PROCESS, with the old value saved in the suspended process Dump Stack and the new value loaded from the activated process Dump Stack. This is excellent evidence for the later architecture, but it must be compared with the original Pocket Reference before assigning those exact semantics to early PP250.

---

# 4. Other Plessey data-processing families found during the sweep

These are genuine Plessey computing patents encountered through the assignee/citation search. They should be reviewed, but the audit does **not** currently classify them as original PP250 processor mechanisms merely because they are nearby in the citation graph.

- **GB1523005A / US3999052A — Data processing apparatus / upper-bounds address checking.** Priority 1975-06-18. It appears in the citation neighbourhood of `GB1536853A`; treat as a later general data-processing protection family pending architectural review.
- A later data-processing family published as **GB1542136A / US4133029A** appears in the same period and citation neighbourhood. It requires direct primary-text review before being classified as PP250 lineage; do not silently include it in the original machine.

These are deliberately separated from the confirmed System-250 set rather than inflated into it.

---

# 5. Correction: unrelated diagnostic patent removed

An earlier draft of this audit incorrectly treated **US4066883A / GB1563288A** as a Plessey diagnostic-lineage patent. It is an **IBM** invention by Delwyn Roche Wheeler Jr. It merely cites Plessey `GB1422952A` as prior art. It is **not part of the PP250 patent corpus** and has been removed from the acquisition list.

This illustrates why citation alone is not evidence of architectural membership.

---

# 6. Principal designer audit

The inventor search was expanded beyond the initial Cosserat/Cotton/Hamer-Hodges trio. The contemporary System 250 bibliography names Cosserat, Cotton, England, Halton, Hamer-Hodges, Hemmings and Repton as principal published authors; the patent corpus adds Arnold, Beck, Boom, Cole, O'Halloran, Sandeman, Trapnell, Venton, Williams and others.

## Original-architecture inventors found

### David Cockburn Cosserat

Confirmed relevant families:

- `US3657736A` — distributed-algorithm/subroutine ancestry
- `US3787813A / GB1329721A` — capability architecture
- `US3757307A` — program interrupt facilities
- `US3787818A / GB1394431A` — multiprocessor architecture
- `US3771146A / GB1410631A` — capability interrupt arrangements

Cosserat is therefore one of the strongest inventor pivots for reconstructing the processor/system architecture.

### John Michael Cotton

Confirmed relevant families:

- `US3657736A` — distributed-algorithm ancestry
- `US3787813A / GB1329721A` — capability architecture
- `US3757307A` — program interrupt facilities
- `US3771146A / GB1410631A` — capability interrupt arrangements

Other inventor results include `US3680053A` (ring data transmission) and substantially later telecommunications/network inventions. These are not PP250 evidence without an independent architectural link.

### Kenneth James Hamer-Hodges

Confirmed relevant families:

- `US3787818A / GB1394431A` — multiprocessor architecture
- `US3814919A / GB1344474A` — fault detection/handling and PP250 checkout
- `US4121286A / GB1548401A` — later capability/MCT allocation-deallocation

Later work with Cotton in telecommunications switching exists but belongs to a subsequent career context, not the original PP250 patent set.

### Charles S. Repton

Confirmed relevant family:

- `US3814919A / GB1344474A` — fault detection and handling

This aligns directly with Repton's contemporary published System 250 reliability work.

### Peter C. Venton

Confirmed relevant families:

- `US3814919A / GB1344474A` — fault detection/handling
- `US4121286A / GB1548401A` — later memory allocation/deallocation

### Roger J. Boom

Confirmed relevant families:

- `US3657736A` — distributed-algorithm ancestry
- `US3787818A / GB1394431A` — multiprocessor architecture

### Michael O'Halloran

Confirmed relevant families:

- `US3757307A` — program interrupt facilities
- `US3787818A / GB1394431A` — multiprocessor architecture

Name searching produces many unrelated later patents belonging to other people with the same/similar name; these were rejected rather than attributed to the System 250 designer.

### James Jeffrey Llewelyn Williams

Confirmed relevant family:

- `US3771146A / GB1410631A` — capability interrupt arrangements

Do not confuse him with **John Lloyd Williams**, who appears on later System-250 bus/access-unit patents.

### Alun J. Cole

Confirmed relevant family:

- `US3787813A / GB1329721A` — foundational capability architecture

### Frederick M. Trapnell

Confirmed relevant family:

- `US3757307A` — program interrupt facilities

### John S. Arnold, George M. Beck, Theodor D. Sandeman, Roger M. Williams

Confirmed relevant family:

- `US3787818A / GB1394431A` — multiprocessor architecture

### Gordon Edge and George Worthington

Confirmed relevant family:

- `US3879712A / GB1422952A` — PP250 fault diagnostic arrangements

Their patent is particularly important because it gives evidence about the processor's microprogram diagnostic interface rather than merely software-visible architecture.

## Later System-250 development inventors

### John Lloyd Williams, Graham J. Wisdom, Peter V. Creteau

- `US4041460A` — System 250 peripheral/store access units

John Lloyd Williams and Graham Wisdom also appear in the read-and-hold family:

- `US4050059A / GB1536853A` — read-and-hold / MCT atomicity

with Roger J. Leaman, Robert V. Moberly and Geoffrey B. K. Stagg.

### Michael Blench, Anthony K. Sutherland

Together with Venton and Hamer-Hodges:

- `US4121286A / GB1548401A` — capability/MCT allocation and deallocation

### Nigel J. Wheatley and Martyn P. Andrews

Confirmed later families already represented in the repo:

- `US4383297A` — internal-register addressing
- `US4408274A` — later capability/memory protection
- `US4486831A` — process suspension/Dump Stack

These are a coherent later design layer and should be treated separately from original PP250.

## Published designers for whom this audit did not establish a PP250 patent family

The contemporary bibliography also names **D. M. England, D. Halton and W. A. C. Hemmings**. Searches did not establish a patent family attributable to them that can presently be classified as PP250 architecture. Their published papers remain primary architectural sources; absence from this patent inventory does not imply absence from the design work.

---

# 7. Designer patents outside the PP250 lineage

A complete inventor-name search inevitably reaches later employment and unrelated technology. Examples found include:

- Cotton and Hamer-Hodges on later distributed digital/remote telecommunications switching systems after the PP250 period;
- Cotton on later communications/network inventions;
- same-name false positives for O'Halloran and other inventors.

These are **not** included as PP250 patents. The audit records the distinction because otherwise a naive "all patents by name" search contaminates the architectural evidence.

---

# 8. Definitive acquisition list for the repository

## Original PP250/System 250 — acquire all

1. **US3757307A** — Program interrupt facilities in data processing systems
2. **US3814919A / GB1344474A** — Fault detection and handling arrangements
3. **US3787818A / GB1394431A** — Multi-processor data processing system
4. **US3771146A / GB1410631A** — Data processing system interrupt arrangements
5. **US3879712A / GB1422952A** — Data processing system fault diagnostic arrangements

`US3787813A / GB1329721A` is already present.

## Later System 250/capability development — acquire missing

6. **US4041460A** — Multi-processor data processing system peripheral equipment access units
7. **US4050059A / GB1536853A** — Data processing read-and-hold facility
8. **US4121286A / GB1548401A** — Data processing memory-space allocation/deallocation

`US4383297A`, `US4408274A`, and `US4486831A` are already present.

## Architectural ancestry — desirable

9. **US3657736A** — Method of assembling subroutines / distributed-algorithm multiprocessor work

## Review before inclusion as lineage

10. **US3999052A / GB1523005A** — Data processing apparatus / upper-bounds address checking
11. **US4133029A / GB1542136A** — later data-processing family; direct text review required before classification

---

# 9. Priority specifically for the M emulator

For the current M/H/T reconstruction, reading order should be:

1. **US3814919 / GB1344474** — fault entry, checkout and restricted authority
2. **US3771146 / GB1410631** — capability restoration and interrupt/process transition
3. **US3757307** — primary interrupt-demand machinery
4. **US3879712 / GB1422952** — microprogram diagnostic visibility
5. **US3787818 / GB1394431** — physical processor/store/bus context
6. **US3787813 / GB1329721** — capability machinery (already in repo)
7. **US4486831** — later comparative description of Dump Stack, special registers and indicators

The first four missing patents directly bear on the questions currently open about MIP/MIS/indicator state, fault entry, SPECIAL, special capability registers, process suspension and the point at which M hands control to H/T.

---

# 10. Corrections established by the completed audit

- **US4001813** is unrelated to PP250 and must not be cited as System 250 evidence.
- **US4066883 / GB1563288** is IBM, not Plessey; it cites a PP250 diagnostic patent but is not part of the PP250 family.
- **GB1410631A** corresponds to US family member **US3771146A**; there is no "US 1,410,631" counterpart.
- **US3757307A** and **US3771146A** are separate interrupt inventions.
- **James Jeffrey Llewelyn Williams** (original interrupt patent) and **John Lloyd Williams** (later bus/access-unit patents) are different inventors.
- The later Wheatley/Andrews patents are valuable evidence but belong to a later architectural generation.
- A citation from a PP250 patent does not make the cited patent a PP250 patent.
- An inventor-name match does not establish identity or architectural relevance.

---

# 11. Contemporary designer/source cross-check

The MIT 1973 *Bibliography of Papers on Plessey System 250* independently identifies contemporary papers by:

- D. C. Cosserat — capability-oriented multiprocessor architecture
- J. M. Cotton — communications-control processor requirements
- D. M. England — operating system and architectural features
- D. Halton — System 250 hardware
- K. J. Hamer-Hodges — fault resistance and recovery
- W. A. C. Hemmings — telephone switching based on System 250
- D. J. Repton — reliability assurance

This cross-check is useful because the patent inventor network and published-author network overlap strongly but are not identical. Both should be used in reconstruction research.

---

# 12. Audit conclusion

The repository's four patent PDFs were not a representative System 250 patent collection. The original architectural nucleus consists of at least the six confirmed families in Section 1, of which only the foundational capability-register family is currently present.

For processor reconstruction, the most serious omissions are the two original interrupt families, the fault-handling family, the multiprocessor family and the PP250 diagnostic/microprogram family. The later repo patents are valuable, but relying on them without the missing original patents risks importing late-1970s mechanisms into the early PP250 model.

This document is therefore the canonical patent inventory for PP250-Reboot. New patents should be added only after family identity, inventor identity, date, assignee and architectural relevance have been checked.