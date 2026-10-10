# PP250 / System 250 — Current Open Architectural Questions

## Status

**CANONICAL CURRENT-STATUS INDEX**

This file is the authoritative index of architectural questions that are **currently open** in the PP250-Reboot reconstruction.

Historical research notes preserve the state of investigation at the time they were written. Their labels `UNKNOWN`, `HYPOTHESIS`, “unresolved”, “open question” and similar wording are **not by themselves evidence that an issue remains open now**.

## Admission rule

A question belongs here only when all of the following are true:

1. the current repository corpus and reconstruction have been checked;
2. a specific missing fact, conflict or mechanism can be stated precisely;
3. the missing item materially prevents or weakens reconstruction of historical System 250 behaviour, rather than merely leaving a bit layout, chronology or implementation detail undocumented;
4. the issue is not simply a difference between architectural generations, operating systems, forms or source viewpoints.

Before proposing a new architectural unknown, first check this file and the current canonical architecture. If an older note appears to expose a new issue, reconcile it against later evidence and conclusions before adding it here.

Generation-specific representations are allowed to differ. There is no requirement to derive a bit-for-bit evolutionary mapping between generations unless such a mapping is itself necessary to explain observed behaviour.

## Current open questions

### Initial protected processor state at cold power-up

**Open question:** what protected and special processor state exists immediately after cold power-up, before the C(S)-rooted start-up sequence establishes the first protected process environment?

The hardware/preset C(S) root accounts for primordial authority, but the exact initial state of the other protected/special registers and indicators has not yet been established. This matters because it defines the architectural starting state from which the documented start-up mechanisms operate.

### Provenance of the capability installed in C(C)

**Open question (narrowed, 10 October 2026):** how does C(S)-rooted startup authority construct and supply a legitimate *first SCT-root capability* to the transition process, before ordinary Inform loading through the runtime SCT is possible?

The present reconstruction distinguishes two cases. At genuine cold start there is no existing runtime SCT, so an empty SCT block must first be created or prepared and a legitimate capability to it supplied for installation in C(C). On processor restart/rejoin, the runtime SCT already exists in a memory module being used by the running system, so the returning processor instead needs a capability identifying that existing SCT. The transition mechanism may be the same in both cases; what remains unresolved is the provenance and delivery of the appropriate source capability.

### Bootstrap from the initial empty SCT

**Open question (narrowed, 10 October 2026):** can the proposed private mixed-access data-write → capability-load construction method, under C(S)-rooted bootstrap authority, establish the initial SCT entries, code capabilities and service environments after C(C) is installed, and what exact validation governs that transition?

Creating an empty SCT and installing its capability in C(C) establishes the normal capability-table root but does not by itself populate the runtime capability universe. The service-level COS `GIV`/`ALO`/`RSP` distinction supports a candidate common construction principle, but does not prove its cold-start use. Verify whether `LC` accepts data-written representations, how the initial pre-SCT descriptor is accepted, how special-register installation works through the established One-Shot Second Group LC path, and the exact limits of C(S)-derived construction authority. COS `LOI` concerns running COS rather than virgin physical loading.

### Multiprocessor cold-start coordination

**Open question:** when a multiprocessor System 250 is powered up from cold, how are the simultaneously starting processors coordinated so that one normal runtime capability environment is established rather than independent competing start-up environments?

In particular, it remains to be established whether all processors initially execute the C(S)-rooted sequence, whether one processor becomes the bootstrap processor while the others wait or remain in a restricted state, how any such processor is selected, and how the remaining processors subsequently acquire the same C(C) and join the newly established runtime system. This is distinct from processor restart/rejoin, where an existing running system and runtime SCT are already available.

### M-extension sealing after One-Shot Second Group LC

**Hypothesis:** C(C), C(I) and C(N) are established from the C(S)-rooted start-up/recovery environment using **One-Shot Second Group LC**. Once that operation has been consumed, no authority accessible to H or T can replace those registers or reproduce their contents. If confirmed, the protected software reached through C(C), C(I) and C(N) is effectively sealed into M until the next C(S)-rooted recovery/start-up sequence.

**Open question:** after C(C), C(I) and C(N) have been established, is there any surviving route by which H or T can replace them, or recreate equivalent authority, without a new C(S)-rooted start-up/recovery sequence?

This matters architecturally because it determines whether the protected software entered through these registers is merely privileged system software or a protected software extension of M whose installation authority disappears after construction.

### Software access to and garbage collection of SCT entries

**Established context:** C(C) identifies the SCT used by the processor to expand compact active capability pointers into capability-register state. Stored active capabilities contain an SCT index/reference; they are not themselves capabilities to the SCT. The SCT entries themselves require lifecycle management, including garbage collection, and later evidence associates garbage-collection state such as `GARBAGE` and `VISITED` with SCT entries.

