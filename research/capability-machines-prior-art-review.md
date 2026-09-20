# Preliminary Prior-Art Review: Capability Machines and PP250 Reboot

**Project:** PP250 Reboot  
**Review date:** 20 September 2026  
**Purpose:** Engineering prior-art review to inform invention capture and possible future patent drafting  
**Status:** Working research note — not a legal opinion or a freedom-to-operate opinion

---

## 1. Executive summary

The central ideas of a **capability machine are old prior art**. A patent application should therefore not attempt to claim, at a high level:

- a processor that uses capabilities;
- a capability containing an address or segment identity plus access rights;
- base/limit bounds checked by hardware;
- capability registers distinct from ordinary data registers;
- non-forgeable or hardware-protected capabilities;
- capability-controlled procedure/domain entry;
- capability attenuation or monotonic reduction of authority;
- tagged memory used to distinguish capabilities from ordinary data;
- capability-aware memory access;
- capability-based DMA merely as a general concept; or
- capability enforcement at a system interconnect merely as a general concept.

Those concepts are represented by a substantial body of work beginning in the 1960s and including the Chicago Magic Number Machine, Plessey System 250, Cambridge CAP, Hydra, IBM System/38, Intel iAPX 432, and, much later, CHERI and related capability-aware SoC research.

This is not bad news for PP250 Reboot. It clarifies where the potentially patentable work would have to lie.

The strongest prospect is **not "a modern capability computer based on PP250."** It would instead be a *specific new mechanism* discovered while constructing that computer: for example, a particular way of preserving PP250-style authority across a modern SoC interconnect, a new representation or derivation mechanism, a novel capability-controlled peripheral/DMA scheme, a new domain-transition mechanism, a new revocation mechanism, or a concrete low-cost microarchitecture that achieves a security property in a materially different way from the prior art.

The project's historical reconstruction is therefore valuable in two directions:

1. it establishes what PP250 itself already taught, preventing the project from mistakenly treating a 1970s mechanism as new; and
2. it provides a precise baseline against which a genuinely new descendant mechanism can be identified.

**Important practical point:** the repository is public. If a potentially patentable implementation is identified, it should be reviewed *before* detailed enabling material is committed publicly. Public disclosure can affect patent rights, particularly outside jurisdictions with grace periods.

---

## 2. Scope and method

This review asks a patent-oriented question:

> If PP250 Reboot develops a small modern processor/SoC whose security model descends from System 250, which parts are already disclosed by the capability-machine literature, and where might a narrower invention still exist?

The review considers:

- foundational capability literature;
- early hardware capability machines;
- Plessey System 250 itself and known Plessey patents;
- later commercial capability systems;
- CHERI;
- capability-aware DMA and system-interconnect work; and
- adjacent modern secure-bus/IOMMU mechanisms.

This is a **preliminary landscape**, not an exhaustive patentability search. A professional search would additionally require systematic searching of patent families, backward and forward citations, non-patent literature, conference proceedings, theses, standards, product manuals, and relevant claims in each target jurisdiction.

Patentability also depends on the precise claims eventually drafted. An old system does not have to look identical to a proposed implementation to be important: one reference may anticipate a claim, while multiple references may be combined in an obviousness/inventive-step argument.

---

## 3. The baseline capability idea predates PP250

The modern capability literature commonly traces the conceptual foundation at least to Dennis and Van Horn:

- Jack B. Dennis and Earl C. Van Horn, **"Programming Semantics for Multiprogrammed Computations"**, *Communications of the ACM*, 1966.

The important consequence for PP250 Reboot is that the abstract proposition that possession of an unforgeable reference conveys authority was already in the literature before System 250.

B. W. Lampson's work on protection and dynamic protection structures also belongs to this foundational period.

Useful bibliography:

- Capability Systems Bibliography: https://hydra-www.ietfng.org/capbib/full/date.html
- Saltzer, **The Protection of Information in Computer Systems / State of the Art** material: https://web.mit.edu/saltzer/www/publications/protection/State.html

