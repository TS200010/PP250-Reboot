# PP250 Attack Surface Analysis

## Status

**Working research note.**

This note examines the attack surfaces exposed by the Plessey System 250 architecture as it is currently reconstructed in this repository.

It deliberately analyses the PP250 on its own terms. It does not use the separate M⟨H,T⟩ interpretation of the architecture.

An **attack surface** is not necessarily a vulnerability. It is a point at which hostile, erroneous, or compromised software can interact with a security-relevant mechanism. In many cases the purpose of the PP250 architecture is precisely to constrain such interactions.

The objective is therefore to distinguish:

- surfaces that are architecturally constrained;
- surfaces whose security depends partly upon software convention or trusted system software;
- surfaces for which the surviving evidence is insufficient to determine the protection mechanism; and
- any genuine paths by which software could acquire authority that it was not intended to possess.

---

## 1. Fundamental security property

The central security property of the PP250 is not that programs cannot make erroneous accesses. They plainly can attempt them.

Rather, access to protected objects is mediated through capabilities, and the processor checks the authority represented by those capabilities when performing protected operations.

Consequently, the most important question for this analysis is:

> **Can software obtain or exercise authority that has not legitimately been made available to it?**

An implementation error, bad pointer, incorrect procedure call, or malicious calculation is security-significant only when it can cross an authority boundary rather than merely cause failure within authority already possessed by the process.

This makes authority acquisition, delegation, retention, transformation and revocation the principal areas of interest.

---

## 2. Ordinary memory addressing

Ordinary erroneous address calculation appears to present a relatively small security surface.

A program may calculate an incorrect offset, but an offset is interpreted relative to an accessible object. Bounds and access permissions are enforced by the capability mechanism.

Thus conventional memory-corruption techniques based upon turning an arbitrary integer into an unrestricted machine address do not appear to translate directly to the PP250 architecture.

An out-of-range reference should result in an exception rather than access to an adjacent unrelated object.

This does not prevent corruption of data lying legitimately within an accessible object. Object layout therefore remains important: placing mutually distrustful data within the same protected object weakens the protection boundary.

**Current assessment:** substantially constrained by architecture.

---

## 3. Capability registers surviving protected CALL

Protected procedure calling changes the caller's protected environment, including C6 and C7, but capabilities in C0–C4 may survive the transition.

Levy explicitly identifies this as a potential route by which capabilities can pass across a protected procedure boundary.

This creates a genuine authority-delegation surface.

A caller that leaves a capability in one of these registers may unintentionally provide it to the called procedure. Conversely, conventions governing returned register contents are important if capabilities can be returned to the caller.

However, this should not automatically be described as an architectural protection failure. The surviving capability was already valid authority possessed by one participant. The issue is unintended **delegation or retention of existing authority**, rather than manufacture of new authority.

The mechanism also appears useful for efficient capability parameter passing and may therefore represent a deliberate architectural trade-off.

The eventual ABI must specify which capability registers constitute arguments, results, preserved state, and scratch state, and whether unused capability registers should be cleared at protection boundaries.

**Current assessment:** real attack surface; principally an authority-delegation and interface-discipline issue.

---

## 4. Capability creation and derivation

The most important unresolved security question concerns the complete set of operations capable of producing a new capability.

For every such operation we ultimately need to establish:

1. the source of the authority;
2. whether the resulting capability can contain greater rights than its source;
3. whether its bounds can exceed those authorised by the source;
4. whether arbitrary data can influence the identity of the protected object; and
5. whether any exceptional mechanism exists for introducing new authority.

Our current reconstruction strongly suggests controlled derivation rather than arbitrary capability construction, but the complete mechanism has not yet been demonstrated from the surviving documentation.

The exact semantics of capability propagation, capability-block creation and `SC` remain particularly important.

The working hypothesis that `SC` may attenuate rights during storage must remain explicitly labelled as a hypothesis until supported by stronger evidence.

**Current assessment:** potentially fundamental surface; reconstruction incomplete.

---

## 5. Pointer registers and LDP

The relationship between pointer registers, `LDP`, object identity and capability loading requires further reconstruction.

A pointer appears capable of identifying something without itself representing the authority associated with a capability.

