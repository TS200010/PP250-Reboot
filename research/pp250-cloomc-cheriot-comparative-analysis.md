# PP250, CLOOMC and CHERIoT — Comparative Architectural Analysis

> Status: research note capturing an exploratory design discussion. Historical claims, modern comparisons, and design hypotheses are deliberately kept distinct. This note preserves the reasoning as well as the current conclusions.

## Why CHERIoT became the important comparator

The initial comparison for PP250-Reboot was naturally with CHERI in general. That comparison is useful historically and conceptually, but CHERIoT is probably the more useful contemporary engineering comparator for a small modern PP250-derived machine.

CHERIoT is aimed at embedded systems rather than a large desktop/server environment. It combines a small RISC-V-derived processor with CHERI-style capabilities, compartments, protected cross-compartment calls, capability-protected MMIO, and an RTOS designed around those facilities.

That puts it much closer to the design space now being considered for PP250-Reboot: a small capability-native processor that could plausibly be implemented in FPGA and eventually silicon.

The comparison should not be reduced to "which is smaller?" because the two architectures put the protection boundary in different places and provide somewhat different guarantees.

## CHERIoT's central approach

At a high level, CHERIoT makes C/C++ pointers capability-bearing values. A pointer can carry bounds, permissions and provenance enforced by the processor. This gives fine-grained spatial protection: a pointer to one allocation cannot simply be advanced into another allocation.

CHERIoT also addresses temporal safety: after storage is freed and reused, stale capabilities must not become valid authority over the replacement object. This requires revocation machinery.

This is an important baseline because it demonstrates that practical C/C++ memory safety can be implemented in a small embedded processor.

## PP250 starts from a different abstraction

The PP250 architecture does not begin by turning every ordinary pointer into a capability. It has a capability architecture based on segments, the Segment Capability Table (SCT), separate capability state, access rights, and protected transitions.

This led to an initially misleading question: if a PP250 capability covers a large data segment containing many language objects, could a bad C pointer move from one object to another inside the segment?

Yes, if many independent objects are deliberately placed inside one protection segment.

But that is not the only PP250 representation. The more important observation is:

> A language-level allocation can itself be represented by a PP250 data segment.

In that case the segment bounds are the object's bounds.

Conceptually:

    allocation
        |
        v
    data segment
        |
        v
    SCT entry
        |
        v
    bounded object storage

An attempted access outside that segment is outside the authority represented by the segment capability.

This means that object-level spatial safety does not inherently require the CHERIoT model of attaching a large capability representation to every C pointer. PP250 can instead place protection metadata at the segment/SCT level.

The detailed representation and cost of pointers, segment references and address formation still need to be derived from the historical architecture rather than assumed.

## C++ classes and Enter Capabilities

A particularly interesting idea emerged from considering C++.

A security-relevant C++ object may map naturally onto the PP250 distinction between executable entry authority and data authority.

An externally visible object reference could be an Enter Capability (EC). The object's private state could reside in one or more data segments. Calling a method would then correspond to a protected entry through the EC rather than handing the caller direct read/write authority over the object's representation.

Conceptually:

    caller
      |
      | Enter Capability
      v
    protected object interface
      |
      +---- executable methods
      |
      +---- private data segment(s)

This is stronger than simply giving the caller a bounded read/write pointer to the object's storage. The authority conveyed can be "you may invoke this interface" rather than "you may read or modify these bytes."

This suggests a possible hardware-enforced object model:

* ordinary local/value objects remain ordinary data where protection adds no value;
* allocations requiring memory bounds can be data segments;
* authority-bearing or encapsulated objects can be represented by ECs;
* class implementation code may potentially be shared while individual instances have distinct data/capability environments;
* different ECs might expose different subsets of authority to the same underlying resource.

Examples of objects where this distinction may matter include files, sockets, devices, queues, memory pools, cryptographic services and other resource-bearing objects.

This is a design exploration, not yet a claim about the exact historical PP250 C++ mapping (C++ post-dates the machine).

## CALL versus CHP

The C++/EC discussion also clarified an important performance distinction.

It would be unattractive if every protected method invocation required a full process change. The relevant PP250 mechanism appears instead to be protected CALL/domain entry. CHP is the heavyweight process-state transition.

The useful conceptual hierarchy is:

    ordinary call
        -> same protection environment

    protected EC call
        -> controlled protection/domain transition

    CHP
        -> complete process change / scheduling transition