The patentable unit of invention in 2026 therefore cannot plausibly be "using capabilities instead of conventional addresses or ACLs."

---

## 4. Chicago Magic Number Machine

The University of Chicago Magic Number Machine work, beginning in the late 1960s, is significant prior art because it attempted an integrated hardware/software capability system.

Features attributed to it in the historical literature include:

- segmented memory;
- addressing through capabilities;
- capabilities held in registers and memory;
- separation of capability storage from ordinary data; and
- hardware enforcement of capability-based primary-memory protection.

This matters because a proposed PP250 descendant cannot rely for novelty merely on:

- separate data and capability registers;
- protected capability storage;
- hardware bounds/protection checks; or
- requiring possession of a capability to address a segment.

A useful secondary source is Henry M. Levy, *Capability-Based Computer Systems*, Chapter 3:

https://book.huihoo.com/pdf/capability-based-computer-systems/Chapter3.pdf

---

## 5. Plessey System 250 / PP250

### 5.1 Why PP250 is unusually important prior art for this project

PP250 is not merely background inspiration. For patent purposes it is prior art against later inventions, including our own future work.

D. M. England's **"Capability Concept Mechanisms and Structure in System 250"** describes a machine in which:

- the system is multiprocessor;
- memory is segmented;
- a single storage address space spans the storage system;
- a segment is referenced through a capability;
- each CPU contains conventional data registers and capability registers;
- a capability register contains addressing information and an access field;
- memory references are made relative to capability registers;
- hardware checks segment bounds and permitted operations; and
- capabilities participate in sharing and protection.

A surviving copy is available at:

https://cs.ucf.edu/courses/cop6614/fall2005/englandplessey250.pdf

System 250 is repeatedly identified in later literature as an early commercial hardware capability system.

### 5.2 Features that PP250 itself places firmly in the prior art

Subject to the continuing historical reconstruction, surviving material already makes the following broad concepts unsafe to treat as new:

| Concept | PP250 relevance |
|---|---|
| Capability registers | Explicit architectural feature |
| Data registers separate from capability registers | Explicit architectural feature |
| Capability contains segment location/bounds plus access authority | Explicit |
| Hardware checks access against capability | Explicit |
| Memory segmented into protected objects/segments | Explicit |
| Capability used as the basis of addressing | Explicit |
| Multiprocessor capability machine | Explicit |
| Shared memory accessed by multiple processors under capability control | Explicit |
| Capability-controlled domain/process transition | Strongly documented in System 250 material |
| C6/C7-style security/execution context | Characteristic System 250 mechanism |
| No need for a conventional privileged-mode protection model as the primary security boundary | Characteristic architectural property |
| Memory-mapped peripheral model | Documented System 250 architecture |
| Fault handling that replaces/limits capability context | Covered by Plessey patent material |

This table is intentionally broad. Exact historical claims in the PP250 Reboot repository should continue to be classified as Documented / Corroborated / Inferred / Uncertain / Speculative according to the project's evidence rules.

---

## 6. Plessey patents already located

### 6.1 US 3,771,146 / GB 1,410,631 — Data Processing System Interrupt Arrangements

Inventors include John Michael Cotton, James Jeffrey Llewelyn Williams, and David Cockburn Cosserat. The British priority date is 26 January 1972.

The patent describes, among other things:

- segmented memory;
- capability registers;
- a capability word containing a segment descriptor and access type;
- base and limit information;
- capability checks on memory access;
- master capability table structures;
- special characteristic codes in capability registers; and
- trapping/interrupt behaviour associated with capability state.

Google Patents:

https://patents.google.com/patent/GB1410631A/en

US grant information:

https://uspto.report/patent/grant/3771146

For present purposes this is strong primary evidence that several mechanisms we might otherwise rediscover from manuals were expressly reduced to patent claims in the early 1970s.