If that interpretation is correct, the critical security question becomes:

> **What authorises the conversion from an object-identifying pointer into a usable capability?**

A freely manipulable pointer must not provide a route by which software can name an arbitrary protected object and thereby acquire authority to it.

The existence of pointer arithmetic is therefore not itself problematic. The security property depends upon the mechanism that combines object identity with legitimate authority.

The exact roles of `LDP`, any corresponding store-pointer operation, and their relationship to `LC` need to be established from the instruction descriptions, patents and surviving software.

**Current assessment:** unresolved pending reconstruction of the pointer/capability relationship.

---

## 6. System Capability Table identity and reuse

Stored capabilities refer indirectly to objects through the System Capability Table.

This creates an important object-lifetime question.

Suppose a capability referring to SCT entry *n* survives after the original object represented by entry *n* has ceased to exist. If that SCT entry were subsequently reused for an unrelated object, the old capability must not silently acquire authority over the new object.

This is essentially a stale-authority problem.

The SCT garbage-collection and object-reclamation mechanisms therefore appear to be security-critical rather than merely storage-management facilities.

Our reconstruction suggests that SCT entries cannot safely be reused while capabilities referring to them remain reachable. The precise historical mechanism by which this condition is guaranteed still requires further reconstruction.

**Current assessment:** architecturally recognised problem; exact protection mechanism not yet completely reconstructed.

---

## 7. Capability-block traversal and garbage collection

Capability blocks create a graph of authority relationships.

Garbage collection must therefore distinguish genuine capability references from ordinary data and determine whether an SCT identity remains reachable.

Two potentially important cases require further investigation.

First, outformed capability blocks may temporarily place parts of the authority graph outside immediately resident memory. The collector must nevertheless preserve the identity relationships represented by them.

Second, concurrent modification creates a classic tracing problem: if a process stores a capability into a block after the collector has scanned that block, the newly reachable object must not subsequently be reclaimed.

The historical synchronization mechanism is not yet known. Possible solutions should not be projected onto the PP250 without documentary evidence.

**Current assessment:** likely addressed by system design, but historically incomplete in our reconstruction.

---

## 8. Process change and Dump Stack state

`CHP` restores a substantial portion of processor state from a Dump Stack, but the workspace capability registers C0–C5 are not restored as attacker-supplied 48-bit base/limit descriptors. The fixed Dump Stack locations corresponding to C0–C5 are 24-bit capability pointers. US3771146A states that these pointers are recorded when the workspace capability registers are loaded and are used through the capability table to reconstruct those registers on process restoration.

The Dump Stack therefore represents security-sensitive process state, but the attack surface is more constrained than a model in which arbitrary expanded capability-register images could simply be written and restored. Physical base/limit state is rematerialised through the capability table current at restoration.

The important questions include:

- who may create a valid Dump Stack;
- who may modify its 24-bit capability-pointer state;
- what access and validation rules prevent fabrication or amplification of authority in those pointers;
- how a process becomes eligible for `CHP`; and
- whether arbitrary software can cause processor state to be reconstructed from an attacker-controlled Dump Stack.

The fixed Dump Stack layout itself is not a weakness. The attack surface lies in authority over the object, legitimate construction or modification of its compact capability-pointer state, and the mechanism selecting it for process change.

**Current assessment:** high-value control surface; the C0–C5 representation/restoration mechanism is now established, while the authority and validation rules governing creation and modification of that saved pointer state remain only partly reconstructed.

---

## 9. Normal interrupts and traps

Normal interrupts enter through the Normal Interrupt mechanism associated with C(N), the Normal Interrupt Block and subsequent process-state machinery.

This is security-sensitive because an interrupt can cause execution to move away from the currently executing program without that program making an ordinary protected call.

The important issue is therefore not merely where the interrupt vector resides, but who can establish or modify the state determining what executes in response to the interrupt.

Our reconstruction currently distinguishes normal interrupts and program traps from the separate fault/startup mechanism.

Further work should verify all writable paths leading to C(N), Normal Interrupt Blocks and associated Dump Stacks.

**Current assessment:** privileged control surface; apparently deliberately structured, with details still to verify.