**Open question:** by what capability or architectural mechanism does the software responsible for SCT garbage collection traverse and manage the SCT entries?

If C(C) is the only reference that identifies the current SCT, it remains to be established whether software can use that authority directly to traverse the table, whether a separate ordinary capability to the storage containing the SCT exists, or whether some other protected mechanism provides the required access.

This also bears on SCT replacement during start-up/rejoin. Replacing C(C) establishes a new SCT for processor capability expansion, but until the management path is understood we must not infer that this necessarily removes every software-accessible route to the previous SCT or establishes how its entries/storage are subsequently reclaimed.

### Loading C7 with LC during execution

**Open question:** is `LC C7, …` a valid instruction form, and, if so, what are its precise execution and instruction-fetch semantics?

In particular, can ordinary LC replace the currently executing C7 capability, and does the replacement take effect for the next instruction fetch, preserve the existing instruction-address offset, or require a separate control transfer? There is no established prohibition in the material reviewed so far, but neither validity nor precise behaviour is documented.

**Why it matters:** Checkout's documented immediate jump from its four-instruction block at octal `034–037` to the complementary-address block raises the possibility of transferring between separately bounded executable capabilities rather than requiring a module-spanning C7. This is a candidate mechanism, **not** a claim that Checkout actually uses LC C7. The Checkout jump alone does not establish C7's bounds.

#### Reasoning trail: implicit transfer and potential security consequences (8 October 2026)

This arose as an aside while analysing Repton's Checkout entry: the paper places a four-instruction block at absolute octal `034–037`, which immediately jumps to a complementary-address block in the same store module. That does **not** by itself imply a C7 capability spanning the module; the program is exercising the *processor's* instruction-address sequencing, and might transfer between narrowly bounded executable capabilities. We considered whether `LC C7, …` could implement such a transfer, without claiming that Checkout uses it.

**Conditional instruction semantics:** Suppose `LC C7` is permitted, changes only C7, and has no special effect on IAR or the normal instruction sequence. If the instruction executes at IAR `042` (octal), the next sequential fetch uses **the new C7** with IAR `043`: physical address = new C7 base + `043` (subject to execute permission and bounds). Under these assumptions there is no separate branch operation or delayed changeover to explain: replacing C7 immediately redirects the next ordinary fetch. The new capability's base must be chosen with the continuing IAR offset in mind. This is economical but awkward/hacky as a general control-transfer idiom; it could nevertheless be useful in diagnostic code. Its legality and actual timing remain unverified.

**Conditional security question:** If ordinary LC can replace C7, could a holder of a legitimate executable capability use this implicit transfer to enter a target code block at an unintended offset, without an enter-capability CALL and without installing the C6 context that the target expects? That would potentially expose control-flow-integrity or authority-context assumptions even though every fetch remains within a valid C7's execute bounds. It does **not** manufacture a capability, bypass base/limit checks, or establish an actual exploit. The risk depends on the rights required to load C7, whether an ordinary executable capability can be held independently of controlled enter authority, what entry restrictions (if any) hardware enforces, and whether target code relies on C6/C7 pairing or CALL/RETURN discipline.

**Tests needed:** establish from instruction definitions or microcode whether LC permits C7 as destination; whether IAR continues normally across such a load; whether a new C7 takes effect on the very next fetch; and whether any protection mechanism restricts executable entry or binds executable authority to an expected C6 context. Keep this thought experiment separate from the documented Checkout jump and from any claim of a proven PP250 vulnerability.

### Power-failure indication in MIP

**Open question:** what are the exact set/reset semantics of the power-failure indication in MIP, particularly across a power-failure/start-up CHANGE PROCESS sequence?

Page 6 of the Pocket Reference shows MIP saved in the Process Dump Stack at offset `020`, while page 8 identifies a power-failure indication in MIP. It remains to be established when hardware or microcode sets that indication, whether it is present before or after MIP is saved/restored by CHANGE PROCESS, and what event or operation clears it.

This matters because start-up or recovery code can only use the indication to distinguish a power-failure entry from another entry if the indication survives with defined semantics into the executing context.

### Fault within the fault-handling process

**Open question:** what happens if the fault-handling process entered by the processor itself generates a capability fault?

The architectural path for a fault arising within the fault-handling process has not yet been established.

### Initial access to the Common Facilities Block (CFB)

**Open question:** by what capability path did a newly constructed ordinary process initially reach the Common Facilities Block and its resource allocators, and was there a conventional location in the initial C6 capability block?

**Established evidence:** D. M. England, *Architectural Features of System 250*, §35 and Fig. 11, describes a read capability to the CFB, which contains enter capabilities to seven allocators: store, process, flag, stream, textfile, directory and job. A process must receive legitimate authority through existing capabilities or authorised protected services; a symbolic name or address alone cannot grant it.