### 6.2 US 3,814,919 — Fault Detection and Isolation in a Data Processing System

Priority-era System 250 work also includes the Plessey fault-isolation patent.

It describes a fault response in which processor capability state is replaced so that a faulty processor is confined to fault check-out storage. It also describes multiple processors/storage modules and multiple copies of check-out code/workspace.

Patent text:

https://www.freepatentsonline.com/3814919.html

This is particularly relevant to any PP250-Reboot idea based on:

- changing the processor's authority set on a fault;
- confining a processor to a recovery domain;
- constructing a restricted execution environment without conventional supervisor privilege; or
- using capability context itself as a fault-containment boundary.

Any new recovery mechanism would need to be distinguished at the implementation level.

---

## 7. Cambridge CAP

The Cambridge CAP computer project began around 1970 and produced a working hardware capability machine.

Key prior-art themes include:

- hardware capability registers;
- capabilities defining base, limit and access rights;
- protected capability manipulation;
- protected procedure/domain mechanisms; and
- capability-oriented operating-system structure.

Primary historical book:

Maurice V. Wilkes and Roger M. Needham, *The Cambridge CAP Computer and Its Operating System*, 1979.

Google Books record:

https://books.google.com/books?id=dK9QAAAAMAAJ

Cambridge teaching material summarises a CAP capability as containing base, limit and access information:

https://www.cl.cam.ac.uk/teaching/0203/AdvSysTopics/ast-smh.pdf

CAP is especially relevant when assessing any claim based on capability memory being structurally separated from ordinary data rather than protected by a per-word tag.

---

## 8. Hydra and capability operating systems

Hydra, developed for Carnegie Mellon's C.mmp, is important even where a proposed invention is hardware-centric because patent obviousness is not restricted to prior machines having the same hardware implementation.

Hydra treated resources as protected objects and developed mechanisms for:

- object references;
- controlled operations;
- protection;
- creation of new object types; and
- secure sharing.

Wulf et al., **"HYDRA: The Kernel of a Multiprocessor Operating System"**, *Communications of the ACM*, 1974, is a key reference.

Hydra therefore weakens any broad attempt to claim object authority, protected operations, or capability-mediated resource sharing as a new software/hardware concept.

---

## 9. IBM System/38

IBM System/38 is important because it demonstrates a different solution to capability integrity.

Historical descriptions identify tagged pointers/capabilities in which tag information is not normally accessible as ordinary program data. Capability and ordinary data can coexist in memory while hardware/microcode preserves the distinction.

Levy, *Capability-Based Computer Systems*, Chapter 8:

https://homes.cs.washington.edu/~levy/capabook/Chapter8.pdf

For PP250 Reboot, System/38 is strong warning prior art against a broad claim such as:

> "associate hidden metadata with memory words so that hardware can determine whether a stored value is a valid protected pointer/capability."

That design space was explored decades ago.

---

## 10. Intel iAPX 432 and other historical machines

A complete professional search should include at least:

- Intel iAPX 432;
- CAL-TSS;
- SUE;
- CMU Cm* and Hydra-related systems;
- SWARD;
- PSOS;
- Monads;
- IBM System/38;
- KeyKOS and related object-capability systems;
- M-Machine;
- EROS/CapROS; and
- capability-oriented language/runtime systems.

Not all are equally close to a small PP250-derived hardware machine, but together they make the capability field densely populated prior art.

The important patent lesson is that novelty is unlikely to exist at the level of the general capability abstraction.

---

## 11. CHERI

### 11.1 What CHERI adds to the modern landscape

CHERI (Capability Hardware Enhanced RISC Instructions), developed by SRI International and the University of Cambridge, is the most important modern comparison.

CHERI combines conventional processor architecture with architectural capabilities that include:

- an address/cursor;
- bounds;
- permissions;
- validity/provenance protected by a tag;
- capability registers;
- controlled capability-manipulation instructions;
- monotonic derivation — derived capabilities cannot simply acquire greater authority;
- sealed capabilities / protected control transfer;
- fine-grained memory safety;
- software compartmentalisation; and
- hybrid operation with conventional virtual memory and software.

Cambridge CHERI FAQ:

https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/cheri-faq.html

CHERI architectural rules:

https://ctsrd-cheri.github.io/cheri-c-programming/background/architectural-rules.html

CHERI ISA material:

https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/cheriformal_abstracts.html

### 11.2 Consequence for PP250 Reboot

A modern implementation does **not** become novel merely because it:

- puts capabilities into a contemporary RISC-like processor;
- uses tags;
- protects pointer provenance;
- provides bounds and permissions;
- prevents arbitrary software from manufacturing valid capabilities;
- attenuates capabilities;
- uses capabilities for compartments;
- provides sealed entry capabilities; or
- runs capability hardware on FPGA or as an ASIC.

CHERI occupies this territory extensively.

### 11.3 PP250 is nevertheless architecturally different

This difference is important for engineering but must not be confused with automatic patentability.

PP250's model is historically centred on segments, capability registers, capability tables and process/domain context. CHERI is deliberately designed to integrate capability semantics into modern pointer-rich software ecosystems and conventional ISAs.

A modern PP250 descendant could therefore be *different from CHERI* without necessarily being *patentably new*. The question is whether the particular new mechanism was disclosed or rendered obvious by PP250, CAP, CHERI, and the rest of the art in combination.

---

## 12. Capability-aware DMA is already an active prior-art field

One of the most attractive PP250-Reboot experiments is to extend authority to DMA:

> instead of programming a DMA engine with an unrestricted physical address, give it authority that identifies exactly the memory region and operations it may perform.

This is a sound research direction, but the broad idea is already known.

The CHERI ISA literature explicitly discusses **capability-aware DMA and IOMMUs**, including DMA engines that preserve capability provenance and capability semantics.

CHERI ISA v7 material:

https://xmpp.ietfng.org/nwf/_downloads/812ecd2adf1484a2569edfc1d1d15213/2018-cheri-isav7.pdf

The current CHERI specification repository goes further and discusses designs in which DMA descriptor/MMIO integer addresses are replaced by capabilities and memory transactions are checked against capability bounds and permissions:

https://github.com/CTSRD-CHERI/cheri-specification/blob/main/app-exp-dma.tex

Consequently, **"a DMA controller that receives a bounded capability and checks DMA accesses against it" is not, by itself, a safe novelty proposition.**

---

## 13. Bus/interconnect-level capability enforcement is also prior art

This area is particularly important because it overlaps our stated modern-descendant objective: a modern transaction must not bypass the capability model.

### 13.1 Northcape

The Northcape research project describes byte-granular capability-based protection enforced at the bus/northbridge level and applied to CPUs, accelerators, DMA peripherals, memory, and memory-mapped I/O, while retaining an AXI system bus.

CISPA summary:

https://cispa.de/en/research/publications/79232-work-in-progress-northcape-embedded-real-time-capability-based-addressing

This is close enough to our broad system-level objective that it must be treated as important prior art.

A claim such as:

> "enforce capability permissions at the interconnect so that CPU and DMA transactions cannot access unauthorised memory or MMIO"

would be far too broad.

### 13.2 Secure-aware buses

Conventional secure-bus architectures also provide adjacent prior art even where they do not implement true capabilities. For example, Nordic Semiconductor's US 11,698,995 describes hardware filtering of secure/non-secure bus transactions to peripherals.

https://patents.justia.com/patent/11698995

These systems matter to obviousness because a patent examiner may combine capability references with established bus-security mechanisms.

### 13.3 IOMMUs

IOMMUs already constrain device DMA to authorised address spaces and are a mature alternative mechanism.

A capability-based DMA design therefore needs a specific technical distinction and advantage over both:

1. capability-aware DMA proposals; and
2. page/table-based IOMMU protection.