---

## 10. Fault and startup path

Fault handling and processor startup use mechanisms distinct from normal procedure calling and normal interrupt handling.

Evidence examined so far indicates that fault entry deliberately invalidates existing processor capability state through reversal of internal capability parity before recovery proceeds.

If correctly understood, this is a particularly significant security property: execution cannot simply continue after a serious fault while retaining an arbitrary collection of capabilities from the failed context.

C(S), SPECIAL, the initial Dump Stack and the subsequent recovery path therefore constitute a critical trusted surface.

The outstanding reconstruction questions concern precisely how the initial recovery authority is established and under what circumstances these mechanisms can be invoked or modified.

**Current assessment:** deliberately hardened architectural boundary; bootstrap details remain partly unresolved.

---

## 11. Internal Mode and processor-internal state

Processor-internal state is accessible through special architectural mechanisms associated with Internal Mode.

This state can affect capability processing, interrupt handling and other fundamental processor behaviour. Consequently, unauthorised access to it would be substantially more serious than access to an ordinary application object.

The principal security question is therefore:

> **What prevents ordinary software from acquiring an environment from which Internal Mode operations are permitted?**

Our reconstruction indicates that Internal Mode is an access condition associated with processor-local internal resources rather than a conventional globally addressed privileged memory space.

The complete authority path into this state nevertheless deserves explicit verification.

**Current assessment:** critical attack surface; architecture appears specifically designed to restrict it, but reconstruction should establish the complete entry conditions.

---

## 12. Dynamic reconfiguration

System 250 was designed to permit modules to be added, removed and reconfigured while the system remained operational.

This creates several security-relevant questions.

Stored capabilities refer through the SCT, permitting object location to be changed without rewriting every stored capability. Loaded capabilities, however, contain expanded information used directly by the processor.

We therefore need to establish what happens when an SCT entry or physical resource changes while an expanded capability referring to the previous state remains in a processor.

This applies particularly to:

- relocation;
- removal of a resource;
- replacement of a resource;
- software replacement; and
- failure recovery.

The emerging generational interpretation of software replacement may explain some semantic replacement cases: existing executions continue using an old generation while new executions are directed to the new generation.

That should not, however, be assumed to solve the lower-level coherency problem associated with physical reconfiguration.

**Current assessment:** important unresolved interaction between capability state and live reconfiguration.

---

## 13. Peripheral and I/O authority

I/O devices and other physical resources ultimately represent authority just as memory objects do.

The PP250 documentation therefore needs to be examined for the complete path by which software obtains authority over devices and initiates I/O.

A particularly important question is whether any I/O mechanism can access storage independently of the processor capability checks.

Any autonomous transfer mechanism analogous to modern DMA potentially creates a path around processor-enforced protection unless its accessible resources are independently constrained.

This distinction will also be important when translating PP250 principles to modern hardware, but the historical PP250 mechanism should first be established independently.

**Current assessment:** insufficiently reconstructed.

---

## 14. Outform and external representation

Outforming a capability or capability block creates an external representation of protected state.

The representation must not become equivalent to unrestricted authority merely because software can copy, modify or replay it.

Our current reconstruction treats an outform as a representation of capability information rather than automatically as a live capability.

The precise validation and informing mechanisms therefore matter. In particular, arbitrary modification of an outform must not permit increased access rights, changed object identity, or manufacture of authority.

This becomes still more significant if capability representations are transported between systems.

**Current assessment:** security-sensitive transformation; detailed mechanism requires further reconstruction.

---

## 15. Resource exhaustion and denial of service

Capability confinement does not by itself guarantee availability.

A process possessing legitimate authority may still attempt to consume:

- processor time;
- SCT entries;
- memory;
- capability blocks;
- I/O bandwidth;
- queue capacity; or
- other finite system resources.

Retention of capabilities may also prevent objects or SCT identities from being reclaimed.

The watchdog mechanism addresses at least one aspect of uncontrolled processor consumption, but scheduling, accounting and resource allocation remain necessary system functions.

These attacks differ fundamentally from authority-forging attacks: the attacker abuses authority already granted rather than acquiring authority that it does not possess.