**Working hypothesis (unverified):** the process constructor installs a **read capability to the CFB at `C6[0]`** in the newly created process's *initial* C6 block. This is a reconstruction convention for current reasoning, **not** an established historical offset or a claim about every protected node's C6.

**Still to establish:** whether offset zero was actually used; whether all or only selected processes received CFB access; whether the path was direct or via an authorised intermediary; and precisely how the constructor obtained and passed on the relevant capability. The diagrams reviewed so far do not establish `C6[0]`.

### Exclusive physical-resource admission and reconfiguration

**Open question:** when a memory module or memory-mapped peripheral is first admitted, does System 250 exclusively allocate its physical address range and manufacture only the requested rights, or can an existing authority holder generate overlapping access? England's Store Allocator creates *virtual storage blocks* with requested access, which does not answer this physical-resource question.

**Open question:** after physical removal, what authorised mechanism can retire the resource and admit another, potentially at the same address, without stale stored capabilities or already-expanded processor-register capabilities reaching the replacement? What authority permits subsequent admission if primordial authority was consumed, and how are SCT/object identities safely reclaimed or distinguished?

See [resource-lifecycle reconstruction, section 22](capability-genesis-and-resource-lifecycle.md#22-8-october-2026--exclusive-primordial-allocation-and-dynamic-physical-reconfiguration).

### COS `LOP` loading command

**Open documentary question:** What does the `LOP` command do, and what does the `P` suffix signify?

The *System 250 Pocket Reference*, pp. 11–12, lists `LOI` (“load assembler output via data break”), `LOO` (“load assembler output from serial medium reader”), and `LOP` with a **genuinely blank description** (confirmed against the original page). Pages 15–16 also give `LP` as the abbreviated form of `LOP`. The commands are adjacent because the list is alphabetical; their adjacency provides no evidence of a common bootstrap function. Neither “Program” nor “Paper tape” is an established expansion of `P`.

This may be relevant to historical software loading or initial installation, but **no bootstrap role for `LOP` has been established**. Seek an independent command description or contemporary operational documentation. This is a documentary gap, not a blocker to the architectural startup reconstruction.

## Resolved, demoted or deliberately parked issues

The following must **not** be resurfaced as current architectural unknowns merely because older research records contain unresolved wording:

- **`PS/DT/RTE` versus `EC/WC/RC/ED/WD/RD`:** generation/version-specific representations. No cross-generation bit mapping is required.
- **SCT state/flag layouts across generations:** generation/version-specific representations. Reconstruct semantics within the relevant generation; cross-generation bit continuity is not required.
- **Inform/Outform:** part of the virtual-memory/storage-management mechanism. Outform/Inform conversion is managed by VM/storage management and VM traps. Exact representation may be generation-specific and is not, by itself, an architectural blocker.
- **Capability creation/resource allocation:** the allocator creates the resource and returns the appropriate capability; this is established architecture, not a missing genesis mechanism for ordinary runtime allocation.
- **VM trap and restart path:** reconstructed; do not reopen merely because an older source note calls some part unresolved.
- **LDP:** accepted architectural meaning is loading the compact capability pointer associated with its operand into a D register. Historical software uses are deliberately parked pending naturally arising evidence.
- **CALL / Enter Capability / RETURN:** reconstructed. CALL saves C6/C7/IAR, establishes the entered C6/C7 context, and RETURN restores it; other general registers survive the call boundary as documented.
- **Exact bit encodings or chronology:** an undocumented generation-specific encoding is a documentary gap, not automatically an architectural open question. Promote it only if the missing encoding prevents reconstruction of behaviour.
- **Cold start / primordial authority:** the startup root is architectural hardware/preset state, not an authority that ordinary software must manufacture. The corpus documents the hard-wired early SSCR/Fault-Block route and the later processor-preset `C(S)`/Special Start-Up Block route, leading through the special capability environment, reserved pointer/Dump Stack and process entry. Exact generation-specific microsequence or physical loading/commissioning details are documentary/implementation questions, not a missing architectural authority mechanism.

## Research-record rule

Do not rewrite completed source reviews merely to make their historical questions agree with this file. They are records of what was learned at that time.

Living topic-research notes may be updated when later evidence resolves an issue. Where retaining old reasoning is useful, mark it as superseded rather than allowing it to masquerade as a current open question.

## Maintenance

When a current open question is resolved:

1. remove it from **Current open questions**;
2. add a concise entry under **Resolved, demoted or deliberately parked issues** if recurrence is likely;
3. update the canonical architecture or relevant living topic note;
4. leave completed source reviews historically intact unless they contain an actual error.

When proposing a new open question, state exactly what historical behaviour cannot presently be reconstructed and why existing evidence does not already answer it.