---

## 14. Recent capability patents and applications

The modern patent landscape is active. One example is Intel's published application **US 2025/0284832**, "Capability-Based Memory Access Control for Graphics Processors and Accelerators," describing capability checks associated with accelerator accesses, including bulk checking.

https://patents.justia.com/patent/20250284832

The significance is not that this application necessarily blocks anything we may build. Its claims would need detailed analysis. The significance is that capability enforcement beyond a scalar CPU is an actively patented field.

Any invention involving accelerators, DMA, bus masters, bulk transactions, caching of capability decisions, or capability propagation through a SoC should therefore trigger a dedicated patent search before disclosure.

---

## 15. Preliminary claim-space map

The following is an engineering triage, not a legal conclusion.

| Proposed idea | Preliminary prior-art risk | Reason |
|---|---:|---|
| Processor with capabilities | Very high | Foundational capability machines |
| Base/limit/permissions capability | Very high | Chicago, PP250, CAP, CHERI |
| Separate capability registers | Very high | Early machines including PP250 |
| Capabilities protected from ordinary manipulation | Very high | Core capability property |
| Capability table | Very high | PP250/CAP and others |
| Protected domain entry using a capability | Very high | PP250/CAP/CHERI-related art |
| No conventional privileged mode as primary protection mechanism | High | PP250 and capability-system literature |
| Tagged capability memory | Very high | System/38, M-Machine, CHERI and others |
| Capability attenuation | Very high | Capability literature / CHERI monotonicity |
| Capability-controlled MMIO | High | Natural consequence of capability addressing; modern bus-security work |
| Bounded DMA capability | Very high | CHERI DMA work and other research |
| Capability checking at SoC interconnect | Very high | Northcape and adjacent work |
| Capability-aware accelerator | Very high | Research plus recent patent activity |
| Modern external bus with internal capability protection | High | Combination likely obvious absent specific mechanism |
| PP250 implemented on FPGA | Very high | Implementation medium alone is not the invention |
| PP250 implemented as ASIC | Very high | Same reason |
| Specific new low-cost capability representation | **Potential** | Depends on exact mechanism and search |
| Specific new revocation mechanism | **Potential** | Active field; exact mechanism essential |
| Specific new domain-transition implementation | **Potential** | Must distinguish old protected-call/sealing mechanisms |
| Specific capability-preserving bridge to legacy peripherals | **Potential** | Must distinguish Northcape/CHERI/IOMMU/secure-bus art |
| Specific way to derive device authority from process authority without a privileged broker | **Potential** | Interesting, but requires detailed search |
| New transaction-level authority representation on a modern interconnect | **Potential** | Only if technically specific and distinct from existing tagged/secure/capability buses |
| New PP250-derived mechanism that eliminates a conventional MMU/IOMMU while preserving confinement | **Potential** | Needs concrete architecture and comparison |
| New capability revocation/recovery scheme integrated with processor fault containment | **Potential** | Plessey fault patent is directly relevant |

"Potential" means **worth invention capture**, not "patentable."

---

## 16. Where PP250 Reboot should look for genuine inventions

### 16.1 Authority-preserving SoC transactions

A promising question is not merely whether the CPU checks a capability before issuing a load/store. It is:

> Can authority itself remain an architectural property of the transaction as it passes through arbitration, bridges, memory controllers, DMA engines and peripheral fabrics, such that no intermediary silently collapses it back into an unrestricted physical address?

This must be compared carefully with Northcape, CHERI physical-capability/DMA work, tagged interconnects, secure-world bus signals, and IOMMUs.

Potentially differentiating details might include:

- how authority is encoded;
- whether the transaction carries a capability, a derived token, or an ephemeral reference;
- whether authority can be amplified by a bridge;
- how a peripheral derives downstream authority;
- whether delegation is monotonic;
- how revocation works for in-flight or cached authority;
- how capabilities survive width conversion or protocol bridging;
- how legacy peripherals are confined;
- how device-generated transactions are linked to the authority of the initiating process; and
- whether all of this is achieved without a conventional privileged monitor.