**Current assessment:** inherent system-level attack surface rather than obvious failure of capability protection.

---

## 16. Current overall assessment

Nothing in the reconstruction so far demonstrates a straightforward route by which an ordinary program can manufacture arbitrary authority.

The attack surfaces identified above instead fall broadly into three groups.

**Authority misuse or unintended delegation.**  
Examples include retaining or passing capabilities through C0–C4 and corrupting data within an object to which the program legitimately has access.

**Security-critical architectural mechanisms.**  
Examples include protected calling, SCT management, process change, interrupt handling, fault recovery and Internal Mode. These are attack surfaces because they control authority, but their existence does not imply that they are vulnerable.

**Mechanisms whose protection merits further analysis.**  
Do not treat this historical list as the current open-question index. In particular, LDP's architectural pointer meaning is established, ordinary runtime capability creation is allocator-mediated, and generation-specific representation differences are not architectural blockers. Current unresolved architecture is governed by `pp250-open-questions.md`; this attack-surface note may still investigate security properties of established mechanisms.

This distinction should be maintained as reconstruction proceeds.

---

## 17. Research method going forward

For each security-sensitive PP250 operation we should eventually be able to answer four questions:

1. **What state can untrusted software supply or influence?**
2. **What authority does the operation consume?**
3. **What authority can exist after the operation that did not exist before it?**
4. **What architectural check prevents authority from being increased illegitimately?**

The useful end product may therefore be an **authority-transition inventory** covering at least:

`LC`, `SC`, `LDP`, protected `CALL`, `RET`, `CHP`, capability-block manipulation, SCT creation/update/reclamation, outform/inform, interrupt entry, fault/startup entry, Internal Mode operations, and module reconfiguration.

This should be built from documentary evidence rather than inferred from a desired security model.

---

## 18. Provisional conclusion

The PP250 unquestionably has attack surfaces. Any useful computer does.

What is striking in the reconstruction so far is that many conventional attack techniques appear to terminate at an existing authority boundary rather than providing an obvious means of escaping it.

The significant unresolved questions therefore lie less in ordinary computation and more in the relatively small number of mechanisms that can **create, transform, preserve, transfer, restore or destroy capability authority**.

Those mechanisms should be the focus of the continuing security analysis.

No claim should yet be made that the PP250 is free of authority-escalation paths. The stronger and more useful claim at this stage is simply:

> **No such path has yet been demonstrated in the reconstructed architecture, and the remaining candidates can now be identified and investigated explicitly.**

## Evidence update — 2 October 2026: relocation and channel protection

**Resolution — 7 October 2026:** the Pocket Reference and US3771146A together establish that the fixed C0–C5 Dump Stack words are 24-bit capability pointers rather than saved 48-bit capability-register images. This removes one candidate authority-escalation path from the attack model: process restoration does not permit stale or fabricated physical base/limit descriptors to be resurrected directly from those words. Restoration instead rematerialises the workspace capability registers through the capability table. The security question moves one level earlier, to who can create or alter valid capability pointers and the SCT state through which they are interpreted.

**DOCUMENTED OBSERVATION:** US3771146A, Description 121, supplies the refresh mechanism missing from section 12: interrupt the affected processors and restore their processes, thereby reloading workspace capabilities through the table. Changing the SCT alone is insufficient. The documented relocation case narrows the unresolved assessment; software replacement, full cross-processor coordination and SCT reuse remain unresolved. See [canonical SCT mechanism](../architecture/system-capability-table.md#already-expanded-capabilities-during-relocation).

**DOCUMENTED OBSERVATION:** US3787818A, Description 48–50 and 63–90, supplies independent channel protection relevant to section 13: channel-owned source/destination capabilities, descriptor sumcheck validation and repeated bounds checks constrain autonomous transfers. Processor backdoor writes to the special channel capabilities require the channel to be offline. The remaining attack-surface questions concern configuration authority, active-channel relocation and the exact rights checks, rather than whether the described channel has capability/bounds machinery at all. See [channel architecture](../architecture/io-and-interconnect.md).

See [2 October 2026 patent-transcription review](pp250-patent-transcriptions-review-2026-10-02.md) for the complete evidence record and textual limitations.
