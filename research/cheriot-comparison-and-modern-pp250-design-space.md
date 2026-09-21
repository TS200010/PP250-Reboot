# CHERIoT comparison and the modern PP250 design space

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