The novelty, if any, will be in the *mechanism*, not the objective.

### 16.2 C6/C7-style domains in a modern microarchitecture

PP250's distinction between current execution/code authority and a broader capability context is architecturally interesting.

A new implementation might become invention-worthy if it develops a materially new mechanism for:

- constant-time domain transition;
- hardware construction of a new execution context;
- very small protected context state;
- capability-mediated call/return without privilege-level switching;
- nested domains;
- interrupt/fault entry preserving least authority;
- multicore migration of a domain; or
- revocation of a domain while retaining deterministic real-time behaviour.

But PP250 itself, CAP protected procedures, Hydra, CHERI sealing, call gates and other protection mechanisms form substantial prior art. "C6/C7 on a modern chip" is not enough.

### 16.3 Capability-safe DMA without an IOMMU

This remains interesting if the implementation solves a problem not already solved in the known art.

Questions worth exploring:

- Can a process delegate a subset of its memory authority directly to a DMA engine?
- Can the DMA engine be physically incapable of accessing anything outside that delegated authority?
- Can this work without page tables, an IOMMU, or privileged reconfiguration?
- Can authority be revoked cheaply and deterministically?
- Can chained descriptors attenuate but never amplify authority?
- Can one peripheral delegate to another without creating ambient authority?
- Can a device receive authority over both an MMIO endpoint and a bounded memory buffer in one protected operation?

Each concrete answer should be searched before publication.

### 16.4 Fault containment as capability replacement

The original Plessey fault patent already used replacement of capability context to confine a suspect processor.

A modern descendant could nevertheless generate new ideas around:

- hardware quarantine of a core;
- automatic least-authority recovery contexts;
- cross-core recovery without a privileged supervisor;
- deterministic recovery from malformed capability state;
- capability revocation on detected hardware fault;
- confinement of accelerators or peripherals after fault detection.

Because US 3,814,919 is unusually close conceptual prior art, this area requires particularly careful claim differentiation.

### 16.5 Minimal capability hardware for microcontrollers

There may be useful invention space in achieving strong capability semantics at very low gate count/power/latency.

But "small CHERI" or "PP250 on a microcontroller" is not a sufficient invention.

Patentable subject matter, if present, might instead be a particular:

- compressed representation;
- register organisation;
- cache/tag arrangement;
- bounds-check circuit;
- context-switch mechanism;
- capability-table cache;
- bus transaction encoding;
- revocation structure; or
- compiler/hardware co-design

that produces a demonstrable technical effect.

---

## 17. A useful distinction: capability *data* versus capability *authority*

An important design principle for PP250 Reboot is to avoid confusing a value labelled as a capability with an architecture that actually enforces capability authority.

Adding a capability field or tag to every word does not by itself create a capability machine.

The relevant questions are:

1. **Provenance:** How can valid authority come into existence?
2. **Integrity:** Can ordinary computation manufacture or alter it?
3. **Non-amplification:** Can delegated authority become stronger?
4. **Use:** Is every protected operation checked against authority?
5. **Propagation:** What happens when authority crosses a CPU/device/interconnect boundary?
6. **Domain transition:** How is execution authority changed?
7. **Revocation:** How is previously delegated authority withdrawn?
8. **Fault behaviour:** What authority remains after an exception or hardware fault?
9. **Persistence:** If capabilities are stored, swapped, transmitted, or cached, how is their integrity maintained?
10. **No bypass:** Is there any alternate address path that escapes the model?

This distinction is also useful for patent work. A patentable invention is more likely to arise from a new answer to one of these mechanism questions than from adding more metadata to an ordinary binary processor.

---

## 18. Quantum resistance is not the relevant novelty axis

The capability mechanisms discussed here are generally **architectural access-control mechanisms**, not cryptographic secrets that an attacker defeats by solving a hard mathematical problem.