The exact historical cost and microsequence of CALL and CHP remain subjects for documentary reconstruction, but modern object invocation should not be assumed to imply CHP.

## Cost comparison with CHERIoT

CHERIoT demonstrates that capability protection is practical in a small embedded core, but its hardware must support capability-bearing pointers throughout the C/C++ execution model.

A PP250-derived design may avoid some of that per-pointer machinery by putting bounds, identity and access information in the SCT/segment architecture and retaining distinct capability operations.

That creates a plausible route to a smaller protected processor, but no cost advantage should yet be claimed. A fair comparison must compare equivalent guarantees.

In particular, the proposed PP250-derived machine must account for:

* object-level spatial safety;
* temporal safety;
* protected object invocation;
* MMIO/resource authority;
* process/domain transitions;
* capability lifetime and SCT reclamation;
* event/fault handling.

Only after those are specified can a gate-count comparison with CHERIoT be meaningful.

## Comparative evaluation: objectives and protection layers (22 October 2026)

This section records the conclusions and open issues from the PP250–CHERIoT–CLOOMC discussion. It is **not** a claim that the three systems have equivalent security guarantees, or that PP250 has already proved cheaper. The historical evidence and technical hypotheses must remain distinct.

### What problem is solved at which level?

| Concern | PP250 / proposed descendant | CHERIoT | CLOOMC |
| --- | --- | --- | --- |
| Fundamental authority | Hardware capabilities, segment access rights and protected transitions; system services distribute authority | Hardware capabilities with bounds, permissions, tags and compartments | Church Machine / Golden Token and Namespace concepts; inspect implementation before assigning identical hardware guarantees |
| Protected service or object interface | Enter capabilities, C6/C7-related execution environment, CALL/RETURN; complete process change via CHP | Compartment entry and sealed references; trusted switcher mediates protected transitions | Object/capability construction is central to the programming model; precise entry and switching cost requires implementation review |
| Ordinary language-level bounds and lifetime errors | Can be handled by compiler/runtime inside authorised segments; per-allocation segments are an optional design choice | Capability-bearing C/C++ pointers provide additional hardware-enforced spatial bounds; allocator and revocation contribute temporal safety | No equivalence of pointer-safety guarantees established |
| Storage and process management | Store Manager and Process Manager are indispensable privileged services built upon machine primitives, not assumed to be wholly hardwired | Capability-aware allocator, loader, scheduler and switcher; compartment/thread construction constraints differ | Map specific allocator/thread-creation facilities from CLOOMC sources before asserting correspondence |
| Malicious or injected machine code | Must not exceed its hardware-delegated authority, although it can harm other objects *within* that authority | Cannot forge capabilities or escape compartment authority; may misuse broader capabilities legitimately available within its compartment | Threat model and enforcement boundaries still require source-level verification |

**Key distinction:** Hardware authority enforcement is not the same objective as preventing every C/C++ pointer mistake. A safe-language compiler can check indexes, ownership and object lifetimes, but injected assembly, unsafe code or faulty compilation can bypass compiler-only rules. Hardware-enforced domain boundaries should remain effective even then. Conversely, bounding only the domain does **not** stop malicious code or a buffer error from corrupting another allocation *inside* that domain. CHERIoT deliberately offers additional per-pointer/per-allocation protection where capability bounds are sufficiently narrow; that is a real additional property rather than redundant protection. It does not mean every pointer is necessarily bounded to the smallest C subobject.

The proposed PP250 division of labour (hardware protecting authority; language and runtime preventing ordinary internal errors) is therefore a **design hypothesis with an explicit security-granularity trade-off**, not evidence that fine-grained CHERIoT pointer protection is unnecessary.

### SCT indirection is not per-reference descriptor lookup

The historically recovered PP250 mechanism loads/expands stored capabilities into C registers holding the information for subsequent access checks. An ordinary reference through an expanded capability does **not** imply another SCT fetch. Later Andrews–Wheatley US4408274A describes P-register/C-register *Load on Use*: pointer movement may defer expansion until first use. Do not project this later feature into the original PP250 generation. See [the modern relevance note](2026-10-18-pp250-modern-relevance-and-memory-safety.md), [capability representation](../architecture/capability-representation.md), and [LC/LDP evolution](lc-ldp-direct-mode-evolution.md).

