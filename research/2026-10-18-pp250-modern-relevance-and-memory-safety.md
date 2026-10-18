# PP250 in 2026 — Architectural Protection versus Fine-Grained Memory Safety

**Date:** 18 October 2026  
**Status:** Research discussion and emerging design direction; **not** a historical architecture specification.

## Purpose and scope

This note preserves the reasoning that began with the cost of capability checking and SCT resolution, considered the later Andrews–Wheatley development and comparisons with CLOOMC and CHERIoT, and ended by questioning the premise that every C/C++ pointer ought to be a hardware capability.

The primary objective of PP250-Reboot remains faithful reconstruction of the historical machine. This note discusses possible modern descendants, without changing the documented historical baseline, declaring a modern ISA, or claiming an unmeasured performance advantage.

## 1. Starting question: per-pointer capabilities and performance

We asked whether an SCT-based capability machine could compete in modern pointer-rich programs if *every* C/C++ pointer denoted an independently protected object. In a design with one SCT descriptor per separately protected allocation, pointer-heavy traversal might incur repeated descriptor resolution; a very large live object population would imply a correspondingly large SCT.

These are conditional costs of choosing **allocation-level hardware capability granularity**, not inevitable consequences of the PP250 itself. A C/C++ pointer, a stored capability pointer, a C-register capability, a protected segment and an individual allocator object are different concepts. Conflating them led us to evaluate the historical architecture against a requirement it did not impose.

For the historical SCT descriptor family, three 24-bit words correspond to nine bytes of raw descriptor information. One million separately represented objects would therefore imply about 9 MB of descriptor data alone *if* every object required one entry. This illustration omits allocator, alignment and coherence costs; it is not a measured historical or proposed-system footprint.

## 2. What the processor evidence establishes

The dedicated processor protection hardware is documented in [processor-hardware-design-description.txt](../architecture/processor-hardware-design-description.txt), especially sections 6, 7.2 and 7.3, drawing on US3814919A, Figures 2a/2b, and the later US4383297A LOAD sequence.

- Address generation uses arithmetic hardware. Store-mode indexed addressing is described in [instruction-set.md](../architecture/instruction-set.md): the instruction address field plus optional D-register modifier forms a segment-relative offset, interpreted through the selected C register.
- Dedicated comparator hardware checks capability bounds, access rights and, in the early patent circuit, base/limit parity. This is integrated into the memory-reference machinery; it is not a separately executed software validation routine.
- Ordinary references through an already expanded C register are not described as re-fetching its SCT descriptor on every access.
- **Unknown:** precise comparator latency, degree of overlap with address arithmetic, and number of microcode slots or clock cycles attributable to protection. Dedicated circuitry does not prove zero additional latency.
- US4383297A describes validation before address transmission to the bus sequence/control path; do not posit speculative unauthorised store accesses and cancellation as an established historical mechanism.

Generation-specific implementations must not be collapsed into a single pin-level reconstruction. Refer also to [pp250-architectural-generations.md](pp250-architectural-generations.md).

## 3. Andrews–Wheatley: reducing unnecessary capability expansion

US4408274A, *Memory protection system using capability registers*, describes the later P-register / C-register pairing and **Load on Use** mechanism. A compact capability pointer is installed in its associated P register, with expanded base/limit state in C deferred until needed. Capability pointers transferred without dereference need not always trigger full descriptor resolution.

The mechanism improves capability movement; it does not make descriptor resolution free when a newly encountered capability is actually used. It does not, by itself, shrink the SCT. Historical chronology and the LC/LDP questions are covered in [lc-ldp-direct-mode-evolution.md](lc-ldp-direct-mode-evolution.md) and [instruction-set.md](../architecture/instruction-set.md). Do not back-project the later enhancement onto the earliest machine.

## 4. The workload question and the discarded premise

Lists, queues, trees and object graphs are common software structures; this alone does **not** establish that each node should occupy a separate protected SCT segment. A million-node linked list might instead live within one protected segment with ordinary internal offsets or pointers. One segment descriptor protects the domain boundary while ordinary addressing traverses its internal layout.

That choice reduces descriptor population and capability turnover, but weakens hardware-enforced *intra-segment* spatial isolation. An incorrect internal pointer may still corrupt another allocation within the same authorised segment. The granularity of protected segments is thus a **security-policy and system-design choice**, not a free performance optimisation.

No representative benchmark in this investigation quantified descriptor hit rates, capability-load frequency, pointer-chasing cost or total-system instruction count. Claims of significant speedup or slowdown remain unproved.

## 5. Comparisons: CLOOMC and CHERIoT

CLOOMC's Church Machine documentation describes compact Golden Tokens, Namespace entries and a capability-register cache of expanded Namespace information on LOAD. The design bears a structural resemblance to compact PP250 pointers plus expanded C-register descriptors, but must not be treated as identical; its exact runtime and FPGA costs were not measured here.