A locally hardware-protected capability can therefore provide non-forgeability without relying on public-key cryptography. In that limited sense its security property is not based on RSA/ECC-style assumptions vulnerable to a sufficiently capable quantum computer.

However:

- network authentication may still use cryptography;
- secure boot may use signatures;
- encrypted storage may use cryptography;
- remote capability transfer may require cryptographic protection; and
- physical attacks may introduce different threat models.

Accordingly, "quantum-proof capability machine" would be an over-broad and potentially misleading invention framing. The more precise property is **hardware-enforced authority that does not depend on computational secrecy for local non-forgeability**.

That property itself is old capability-system art.

---

## 19. What could make a future claim defensible?

Before treating an idea as an invention candidate, write it in this form:

### Problem

What specific technical problem exists in prior systems?

### Mechanism

What exact hardware/software mechanism solves it?

### Security invariant

What must remain impossible?

For example:

> A device transaction cannot acquire authority to an address range or operation not derivable from the authority delegated by its initiating execution domain.

### Technical effect

What measurable consequence follows?

Examples:

- removes an IOMMU/page-table walk;
- reduces context-transition cycles;
- removes privileged software from the DMA setup path;
- reduces capability storage bits;
- makes revocation bounded-time;
- prevents confused-deputy amplification at a bridge;
- confines a bus master despite compromised firmware;
- preserves authority across heterogeneous interconnects.

### Distinction

Which feature is absent from PP250, CAP, System/38, CHERI, Northcape, IOMMUs and secure-bus architectures?

If we cannot state that distinction precisely, the idea is not ready for a patentability assessment.

---

## 20. Suggested invention-capture rule for this repository

For every new architectural idea, before implementation detail is committed publicly, record privately:

1. **Title**
2. **Date conceived**
3. **Contributors**
4. **Problem being solved**
5. **Exact mechanism**
6. **Security invariant**
7. **Why PP250 did not already do it**
8. **Why CHERI does not already do it**
9. **Why a conventional MMU/IOMMU/TrustZone-style system does not already do it**
10. **Closest paper/patent found**
11. **Differences from that reference**
12. **Whether public disclosure should be held pending patent review**

The repository's `AGENTS.md` should continue to require agents to flag potentially patentable material before committing enabling implementation detail.

---

## 21. Public-repository warning

This project is deliberately open and historically oriented, but patent work and immediate public disclosure can conflict.

A sensible workflow is:

**idea → private invention note → focused prior-art search → patent decision → public repository**

rather than:

**idea → detailed public commit → later patent discussion**

This matters particularly if protection outside the United States is contemplated. Patent systems differ in how they treat an inventor's own pre-filing disclosure. Do not assume that a later filing can always recover rights after publication.

This note itself intentionally identifies research directions at a relatively high level. If one of them develops into a concrete new implementation, the detailed mechanism should be reviewed before publication.

---

## 22. Recommended next searches

A professional or second-stage search should concentrate on the exact mechanisms we actually design, but the following subject clusters should be covered:

- capability processor patents, 1965-present;
- Plessey/System 250 patent families and citations;
- Cambridge CAP patents/publications;
- IBM System/38 pointer/tag patents;
- Intel iAPX 432 protection/object patents;
- tagged-memory architectures;
- CHERI patents and publications;
- capability compression;
- capability revocation;
- sealed/protected domain transitions;
- object-capability hardware;
- capability-aware DMA;
- capability IOMMUs;
- device capability delegation;
- secure AXI/AHB/Wishbone interconnects;
- bus-level access-control metadata;
- transaction-level security labels/tags;
- accelerator capability checking;
- memory-safe microcontrollers;
- hardware compartmentalisation;
- protection without privilege rings;
- fault containment through protection-context replacement; and
- secure delegation between bus masters.

Backward and forward citations from the closest references are essential.

---

## 23. Preliminary conclusion