CHERIoT's public explanation of table-based capabilities argues that lookup can cost extra accesses and descriptor caching incurs hardware/power cost. **That general objection is insufficient by itself to evaluate PP250's already-expanded C registers.** It does *not* establish that CHERIoT's designers misunderstood PP250 or had evaluated that architecture specifically. The real comparison is *first-use expansion frequency, cache/register residency, coherent revocation and total cost*, not a fictional SCT access for every PP250 data transfer. Primary CHERIoT reference: [CHERIoT Programmer's Guide](https://cheriot.org/book/concepts.html); exact wording and context should be checked against a fixed edition before quoting.

### Mixed pointers and data; C/C++ compatibility

An SCT need not keep all stored capability *references* apart from ordinary data: descriptors live in the SCT, whereas capability references can be fields of structures, arrays and linked data. A proposed PP250 C ABI could represent data pointers, pointers to pointers, function pointers, and arrays of function pointers by stored capability references plus necessary offsets, loading an active reference into a C register when used. More C registers in a successor might reduce turnover. **This is not yet an implemented or historically established general C ABI.**

Open details include pointer arithmetic and interior pointers; storage of offsets; mixed-layout alignment; ordinary memcpy, unions and bytewise overwrite; validity/non-forgeability when data are interpreted as stored capabilities; and distinctions between executable/enter authority and ordinary data authority. Neither interleaving nor a large capability-register file alone settles those questions. Compare the draft [PP250 ABI](PP250-ABI.md) and historical [ISA notes](../architecture/instruction-set.md).

### Hardware economics and modern ISA

The historical three-word SCT entry is 3 × 24 = 72 bits. It therefore **cannot fit unchanged into one 64-bit transfer**. A redesigned 64-bit descriptor would require field-budget choices; an unchanged historical entry on a 64-bit interface takes at least two transfers, absent a suitable different physical layout. A 128-bit memory interface could fetch 72 bits in one transfer, but is not automatically economical for a low-cost board, especially with off-chip pins/routing. Transfer count is **not** clock-cycle latency: memory timing, arbitration, caching and descriptor coherence remain unknown.

The 24-bit PP250 ISA already includes integer arithmetic, direct/store operand modes, capability registers and protected call/process mechanisms. Possible 32/64-bit successors with more data and capability registers, a changed encoding, LLVM support and contemporary byte/atomic/numerical facilities are proposals, not historical facts or completed designs. They should be judged against RV32/RV64 and CHERIoT on code density, throughput, gates, energy, memory footprint and full trusted software. Store allocation and process management must be counted in total-system cost.

### Competitive and commercial position

CHERIoT already provides real hardware and a C/C++ toolchain, RTOS and ecosystem. The potential niche for a PP250-derived embedded board or licensable processor IP is **unproven**. The relevant possible differentiators to test are compact SCT-backed stored references; reduced descriptor overhead from expanded C registers and Load on Use; dynamic protected execution construction; and protected-object invocation with native architectural support. CLOOMC offers an additional capability/object-oriented comparison, but is not interchangeable with either hardware ISA. A credible comparison must use matched security properties, real workloads, compiler maturity, protection transitions, object allocation/reclamation and cost data. Do not infer superiority or prospective sales from the size of the Raspberry Pi market.

### Questions requiring measurement or primary-source checking

- How do PP250's expanded capability register loads, descriptor invalidation and multiprocessor SCT coherence behave in each historical generation?
- What are CHERIoT's exact documented pointer/subobject and temporal guarantees, trusted-switcher costs and restrictions on dynamic protected construction, versus the semantics we would require?
- Which capabilities does CLOOMC actually enforce in hardware versus software, and what are its concrete object and process-management interfaces?
- Can a specified modern PP250 ABI handle arbitrary C pointer nesting, mutation and function-pointer dispatch without opening a capability-forgery path?
- For equivalent security boundaries, what are gate area, memory transfers, first-use expansion cost, compiler-generated code size, execution time and energy?

## Constrained by construction — comparison of PP250, CLOOMC and CHERIoT

**Comparative interpretation (23 October 2026).** *Constrained by construction* means that fundamental constraints are embedded in the mechanisms by which computation and authority operate, not merely imposed as optional checks by cooperating application software. It does **not** imply that all protection policy is hardwired or that trusted software is absent. The fuller PP250 reasoning, beginning with its clocked control of state transitions and external events, is recorded in [PP250 Beyond the Capability Machine](pp250-beyond-the-capability-machine.md).

| Dimension | PP250 | CLOOMC | CHERIoT |
|---|---|---|---|
| Intrinsic enforcement | Hardware capability checks and microprogrammed execution, state transitions and fault pathways. | Golden Token validation and capability confinement are design objectives of its hardware pipeline. | Hardware capability tags, bounds, permissions and sealing restrict ordinary memory and control operations. |
| Trusted software | The Store Allocator and Process Manager establish and distribute protected resources and processes within machine-enforced mechanisms. | Mint, memory management, loading and scheduling are envisaged as capability-confined abstractions, alongside a minimal trusted boot foundation. | A trusted switcher, allocator, loader and other RTOS services implement higher-level compartment and lifetime policies on top of hardware capability enforcement. |
| Faults and containment | Documented checking, interrupts, store response/timeout mechanisms and recovery are integrated with processor control. | Its foundation document explicitly claims *fail-safe by construction*: faults should be confined to the capability envelope, with structured fault records. This is a design claim, not an independently verified guarantee. | Capability violations cause hardware exceptions; protected switching and software services participate in fault response and system containment. |
| Scope of demonstrated integration | Synchronous microprogram control of incoming conditions is documented, though exact next-slot and wait circuitry remains under reconstruction. | A similarly comprehensive, independently verified account of clocked external-response and fault-state sequencing has **not** been established in this comparison. | A synchronous hardware core is not in doubt, but higher-level isolation and recovery are shared with explicitly trusted software; equivalence to PP250's proposed unified interpretation has **not** been established. |

The initial comparison risked treating CHERIoT's reliance on a trusted switcher or allocator as a weakness relative to PP250. **That was mistaken.** PP250 also uses trusted software to implement important protection policy. In particular, its Store Allocator controls segment creation and the distribution of capabilities; software authority is exercised *within* hardware-enforced bounds. A mixed capability/data Dump Stack is a known special use. Whether mixed storage is needed for capability manufacture is an **open reconstruction question**: the proposed alternative capability-creation mechanism need not require it. We cannot currently assert that there are exactly two permitted mixed-segment cases, nor that their admission rule is exclusively hardware-enforced.

CLOOMC is conceptually close to this thesis because its documentation deliberately relates authority confinement, fault reporting and reliability, and uses the term *fail-safe by construction*. That similarity must not be mistaken for proof that its implemented hardware provides the same control or fault-containment guarantees as the historical PP250. Conversely, CHERIoT's deliberate hardware/software partition is an architectural choice, not evidence that its protections are merely conventional software policy.

**Comparison criterion:** for each machine, identify which invariants hardware makes non-bypassable, which additional invariants a restricted software service maintains, how that service receives and exercises authority, and what happens when software, external devices or hardware misbehave. PP250's possible distinction is the *integration* of ordinary execution, external conditions and fault responses into its constructed control mechanism—not the mere presence of hardware capabilities or the absence of trusted software. Whether the other two embody equivalent integration remains to be determined from their implementations.

**Sources and status:** PP250 timing and microprogram control: [processor self-test](../transcriptions/processor-self-test-program.md) and [processor control reconstruction](../architecture/processor-control.md); PP250 Store Allocator: [working reconstruction](../reconstructed-common-facilities/store-allocator.md). CLOOMC: [foundation document](https://github.com/khhodges/cloomc-project/blob/main/docs/cloomc-foundation.md) and [repository overview](https://github.com/khhodges/cloomc-project/blob/main/README.md). CHERIoT: existing references and analysis in this note. Comparative judgments above are interpretations, not measured equivalence claims.

## Current working conclusion

CHERIoT should be treated as a primary modern reference architecture for PP250-Reboot.

The interesting comparison is not "PP250 versus CHERI" in the abstract. It is increasingly:

> CHERIoT carries fine-grained authority with pointers, whereas a PP250-derived machine may be able to make segments and protected objects the units of authority and use the SCT as the indirection/lifetime structure.

Whether that produces comparable C/C++ safety with less hardware is an engineering question to be tested, not assumed.

## Questions to carry forward

1. What is the exact historical PP250 data-pointer/segment-addressing model relevant to small data segments?
2. What is the minimum practical segment size and SCT cost in a modern implementation?
3. Can a compiler map selected C++ classes cleanly to EC-mediated protected objects?
4. What does a protected object call cost relative to an ordinary call?
5. Which CHERIoT safety guarantees would the PP250-derived model match, and which would it intentionally not provide?
6. What is the resulting gate/area budget when compared on equal security properties?