CHERIoT targets fine-grained memory safety and compartmentalisation for embedded software. Its 64-bit capabilities, associated validity metadata and revocation machinery involve different trade-offs from SCT indirection. In particular, CHERIoT does not require a separate SCT descriptor lookup for each distinct object reference. It also supports architectural compartment boundaries, so it is not properly compared with an unprotected conventional operating system.

The relevant comparator is **CHERIoT**, not generic 128-bit CHERI, and not Windows message-dispatch overhead. Differences in purpose and software composition must be measured using equivalent workloads and security guarantees.

## 6. Emerging architectural separation for a possible successor

The key interpretive conclusion from the discussion is:

> A PP250 protected object or execution domain need not coincide with an individual programming-language allocation or pointer.

A possible 2026 design could divide responsibilities as follows:

| Layer | Candidate responsibility |
| --- | --- |
| Hardware authority | Capabilities for protected segments, devices, services and other resources; enforced bounds/rights and transitions |
| Protected execution | C6/C7-like code/environment state, CALL/RETURN, process construction and switching |
| Programming language | Memory-safe manipulation of pointers, arrays, object ownership and lifetimes *within* a domain |
| Runtime/allocator | Allocation, reclamation and optional dynamic checks within authorised storage |
| SCT | Protected-segment identity, authority, sharing and location indirection, with defined relocation and cache coherence |

Rust and Swift offer ways to reduce pointer-related bugs within suitable safe language subsets; they do not inherently enforce hardware isolation between untrusted code components, and unsafe code / interoperability remain security considerations. Hardware retains responsibility for authority boundaries, including cases where code inside a domain is defective or adversarial.

This division is an **exploratory design direction**, not a claim that language-level safety makes fine-grained hardware memory safety unnecessary for all threat models. Intra-domain safety and cross-domain isolation are complementary requirements.

## 7. SCT implementation improvements worth testing

A modern implementation could investigate:
- More resident C registers and Andrews–Wheatley-style deferred expansion.
- A small dedicated on-chip SCT descriptor cache or, in a bounded embedded configuration, on-chip SRAM holding the entire SCT.
- Full rules for invalidating expanded C registers and cached descriptors after SCT relocation, change of rights, or revocation in a multiprocessor.
- Allocation granularity chosen explicitly from the protection requirements, not imposed by the source language's every-pointer representation.

A descriptor cache only helps when useful entries are revisited. One-way traversal of many distinct protected objects can remain latency-limited. Neither SRAM placement nor caching has been demonstrated here to remove all performance costs.

## 8. System-level efficiency rather than isolated cycle counts

The original PP250 is notable for the integration of capability authority, protected execution, process-state handling and resource management. An architectural CALL/RETURN or fault-controlled transition may be compared against a complete software service path, rather than solely against a conventional pointer load. But the claim that a PP250 descendant would execute substantially fewer instructions than CHERIoT for equivalent secure functionality remains a **hypothesis** requiring implementation and measurement.

The important tests would compare identical embedded functions, security boundaries and fault-containment guarantees, including code size, memory footprint, descriptor footprint, cache behaviour, worst-case latency, system-call / compartment-call overhead and failure handling.

## 9. Current conclusions and limits

**Established from sources:** PP250 capability-qualified memory references use hardware checking; ordinary access through a loaded C register does not require a fresh SCT expansion; the later Andrews–Wheatley design adds Load on Use; the SCT offers location indirection at segment granularity.

**Architectural interpretation:** The PP250's natural protection unit may be a meaningful system object or execution domain, not every language-level pointer or allocation. This accords with the larger C6/C7, CALL, CHP and protected-service reconstruction, without asserting a single mandatory original segmentation policy.

**Open engineering questions:** Appropriate domain granularity; actual original and modern timing; SCT cache size/hit rate and coherence; temporal and spatial safety *within* segments; realistic software/code-size savings against CHERIoT; effective treatment of unsafe/foreign-language code.

**Decision for now:** Continue faithful historical reconstruction. Do not revise the historical processor or SCT architecture to imitate fine-grained CHERIoT pointers. Treat the combination of PP250 protected domains with modern memory-safe languages as a research hypothesis for a possible successor, and validate it before promoting it to an architectural design specification.

## References within repository

- [Processor hardware design description](../architecture/processor-hardware-design-description.txt)
- [Instruction set and address formation](../architecture/instruction-set.md)
- [Capability representation](../architecture/capability-representation.md)
- [LC/LDP direct-mode evolution](lc-ldp-direct-mode-evolution.md)
- [Architectural generations](pp250-architectural-generations.md)
- [Processor microarchitecture reconstruction](pp250-microarchitecture-reconstruction.md)

Primary patent evidence: US3814919A, US4383297A and US4408274A (see the repository `patents/` and `transcriptions/` inventories).