The PP250 Reboot project sits in a historically rich and heavily researched field. **The capability concept is not the invention.** Neither is putting it on an FPGA, shrinking it onto a modern chip, adding tags, or extending bounds checking to DMA in the abstract.

PP250's value to a new design is deeper: it gives us a different architectural starting point from the conventional processor-plus-MMU model and, in some respects, from CHERI's modern pointer-capability model.

The potential invention is therefore likely to appear when we ask:

> What mechanism becomes possible when the entire machine — CPU execution context, memory, peripherals, DMA and interconnect — is designed so that authority is native and there is no ordinary unrestricted address path underneath it?

Even that objective is not new in the abstract. Northcape, CHERI research and historical capability systems already occupy parts of the territory.

The patent opportunity, if one emerges, will be a **specific implementation that solves a concrete remaining problem in a new way**.

That is the standard the PP250 Reboot project should apply before labelling any new feature "patentable."

---

## 24. Key references

### Foundational / historical

- Dennis, J. B. and Van Horn, E. C., "Programming Semantics for Multiprogrammed Computations," *Communications of the ACM*, 1966.
- Capability Systems Bibliography: https://hydra-www.ietfng.org/capbib/full/date.html
- Levy, H. M., *Capability-Based Computer Systems*: https://homes.cs.washington.edu/~levy/capabook/
- Early capability architectures (Levy, Chapter 3): https://book.huihoo.com/pdf/capability-based-computer-systems/Chapter3.pdf
- Saltzer protection history: https://web.mit.edu/saltzer/www/publications/protection/State.html

### Plessey System 250

- England, D. M., "Capability Concept Mechanisms and Structure in System 250": https://cs.ucf.edu/courses/cop6614/fall2005/englandplessey250.pdf
- Plessey System 250 overview/bibliographic leads: https://en.wikipedia.org/wiki/Plessey_System_250
- GB 1,410,631, "Data Processing System Interrupt Arrangements": https://patents.google.com/patent/GB1410631A/en
- US 3,771,146: https://uspto.report/patent/grant/3771146
- US 3,814,919, "Fault Detection and Isolation in a Data Processing System": https://www.freepatentsonline.com/3814919.html

### Cambridge CAP

- Wilkes, M. V. and Needham, R. M., *The Cambridge CAP Computer and Its Operating System*: https://books.google.com/books?id=dK9QAAAAMAAJ
- Cambridge capability-systems teaching material: https://www.cl.cam.ac.uk/teaching/0203/AdvSysTopics/ast-smh.pdf

### IBM System/38

- Levy, Chapter 8: https://homes.cs.washington.edu/~levy/capabook/Chapter8.pdf

### CHERI

- CHERI FAQ: https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/cheri-faq.html
- CHERI architectural rules: https://ctsrd-cheri.github.io/cheri-c-programming/background/architectural-rules.html
- CHERI formal/ISA references: https://www.cl.cam.ac.uk/research/security/ctsrd/cheri/cheriformal_abstracts.html
- CHERI capability-aware DMA discussion: https://github.com/CTSRD-CHERI/cheri-specification/blob/main/app-exp-dma.tex

### Modern SoC / DMA / interconnect

- Northcape capability-based bus-level protection: https://cispa.de/en/research/publications/79232-work-in-progress-northcape-embedded-real-time-capability-based-addressing
- US 11,698,995, secure-aware peripheral bus: https://patents.justia.com/patent/11698995
- Intel US 2025/0284832, capability-based accelerator memory access: https://patents.justia.com/patent/20250284832

---

## 25. Repository status

This document is a research aid for PP250 Reboot. It should be revised whenever:

- a new historical PP250 source changes our understanding of the original machine;
- a close modern patent or paper is found;
- the modern-descendant architecture becomes more concrete; or
- a specific invention candidate is identified.

When that happens, the next review should move from this broad landscape to a **feature-by-feature claim chart against the closest references**.
